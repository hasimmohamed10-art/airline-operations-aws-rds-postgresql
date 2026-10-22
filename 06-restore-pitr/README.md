# Lab 06 — Snapshot Restore and Point-in-Time Recovery (PITR)

## Overview

This lab covers two Amazon RDS for PostgreSQL recovery methods: restoring a DB instance from a manual snapshot and restoring a DB instance to a selected point in time (PITR).

Both methods were performed using the Airline Operations database. After each restore, the restored instance was accessed using PostgreSQL `psql` over SSL/TLS, and the database structure and sample data were validated.

## Objectives

- Restore an RDS PostgreSQL DB instance from a manual snapshot.
- Restore a DB instance to a selected point in time using automated backups.
- Understand the difference between snapshot restore and PITR.
- Connect to restored instances using `psql` and SSL/TLS.
- Validate database tables, row counts, and sample records.
- Review the lifecycle and cost implications of temporary recovery instances.

## Environment

| Component | Details |
|---|---|
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL engine version | 18.3 |
| Source DB instance | `database-1` |
| AWS Region | Sydney (`ap-southeast-2`) |
| Database | `airlinedb` |
| Schemas | `airline`, `booking` |
| SQL client | `psql` 18.4 |
| Connection security | SSL/TLS |
| Manual snapshot used | `airline-lab5-manual-snapshot` |
| Snapshot-restored instance | `airline-restore-from-snapshot` |
| PITR-restored instance | `airline-pitr-recovery` |

## Part A — Restore from a Manual Snapshot

### Step 1 — Select the manual snapshot

1. Open the AWS Management Console.
2. Navigate to **Amazon RDS → Snapshots**.
3. Select the **Manual** snapshots view.
4. Locate `airline-lab5-manual-snapshot`.
5. Choose **Actions → Restore snapshot**.

The selected snapshot was created in Lab 05 from the source database instance `database-1`.

### Step 2 — Configure the restored instance

The following configuration was used during the restore:

| Setting | Recorded value |
|---|---|
| New DB instance identifier | `airline-restore-from-snapshot` |
| Instance class | `db.t3.micro` |
| Availability Zone preference | No preference |
| Observed Availability Zone | `ap-southeast-2b` |
| Public access | Enabled for the Windows client connection |

The restored instance was created separately from the source instance. The original `database-1` instance was not replaced.

### Step 3 — Wait for the instance to become available

1. Open **RDS → Databases**.
2. Locate `airline-restore-from-snapshot`.
3. Monitor its status.
4. Wait until the status becomes `Available`.

The restored instance reached `Available` and was ready for connection and validation.

### Step 4 — Connect to the restored instance

The connection was made from Windows PowerShell using PostgreSQL `psql` and the RDS CA certificate bundle.

Use the following pattern when reproducing the test. Replace the placeholder locally with the restored instance's endpoint; do not publish the live endpoint.

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<RESTORED_RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

After connecting, switch to the Airline database:

```sql
\c airlinedb
```

The connection succeeded using SSL/TLS.

### Step 5 — Validate the restored database

