# Lab 09 — PostgreSQL Read Replica: Commands with Output

## Environment

| Component | Recorded value |
|---|---|
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL server version | 18.3 |
| PostgreSQL client | `psql` 18.4 |
| Primary DB instance | `database-1` |
| Read replica | `airline-sydney-replica` |
| AWS Region | Sydney (`ap-southeast-2`) |
| Database | `airlinedb` |
| Schemas | `airline`, `booking` |
| Port | 5432 |
| Connection security | SSL/TLS |

Live database endpoints, passwords, AWS account identifiers, and private key identifiers are excluded.

**Scope clarification:** A Hyderabad cross-Region replica was attempted but not created because the KMS key-loading error persisted. The successful practical test in this lab used a same-Region Sydney read replica.

## 1. Inspect the Hyderabad Cross-Region Attempt

### Recorded console error

During the attempt to select Hyderabad (`ap-south-2`) as the destination Region, the console displayed:

```text
Error loading KMS Keys
The security token included in the request is invalid
```

The workflow was retried after signing in again, but the same error persisted.

### Recorded result

- Hyderabad read replica: Not created.
- KMS key list: Failed to load.
- Root cause: Not confirmed.

This error is recorded as an observed result. It does not prove that the AWS Free plan or the Hyderabad Region itself was the cause.

## 2. Create the Sydney Read Replica

### Recorded configuration

| Setting | Value |
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

After Sydney was selected, the red KMS error was no longer displayed.

### Recorded console output

The AWS RDS console displayed the success message:

```text
Successfully created replica airline-sydney-replica in Asia Pacific (Sydney)
```

The database list then displayed:

| DB identifier | Role | Status | Engine |
|---|---|---|---|
| `database-1` | Primary | Available | PostgreSQL |
| `airline-sydney-replica` | Replica | Available | PostgreSQL |

**Result:** The same-Region Sydney read replica was created successfully.

## 3. Connect to the Read Replica

### Command

Run the following command in Windows PowerShell. Replace `<REPLICA_RDS_ENDPOINT>` locally with the real replica endpoint.

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<REPLICA_RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

### Recorded output

```text
Password for user postgres:

psql (18.4, server 18.3)
WARNING: Console code page (850) differs from Windows code page (1252)
         8-bit characters might not work correctly. See psql reference
         page "Notes for Windows users" for details.
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

mydb=>
```

The password was entered interactively and is not included in this document.

The Windows console code-page warning did not prevent the connection.

## 4. Verify the Replica Recovery State

### Query

The query was executed while connected to `mydb` on the replica:

```sql
SELECT current_database(), current_user, pg_is_in_recovery();
```

### Recorded output

```text
 current_database | current_user | pg_is_in_recovery
------------------+--------------+-------------------
 mydb             | postgres     | t
(1 row)
```

**Result:** `pg_is_in_recovery()` returned `t`, confirming that the PostgreSQL server was operating in recovery mode as a read replica.

This query reports PostgreSQL server state. It does not independently identify the AWS DB instance identifier.

## 5. Connect to the Airline Database

### Command

```sql
\c airlinedb
```

### Recorded output

```text
psql (18.4, server 18.3)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
You are now connected to database "airlinedb" as user "postgres".
```

**Result:** The session switched from `mydb` to `airlinedb` successfully over SSL/TLS.

## 6. Validate Table Presence

### Query

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_schema IN ('airline', 'booking')
ORDER BY table_schema, table_name;
```

### Recorded output

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

**Result:** All six expected tables were present on the replica.

## 7. Validate Table Row Counts

### Query

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

### Recorded output

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

**Result:** The row counts matched the expected sample data.

## 8. Insert a Temporary Test Row on the Primary

The insert was executed on the primary `database-1`, not on the read replica.

### Command

```sql
INSERT INTO airline.airports
    (airport_code, airport_name, city, country)
VALUES
    ('TST', 'Replication Test Airport', 'Test City', 'Test Country');
