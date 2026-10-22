# Lab 09 — PostgreSQL Read Replica: Commands and Procedures

## Overview

This document records the AWS Console procedures and PostgreSQL commands used to create and validate a same-Region Amazon RDS for PostgreSQL read replica.

The lab also records the unsuccessful attempt to configure a cross-Region replica in Hyderabad. The Hyderabad attempt returned a KMS security-token error, and no replica was created in that Region.

**Important:** The completed practical exercise was a same-Region Sydney read replica. Do not describe it as a successful Hyderabad cross-Region deployment.

## Environment

| Component | Details |
|---|---|
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL server version | 18.3 |
| PostgreSQL client | `psql` 18.4 |
| Primary instance | `database-1` |
| Read replica | `airline-sydney-replica` |
| AWS Region | Sydney (`ap-southeast-2`) |
| Database | `airlinedb` |
| Port | 5432 |
| Connection security | SSL/TLS |
| Certificate bundle | `global-bundle.pem` |

Do not publish live RDS endpoints, passwords, AWS account identifiers, or KMS key IDs. Replace endpoint values with placeholders in public commands.

# Part A — Inspect the Cross-Region Attempt

## Step 1 — Open the Read Replica Workflow

1. Sign in to the AWS Management Console.
2. Open **Amazon RDS → Databases**.
3. Select `database-1`.
4. Choose **Actions → Create read replica**.
5. Review the replica configuration.

## Step 2 — Select Hyderabad

1. Locate **Destination Region**.
2. Select **Asia Pacific (Hyderabad)** (`ap-south-2`).
3. Review the KMS key settings.

### Recorded observation

The console displayed this KMS key-loading error:

```text
Error loading KMS Keys
The security token included in the request is invalid
```

The error persisted after signing in again. The exact cause was not confirmed.

No Hyderabad replica was created. Do not assume that this error proves the Free plan or the Region itself was the cause.

# Part B — Create the Same-Region Sydney Replica

## Step 3 — Open the Workflow Again

1. Return to **Amazon RDS → Databases**.
2. Select the primary instance `database-1`.
3. Choose **Actions → Create read replica**.
4. Locate **Destination Region**.

## Step 4 — Select Sydney

Select:

`Asia Pacific (Sydney)` (`ap-southeast-2`)

The KMS error was no longer displayed after Sydney was selected.

## Step 5 — Configure the Replica

The following configuration was used during the completed exercise:

| Setting | Recorded value |
|---|---|
| Source instance | `database-1` |
| Replica identifier | `airline-sydney-replica` |
| Destination Region | Sydney (`ap-southeast-2`) |
| Instance class | `db.t4g.micro` |
| Allocated storage | 20 GiB |
| Storage type | General Purpose SSD (`gp2`) |
| KMS selection | Default `aws/rds` |
| Enhanced Monitoring | Disabled before creation |
| Storage autoscaling | Disabled before creation |

Review the current pricing, encryption, networking, and security settings before creating a replica in another environment.

## Step 6 — Create and Verify the Replica

1. Review the configured replica identifier and source database.
2. Confirm that the destination Region is Sydney.
3. Review the estimated charges and selected settings.
4. Choose **Create read replica**.
5. Open **RDS → Databases**.
6. Wait until `airline-sydney-replica` becomes `Available`.

### Recorded result

The RDS console displayed a successful replica creation message. Both `database-1` and `airline-sydney-replica` were shown as `Available`.

# Part C — Connect to the Read Replica

## Step 7 — Connect Using Windows PowerShell

Open a new PowerShell window and run the following command pattern. Replace `<REPLICA_RDS_ENDPOINT>` locally with the actual endpoint from the replica's **Connectivity & security** tab.

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<REPLICA_RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

Enter the PostgreSQL password when prompted. Do not include the password in the command or documentation.

## Step 8 — Verify the SSL/TLS Connection

The connection should report successful SSL/TLS negotiation. During the completed exercise, the connection reported TLS 1.3.

Switch to the Airline database:

```sql
\c airlinedb
```

## Step 9 — Verify Replica Recovery Mode

Run the following query on the replica:

```sql
SELECT
    current_database(),
    current_user,
    pg_is_in_recovery();
```

During the recorded exercise, the query was run while connected to `mydb` and returned:

```text
 current_database | current_user | pg_is_in_recovery
------------------+--------------+-------------------
 mydb             | postgres     | t
(1 row)
```

A result of `t` for `pg_is_in_recovery()` indicates that the PostgreSQL server is operating in recovery mode, which is expected for a physical read replica.

This query does not independently identify the AWS DB instance identifier.

# Part D — Validate Database Structure and Data

## Step 10 — Verify the Six Tables

