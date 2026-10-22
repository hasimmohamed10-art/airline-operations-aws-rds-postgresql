# Lab 07 — Monitoring and CloudWatch Commands

## Overview

This document records the AWS Console procedures used to inspect Amazon RDS monitoring metrics, review PostgreSQL log files, examine recent RDS events, and inspect monitoring configuration.

This lab was performed using the AWS Management Console. The main activities were console-based, so no SQL commands were required for monitoring metric or log inspection.

## Environment

| Component | Details |
|---|---|
| Cloud provider | Amazon Web Services |
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL engine version | 18.3 |
| DB instance | `database-1` |
| AWS Region | Sydney (`ap-southeast-2`) |
| Database | `airlinedb` |
| Monitoring service | Amazon CloudWatch |

## Step 1 — Open RDS Monitoring

1. Sign in to the AWS Management Console.
2. Open **Amazon RDS**.
3. Select **Databases**.
4. Select `database-1`.
5. Open the **Monitoring** tab.
6. Review the available metric graphs.

Record the metrics inspected and the time range displayed. Do not infer a performance issue from a graph without examining the values and time range.

## Step 2 — Review RDS Metrics

Inspect the following metrics where available:

- CPU utilization
- Database connections
- Free storage space
- Freeable memory
- Read latency

For each metric:

1. Open the relevant graph.
2. Review the displayed time range.
3. Observe the values and any visible trends.
4. Capture a screenshot if it provides useful evidence.
5. Avoid recording a numeric result unless it is clearly visible and can be read accurately from the graph.

### Metric interpretation

| Metric | What to inspect |
|---|---|
| CPU utilization | CPU usage over the selected period |
| Database connections | Connection activity and changes over time |
| Free storage space | Remaining storage capacity |
| Freeable memory | Memory available for use by the operating system |
| Read latency | Time taken for read I/O operations |

## Step 3 — Open CloudWatch Metrics

1. Open the Amazon CloudWatch console.
2. Ensure the selected Region is Sydney (`ap-southeast-2`).
3. Navigate to **Metrics**.
4. Find the Amazon RDS metric namespace or corresponding RDS metric view.
5. Locate metrics associated with `database-1`.
6. Open the relevant metric graphs.
7. Review the time range and displayed values.

During this lab, CloudWatch views for CPU utilization, database connections, free storage space, freeable memory, and read latency were inspected.

Capture screenshots of the metric views that are relevant to the lab. Hide private AWS account information before publishing screenshots.

## Step 4 — Open PostgreSQL Log Files

1. Return to **Amazon RDS → Databases**.
2. Select `database-1`.
3. Open **Logs & events**.
4. Locate the PostgreSQL log files.
5. Open `error/postgres.log`.
6. Review the available log entries.

### Recorded observations

- Log file: `error/postgres.log`
- Displayed size: approximately 7 kB
- Last-written time observed: October 9, 2026, at 21:44 (UTC+05:30)
- Reviewed entries included PostgreSQL `LOG` severity messages.

The reviewed excerpt showed messages about redirection to the logging collector. The inspected excerpt did not show `ERROR` or `FATAL` entries.

This applies only to the excerpt that was reviewed; it does not establish that no errors occurred elsewhere or at other times.

## Step 5 — Review Recent RDS Events

1. Select `database-1`.
2. Open **Logs & events**.
3. Review the **Recent events** section.
4. Read the event descriptions and timestamps.
5. Capture a screenshot of relevant events.

The reviewed event history included monitoring changes and database lifecycle events such as start, stop, restart, and recovery events.

Event descriptions provide context, but they do not necessarily establish the cause of an incident. Correlate events with metrics and logs before drawing conclusions.

## Step 6 — Inspect Database Insights Configuration

1. Open **RDS → Databases**.
2. Select `database-1`.
3. Choose **Modify** if needed to inspect the monitoring options.
4. Review the Database Insights and database counter metrics settings.
5. Record the currently displayed settings.
6. Exit without applying modifications unless a change has been specifically approved.

### Configuration observed during this lab

| Setting | Recorded value |
|---|---|
| Database Insights | Standard |
| Database counter metrics | Enabled |
| Displayed retention period | 7 days |

These values describe the configuration observed during the lab. Verify the current console before treating them as the current configuration.

## Step 7 — Inspect Enhanced Monitoring

1. Open the DB instance modification page.
2. Locate **Monitoring** or **Enhanced Monitoring**.
3. Review whether Enhanced Monitoring is enabled.
4. Record the setting.
5. Do not enable it as part of this lab.

**Recorded setting:** Enhanced Monitoring was disabled.

Enhanced Monitoring provides additional operating-system-level metrics. Review AWS pricing before enabling it.

## Step 8 — Inspect PostgreSQL Log Export Settings

1. Open **RDS → Databases**.
2. Select `database-1`.
3. Choose **Modify**.
4. Locate the log export configuration.
5. Inspect the PostgreSQL log export checkbox.
6. Record the setting.
7. Leave PostgreSQL log exports disabled for this exercise.
8. Exit without applying changes.

**Recorded setting:** PostgreSQL log exports to CloudWatch Logs were unchecked.

The PostgreSQL log was inspected through the RDS console. This lab does not claim that the logs were exported to CloudWatch Logs.

## Step 9 — Record Results

Record the following items in your lab notes:

- The monitoring metrics inspected
- The time range shown in the graphs
- The PostgreSQL log file reviewed
- Relevant log observations
- Recent RDS events inspected
- Database Insights configuration
- Enhanced Monitoring setting
- PostgreSQL log export setting

Separate direct observations from interpretations. Do not invent metric values or claim performance improvements that were not measured.

## Step 10 — Review Cost and Security

Before changing monitoring settings:

1. Review the applicable AWS pricing.
2. Consider CloudWatch log ingestion and storage charges if log export is enabled.
3. Consider any additional charges associated with enhanced monitoring or retention options.
4. Keep private AWS account information and live database endpoints out of public screenshots.
5. Do not publish passwords or other private connection details.

No monitoring configuration changes were recorded in this lab.

## Related Files

- [`README.md`](README.md) — Lab overview, observations, and validation results.
- [`commands-with-output.md`](commands-with-output.md) — Recorded configuration observations and outputs.
- [`screenshots/`](screenshots/) — Supporting evidence.

## Official AWS References

- [Monitoring Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MonitoringOverview.html)
- [Viewing Amazon RDS metrics](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/monitoring-cloudwatch.html)
- [Monitoring Amazon RDS log files](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.html)
- [Publishing Amazon RDS logs to CloudWatch Logs](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_LogAccess.Procedural.UploadtoCloudWatch.html)
