# Lab 06 — Snapshot Restore and PITR: Commands with Output

## Environment

- Database service: Amazon RDS for PostgreSQL
- PostgreSQL server version: 18.3
- PostgreSQL client: `psql` 18.4
- Source DB instance: `database-1`
- AWS Region: Sydney (`ap-southeast-2`)
- Database: `airlinedb`
- Schemas: `airline`, `booking`
- Connection security: SSL/TLS

This lab contains two recovery exercises: restoring from a manual snapshot and performing point-in-time recovery (PITR).

Live database endpoints and passwords are excluded.

# Part A — Restore from a Manual Snapshot

## 1. Restore the Snapshot

### Procedure

1. Open **Amazon RDS → Snapshots → Manual**.
2. Select `airline-lab5-manual-snapshot`.
3. Choose **Actions → Restore snapshot**.
4. Set the new instance identifier to `airline-restore-from-snapshot`.
5. Review the configuration and start the restore.
6. Wait until the restored instance becomes `Available`.

### Recorded configuration

| Setting | Value |
|---|---|
| Snapshot identifier | `airline-lab5-manual-snapshot` |
| Restored instance | `airline-restore-from-snapshot` |
| Instance class | `db.t3.micro` |
| Availability Zone preference | No preference |
| Observed Availability Zone | `ap-southeast-2b` |
| Public access | Enabled |

The restored instance became available. It remained separate from the source instance `database-1`.

## 2. Connect to the Restored Database

### Command

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<RESTORED_RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

### Recorded output

```text
psql (18.4, server 18.3)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.
```

The connection succeeded using SSL/TLS. The Windows console also displayed a code-page warning during the connection; it did not prevent access.

Switch to the Airline database:

```sql
\c airlinedb
```

### Recorded output

```text
You are now connected to database "airlinedb" as user "postgres".
```

## 3. Validate Table Presence

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

**Result:** All six expected tables were present after the snapshot restore.

## 4. Validate Row Counts

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

**Result:** The restored database contained the expected sample row counts.

## 5. Inspect Restored Airport and Flight Data

### Queries

```sql
SELECT * FROM airline.airports;
SELECT * FROM airline.flights;
```

### Recorded result

The airport query returned three records:

| Airport code | Airport name | City | Country |
|---|---|---|---|
| MAA | Chennai International Airport | Chennai | India |
| DEL | Indira Gandhi International Airport | Delhi | India |
| DXB | Dubai International Airport | Dubai | UAE |

The flight query returned two records: `AI101` and `AI202`. Their departure, arrival, aircraft, and timestamp values matched the sample data documented in Lab 02.

**Result:** The snapshot-restored database contained the expected airport and flight records.

## 6. Snapshot Restore Cleanup

### Recorded outcome

| Resource | Result |
|---|---|
| `airline-restore-from-snapshot` | Deleted |
| `airline-lab5-manual-snapshot` | Deleted |

Both deletions were confirmed after validation. The manual snapshot is no longer available for another restore.

# Part B — Point-in-Time Recovery (PITR)

## 7. Inspect the Recovery Window

### Procedure

1. Open **Amazon RDS → Automated backups**.
2. Select **Current Region**.
3. Locate the backup for `database-1`.
4. Review the earliest and latest restorable times.
5. Choose a target time within the available recovery window.

### Recorded values

| Setting | Value observed during the lab |
|---|---|
| Latest restorable time | October 8, 2026, 12:28:40 (UTC+05:30) |
| Selected recovery time | October 8, 2026, 12:25:00 (UTC+05:30) |
| Source instance | `database-1` |
| Backup retention period | 1 day |

These timestamps describe the completed exercise. When repeating PITR, use the recovery window currently displayed in AWS.

## 8. Configure and Start PITR

### Procedure

1. Open **Amazon RDS → Automated backups → Current Region**.
2. Select the automated backup for `database-1`.
3. Choose **Actions → Restore to point in time**.
4. Select **Custom date and time**.
5. Enter the recovery time recorded above.
6. Set the new instance identifier to `airline-pitr-recovery`.
7. Review the configuration.
8. Start the restore and wait until the new instance becomes `Available`.

