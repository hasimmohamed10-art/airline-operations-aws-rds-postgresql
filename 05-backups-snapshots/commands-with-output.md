# Lab 05 — Automated Backups and Manual Snapshots: Commands with Output

## Environment

- Database service: Amazon RDS for PostgreSQL
- PostgreSQL engine version: 18.3
- Source DB instance: `database-1`
- AWS Region: Sydney (`ap-southeast-2`)
- Backup retention period: 1 day
- Manual snapshot identifier: `airline-lab5-manual-snapshot`

This lab was performed through the AWS Management Console. The outputs below record the settings and results observed during the exercise; no SQL commands were required to create or manage RDS snapshots.

## 1. Review Automated Backup Configuration

### Procedure

1. Open **Amazon RDS → Databases**.
2. Select `database-1`.
3. Review the automated backup configuration.
4. Open **Automated backups → Current Region**.
5. Locate the backup entry for the source DB instance.

### Recorded output

| Setting | Recorded value |
|---|---|
| Source DB instance | `database-1` |
| Automated backups | Enabled |
| Backup retention period | 1 day |
| Backup target | AWS Cloud |
| Region | Sydney (`ap-southeast-2`) |
| Backup entry | One entry was shown in the Current Region view |

**Result:** Automated backups were enabled, and the retention period was verified as one day.

## 2. Inspect the Recovery Window

### Procedure

1. Open **Amazon RDS → Automated backups → Current Region**.
2. Select the backup entry associated with `database-1`.
3. Inspect the earliest restorable time.
4. Inspect the latest restorable time.

### Recorded output

The console displayed the earliest and latest restorable times for the source database.

The exact timestamps are not reproduced here because the recovery window changes over time. Refer to the relevant AWS Console screenshot for the values visible during the inspection.

**Result:** The available point-in-time recovery window was reviewed.

## 3. Create a Manual Snapshot

### Procedure

1. Open **Amazon RDS → Snapshots**.
2. Select the **Manual** snapshots view.
3. Start the snapshot creation process.
4. Select `database-1` as the source DB instance.
5. Enter the snapshot identifier `airline-lab5-manual-snapshot`.
6. Review the configuration and create the snapshot.

### Recorded configuration

| Setting | Value |
|---|---|
| Snapshot identifier | `airline-lab5-manual-snapshot` |
| Source DB instance | `database-1` |
| Region | Sydney (`ap-southeast-2`) |
| Snapshot type | Manual |

**Result:** The manual snapshot was created from `database-1`.

## 4. Monitor the Snapshot Status

### Procedure

1. Return to **Amazon RDS → Snapshots → Manual**.
2. Locate `airline-lab5-manual-snapshot`.
3. Refresh the snapshot list as needed.
4. Wait until the status becomes `Available`.
5. Review the snapshot details.

### Recorded output

| Field | Recorded value |
|---|---|
| Snapshot identifier | `airline-lab5-manual-snapshot` |
| Snapshot status | `Available` |
| Full snapshot size displayed by the console | Approximately 3.09 GiB |

**Result:** The snapshot reached `Available` and was ready to be used for the subsequent restore exercise.

The displayed full snapshot size is a console observation, not an exact storage charge.

## 5. Use the Snapshot for Restore Validation

The manual snapshot created in this lab was used in Lab 06 to restore a separate DB instance.

The restored instance was:
- `airline-restore-from-snapshot`

The restore was validated using:
- A `psql` connection over SSL/TLS
- Table-presence checks
- Row-count validation across the six Airline tables
- Inspection of airport and flight records

The restored instance and source manual snapshot were subsequently deleted after successful validation.

### Recorded outcome

| Item | Result |
|---|---|
| Snapshot used for restore | `airline-lab5-manual-snapshot` |
| Restored DB instance | `airline-restore-from-snapshot` |
| Database validation | Completed |
| Restored instance cleanup | Confirmed deleted |
| Manual snapshot cleanup | Confirmed deleted |

**Result:** The snapshot was used successfully in the subsequent restore demonstration and was deleted after validation.

The deleted snapshot is no longer available for another restore. A new manual snapshot would be needed to repeat the exercise.

## 6. Understand Backup Retention

### Recorded configuration

| Setting | Value |
|---|---|
| Automated backups | Enabled |
| Retention period | 1 day |

A one-day retention period represents a rolling recovery window. It does not mean that RDS retains one independent full backup file per day.

RDS combines backup data and transaction-log information to support point-in-time recovery within the available recovery window.

## 7. Cost and Cleanup Review

### Recorded observations

- The console displayed a full manual snapshot size of approximately 3.09 GiB.
- The snapshot was retained until it had been used for the Lab 06 restore validation.
- The snapshot was subsequently deleted.
- The restored DB instance was also deleted after validation.

### Cost considerations

- Manual snapshots can incur backup-storage charges while retained.
- Automated backup storage may also incur charges depending on usage and applicable AWS allowances.
- Stopping an RDS DB instance does not necessarily stop charges for provisioned storage, backup storage, or public IPv4 addresses.
- The displayed snapshot size must not be treated as an exact cost.
- Review AWS billing information before creating or retaining additional snapshots.

## Final Validation Summary

| Validation item | Recorded result |
|---|---|
| Automated backups | Enabled |
| Backup retention period | 1 day |
| Recovery window | Inspected |
| Manual snapshot created | Confirmed |
| Snapshot reached `Available` | Confirmed |
| Snapshot size displayed | Approximately 3.09 GiB |
| Snapshot used for Lab 06 restore | Confirmed |
| Restored database validation | Completed |
| Manual snapshot deleted | Confirmed |
| Restored instance deleted | Confirmed |

## Related Files

- [`README.md`](README.md) — Lab overview and implementation results.
- [`commands.md`](commands.md) — AWS Console procedures.
- [`screenshots/`](screenshots/) — Supporting evidence.

## Security Note

Do not publish AWS account identifiers, live RDS endpoints, passwords, or other private details. Review screenshots before uploading them to GitHub.
