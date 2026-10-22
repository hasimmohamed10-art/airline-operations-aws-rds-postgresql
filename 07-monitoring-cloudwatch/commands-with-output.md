# Lab 07 — Monitoring and CloudWatch: Commands with Output

## Environment

- Cloud service: Amazon Web Services (AWS)
- Database service: Amazon RDS for PostgreSQL
- PostgreSQL engine version: 18.3
- DB instance: `database-1`
- AWS Region: Sydney (`ap-southeast-2`)
- Database: `airlinedb`

**Execution method:** AWS Management Console.

This lab mainly involved reviewing console metrics, PostgreSQL logs, RDS events, and monitoring settings. Since these operations were performed in the AWS Console, the results below are recorded observations rather than SQL terminal output.

## 1. Open RDS Monitoring

### Procedure

1. Navigate to **Amazon RDS → Databases**.
2. Select `database-1`.
3. Open the **Monitoring** tab.
4. Review the available metric graphs.

### Recorded output

| Item | Result |
|---|---|
| DB instance | `database-1` |
| Monitoring page | Opened |
| Available metric graphs | Inspected |
| Monitoring screenshots | Captured |

**Result:** The RDS monitoring interface was inspected successfully.

## 2. Review CPU Utilization

### Procedure

1. Open the RDS Monitoring page.
2. Locate CPU utilization.
3. Review the graph and its displayed time range.
4. Capture the graph for documentation.

### Recorded output

| Item | Result |
|---|---|
| Metric | CPU utilization |
| Graph | Reviewed |
| Numeric CPU reading | Not recorded in the available notes |

**Result:** CPU utilization monitoring was reviewed. No specific CPU percentage or performance improvement is claimed.

## 3. Review Database Connections

### Procedure

1. Open the RDS Monitoring page or the relevant CloudWatch metric view.
2. Locate the database connections metric.
3. Review the displayed graph.
4. Capture the relevant screenshot.

### Recorded output

| Item | Result |
|---|---|
| Metric | Database connections |
| Graph | Reviewed |
| Numeric connection count | Not recorded in the available notes |

**Result:** The connection metric was inspected. No specific connection count is claimed.

## 4. Review Free Storage Space

### Procedure

1. Open the RDS or CloudWatch monitoring view.
2. Locate the free storage space metric.
3. Review the graph and time range.
4. Capture a screenshot.

### Recorded output

| Item | Result |
|---|---|
| Metric | Free storage space |
| Graph | Reviewed |
| Numeric storage reading | Not recorded in the available notes |

**Result:** The storage metric was reviewed. No specific storage value is claimed.

## 5. Review Freeable Memory

### Procedure

1. Open the RDS or CloudWatch monitoring view.
2. Locate the freeable memory metric.
3. Review the graph.
4. Capture a screenshot.

### Recorded output

| Item | Result |
|---|---|
| Metric | Freeable memory |
| Graph | Reviewed |
| Numeric memory reading | Not recorded in the available notes |

**Result:** The memory metric was inspected. No specific memory reading is claimed.

## 6. Review Read Latency

### Procedure

1. Open the RDS or CloudWatch monitoring view.
2. Locate the read latency metric.
3. Review the graph and time range.
4. Capture a screenshot.

### Recorded output

| Item | Result |
|---|---|
| Metric | Read latency |
| Graph | Reviewed |
| Numeric latency reading | Not recorded in the available notes |

**Result:** The read latency metric was reviewed. No specific latency value or performance claim is made.

## 7. Review CloudWatch RDS Metrics

### Procedure

1. Open the Amazon CloudWatch console.
2. Select the Sydney Region (`ap-southeast-2`).
3. Navigate to **Metrics**.
4. Locate the Amazon RDS metrics associated with `database-1`.
5. Open and review the relevant graphs.

### Recorded output

The following metric views were inspected and captured as screenshots:

- CPU utilization
- CPU utilization and database connections
- Database connections
- Free storage space
- Freeable memory
- Read latency
- RDS metrics overview

**Result:** CloudWatch metric views were inspected. The screenshots provide evidence of the displayed metric views; exact numeric readings were not recorded in the available notes.