### Recorded result

The PITR operation completed, and `airline-pitr-recovery` reached `Available`.

RDS created a separate recovery instance; the source instance was not overwritten.

## 9. Connect to the PITR-Restored Database

### Command

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<PITR_RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

### Recorded output

The connection succeeded using `psql` 18.4 and SSL/TLS. The connection reported TLS 1.3.

Switch to the Airline database:

```sql
\c airlinedb
```

### Recorded output

```text
You are now connected to database "airlinedb" as user "postgres".
```

## 10. Validate PITR Table Presence

### Query

```sql
SELECT table_schema, table_name
FROM information_schema.tables
WHERE table_schema IN ('airline', 'booking')
ORDER BY table_schema, table_name;
```

### Recorded result

The query returned all six expected tables:

```text
airline.aircraft
airline.airports
airline.flights
booking.bookings
booking.passengers
booking.tickets
```

```text
(6 rows)
```

**Result:** All six expected tables were present in the PITR-restored database.

## 11. Validate PITR Row Counts

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

**Result:** The PITR-restored database returned the expected sample row counts.

## 12. Inspect PITR Airport Records

### Query

```sql
SELECT * FROM airline.airports;
```

### Recorded result

| airport_id | airport_code | airport_name | city | country |
|---:|---|---|---|---|
| 1 | MAA | Chennai International Airport | Chennai | India |
| 2 | DEL | Indira Gandhi International Airport | Delhi | India |
| 3 | DXB | Dubai International Airport | Dubai | UAE |

```text
(3 rows)
```

## 13. Inspect PITR Flight Records

### Query

```sql
SELECT * FROM airline.flights;
```

### Recorded result

The query returned two flight records:

| Flight | Departure airport ID | Arrival airport ID | Aircraft ID |
|---|---:|---:|---:|
| AI101 | 1 | 2 | 1 |
| AI202 | 2 | 3 | 2 |

The recorded timestamp values were displayed in UTC during the `psql` session.

**Result:** The expected airport and flight records were present after PITR.

These data checks validate the recovered database contents. They do not independently prove that every operation was recovered at the exact target second.

## 14. PITR Cleanup Status

### Recorded status

| Resource | Status |
|---|---|
| `database-1` | Source instance; keep |
| `airline-restore-from-snapshot` | Deleted |
| `airline-lab5-manual-snapshot` | Deleted |
| `airline-pitr-recovery` | Must be checked in the current AWS Console |

The available notes do not confirm that `airline-pitr-recovery` was deleted.

To verify cleanup:

1. Open **Amazon RDS → Databases** in Sydney.
2. Confirm `database-1` remains available.
3. Check whether `airline-pitr-recovery` still exists.
4. If it remains and is no longer needed, review the deletion settings and delete only that recovery instance.

Do not delete `database-1` or disable its automated backups as part of this lab.

## Final Validation Summary

| Validation | Result |
|---|---|
| Snapshot restore completed | Confirmed |
| Snapshot-restored table validation | Successful |
| Snapshot-restored row-count validation | Successful |
| PITR completed | Confirmed |
| PITR table validation | Successful |
| PITR row-count validation | Successful |
| Sample airport and flight records | Validated |
| Snapshot-restored instance cleanup | Confirmed |
| Manual snapshot cleanup | Confirmed |
| PITR instance cleanup | Requires verification |

## Related Files

- [`README.md`](README.md) — Lab overview and recovery results.
- [`commands.md`](commands.md) — Step-by-step restore and PITR procedures.
- [`screenshots/`](screenshots/) — Supporting evidence.

## Official AWS References

- [Restore a DB instance from a snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_RestoreFromSnapshot.html)
- [Restore a DB instance to a specified point in time](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html)
- [Amazon RDS backup and restore overview](https://docs.aws.amazon.com/AmazonRDS/latest/gettingstartedguide/managing-backup-restore.html)

## Security Note

Do not publish passwords, live RDS endpoints, AWS account identifiers, or other private connection details. Replace endpoints with placeholders in public commands and review screenshots before uploading them to GitHub.
