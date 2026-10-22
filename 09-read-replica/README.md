# Lab 09 — Amazon RDS PostgreSQL Read Replica and Replication Validation

## Overview

This lab covers creating an Amazon RDS for PostgreSQL read replica, connecting to the replica using `psql` over SSL/TLS, validating the restored database structure, and testing replication of data changes from the primary instance.

The original objective included cross-Region replication to Hyderabad. During the exercise, the Hyderabad configuration workflow returned a KMS security-token error. The root cause was not confirmed, and no Hyderabad replica was created.

A same-Region read replica was subsequently created in Sydney and successfully tested. The results documented in this lab therefore describe **same-Region read replication**, not a successful cross-Region deployment.

## Objectives

- Inspect the cross-Region read replica configuration options.
- Create a read replica in the same AWS Region as the primary instance.
- Connect to the replica using PostgreSQL `psql` and SSL/TLS.
- Verify that the instance is operating as a read replica.
- Validate the Airline database tables and row counts.
- Test replication of an inserted row.
- Test replication of a deleted row.
- Review replica cleanup and cost considerations.

## Environment

| Component | Details |
|---|---|
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL server version | 18.3 |
| PostgreSQL client | `psql` 18.4 |
| Primary DB instance | `database-1` |
| Replica DB instance | `airline-sydney-replica` |
| AWS Region | Sydney (`ap-southeast-2`) |
| Instance class | `db.t4g.micro` |
| Allocated storage | 20 GiB |
| Storage type selected | General Purpose SSD (`gp2`) |
| Database | `airlinedb` |
| Schemas | `airline`, `booking` |
| Port | 5432 |
| Connection security | SSL/TLS |

Live database endpoints, passwords, AWS account identifiers, and private key identifiers are excluded from this documentation.

## Part A — Cross-Region Attempt

### Step 1 — Open the Read Replica Workflow

1. Open **Amazon RDS → Databases**.
2. Select the primary instance `database-1`.
3. Choose **Actions → Create read replica**.
4. Inspect the destination Region and replica configuration options.

### Step 2 — Select Hyderabad

The intended destination Region was:

`Asia Pacific (Hyderabad)` — `ap-south-2`

During the Hyderabad attempt, the console displayed the following error while loading KMS keys:

`The security token included in the request is invalid`

The AWS Console was reopened and the workflow was retried, but the error persisted.

### Recorded result

- The KMS key list did not load successfully.
- The exact root cause was not confirmed.
- No Hyderabad read replica was created.
- The issue was not proven to be caused by the AWS Free plan or by the Region itself.

The cross-Region attempt is documented as an incomplete attempt, not a successful deployment.

## Part B — Create a Same-Region Sydney Read Replica

### Step 1 — Select Sydney as the Destination

The replica workflow was reopened, and the destination Region was set to:

`Asia Pacific (Sydney)` — `ap-southeast-2`

When Sydney was selected, the earlier red KMS security-token error was no longer displayed.

### Step 2 — Configure the Replica

The replica was configured with the following settings:

| Setting | Recorded value |
|---|---|
| Source instance | `database-1` |
| Replica identifier | `airline-sydney-replica` |
| Destination Region | Sydney (`ap-southeast-2`) |
| Instance class | `db.t4g.micro` |
| Allocated storage | 20 GiB |
| Storage type | General Purpose SSD (`gp2`) |
| Enhanced Monitoring | Disabled before creation |
| Storage autoscaling | Disabled before creation |

The KMS selection remained `(default) aws/rds`.

The replica was created after reviewing the configuration.

### Step 3 — Verify Replica Availability

The RDS Databases page displayed a successful creation message for `airline-sydney-replica`.

Both instances appeared with status `Available`:

- `database-1` — Primary
- `airline-sydney-replica` — Replica

This confirmed that the same-Region replica creation operation completed successfully.

## Part C — Connect to the Read Replica

### Step 1 — Connect from Windows PowerShell

The replica was accessed using PostgreSQL `psql` 18.4 and the RDS CA certificate bundle.

Use the following command pattern when reproducing the connection. Replace the endpoint locally; do not publish a live endpoint.

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<REPLICA_RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

The connection succeeded using SSL/TLS.

### Recorded connection information

```text
psql (18.4, server 18.3)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.
```

The Windows console also displayed a code-page warning. It did not prevent the connection.

### Step 2 — Switch to the Airline Database

```sql
\c airlinedb
```

Recorded result:

```text
You are now connected to database "airlinedb" as user "postgres".
```

## Part D — Verify the Replica Role

Run the following query in the replica session:

```sql
SELECT
    current_database(),
    current_user,
    pg_is_in_recovery();
```

Recorded output:

```text
 current_database | current_user | pg_is_in_recovery
------------------+--------------+-------------------
 mydb             | postgres     | t
(1 row)
```

The query was run while connected to `mydb`. The `pg_is_in_recovery()` value of `t` indicated that this PostgreSQL instance was operating in recovery mode as a read replica.

The session was then switched to `airlinedb` for the table validation steps.

## Part E — Validate Table Presence

Run:

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_schema IN ('airline', 'booking')
ORDER BY table_schema, table_name;
```

Recorded output:

```text
 table_schema | table_name
--------------+------------
 airline      | aircraft
 airline      | airports
 airline      | flights
 booking      | bookings
 booking      | passengers
 booking      | tickets
