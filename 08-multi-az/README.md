# Lab 08 — Amazon RDS Multi-AZ High Availability

## Overview

This lab covers Amazon RDS Multi-AZ deployment concepts and the configuration options available for the Airline Operations PostgreSQL database.

The existing database instance, `database-1`, was inspected through the AWS Management Console. Its deployment configuration was reviewed, and the Multi-AZ deployment options were examined.

The console displayed a Free plan restriction notice, and the Multi-AZ options were unavailable in the configuration screen. Therefore, a Multi-AZ deployment and a live failover test were not completed.

## Objectives

- Review the current RDS database deployment configuration.
- Understand the purpose of Multi-AZ high availability.
- Inspect the Multi-AZ deployment options in the AWS Console.
- Understand the difference between a Single-AZ deployment and a Multi-AZ deployment.
- Record the observed configuration and the limitation encountered.
- Review a reference workflow for deploying and validating Multi-AZ when supported.

## Environment

| Component | Details |
|---|---|
| Cloud provider | Amazon Web Services (AWS) |
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL engine version | 18.3 |
| DB instance | `database-1` |
| AWS Region | Sydney (`ap-southeast-2`) |
| Instance class | `db.t4g.micro` |
| Deployment observed | Single-AZ |

## Step 1 — Review the Current Database Configuration

1. Open the AWS Management Console.
2. Navigate to **Amazon RDS → Databases**.
3. Select `database-1`.
4. Open the **Configuration** tab.
5. Review the deployment and Availability Zone details.

### Recorded configuration

| Setting | Observed value |
|---|---|
| DB instance | `database-1` |
| Deployment configuration | Single-AZ |
| Multi-AZ | No |
| Secondary Availability Zone | Not configured |

This confirms that the inspected database was configured as a Single-AZ deployment during the lab.

## Step 2 — Inspect Multi-AZ Deployment Options

1. Open the RDS database creation or modification workflow.
2. Locate the availability and durability deployment settings.
3. Review the available Multi-AZ options.
4. Record any restriction or warning displayed by the console.

### Observed result

The console displayed a Free plan restriction notice, and the Multi-AZ deployment options were unavailable in the screen inspected.

No Multi-AZ deployment was created. The existing `database-1` instance was not replaced.

This is a record of what the console showed during the lab. It should not be interpreted as a claim that Multi-AZ is universally unavailable for all AWS accounts or RDS configurations.

## Step 3 — Understand Single-AZ Deployment

In a Single-AZ deployment, the DB instance runs in one Availability Zone.

The deployment inspected in this lab had:

- A primary DB instance in the Sydney Region.
- No configured Multi-AZ standby.
- No Multi-AZ automatic failover capability from a standby instance.

A Single-AZ database can still have automated backups and recovery options, but these do not provide the same failover architecture as a Multi-AZ deployment.

## Step 4 — Understand Multi-AZ High Availability

Amazon RDS Multi-AZ deployment options provide higher availability by using infrastructure across Availability Zones.

For a Multi-AZ DB instance deployment, RDS maintains a standby in another Availability Zone and uses synchronous replication for the standby. The standby is intended for failover and is not normally used to serve read traffic.

If the primary becomes unavailable, RDS can fail over to the standby. The DB endpoint is designed to direct connections to the current primary after the failover process.

Failover can interrupt existing connections, so applications should be able to reconnect and retry appropriate transactions.

## Step 5 — Understand Multi-AZ Deployment Types

Amazon RDS provides different deployment configurations. Do not treat all Multi-AZ options as the same architecture.

| Deployment type | General purpose |
|---|---|
| Single-AZ DB instance | Runs the DB instance in one Availability Zone |
| Multi-AZ DB instance deployment | Uses a primary and standby across Availability Zones for high availability |
| Multi-AZ DB cluster | Uses a different cluster architecture with a writer and readable standby instances, subject to engine and configuration support |

The available deployment types depend on the database engine, configuration, Region, and account eligibility.

## Step 6 — Reference Workflow for a Multi-AZ Deployment

The following is a reference procedure for an environment where the chosen Multi-AZ option is available. It was not executed as part of this lab.

1. Open **Amazon RDS → Databases**.
2. Choose to create or modify the intended DB instance.
3. Select a Multi-AZ deployment option supported by the selected engine and instance configuration.
4. Review the Availability Zone, networking, storage, and security settings.
5. Review the applicable cost before applying the change.
6. Apply the configuration.
7. Wait for the database to return to `Available`.
8. Verify the Multi-AZ configuration in the database's Configuration tab.
9. If a failover test is part of an approved exercise, plan it carefully and record the interruption and recovery behaviour.
10. Confirm that the application or client can reconnect after failover.

Do not follow this procedure against a billable environment without understanding the costs and expected service interruption.

## Step 7 — Reference Failover Validation

The following checks can be used when a Multi-AZ deployment and failover test are actually performed.

### Before failover

- Confirm the DB instance is available.
- Record the Multi-AZ status and Availability Zone information.
- Verify connectivity through the RDS endpoint.
- Record the time and result of a test query.

### After an approved failover test

- Check the RDS event history for failover-related events.
- Confirm that the database returns to `Available`.
- Reconnect through the RDS endpoint.
- Verify that the database and expected tables remain accessible.
- Record any interruption or reconnection behaviour that was actually observed.

These validation steps are reference guidance only. This lab did not execute a Multi-AZ failover test.

## Validation Results

| Validation item | Result |
|---|---|
| Existing RDS instance inspected | Completed |
| Single-AZ configuration reviewed | Completed |
| Multi-AZ options inspected | Completed |
| Free plan restriction observed | Recorded |
| Multi-AZ deployment created | Not completed |
| Live failover test | Not performed |
| Post-failover validation | Not applicable |

## Cost and Safety Considerations

- Multi-AZ configurations can cost more than a comparable Single-AZ configuration.
- Review the current AWS pricing for the exact deployment type and Region before enabling it.
- A database modification or failover test may interrupt active connections.
- Do not enable paid resources simply to complete documentation.
- Keep the existing primary database separate from any temporary test resources.
- Never publish passwords, live endpoints, AWS account identifiers, or other private information in screenshots.

## Screenshots

The screenshot for this lab documents the Multi-AZ options inspected in the AWS Console.

- [`27-rds-multi-az-options.png`](screenshots/27-rds-multi-az-options.png) — Multi-AZ deployment options and the restriction shown during the inspection.

Review the screenshot before publishing and hide private account information if visible.

## Related Documentation

- [`commands.md`](commands.md) — Console inspection and reference validation procedures.
- [`commands-with-output.md`](commands-with-output.md) — Recorded configuration observations and results.
- [`screenshots/`](screenshots/) — Supporting evidence.

## Official AWS References

- [Amazon RDS Multi-AZ deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)
- [High availability for Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZSingleStandby.html)
- [Modifying an Amazon RDS DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Overview.DBInstance.Modifying.html)

## Outcome

The Single-AZ configuration of `database-1` was inspected, and the available Multi-AZ deployment options were reviewed. The console displayed a Free plan restriction, so Multi-AZ deployment and failover testing were not completed. The lab records the observed limitation and provides a reference workflow without claiming that high availability was deployed or tested.
