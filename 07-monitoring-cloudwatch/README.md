# Lab 07 — Amazon RDS Monitoring, PostgreSQL Logs and CloudWatch

## Overview

This lab covers monitoring an Amazon RDS for PostgreSQL instance, reviewing database metrics, inspecting PostgreSQL log files, and examining recent RDS events.

The work was performed on the Airline Operations database using the AWS Management Console. Monitoring graphs, database logs, event history, and monitoring configuration were inspected to understand the available monitoring and troubleshooting information.

## Objectives

- Review Amazon RDS monitoring metrics.
- Inspect CPU utilization, database connections, free storage space, freeable memory, and read latency.
- Review PostgreSQL log files in the RDS console.
- Inspect recent database events.
- Check Database Insights and Enhanced Monitoring settings.
- Review the PostgreSQL log export configuration.
- Understand monitoring, logging, and cost considerations.

## Environment

| Component | Details |
|---|---|
| Cloud service | Amazon Web Services (AWS) |
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL engine version | 18.3 |
| DB instance identifier | `database-1` |
| AWS Region | Sydney (`ap-southeast-2`) |
| Database | `airlinedb` |
| Monitoring interface | Amazon RDS and Amazon CloudWatch |

## Step 1 — Open the RDS Monitoring Page

1. Sign in to the AWS Management Console.
2. Open **Amazon RDS → Databases**.
3. Select `database-1`.
4. Open the **Monitoring** tab.
5. Review the available metric graphs.

The monitoring interface provides visibility into database instance activity and resource utilization.

## Step 2 — Review RDS Monitoring Metrics

The following monitoring metrics were reviewed during this lab.

| Metric | Purpose |
|---|---|
| CPU utilization | Shows the percentage of allocated CPU capacity being used. |
| Database connections | Shows the number of database connections. |
| Free storage space | Shows the available storage space. |
| Freeable memory | Shows memory available for use by the operating system. |
| Read latency | Shows the time taken for read I/O operations. |

These metrics help a DBA investigate resource utilization, connection activity, storage capacity, memory pressure, and storage I/O behaviour.

No performance improvement or specific resource-utilization percentage is claimed because this lab did not record a before-and-after performance comparison.

## Step 3 — Open Amazon CloudWatch Metrics

1. Open the Amazon CloudWatch console in the Sydney Region.
2. Navigate to **Metrics**.
3. Locate the Amazon RDS metrics.
4. Find the metrics associated with `database-1`.
5. Open the relevant graphs and review their time ranges.

The following CloudWatch metric views were captured during the lab:

- CPU utilization
- CPU utilization and database connections
- Database connections
- Free storage space
- Freeable memory
- Read latency
- RDS metrics overview

These graphs provide supporting evidence of the monitoring information available for the RDS instance.

### Monitoring observations

The metrics were inspected and screenshots were captured. The screenshots document the available metric views; they should not be interpreted as proof of a performance issue unless a specific abnormal pattern was identified and investigated.

## Step 4 — Review PostgreSQL Log Files

1. Return to **Amazon RDS → Databases**.
2. Select `database-1`.
3. Open **Logs & events**.
4. Locate the PostgreSQL log files.
5. Open `error/postgres.log`.
6. Review the available log entries.

### Recorded observations

| Item | Observation |
|---|---|
| Log file | `error/postgres.log` |
| Displayed file size | Approximately 7 kB |
| Last-written time observed | October 9, 2026, 21:44 (UTC+05:30) |
| Log entries reviewed | PostgreSQL `LOG` severity messages |

The inspected excerpt included messages about redirection to the logging collector. The reviewed excerpt did not show `ERROR` or `FATAL` entries.

This observation applies only to the excerpt inspected. It does not establish that the database had no errors in other log entries or at other times.

## Step 5 — Review Recent RDS Events

1. Select `database-1` in the RDS console.
2. Open **Logs & events**.
3. Review the **Recent events** section.
4. Inspect the event descriptions and timestamps.

The displayed event history included monitoring changes and database lifecycle events such as start, stop, restart, and recovery events.

RDS events provide useful context when investigating database changes or availability. An event description alone does not establish the cause of an incident; it should be correlated with logs, metrics, and the operational timeline.

## Step 6 — Review Monitoring Configuration

The monitoring configuration was inspected through the RDS console.

### Recorded configuration

| Setting | Recorded value |
|---|---|
| Database Insights | Standard |
| Database counter metrics | Enabled |
| Displayed retention period | 7 days |
| Enhanced Monitoring | Disabled |
| PostgreSQL log exports to CloudWatch Logs | Unchecked / disabled |
| Encryption key shown | AWS-managed `aws/rds` |

These values describe the configuration observed during the lab. Check the current AWS Console before relying on them as the present configuration.

### Enhanced Monitoring

Enhanced Monitoring can provide additional operating-system-level metrics. It was disabled during this lab.

### PostgreSQL log exports