(6 rows)
```

**Result:** All six expected Airline database tables were present on the replica.

## Part F — Validate Row Counts

Run the following query on the replica:

```sql
SELECT 'airline.airports' AS table_name, COUNT(*) AS rows
FROM airline.airports
UNION ALL
SELECT 'airline.aircraft', COUNT(*)
FROM airline.aircraft
UNION ALL
SELECT 'airline.flights', COUNT(*)
FROM airline.flights
UNION ALL
SELECT 'booking.passengers', COUNT(*)
FROM booking.passengers
UNION ALL
SELECT 'booking.bookings', COUNT(*)
FROM booking.bookings
UNION ALL
SELECT 'booking.tickets', COUNT(*)
FROM booking.tickets
ORDER BY table_name;
```

Recorded output:

```text
     table_name     | rows
--------------------+------
 airline.aircraft   |    2
 airline.airports   |    3
 airline.flights    |    2
 booking.bookings   |    3
 booking.passengers |    3
 booking.tickets    |    3
(6 rows)
```

**Result:** The table counts matched the expected sample data.

## Part G — Test Replication of an Insert

### Step 1 — Insert a Temporary Test Row on the Primary

The primary instance was accessed in a separate PowerShell session.

After switching to `airlinedb`, the following statement was executed on **the primary only**:

```sql
INSERT INTO airline.airports
    (airport_code, airport_name, city, country)
VALUES
    ('TST', 'Replication Test Airport', 'Test City', 'Test Country');
```

Recorded output:

```text
INSERT 0 1
```

### Step 2 — Check the Replica

On the replica session, the airport table was queried:

```sql
SELECT *
FROM airline.airports;
```

The replica returned four rows, including the temporary `TST` airport record:

| airport code | Airport name | City | Country |
|---|---|---|---|
| MAA | Chennai International Airport | Chennai | India |
| DEL | Indira Gandhi International Airport | Delhi | India |
| DXB | Dubai International Airport | Dubai | UAE |
| TST | Replication Test Airport | Test City | Test Country |

**Result:** The inserted record appeared on the replica, confirming that the insert was replicated.

## Part H — Test Replication of a Delete

### Step 1 — Delete the Temporary Row on the Primary

The following statement was executed on **the primary only**:

```sql
DELETE FROM airline.airports
WHERE airport_code = 'TST';
```

Recorded output:

```text
DELETE 1
```

### Step 2 — Verify the Replica

The airport table was queried again on the replica:

```sql
SELECT *
FROM airline.airports;
```

The first query after the delete still showed four rows, including `TST`. A subsequent query showed only the original three airport rows:

| airport code | Airport name | City | Country |
|---|---|---|---|
| MAA | Chennai International Airport | Chennai | India |
| DEL | Indira Gandhi International Airport | Delhi | India |
| DXB | Dubai International Airport | Dubai | UAE |

**Result:** The temporary row disappeared from the replica after replication caught up. The insert and delete operations were both successfully replicated.

This test demonstrates asynchronous replication behaviour: a change may not appear on the replica immediately after the primary transaction completes.

## Validation Summary

| Validation item | Result |
|---|---|
| Hyderabad cross-Region attempt | Incomplete; KMS error persisted |
| Sydney same-Region replica creation | Successful |
| Replica status | `Available` after creation |
| SSL/TLS connection | Successful |
| `pg_is_in_recovery()` on replica | `t` |
| Expected tables | 6 present |
| Row counts | Matched expected sample data |
| Temporary insert on primary | Successful |
| Insert observed on replica | Confirmed |
| Temporary delete on primary | Successful |
| Delete observed on replica | Confirmed after replication caught up |

## Cleanup and Cost Considerations

The replica was used for connection and replication testing.

Before considering cleanup complete, verify the current state in **Amazon RDS → Databases**:

1. Confirm `database-1` remains available.
2. Check whether `airline-sydney-replica` still exists.
3. If it remains and is no longer needed, select only `airline-sydney-replica` and review the deletion settings.
4. Do not delete the primary `database-1`.

The available conversation records the replica deletion confirmation screen, but does not conclusively confirm that deletion completed.

A read replica is a separate RDS DB instance and can incur instance-hour and storage charges while it exists. Same-Region replication data transfer is not charged, but this does not eliminate the replica's DB instance and storage costs.

## Screenshots

Supporting evidence is stored in the `screenshots/` directory.

| Screenshot | Evidence |
|---|---|
| `28-rds-sydney-read-replica-available.png` | Primary and replica displayed as `Available` |
| `29-rds-replica-ssl-connection.png` | SSL/TLS connection and replica validation |
| `30-rds-replica-table-validation.png` | Table and row-count validation |
| `31-rds-primary-replication-test-cleanup.png` | Primary-side test insert and delete commands |
| `32-rds-replication-test-validation.png` | Test row appeared on the replica and disappeared after the delete replicated |

Review screenshots before publishing. Hide account identifiers, private endpoints, KMS key IDs, and other sensitive AWS console information.

## Related Documentation

- [`commands.md`](commands.md) — Step-by-step read replica creation and validation procedures.
- [`commands-with-output.md`](commands-with-output.md) — SQL commands and recorded outputs.
- [`screenshots/`](screenshots/) — Supporting evidence.

## Official AWS References

- [Working with PostgreSQL read replicas](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.Replication.ReadReplicas.html)
- [Creating a PostgreSQL read replica](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.Replication.ReadReplicas.Create.html)
- [Monitoring PostgreSQL read replicas](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.Replication.ReadReplicas.Monitoring.html)
- [Deleting an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_DeleteInstance.html)

## Outcome

A same-Region PostgreSQL read replica was created in Sydney, connected to using SSL/TLS, and validated through table and row-count checks. A temporary insert and subsequent delete on the primary were both observed on the replica, confirming replication of data changes.

The Hyderabad cross-Region attempt remains incomplete because the KMS key-loading error was not resolved. The final deletion status of the temporary Sydney replica must be verified before cleanup is described as complete.