```

### Recorded output

```text
INSERT 0 1
```

**Result:** One temporary test row was inserted successfully on the primary.

## 9. Verify the Insert on the Replica

### Query

The following query was executed on the replica:

```sql
SELECT *
FROM airline.airports;
```

### Recorded output

```text
 airport_id | airport_code |            airport_name             |   city    |   country
------------+--------------+-------------------------------------+-----------+--------------
          1 | MAA          | Chennai International Airport       | Chennai   | India
          2 | DEL          | Indira Gandhi International Airport | Delhi     | India
          3 | DXB          | Dubai International Airport         | Dubai     | UAE
          4 | TST          | Replication Test Airport            | Test City | Test Country
(4 rows)
```

**Result:** The `TST` row appeared on the replica, confirming that the insert had replicated.

## 10. Delete the Temporary Test Row on the Primary

The delete was executed on the primary `database-1`.

### Command

```sql
DELETE FROM airline.airports
WHERE airport_code = 'TST';
```

### Recorded output

```text
DELETE 1
```

**Result:** One test row was deleted successfully on the primary.

## 11. Verify the Delete on the Replica

The airport table was queried again on the replica.

### Query

```sql
SELECT * FROM airline.airports;
```

### First recorded output after the delete

The first query still showed four rows, including `TST`.

This demonstrated that the delete was not visible on the replica at the instant of that query.

### Later recorded output

A subsequent query returned:

```text
 airport_id | airport_code |            airport_name             |  city   | country
------------+--------------+-------------------------------------+---------+---------
          1 | MAA          | Chennai International Airport       | Chennai | India
          2 | DEL          | Indira Gandhi International Airport | Delhi   | India
          3 | DXB          | Dubai International Airport         | Dubai   | UAE
(3 rows)
```

**Result:** The `TST` row disappeared from the replica after replication caught up.

The insert and delete changes were both successfully replicated. The temporary test row was not retained in the final three-row airport result.

## 12. Replication Test Summary

| Test | Primary result | Replica result |
|---|---|---|
| Insert `TST` | `INSERT 0 1` | `TST` appeared |
| Delete `TST` | `DELETE 1` | `TST` disappeared after replication caught up |
| Final airport count | 3 original airports | 3 original airports |

This test demonstrated asynchronous replication behaviour: a primary change may not be visible on the replica immediately after the primary transaction completes.

## 13. Cleanup Status

The replica delete-confirmation screen was reached, but the available project record does not conclusively confirm that deletion completed.

### Current verification required

| Resource | Required status |
|---|---|
| `database-1` | Keep; confirm it remains Available |
| `airline-sydney-replica` | Check whether deletion completed |

To verify cleanup, open **Amazon RDS → Databases** in Sydney.

If `airline-sydney-replica` still exists and is no longer needed, delete only that replica. Do not delete `database-1`.

A read replica can incur instance-hour and storage charges while it exists. Same-Region replication data transfer is not charged, but the replica's own instance and storage charges can still apply.

## Final Validation Summary

| Validation | Recorded result |
|---|---|
| Hyderabad cross-Region attempt | KMS error; no replica created |
| Sydney read replica creation | Successful |
| Replica status after creation | `Available` |
| SSL/TLS connection | Successful |
| `pg_is_in_recovery()` | `t` |
| Expected tables | 6 present |
| Row counts | Matched expected sample data |
| Insert replication | Confirmed |
| Delete replication | Confirmed after replication caught up |
| Final replica cleanup | Must be verified |

## Related Files

- [`README.md`](README.md) — Lab overview and validation results.
- [`commands.md`](commands.md) — Read replica creation and validation procedures.
- [`screenshots/`](screenshots/) — Supporting evidence.

## Security Note

Do not publish passwords, live RDS endpoints, AWS account identifiers, KMS key IDs, or other private details. Use placeholders in public commands and review screenshots before uploading them to GitHub.
