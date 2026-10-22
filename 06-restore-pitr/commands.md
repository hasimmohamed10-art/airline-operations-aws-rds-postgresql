# Lab 06 — Snapshot Restore and Point-in-Time Recovery Commands

## Overview

This document records the console procedures and PostgreSQL commands used to restore the Airline Operations database from a manual snapshot and perform point-in-time recovery (PITR).

The lab contains two separate recovery exercises:

- Part A — Restore from a manual snapshot.
- Part B — Restore to a selected point in time using automated backups.

The AWS Console is used to initiate both recovery operations. PostgreSQL `psql` is used to connect to the restored instances and validate the databases.

## Environment

| Component | Details |
|---|---|
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL engine version | 18.3 |
| Source DB instance | `database-1` |
| AWS Region | Sydney (`ap-southeast-2`) |
| Database | `airlinedb` |
| Schemas | `airline`, `booking` |
| PostgreSQL client | `psql` 18.4 |
| Connection security | SSL/TLS |
| Certificate bundle | `global-bundle.pem` |

Use `<RESTORED_RDS_ENDPOINT>` or `<PITR_RDS_ENDPOINT>` as a placeholder in public documentation. Replace the placeholder locally when connecting; never publish live database endpoints or passwords.

# Part A — Restore from a Manual Snapshot

## Step 1 — Open the Manual Snapshot

1. Sign in to the AWS Management Console.
2. Open **Amazon RDS → Snapshots**.
3. Select the **Manual** snapshots view.
4. Locate `airline-lab5-manual-snapshot`.
5. Confirm that the snapshot is the intended recovery source.
6. Choose **Actions → Restore snapshot**.

The snapshot was created in Lab 05 from `database-1`.

## Step 2 — Configure the Restored DB Instance

Use the following details from the completed restore exercise:

| Setting | Recorded value |
|---|---|
| New DB instance identifier | `airline-restore-from-snapshot` |
| Instance class | `db.t3.micro` |
| Availability Zone preference | No preference |
| Observed Availability Zone | `ap-southeast-2b` |
| Public access | Enabled |

Review the networking, storage, and security settings before initiating a restore. Keep the security-group inbound rule restricted to the required client IP address.

## Step 3 — Start and Monitor the Restore

1. Review the restore configuration.
2. Start the restore operation.
3. Open **RDS → Databases**.
4. Select `airline-restore-from-snapshot`.
5. Monitor the instance status.
6. Wait until the status becomes `Available`.

A snapshot restore creates a separate DB instance. It does not replace the original `database-1` instance.

## Step 4 — Connect to the Restored Instance

Open Windows PowerShell:

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<RESTORED_RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

Enter the PostgreSQL password when prompted. Do not place the password in the command or documentation.

Switch to the Airline database:

```sql
\c airlinedb
```

