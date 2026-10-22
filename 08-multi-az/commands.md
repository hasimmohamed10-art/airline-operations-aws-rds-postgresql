# Lab 08 — Multi-AZ High Availability: Commands and Procedures

## Overview

This document records the AWS Console procedures used to inspect the current Amazon RDS PostgreSQL deployment and review the available Multi-AZ high-availability options.

This lab was performed through the AWS Management Console. Multi-AZ was not deployed, and no live failover test was performed.

## Environment

| Component | Details |
|---|---|
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL engine version | 18.3 |
| DB instance | `database-1` |
| AWS Region | Sydney (`ap-southeast-2`) |
| Instance class | `db.t4g.micro` |
| Observed deployment | Single-AZ |

## Step 1 — Open the RDS Database

1. Sign in to the AWS Management Console.
2. Open **Amazon RDS**.
3. Select **Databases**.
4. Select `database-1`.
5. Open the **Configuration** tab.
6. Review the deployment and Availability Zone details.

### Recorded configuration

| Setting | Observed value |
|---|---|
| DB instance | `database-1` |
| Deployment | Single-AZ |
| Multi-AZ | No |
| Secondary Availability Zone | Not configured |

The existing database was running as a Single-AZ deployment when inspected.

## Step 2 — Inspect the Multi-AZ Options

1. Open the RDS database creation or modification workflow.
2. Locate the availability and durability deployment settings.
3. Review the Multi-AZ deployment options shown by the console.
4. Read any restrictions or notices displayed.
5. Do not apply changes during this inspection.

### Recorded observation

The console displayed a Free plan restriction notice, and the Multi-AZ options were unavailable in the screen inspected.

No Multi-AZ configuration was applied. The source instance `database-1` was not replaced or deleted.

This is an observation from the environment used during the lab. It is not a claim that Multi-AZ is unavailable for every AWS account or RDS configuration.

## Step 3 — Understand Single-AZ and Multi-AZ

### Single-AZ

A Single-AZ deployment runs the DB instance in one Availability Zone. It does not provide the Multi-AZ standby architecture associated with a Multi-AZ DB instance deployment.

### Multi-AZ DB instance deployment

A Multi-AZ DB instance deployment uses a primary DB instance and a standby in another Availability Zone. RDS uses synchronous replication for the standby and can initiate failover when the primary becomes unavailable.

The standby is intended for high availability and is not normally used to serve read traffic.

### Multi-AZ DB cluster

A Multi-AZ DB cluster is a different architecture that includes a writer and readable standby instances. Availability depends on the selected engine and configuration.

These deployment types have different architecture, availability, and cost characteristics. They should not be treated as interchangeable.

## Step 4 — Reference Workflow for Multi-AZ Deployment

The following procedure is for an environment where the chosen Multi-AZ option is available. It was not executed in this lab.

1. Open **Amazon RDS → Databases**.
2. Select the intended DB instance or start the database creation workflow.
3. Locate the deployment availability settings.
4. Select a Multi-AZ option supported by the engine and instance configuration.
5. Review the Availability Zone, networking, storage, security, and pricing details.
6. Review the planned modification before applying it.
7. Apply the change only when its cost and operational impact are understood.
8. Wait for the DB instance to return to `Available`.
9. Open the **Configuration** tab.
10. Verify the deployed Multi-AZ configuration and Availability Zone details.

Do not apply these changes to a billable environment merely to reproduce documentation. Confirm current AWS plan eligibility and pricing first.

## Step 5 — Reference Failover Validation

The following workflow applies only when a Multi-AZ deployment exists and a failover test has been approved.

### Before failover

1. Confirm that the DB instance is `Available`.
2. Record the current deployment configuration.
3. Test a database connection through the RDS endpoint.
4. Run a simple read-only query.
5. Record the time and result.

Example validation query:

```sql
SELECT
    current_database(),
    current_user,
    inet_server_port(),
    pg_is_in_recovery();
```

This query reports the connected database, user, server port, and whether the connected PostgreSQL server is in recovery. It does not independently identify the AWS DB instance.

### After an approved failover test

1. Review the RDS event history for failover-related events.
2. Wait for the database to return to `Available`.
3. Reconnect through the RDS endpoint.
4. Repeat the read-only validation query.
5. Verify that the expected database and tables remain accessible.
6. Record the actual interruption and reconnection behaviour.

A failover can interrupt existing connections. Applications should be designed to reconnect and retry appropriate transactions.

**This lab did not execute these failover steps.** They are reference procedures, not recorded test results.

## Step 6 — Review Cost and Operational Impact

Before enabling Multi-AZ, review the current AWS pricing for the exact deployment type, instance class, Region, storage, and licensing requirements.

Consider:

- Additional DB instance or standby-related charges.
- Storage and backup charges.
- Potential interruptions during modifications or failover.
- Application reconnection behaviour.
- Account and Region eligibility.

Do not claim that Multi-AZ was deployed or tested unless the AWS Console and validation results confirm it.

## Validation Summary

| Validation item | Result |
|---|---|
| Existing RDS instance inspected | Completed |
| Single-AZ configuration reviewed | Completed |
| Multi-AZ options inspected | Completed |
| Free plan restriction observed | Recorded |
| Multi-AZ deployment created | Not completed |
| Failover test performed | Not performed |
| Post-failover database validation | Not applicable |

## Related Files

- [`README.md`](README.md) — Lab overview and recorded findings.
- [`commands-with-output.md`](commands-with-output.md) — Observed configuration and results.
- [`screenshots/`](screenshots/) — Supporting evidence.

## Official AWS References

- [Amazon RDS Multi-AZ deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
- [High availability for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZSingleStandby.html)
- [Modifying an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.Modifying.html)
