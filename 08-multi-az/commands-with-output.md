# Lab 08 — Multi-AZ High Availability: Commands with Output

## Environment

- Database service: Amazon RDS for PostgreSQL
- PostgreSQL engine version: 18.3
- DB instance: `database-1`
- AWS Region: Sydney (`ap-southeast-2`)
- Instance class: `db.t4g.micro`
- Deployment observed: Single-AZ

This lab was performed through the AWS Management Console. The outputs below document the configuration and observations recorded during the inspection. A Multi-AZ deployment and a live failover test were not performed.

## 1. Inspect the Current DB Instance Configuration

### Procedure

1. Open **Amazon RDS → Databases**.
2. Select `database-1`.
3. Open the **Configuration** tab.
4. Review the deployment configuration and Availability Zone details.

### Recorded output

| Setting | Observed value |
|---|---|
| DB instance | `database-1` |
| AWS Region | Sydney (`ap-southeast-2`) |
| Instance class | `db.t4g.micro` |
| Deployment configuration | Single-AZ |
| Multi-AZ | No |
| Secondary Availability Zone | Not configured |

**Result:** The inspected DB instance was configured as Single-AZ.

## 2. Inspect the Multi-AZ Deployment Options

### Procedure

1. Open the RDS database creation or modification workflow.
2. Locate the availability and durability deployment settings.
3. Review the Multi-AZ options displayed by the console.
4. Inspect any restriction notice.
5. Do not apply a configuration change as part of this inspection.

### Recorded output

| Item | Observation |
|---|---|
| Multi-AZ options | Inspected |
| Restriction notice | A Free plan restriction was displayed |
| Multi-AZ deployment created | No |
| Existing source instance modified for Multi-AZ | No |

The console displayed a Free plan restriction notice, and the Multi-AZ options were unavailable in the screen inspected.

**Result:** The available options were reviewed, but Multi-AZ was not deployed.

This records what was observed in the account during this lab. It does not establish that Multi-AZ is unavailable for every AWS account or RDS configuration.

## 3. Understand the Single-AZ Configuration

### Recorded configuration

| Feature | Observed state |
|---|---|
| Primary DB instance | `database-1` |
| Availability model | Single-AZ |
| Multi-AZ standby | Not configured |
| Multi-AZ failover test | Not performed |

A Single-AZ deployment runs the DB instance in one Availability Zone. Automated backups and recovery options can still be available, but they are different from maintaining a Multi-AZ standby for high availability.

## 4. Review the Multi-AZ Architecture

### Reference output — architecture concepts

The following table summarizes the deployment concepts reviewed for this lab. These are architectural descriptions, not results of a deployed Multi-AZ test.

| Deployment type | General behaviour |
|---|---|
| Single-AZ DB instance | Runs the DB instance in one Availability Zone |
| Multi-AZ DB instance deployment | Uses a primary and a standby in separate Availability Zones for high availability |
| Multi-AZ DB cluster | Uses a different cluster architecture with a writer and readable standby instances, subject to engine and configuration support |

For a Multi-AZ DB instance deployment, RDS uses synchronous replication for the standby. The standby is intended for failover and is not normally used to serve read traffic.

A standby-based failover can interrupt existing connections. Applications must be able to reconnect after the failover process.

## 5. Reference Workflow for Multi-AZ Deployment

The following steps describe a possible workflow for an environment where the desired Multi-AZ option is available. They were not executed during this lab.

1. Open **Amazon RDS → Databases**.
2. Select the intended instance or start the database creation workflow.
3. Locate the availability and durability deployment settings.
4. Select a supported Multi-AZ configuration.
5. Review networking, storage, security, and pricing.
6. Review the proposed change before applying it.
7. Apply the configuration only after confirming eligibility and cost.
8. Wait for the DB instance to return to `Available`.
9. Inspect the Configuration tab to confirm the deployed configuration.

### Recorded result

| Check | Result |
|---|---|
| Reference workflow documented | Yes |
| Multi-AZ configuration applied | No |
| New standby created and verified | No |

**Result:** The workflow is documented for reference only. No deployment result is claimed.

## 6. Reference Failover Validation

The following checks would be useful if a Multi-AZ deployment were available and a failover test were approved. They were not performed in this lab.

### Before failover

1. Confirm the DB instance is `Available`.
2. Record its current deployment configuration.
3. Connect through the RDS endpoint.
4. Run a read-only validation query.

Example query:

```sql
SELECT
    current_database(),
    current_user,
    inet_server_port(),
    pg_is_in_recovery();
```

### After an approved failover test

1. Review RDS events for failover-related notifications.
2. Wait for the DB instance to return to `Available`.
3. Reconnect through the RDS endpoint.
4. Repeat the read-only validation query.
5. Verify that the expected database and tables are accessible.
6. Record the actual reconnection and interruption behaviour.

### Recorded result

| Validation item | Result |
|---|---|
| Pre-failover connection test | Not performed in this lab |
| Failover initiated | No |
| Post-failover connection test | Not applicable |
| Failover interruption measured | Not measured |

The sample SQL query is provided as a reference command only. No output from a Multi-AZ failover test is available to record.

## 7. Review Cost and Operational Impact

### Recorded outcome

No Multi-AZ configuration changes were applied, and no live failover test was run.

Before attempting this workflow in another environment, review the current AWS pricing for the exact deployment type, Region, instance class, and storage configuration. Also consider the potential connection interruption during changes or failover.

## Final Validation Summary

| Validation item | Result |
|---|---|
| Existing RDS instance inspected | Completed |
| Single-AZ deployment confirmed | Confirmed |
| Multi-AZ options reviewed | Completed |
| Free plan restriction observed | Recorded |
| Multi-AZ deployment created | Not completed |
| Standby verified | Not applicable |
| Live failover test | Not performed |
| Post-failover database validation | Not applicable |

## Related Files

- [`README.md`](README.md) — Lab overview and recorded findings.
- [`commands.md`](commands.md) — Console inspection and reference procedures.
- [`screenshots/`](screenshots/) — Supporting evidence.

## Security Note

Do not publish passwords, live database endpoints, AWS account identifiers, or other private connection details. Review screenshots before uploading them to GitHub.
