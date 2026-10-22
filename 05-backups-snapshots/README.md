# Lab 05 — Automated Backups and Manual Snapshots

## Overview

This lab covers Amazon RDS for PostgreSQL automated backups, backup retention, recovery-window inspection, and manual DB snapshots.

Automated backups support point-in-time recovery within the available retention window. Manual snapshots provide a retained recovery point that can be used to restore a DB instance.

## Objectives

- Review the automated backup configuration.
- Inspect the backup retention period and available recovery window.
- Create a manual DB snapshot.
- Monitor the snapshot until it becomes available.
- Understand how snapshots support database recovery.
- Review backup storage and resource cost considerations.

## Environment

| Component | Details |
|---|---|
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL engine version | 18.3 |
| Source DB instance | `database-1` |
| AWS Region | Sydney (`ap-southeast-2`) |
| Backup retention period | 1 day |
| Manual snapshot identifier | `airline-lab5-manual-snapshot` |
| Database | `airlinedb` |

## Step 1 — Review Automated Backups

1. Open the AWS Management Console.
2. Navigate to **Amazon RDS → Databases**.
3. Select `database-1`.
4. Review the backup settings.
5. Open **Automated backups** for the current Region.
6. Inspect the backup entry and the earliest and latest restorable times.

### Observed Configuration

- Automated backups were enabled.
- The backup retention period was set to 1 day.
- The Automated Backups page showed a backup entry in the current Region.
- The earliest and latest restorable times were visible in the console.
- The backup target showed AWS Cloud in Sydney.

## Step 2 — Understand the Retention Window

The backup retention period determines how far back RDS retains automated backup information for recovery.

A one-day retention period provides a rolling recovery window. It does not mean that RDS keeps only one independent full backup file.

RDS uses backup data and transaction logs to support point-in-time recovery within the available recovery window.

## Step 3 — Create a Manual Snapshot

1. Open **Amazon RDS → Snapshots**.
2. Select the **Manual** snapshots view.
3. Choose **Take snapshot** or the equivalent snapshot creation action.
4. Select `database-1` as the source DB instance.
5. Enter the snapshot identifier:

   `airline-lab5-manual-snapshot`

6. Review the details.
7. Create the snapshot.

## Step 4 — Monitor Snapshot Status

1. Return to the Manual snapshots list.
2. Locate `airline-lab5-manual-snapshot`.
3. Monitor its status.
4. Wait until the snapshot status becomes `Available`.
5. Record the snapshot details for validation.

### Recorded Observation

The manual snapshot reached `Available`. The console displayed a full snapshot size of approximately 3.09 GiB.

The displayed snapshot size is an observed console value, not an exact measure of the storage charge.

## Step 5 — Use the Snapshot for Recovery

The manual snapshot created in this lab was later used in Lab 06 to restore a separate DB instance.

The restored instance was validated using `psql`, SSL/TLS connectivity, table checks, and row-count queries.

After successful validation, the manual snapshot `airline-lab5-manual-snapshot` was deleted, as was the restored instance `airline-restore-from-snapshot`.

The deleted snapshot is no longer available for another restore. A new snapshot would be needed to repeat the snapshot-restore demonstration.

## Automated Backups and Manual Snapshots

| Feature | Automated backups | Manual snapshots |
|---|---|---|
| Main purpose | Support point-in-time recovery | Preserve a chosen recovery point |
| Retention | Controlled by the configured retention period | Remain until deleted, subject to AWS policies |
| Recovery | Restore to an available point within the recovery window | Restore from the snapshot |
| Management | RDS manages the automated backup process | User initiates and manages the snapshot lifecycle |

## Validation Results

- Automated backups were enabled for `database-1`.
- The backup retention period was verified as 1 day.
- The available recovery window was inspected.
- A manual snapshot was created and reached `Available`.
- The snapshot was used in the subsequent Lab 06 recovery exercise.
- The manual snapshot was deleted after recovery validation.

## Cost and Safety Considerations

- Manual snapshots can incur backup-storage charges while retained.
- Automated backup storage and other RDS resources may incur charges depending on usage and applicable AWS allowances.
- The displayed full snapshot size should not be treated as the exact cost.
- Stopping an RDS DB instance does not necessarily stop storage, backup-storage, or public IPv4 charges.
- Retain only snapshots that are needed for recovery or documented exercises.
- Review the source instance and snapshot identifiers carefully before deleting resources.

## Screenshots

Supporting evidence is stored in the `screenshots/` folder.

Use the screenshot index to identify the images showing the automated backup recovery window, manual snapshot creation, and snapshot availability. Review images before publishing to ensure they do not expose private AWS account details.

## Related Documentation

- [`commands.md`](commands.md) — Backup inspection and manual snapshot procedure.
- [`commands-with-output.md`](commands-with-output.md) — Recorded observations and results.
- [`screenshots/`](screenshots/) — Supporting evidence for the lab.

## References

- [Amazon RDS automated backups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html)
- [Restoring from a DB snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_RestoreFromSnapshot.html)
- [Amazon RDS point-in-time recovery](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html)

## Outcome

The automated backup configuration and recovery window were reviewed. A manual snapshot was created, monitored until available, used in the following recovery lab, and subsequently deleted after validation.