The log export configuration was reviewed, but PostgreSQL log exports to CloudWatch Logs were not enabled. The PostgreSQL log was inspected through the RDS console.

Therefore, this lab documents log inspection and export-setting review; it does not claim that PostgreSQL logs were successfully exported to CloudWatch Logs.

## Step 7 — Understand Monitoring and Logging

### Metrics

Metrics provide numerical measurements of instance activity and resource utilization. They help DBAs identify patterns and determine whether further investigation is needed.

### Logs

PostgreSQL logs provide event-level details that can help investigate database activity and errors. The value of a log entry depends on its context, severity, and surrounding events.

### Events

RDS events record notifications about database instance changes and other service activity. They help correlate operational changes with metrics and logs.

Using metrics, logs, and events together provides better troubleshooting context than reviewing any one source in isolation.

## Validation Results

| Validation item | Result |
|---|---|
| RDS Monitoring page | Inspected |
| CPU utilization graph | Reviewed |
| Database connections graph | Reviewed |
| Free storage space graph | Reviewed |
| Freeable memory graph | Reviewed |
| Read latency graph | Reviewed |
| PostgreSQL log file | Inspected |
| Recent RDS events | Reviewed |
| Database Insights configuration | Inspected |
| Enhanced Monitoring setting | Inspected |
| PostgreSQL log export setting | Inspected |
| Monitoring configuration changes | None recorded |

## Cost and Security Considerations

- Review the applicable AWS pricing before enabling additional monitoring or log exports.
- CloudWatch log ingestion, storage, and other monitoring features can incur charges depending on the configuration and usage.
- Enhanced Monitoring was left disabled during this lab.
- PostgreSQL log exports were left disabled during this lab.
- Keep monitoring retention periods appropriate for operational and troubleshooting requirements.
- Never publish AWS account identifiers, live database endpoints, passwords, or other private information in screenshots.

## Screenshots

Supporting evidence is stored in the `screenshots/` directory.

### RDS Monitoring and Logs

| Screenshot | Evidence |
|---|---|
| [RDS monitoring 01](screenshots/15-rds-monitoring-01.png) | RDS monitoring view |
| [RDS monitoring 02](screenshots/15-rds-monitoring-02.png) | Additional monitoring view |
| [RDS monitoring 03](screenshots/15-rds-monitoring-03.png) | Additional monitoring view |
| [RDS monitoring 04](screenshots/15-rds-monitoring-04.png) | Additional monitoring view |
| [RDS monitoring 05](screenshots/15-rds-monitoring-05.png) | Additional monitoring view |
| [PostgreSQL log files](screenshots/16-rds-postgresql-log-files.png) | Available PostgreSQL log files |
| [PostgreSQL log review](screenshots/16-rds-postgresql-log-review.png) | Inspected PostgreSQL log entries |
| [Recent RDS events](screenshots/17-rds-recent-events.png) | RDS event history |
| [Monitoring configuration](screenshots/18-rds-monitoring-configuration.png) | Database Insights and monitoring settings |
| [Log export settings](screenshots/19-rds-log-export-settings.png) | PostgreSQL log export configuration |

### CloudWatch Metrics

| Screenshot | Evidence |
|---|---|
| [CPU utilization](screenshots/20-cloudwatch-cpu-utilization.png) | CPU utilization metric |
| [CPU and connections](screenshots/21-cloudwatch-cpu-and-connections.png) | CPU utilization and database connections |
| [Database connections](screenshots/22-cloudwatch-database-connections.png) | Connection metric |
| [Free storage space](screenshots/23-cloudwatch-free-storage-space.png) | Available storage metric |
| [Freeable memory](screenshots/24-cloudwatch-freeable-memory.png) | Memory metric |
| [Read latency](screenshots/25-cloudwatch-read-latency.png) | Read latency metric |
| [RDS metrics overview](screenshots/26-cloudwatch-rds-metrics-overview.png) | CloudWatch metrics overview |

Review each image before publishing and hide any private AWS account information or live database endpoint.

## Related Documentation

- [`commands.md`](commands.md) — Step-by-step monitoring, log inspection, and event review procedures.
- [`commands-with-output.md`](commands-with-output.md) — Recorded configuration observations and outputs.
- [`screenshots/`](screenshots/) — Supporting evidence for this lab.

## Official AWS References

- [Monitoring Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MonitoringOverview.html)
- [Viewing Amazon RDS metrics](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/monitoring-cloudwatch.html)
- [Monitoring Amazon RDS log files](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.html)
- [Publishing Amazon RDS logs to CloudWatch Logs](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.Procedural.UploadtoCloudWatch.html)

## Outcome

RDS monitoring metrics, PostgreSQL log files, recent database events, and monitoring configuration were reviewed. The captured evidence documents the monitoring views and configuration observed during the lab. Enhanced Monitoring and PostgreSQL log exports remained disabled, and no performance improvement is claimed.
