# Lab 04 — Custom PostgreSQL DB Parameter Group Commands

## Overview

This document records the AWS Console steps used to create a custom Amazon RDS PostgreSQL DB parameter group, associate it with the database instance, and inspect its status.

**Environment:**
- Database service: Amazon RDS for PostgreSQL
- Engine version: PostgreSQL 18.3
- DB instance: `database-1`
- AWS Region: Sydney (`ap-southeast-2`)
- Parameter group: `airline-postgres18-params`

No PostgreSQL SQL commands were required for this lab because parameter group creation and association were performed in the AWS Console.

## Step 1 — Open Parameter Groups

1. Sign in to the AWS Management Console.
2. Open **Amazon RDS**.
3. In the navigation pane, choose **Parameter groups**.
4. Review the available parameter groups.

## Step 2 — Create the Custom Parameter Group

1. Click **Create parameter group**.
2. Select the parameter group family that matches PostgreSQL 18.
3. Enter the parameter group name:

   `airline-postgres18-params`

4. Enter a description for the custom PostgreSQL parameter group.
5. Review the selected engine family and other details.
6. Create the parameter group.

**Important:** The parameter group family must match the database engine family. Do not select a parameter group intended for Aurora MySQL or another unrelated engine.

## Step 3 — Verify the Parameter Group

1. Return to **RDS → Parameter groups**.
2. Find `airline-postgres18-params`.
3. Open the parameter group.
4. Confirm that the parameter group exists and belongs to the PostgreSQL 18 family.

The parameter group was created for use with the existing RDS PostgreSQL instance.

## Step 4 — Open the Database Modification Page

1. Open **RDS → Databases**.
2. Select `database-1`.
3. Choose **Actions → Modify**.
4. Locate the DB parameter group setting.
5. Select `airline-postgres18-params`.
6. Review the modification summary before applying the change.

## Step 5 — Apply the Association

The parameter group association was applied to `database-1`.

After applying a modification, RDS can show a status such as `pending-reboot` when a change requires a database restart before it takes effect.

Do not assume that a reboot has occurred merely because the parameter group is associated.

## Step 6 — Inspect the Current Configuration

1. Return to **RDS → Databases**.
2. Select `database-1`.
3. Open the **Configuration** tab.
4. Locate the DB parameter group field.
5. Record the associated parameter group and its current status.

The parameter group association was recorded as `airline-postgres18-params`. The latest available notes do not confirm that the `pending-reboot` status was cleared, so verify the current status in the AWS Console before claiming that no reboot is required.

## Step 7 — Review Parameter Settings

1. Open **RDS → Parameter groups**.
2. Select `airline-postgres18-params`.
3. Review the available parameters and their apply types.
4. Distinguish between dynamic parameters and static parameters.
5. Do not modify parameter values unless there is a specific, reviewed requirement.

No custom parameter value change was recorded in this lab. In particular, `log_min_duration_statement` was not changed.

## Important Notes

- Dynamic parameters can generally take effect without a database restart, depending on the specific parameter and its context.
- Static parameters require a database restart before changed values take effect.
- Check each parameter's apply type and the RDS status before deciding whether a reboot is needed.
- A database reboot may interrupt application connections.
- Do not claim performance improvement without before-and-after measurements.
- No reboot completion should be documented unless verified in the current console.

## Related Files

- [`README.md`](README.md) — Lab overview and implementation steps.
- [`commands-with-output.md`](commands-with-output.md) — Recorded configuration observations and outputs.
- [`screenshots/`](screenshots/) — Supporting evidence.

## References

- [Amazon RDS DB parameter groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/parameter-groups-overview.html)
- [Modifying parameters in a DB parameter group](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.Modifying.html)