## 8. Inspect PostgreSQL Log Files

### Procedure

1. Navigate to **Amazon RDS → Databases**.
2. Select `database-1`.
3. Open **Logs & events**.
4. Locate `error/postgres.log`.
5. Open the log and review the available entries.

### Recorded output

| Item | Observed value |
|---|---|
| Log file | `error/postgres.log` |
| Displayed file size | Approximately 7 kB |
| Last-written time observed | October 9, 2026, 21:44 (UTC+05:30) |
| Log severity reviewed | `LOG` |
| Messages observed | Messages about redirection to the logging collector |

The inspected excerpt did not show `ERROR` or `FATAL` entries.

**Result:** The PostgreSQL log file was opened and inspected. This observation applies only to the excerpt reviewed; it does not establish that no errors existed elsewhere in the logs.

## 9. Inspect Recent RDS Events

### Procedure

1. Select `database-1`.
2. Open **Logs & events**.
3. Review the **Recent events** section.
4. Inspect the available descriptions and timestamps.

### Recorded output

The event history reviewed during the lab included:

- Monitoring-related changes
- Database start events
- Database stop events
- Restart events
- Recovery-related events

**Result:** Recent RDS events were reviewed. Event descriptions alone were not treated as proof of the cause of an incident.

## 10. Inspect Database Insights Configuration

### Procedure

1. Open `database-1` in the RDS Console.
2. Review the monitoring configuration.
3. Record the displayed Database Insights and database counter metrics settings.

### Recorded output

| Setting | Recorded value |
|---|---|
| Database Insights | Standard |
| Database counter metrics | Enabled |
| Displayed retention period | 7 days |

**Result:** The monitoring configuration was inspected. These values represent observations made during the lab and should not be assumed to represent the current configuration without checking the console again.

## 11. Inspect Enhanced Monitoring

### Procedure

1. Open the DB instance modification page.
2. Locate the Enhanced Monitoring setting.
3. Inspect whether it is enabled.
4. Exit without applying changes.

### Recorded output

| Setting | Recorded value |
|---|---|
| Enhanced Monitoring | Disabled |

**Result:** Enhanced Monitoring remained disabled during this lab.

## 12. Inspect PostgreSQL Log Export Settings

### Procedure

1. Open **RDS → Databases**.
2. Select `database-1`.
3. Choose **Modify**.
4. Locate the log export configuration.
5. Inspect the PostgreSQL log export option.
6. Exit without applying changes.

### Recorded output

| Setting | Recorded value |
|---|---|
| PostgreSQL log export to CloudWatch Logs | Unchecked / disabled |

**Result:** The log export setting was reviewed but was not enabled. PostgreSQL logs were inspected through the RDS Console; this lab does not claim that they were exported to CloudWatch Logs.

## Final Validation Summary

| Validation item | Recorded result |
|---|---|
| RDS Monitoring page | Inspected |
| CPU utilization graph | Reviewed |
| Database connections graph | Reviewed |
| Free storage space graph | Reviewed |
| Freeable memory graph | Reviewed |
| Read latency graph | Reviewed |
| CloudWatch RDS metric views | Inspected |
| PostgreSQL log file | Inspected |
| Recent RDS events | Reviewed |
| Database Insights configuration | Inspected |
| Enhanced Monitoring | Disabled |
| PostgreSQL log exports | Disabled |
| Monitoring configuration changes | None recorded |

## Important Notes

- The numeric metric readings were not recorded in the available notes.
- No performance improvement or database incident resolution is claimed.
- The reviewed PostgreSQL log excerpt did not show errors, but this does not prove that errors were absent from all logs.
- CloudWatch log ingestion and storage, Enhanced Monitoring, and retention options can incur charges depending on configuration and usage.
- The settings documented here are historical observations from the lab.

## Related Files

- [`README.md`](README.md) — Lab overview and findings.
- [`commands.md`](commands.md) — Step-by-step monitoring and log inspection procedures.
- [`screenshots/`](screenshots/) — Supporting evidence.

## Security Note

Do not publish passwords, live RDS endpoints, AWS account identifiers, or other private information. Review every screenshot before uploading it to GitHub.