After switching to `airlinedb`, run:

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_schema IN ('airline', 'booking')
ORDER BY table_schema, table_name;
```

The recorded query returned these six tables:

- `airline.aircraft`
- `airline.airports`
- `airline.flights`
- `booking.bookings`
- `booking.passengers`
- `booking.tickets`

## Step 11 — Validate Row Counts

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

Recorded row counts:

| Table | Rows |
|---|---:|
| `airline.airports` | 3 |
| `airline.aircraft` | 2 |
| `airline.flights` | 2 |
| `booking.passengers` | 3 |
| `booking.bookings` | 3 |
| `booking.tickets` | 3 |

These counts matched the expected sample data during the recorded validation.

# Part E — Test Replication of an Insert

## Step 12 — Connect to the Primary Database

Open a separate PowerShell window. Replace `<PRIMARY_RDS_ENDPOINT>` locally with the actual endpoint for `database-1`.

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<PRIMARY_RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

Switch to the Airline database:

```sql
\c airlinedb
```

## Step 13 — Insert a Temporary Test Record on the Primary

**Run this statement on the primary database only. Do not execute it on the read replica.**

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

## Step 14 — Check the Insert on the Replica

Return to the separate session connected to `airline-sydney-replica` and run:

```sql
SELECT *
FROM airline.airports
WHERE airport_code = 'TST';
```

During the recorded test, the replica's airport query returned the temporary `TST` row.

The subsequent full-table query showed four airport rows, including the test record.

**Result:** The inserted row appeared on the replica, confirming that the insert was replicated.

# Part F — Test Replication of a Delete

## Step 15 — Delete the Test Record on the Primary

Return to the primary `database-1` session and run:

```sql
DELETE FROM airline.airports
WHERE airport_code = 'TST';
```

Recorded output:

```text
DELETE 1
```

## Step 16 — Verify the Delete on the Replica

In the replica session, run:

```sql
SELECT *
FROM airline.airports
WHERE airport_code = 'TST';
```

During the recorded exercise, the first full-table query after the primary-side deletion still showed the `TST` row. A later query showed only the original three airport rows.

The final airport data consisted of:

| Airport code | City | Country |
|---|---|---|
| MAA | Chennai | India |
| DEL | Delhi | India |
| DXB | Dubai | UAE |

**Result:** The temporary row disappeared from the replica after replication caught up. Both the insert and the delete were replicated successfully.

This demonstrates that changes may not appear on an asynchronous read replica immediately after a primary transaction completes.

# Part G — Cleanup

## Step 17 — Exit the PostgreSQL Sessions

After validation, leave the `psql` sessions:

```sql
\q
```

Repeat this in each session you opened.

## Step 18 — Verify the Replica's Deletion Status

The project record shows that the delete-confirmation screen for `airline-sydney-replica` was reached, but it does not conclusively confirm that the deletion finished.

To check:

1. Open **Amazon RDS → Databases** in Sydney.
2. Verify that `database-1` remains available.
3. Check whether `airline-sydney-replica` still exists.
4. If it exists and is no longer needed, select **only** `airline-sydney-replica`.
5. Review the deletion confirmation and proceed only after verifying the selected identifier.

Do not delete `database-1` as part of this lab.

A read replica can incur DB instance-hour and storage charges while it exists. Deleting a temporary replica helps stop those charges once deletion takes effect; review any remaining resources and charges separately.

# Final Validation Summary

| Validation | Recorded result |
|---|---|
| Hyderabad cross-Region attempt | Incomplete; KMS error persisted |
| Sydney same-Region replica creation | Successful |
| Replica status after creation | `Available` |
| SSL/TLS connection | Successful |
| `pg_is_in_recovery()` on replica | `t` |
| Expected tables | 6 present |
| Row counts | Matched expected sample data |
| Test insert replicated | Confirmed |
| Test delete replicated | Confirmed after replication caught up |
| Final replica deletion status | Must be verified |

## Related Files

- [`README.md`](README.md) — Lab overview and recorded results.
- [`commands-with-output.md`](commands-with-output.md) — SQL commands and observed outputs.
- [`screenshots/`](screenshots/) — Supporting evidence.

## Official AWS References

- [Working with PostgreSQL read replicas](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.Replication.ReadReplicas.html)
- [Creating a PostgreSQL read replica](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.Replication.ReadReplicas.Create.html)
- [Monitoring PostgreSQL read replicas](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.Replication.ReadReplicas.Monitoring.html)
- [Deleting an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_DeleteInstance.html)

## Security Note

Do not publish passwords, live RDS endpoints, AWS account identifiers, KMS key IDs, or other private details. Use placeholders in public commands and inspect screenshots before uploading them to GitHub.
