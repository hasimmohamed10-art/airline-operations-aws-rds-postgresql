# Lab 05 — Automated Backups and Manual Snapshots: Commands and Procedures

## Overview

This document records the AWS Console procedures used to inspect automated backups, review the recovery window, create a manual DB snapshot, and verify its availability.

**Environment**
- Service: Amazon RDS for PostgreSQL
- DB instance: `database-1`
- AWS Region: Sydney (`ap-southeast-2`)
- Backup retention period: 1 day
- Manual snapshot identifier: `airline-lab5-manual-snapshot`

This lab was performed mainly through the AWS Management Console. No SQL commands were required to create or manage the RDS snapshots.

## Step 1 — Open the RDS Database

1. Sign in to the AWS Management Console.
2. Open **Amazon RDS**.
3. Choose **Databases** in the navigation pane.
4. Select `database-1`.
5. Review the database's current status and configuration.

Before performing any backup operation, confirm that the selected DB instance is the intended source.

## Step 2 — Review Automated Backup Settings

1. Select `database-1`.
2. Open the backup-related settings in the database configuration.
3. Confirm that automated backups are enabled.
4. Check the backup retention period.
5. Record the observed retention period.

### Recorded configuration

- Automated backups: Enabled
- Backup retention period: 1 day

A one-day retention period provides a rolling recovery window. It does not mean RDS retains only one independent full backup file.

## Step 3 — Inspect the Automated Backup Recovery Window

1. Open **Amazon RDS → Automated backups**.
2. Select the **Current Region** view.
3. Locate the backup entry associated with `database-1`.
4. Review the earliest restorable time.
5. Review the latest restorable time.
6. Record the values displayed by the console.

The available recovery window indicates the range of times for which point-in-time recovery can be requested.

The exact recovery window changes over time. Record the values shown during the actual inspection rather than copying old timestamps into new observations.

## Step 4 — Open Manual Snapshots

1. Navigate to **Amazon RDS → Snapshots**.
2. Select the **Manual** snapshots view.
3. Review the existing manual snapshots.
4. Confirm that `database-1` is the intended source before creating a new snapshot.

A manual snapshot preserves a database recovery point that can be used later to restore a DB instance.

## Step 5 — Create a Manual Snapshot

1. Select **Take snapshot** or the corresponding snapshot creation action.
2. Select `database-1` as the source DB instance.
3. Enter this snapshot identifier:

   `airline-lab5-manual-snapshot`

4. Review the snapshot details.
5. Start snapshot creation.

Do not delete the source database when creating a snapshot.

## Step 6 — Monitor Snapshot Creation

1. Return to **Amazon RDS → Snapshots → Manual**.
2. Locate `airline-lab5-manual-snapshot`.
3. Refresh the list as needed.
4. Wait until the snapshot status becomes `Available`.
5. Record the final status and other relevant details displayed by the console.

### Recorded observation

The snapshot reached `Available`. The console displayed a full snapshot size of approximately 3.09 GiB.

The displayed size is a console observation; it should not be treated as the exact storage charge.

## Step 7 — Use the Snapshot for Restore Validation

The manual snapshot created in this lab was subsequently used in Lab 06.

The restored instance was validated using:
- An SSL/TLS connection through `psql`
- Table-presence checks
- Row-count validation
- Review of sample airport and flight records

The restored instance and the manual snapshot were deleted after the recovery validation was completed.

The snapshot has been deleted and is no longer available for another restore. Create a new manual snapshot if the demonstration needs to be repeated.

## Step 8 — Review Snapshot and Backup Costs

Before retaining or creating snapshots, review the applicable AWS billing information.

Consider:
- The configured automated backup retention period
- The number and size of retained manual snapshots
- Backup-storage usage
- Any applicable free allowances or credits

Stopping an RDS instance does not necessarily stop charges for its provisioned storage, backup storage, or public IPv4 address.

## Step 9 — Security and Cleanup Checks

Before completing the lab:

1. Verify that the snapshot belongs to the correct source database.
2. Avoid publishing private AWS account information or live database endpoints.
3. Retain only the snapshots required for recovery or documentation.
4. Verify deletion status in the AWS Console before documenting a resource as deleted.
5. Keep the primary database and its automated backups unless there is a deliberate reason to change them.

## Related Files

- [`README.md`](README.md) — Lab overview and implementation results.
- [`commands-with-output.md`](commands-with-output.md) — Recorded observations and outputs.
- [`screenshots/`](screenshots/) — Supporting evidence.

## Official AWS References

- [Amazon RDS automated backups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html)
- [Restoring from a DB snapshot](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_RestoreFromSnapshot.html)
- [Amazon RDS point-in-time recovery](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html)