## Step 5 — Verify the Expected Tables

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_schema IN ('airline', 'booking')
ORDER BY table_schema, table_name;
```

The recorded result contained these six tables:

- `airline.aircraft`
- `airline.airports`
- `airline.flights`
- `booking.bookings`
- `booking.passengers`
- `booking.tickets`

## Step 6 — Validate Row Counts

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

## Step 7 — Inspect Airport and Flight Records

```sql
SELECT * FROM airline.airports;
```

```sql
SELECT * FROM airline.flights;
```

The recorded validation confirmed three airport records and two flight records.

The flight records included AI101 and AI202. These queries were used to verify that the restored database contained the expected sample data.

## Step 8 — Snapshot Restore Cleanup

After the restored database was validated:

- `airline-restore-from-snapshot` was deleted.
- `airline-lab5-manual-snapshot` was deleted.

Both deletions were confirmed during the project.

The manual snapshot is no longer available for another restore. Create a new snapshot if the exercise needs to be repeated.

# Part B — Point-in-Time Recovery (PITR)

## Step 9 — Inspect the Automated Backup Recovery Window

1. Open **Amazon RDS → Automated backups**.
2. Select the **Current Region** view.
3. Locate the backup entry for `database-1`.
4. Review the earliest restorable time.
5. Review the latest restorable time.
6. Choose a target time within the available recovery window.

The source database had automated backups enabled with a one-day retention period.

The available recovery window changes over time. Always use the values shown in the current console when repeating this procedure.

## Step 10 — Select the Recovery Time

During the completed exercise, the console displayed a latest restorable time of October 8, 2026, 12:28:40 (UTC+05:30).

The selected custom recovery time was:

**October 8, 2026, 12:25:00 (UTC+05:30)**

This timestamp is the historical value recorded for this exercise. When performing a new PITR operation, select a time within the recovery window currently displayed by AWS.

## Step 11 — Configure Point-in-Time Recovery

1. Open **Amazon RDS → Automated backups**.
2. Locate the automated backup for `database-1`.
3. Choose **Actions → Restore to point in time**.
4. Select **Custom date and time**.
5. Enter the recovery time selected in Step 10.
6. Set the new DB instance identifier to `airline-pitr-recovery`.
7. Review the engine, instance class, storage, connectivity, and security settings.
8. Start the restore operation.

RDS creates a separate DB instance for PITR. The original `database-1` instance remains separate.

## Step 12 — Monitor the PITR Restore

1. Open **RDS → Databases**.
2. Locate `airline-pitr-recovery`.
3. Monitor its status.
4. Wait until the status becomes `Available`.

The PITR-restored instance reached `Available` during the completed exercise.

## Step 13 — Connect to the PITR-Restored Instance

Open Windows PowerShell:

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<PITR_RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

Enter the PostgreSQL password when prompted.

Switch to the Airline database:

```sql
\c airlinedb
```

The recorded connection used SSL/TLS.

## Step 14 — Validate PITR Table Presence

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_schema IN ('airline', 'booking')
ORDER BY table_schema, table_name;
```

The recorded result contained all six expected tables.

## Step 15 — Validate PITR Row Counts

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

## Step 16 — Inspect PITR Data

```sql
SELECT * FROM airline.flights;
```

```sql
SELECT * FROM airline.airports;
```

The recorded results showed:

- AI101 — departure airport ID 1, arrival airport ID 2, aircraft ID 1.
- AI202 — departure airport ID 2, arrival airport ID 3, aircraft ID 2.
- Airports MAA (Chennai), DEL (Delhi), and DXB (Dubai).

These queries verified that the expected sample records were present after the PITR restore.

They do not independently prove that every change was recovered at the exact target second. The chosen restore time and the observed validation results should be considered together.

# Part C — Cleanup and Cost Review

## Step 17 — Verify Temporary Instance Cleanup

The following cleanup was confirmed:

- `airline-restore-from-snapshot` was deleted.
- `airline-lab5-manual-snapshot` was deleted.

The deletion status of `airline-pitr-recovery` was not confirmed in the available project notes.

To check its status:

1. Open **Amazon RDS → Databases** in Sydney.
2. Confirm that `database-1` remains available.
3. Check whether `airline-pitr-recovery` still exists.
4. If it exists and is no longer needed, review the deletion settings and delete only that temporary instance.
5. Do not delete `database-1` as part of this lab.

Deleting a temporary recovery instance stops its DB instance-hour charges once deletion takes effect. Storage, retained backups, snapshots, and public IPv4 charges must be reviewed separately.

## Validation Summary

| Check | Result |
|---|---|
| Snapshot restore completed | Confirmed |
| Snapshot-restored database validated | Confirmed |
| PITR restore completed | Confirmed |
| PITR-restored database validated | Confirmed |
| Snapshot-restored instance deleted | Confirmed |
| Manual snapshot deleted | Confirmed |
| PITR instance cleanup | Must be verified |

## Related Documentation

- [`README.md`](README.md) — Lab overview and results.
- [`commands-with-output.md`](commands-with-output.md) — Commands and recorded outputs.
- [`screenshots/`](screenshots/) — Supporting evidence.

## Official AWS References

- [Restore a DB instance from a snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_RestoreFromSnapshot.html)
- [Restore a DB instance to a specified point in time](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html)
- [Amazon RDS backup and restore overview](https://docs.aws.amazon.com/AmazonRDS/latest/gettingstartedguide/managing-backup-restore.html)

## Security Note

Never publish passwords, live database endpoints, AWS account identifiers, or other private connection details. Use placeholders in public commands and review screenshots before uploading them to GitHub.
