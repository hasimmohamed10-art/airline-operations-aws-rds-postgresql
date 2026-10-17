# Lab 04 — Custom PostgreSQL DB Parameter Group

## Overview

This lab covers creating a custom DB parameter group for Amazon RDS for PostgreSQL and associating it with the existing database instance.

A DB parameter group provides engine configuration settings that control PostgreSQL database behavior.

## Objectives

- Identify the correct PostgreSQL parameter group family.
- Create a custom parameter group.
- Associate the custom parameter group with the RDS PostgreSQL instance.
- Verify the parameter group association and status.
- Understand the difference between dynamic and static parameters.

## Environment

| Component | Details |
|---|---|
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL engine version | 18.3 |
| DB instance | `database-1` |
| AWS Region | Sydney (`ap-southeast-2`) |
| Custom parameter group | `airline-postgres18-params` |
| Parameter group family | PostgreSQL 18 |

## Implementation Steps

### Step 1 — Open the Parameter Groups page

1. Open the AWS Management Console.
2. Navigate to **RDS → Parameter groups**.
3. Review the available parameter groups.

### Step 2 — Create a custom parameter group

1. Choose **Create parameter group**.
2. Select the PostgreSQL 18 parameter group family.
3. Enter `airline-postgres18-params` as the parameter group name.
4. Add a description identifying its purpose.
5. Create the parameter group.

The PostgreSQL family must match the database engine family. An unrelated Aurora or MySQL parameter group must not be selected.

### Step 3 — Associate the parameter group

1. Navigate to **RDS → Databases**.
2. Select `database-1`.
3. Choose **Modify**.
4. Locate the DB parameter group setting.
5. Select `airline-postgres18-params`.
6. Review the modification summary.
7. Apply the change when ready.

### Step 4 — Review the association status

Open the database's **Configuration** tab and inspect the DB parameter group field and status.

RDS may initially display `pending-reboot` after a parameter group association or modification. The status must be checked in the current console before deciding whether a reboot is required.

## Parameter Group Concepts

### Dynamic parameters

Dynamic parameters can generally be applied without restarting the database, although the applicable setting and context determine when a change takes effect.

### Static parameters

Static parameters require a database restart before their changed values take effect.

The parameter's AWS documentation and the RDS console indicate whether a parameter is dynamic or static and whether a reboot is required.

## Implementation Outcome

The custom parameter group `airline-postgres18-params` was created and associated with `database-1`.

This lab covers parameter group creation and association. It does not claim that PostgreSQL performance was improved or that custom parameter values were tuned.

## Screenshots

Supporting evidence is stored in the `screenshots/` folder. Review the available screenshots and use the filename index to identify the evidence for parameter group creation, association, and status.

## Cost and Safety Notes

- Review parameter changes before applying them.
- Understand whether a parameter requires a reboot.
- Rebooting an RDS instance can interrupt database connections.
- No performance improvement should be claimed without appropriate measurements.
- Do not publish AWS account identifiers, database endpoints, or other private information in screenshots.

## Related Documentation

- [`commands.md`](commands.md) — Console workflow and parameter group inspection notes.
- [`commands-with-output.md`](commands-with-output.md) — Recorded configuration observations.
- [`screenshots/`](screenshots/) — Supporting screenshots.

## References

- [Amazon RDS DB parameter groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/parameter-groups-overview.html)
- [Modifying parameters in a DB parameter group](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.Modifying.html)