Check that all expected tables exist:

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_schema IN ('airline', 'booking')
ORDER BY table_schema, table_name;
```

The query returned six tables:

- `airline.aircraft`
- `airline.airports`
- `airline.flights`
- `booking.bookings`
- `booking.passengers`
- `booking.tickets`

### Step 6 — Validate row counts

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

### Snapshot restore result

The restored instance contained the six expected tables and the expected sample row counts. The database was also checked by reviewing its airport and flight records.

After validation, the restored instance `airline-restore-from-snapshot` and the source manual snapshot `airline-lab5-manual-snapshot` were deleted.

## Part B — Point-in-Time Recovery (PITR)

### Step 1 — Inspect the recovery window

1. Open **Amazon RDS → Automated backups**.
2. Select the **Current Region** view.
3. Locate the automated backup entry for `database-1`.
4. Review the earliest and latest restorable times.
5. Identify a target time within the displayed recovery window.

The source database had automated backups enabled with a retention period of one day.

### Step 2 — Select the target recovery time

During the lab, the console showed a latest restorable time of **October 8, 2026, 12:28:40 (UTC+05:30)**.

The selected custom recovery time was:

**October 8, 2026, 12:25:00 (UTC+05:30)**

This was within the recovery window displayed by the console at that time.

The recovery time must always be chosen from the currently available window when repeating the procedure. The historical timestamps above describe this completed exercise.

### Step 3 — Configure the PITR instance

1. Select the source backup for `database-1`.
2. Choose **Actions → Restore to point in time**.
3. Select **Custom date and time**.
4. Enter the chosen recovery timestamp.
5. Set the new DB instance identifier to `airline-pitr-recovery`.
6. Review the instance, storage, networking, and backup settings.
7. Start the restore.

RDS creates a new DB instance for PITR; it does not overwrite the original source instance.

### Step 4 — Monitor the restored instance

1. Navigate to **RDS → Databases**.
2. Locate `airline-pitr-recovery`.
3. Monitor its status.
4. Wait until the status becomes `Available`.

The PITR-restored instance reached `Available`.

### Step 5 — Connect and validate the PITR instance

Connect using the new instance's endpoint, PostgreSQL `psql`, and SSL/TLS.

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<PITR_RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

Then switch to the Airline database:

```sql
\c airlinedb
```

The connection succeeded, and the database was inspected using SQL queries.

### Step 6 — Verify the tables and row counts

The table-presence query returned all six expected tables. The row-count validation returned:

| Table | Rows |
|---|---:|
| `airline.airports` | 3 |
| `airline.aircraft` | 2 |
| `airline.flights` | 2 |
| `booking.passengers` | 3 |
| `booking.bookings` | 3 |
| `booking.tickets` | 3 |

The following queries were also used to inspect the recovered records:

```sql
SELECT * FROM airline.flights;
SELECT * FROM airline.airports;
```

The two flights were present:

- `AI101`: departure airport ID 1, arrival airport ID 2, aircraft ID 1.
- `AI202`: departure airport ID 2, arrival airport ID 3, aircraft ID 2.

The airport table contained MAA (Chennai), DEL (Delhi), and DXB (Dubai).

### PITR result

The PITR-restored instance reached `Available`, connected successfully over SSL/TLS, and returned the expected database tables, row counts, airport records, and flight records.

These checks validate the restored database contents. They do not independently prove that every database operation was recovered at the exact target second; the restore timestamp and the validation evidence should be considered together.

## Snapshot Restore vs. PITR

| Feature | Snapshot restore | Point-in-time recovery |
|---|---|---|
| Recovery source | A selected manual DB snapshot | Automated backups and transaction logs |
| Recovery target | The state represented by the selected snapshot | A specified recovery time within the available window |
| Result | A new DB instance | A new DB instance |
| Original source | Remains separate | Remains separate |
| Validation used | Tables, row counts, and sample records | Tables, row counts, and sample records |

## Cleanup and Cost Considerations

Temporary restored DB instances can incur instance-hour and storage charges while they exist.

The following cleanup was confirmed:

- `airline-restore-from-snapshot` was deleted.
- `airline-lab5-manual-snapshot` was deleted.

The cleanup status of `airline-pitr-recovery` must be verified in the current AWS Console before documenting it as deleted.

To verify cleanup:

1. Open **Amazon RDS → Databases** in Sydney.
2. Confirm that `database-1` remains available.
3. Check whether `airline-pitr-recovery` still exists.
4. If it exists and is no longer needed, review the deletion options and remove only that temporary instance.
5. Do not delete `database-1` or disable its automated backups as part of this lab.

Deleting a temporary recovery instance helps stop its DB instance charges once deletion takes effect. Storage, snapshots, backups, and public IPv4 charges must be reviewed separately.

## Screenshots

Supporting screenshots are stored in the `screenshots/` folder.

| Screenshot | Evidence |
|---|---|
| `06-restore-from-snapshot-configuration.png` | Snapshot restore configuration |
| `07-restored-instance-available.png` | Snapshot-restored instance status |
| `08-restored-database-connection-validation.png` | Database connection validation |
| `08-restored-database-validation.png` | Table validation after snapshot restore |
| `06-restore-deployment-options-inspection.png` | Restore configuration inspection |
| `09-pitr-available-recovery-window.png` | Available recovery window |
| `06-pitr-custom-restore-time.png` | Selected custom PITR time |
| `06-pitr-target-instance-configuration.png` | PITR instance configuration |
| `09-pitr-additional-configuration.png` | PITR configuration details |
| `10-pitr-restored-instance-available.png` | PITR-restored instance status |
| `11-pitr-database-validation.png` | PITR table validation |
| `12-pitr-row-validation.png` | PITR row-count validation |
| `13-pitr-table-data-validation.png` | PITR airport and flight data validation |

Review screenshots before publishing to ensure they do not expose private AWS account details or live database endpoints.

## Related Documentation

- [`commands.md`](commands.md) — Step-by-step snapshot restore and PITR procedures.
- [`commands-with-output.md`](commands-with-output.md) — SQL commands and recorded outputs.
- [`screenshots/`](screenshots/) — Supporting evidence for the lab.

## Official AWS References

- [Restore a DB instance from a snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_RestoreFromSnapshot.html)
- [Restore a DB instance to a specified point in time](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html)
- [Amazon RDS backup and restore overview](https://docs.aws.amazon.com/AmazonRDS/latest/gettingstartedguide/managing-backup-restore.html)

## Outcome

Both snapshot restore and point-in-time recovery were performed on Amazon RDS for PostgreSQL. The two restored databases were connected to and validated using table checks, row counts, and sample data queries. The snapshot-restored instance and its manual snapshot were deleted after validation. The final cleanup status of the PITR instance remains to be checked.
