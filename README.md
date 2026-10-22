# Airline Operations Database — AWS RDS PostgreSQL

A structured PostgreSQL DBA implementation on Amazon RDS, covering database connectivity, relational database design, access control, backup and recovery, monitoring, high availability concepts, and read replication.

## Project Overview

This project documents the implementation and validation of PostgreSQL database administration tasks using Amazon RDS and a sample Airline Operations database.

The project progresses from initial database connectivity and schema design to security, recovery, monitoring, and replication. Each lab includes implementation steps, commands, validation results, and supporting screenshots where available.

The repository distinguishes completed practical work from configurations that were inspected but not deployed.

## Technologies Used

- **Database:** PostgreSQL
- **Cloud database:** Amazon RDS for PostgreSQL
- **Cloud platform:** Amazon Web Services (AWS)
- **SQL client:** PostgreSQL `psql`
- **Client environment:** Windows and PowerShell
- **Secure connectivity:** SSL/TLS with certificate verification
- **Monitoring:** Amazon RDS monitoring and Amazon CloudWatch
- **Documentation:** Markdown and screenshots

## Current RDS Environment

| Component | Configuration |
|---|---|
| AWS Region | Sydney (`ap-southeast-2`) |
| PostgreSQL engine version | 18.3 |
| DB instance class | `db.t4g.micro` |
| Deployment | Single-AZ |
| Allocated storage | 20 GiB |
| Database names | `mydb`, `airlinedb` |
| PostgreSQL port | 5432 |
| Encryption | Enabled |
| Automated backup retention | 1 day |
| SSL/TLS | Verified using `sslmode=verify-full` |

Sensitive connection information is intentionally excluded. The repository does not publish database passwords, live RDS endpoints, AWS account identifiers, private IP addresses, or security-group IDs.

## Database Architecture

The Airline Operations database contains two schemas that separate airline operational information from booking information.

### Schema design

| Schema | Tables | Purpose |
|---|---|---|
| `airline` | `airports`, `aircraft`, `flights` | Airport, aircraft, and flight information |
| `booking` | `passengers`, `bookings`, `tickets` | Passenger bookings and ticket information |

The six tables use primary keys, uniqueness constraints, required fields, and foreign-key relationships.

### Architecture overview

```text
Windows / PowerShell
        |
        | PostgreSQL psql
        | SSL/TLS - TCP 5432
        v
Amazon RDS for PostgreSQL
        |
        v
    airlinedb
        |
        +---------------------+
        |                     |
        v                     v
  airline schema        booking schema
        |                     |
        +-- airports          +-- passengers
        +-- aircraft          +-- bookings
        +-- flights           +-- tickets
```

## Project Progress — Labs 01–09

| Lab | Topic | Status |
|---|---|---|
| [Lab 01](01-rds-architecture-connectivity/README.md) | RDS architecture, connectivity and SSL/TLS | Completed |
| [Lab 02](02-database-design/README.md) | Database design and sample data | Completed |
| [Lab 03](03-users-roles-security/README.md) | Users, roles and security | Completed |
| [Lab 04](04-custom-parameter-group/README.md) | Custom DB parameter group | Created and associated; configuration documented |
| [Lab 05](05-backups-snapshots/README.md) | Automated backups and manual snapshots | Completed |
| [Lab 06](06-restore-pitr/README.md) | Snapshot restore and point-in-time recovery | Recovery validation completed |
| [Lab 07](07-monitoring-cloudwatch/README.md) | Monitoring, PostgreSQL logs and CloudWatch | Inspection completed |
| [Lab 08](08-multi-az/README.md) | Multi-AZ high availability | Options inspected; deployment not performed |
| [Lab 09](09-read-replica/README.md) | Read replica and replication validation | Same-Region replication tested successfully |

### Lab 01 — RDS Architecture, Connectivity and SSL/TLS

Created the RDS PostgreSQL instance and established a secure connection from Windows using `psql` and SSL/TLS certificate verification.

### Lab 02 — Database Design and Sample Data

Created the Airline Operations database with two schemas and six relational tables. Validated table presence, row counts, flight routes, bookings, passengers, and tickets.

### Lab 03 — Users, Roles and Security

Configured read-only, application-login, and administrative roles. Validated role membership, effective table privileges, and rejected write operations.

### Lab 04 — Custom DB Parameter Group

Created and associated the custom PostgreSQL 18 parameter group `airline-postgres18-params`. The configuration and association observations are documented in the lab.

### Lab 05 — Automated Backups and Manual Snapshots

Reviewed the automated backup retention period and recovery window. Created a manual snapshot, verified that it became available, and used it in the following restore exercise.

### Lab 06 — Snapshot Restore and PITR

Restored a DB instance from a manual snapshot and performed a separate point-in-time recovery operation. Validated the restored tables, sample data, and row counts.

### Lab 07 — Monitoring and CloudWatch

Reviewed RDS monitoring graphs, CloudWatch metric views, PostgreSQL log files, recent RDS events, and monitoring configuration.

Enhanced Monitoring and PostgreSQL log exports to CloudWatch Logs remained disabled during the documented inspection.

### Lab 08 — Multi-AZ High Availability

Inspected the Single-AZ configuration and the Multi-AZ deployment options. The console displayed a Free plan restriction, and Multi-AZ deployment was not performed. No live failover test is claimed.

### Lab 09 — Read Replica and Replication Validation

Created a same-Region read replica in Sydney. Validated its PostgreSQL recovery state, table structure, and row counts. A temporary airport record was inserted and deleted on the primary; both changes were observed on the replica after replication caught up.

A separate attempt to create a cross-Region replica in Hyderabad encountered a KMS security-token error. Hyderabad replication was not completed, and the root cause was not confirmed.

**Resource cleanup note:** Verify the current deletion status of `airline-sydney-replica` and the temporary PITR recovery instance in the AWS Console before considering cleanup complete. Keep the primary `database-1` unless there is a deliberate reason to remove it.

## Documentation Structure

Each lab uses a consistent structure:

- `README.md` — Objective, environment, procedures, results, and notes
- `commands.md` — Step-by-step commands and console procedures
- `commands-with-output.md` — Commands paired with outputs or recorded observations
- `screenshots/` — Supporting visual evidence

### Repository layout

```text
airline-operations-aws-rds-postgresql/
|
+-- README.md
|
+-- 01-rds-architecture-connectivity/
|   +-- README.md
|   +-- commands.md
|   +-- commands-with-output.md
|   +-- screenshots/
|
+-- 02-database-design/
|   +-- README.md
|   +-- commands.md
|   +-- commands-with-output.md
|   +-- screenshots/
|
+-- 03-users-roles-security/
|   +-- README.md
|   +-- commands.md
|   +-- commands-with-output.md
|   +-- screenshots/
|
+-- 04-custom-parameter-group/
|   +-- README.md
|   +-- commands.md
|   +-- commands-with-output.md
|   +-- screenshots/
|
+-- 05-backups-snapshots/
|   +-- README.md
|   +-- commands.md
|   +-- commands-with-output.md
|   +-- screenshots/
|
+-- 06-restore-pitr/
|   +-- README.md
|   +-- commands.md
|   +-- commands-with-output.md
|   +-- screenshots/
|
+-- 07-monitoring-cloudwatch/
|   +-- README.md
|   +-- commands.md
|   +-- commands-with-output.md
|   +-- screenshots/
|
+-- 08-multi-az/
|   +-- README.md
|   +-- commands.md
|   +-- commands-with-output.md
|   +-- screenshots/
|
+-- 09-read-replica/
    +-- README.md
    +-- commands.md
    +-- commands-with-output.md
    +-- screenshots/
```

## Cost and Security Practices

- Use appropriately sized resources for the required database tasks.
- Review pricing before creating additional DB instances or enabling chargeable features.
- Keep security-group rules restricted to the required client IP addresses.
- Use SSL/TLS certificate verification for client connections.
- Avoid publishing database passwords and private connection details.
- Delete temporary recovery and replication resources when they are no longer needed, after verifying their identifiers.
- Keep completion claims aligned with recorded evidence and actual configuration.

## Important Scope Notes

- Multi-AZ deployment and live failover testing were not completed in Lab 08.
- The successful read replica in Lab 09 was in the same Region as the primary.
- Hyderabad cross-Region replication remains incomplete.
- Monitoring screenshots document inspected metric views; no unmeasured performance improvement is claimed.
- Temporary instance cleanup must be checked against the current AWS Console before it is reported as complete.

## Official AWS References

- [Amazon RDS for PostgreSQL](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_PostgreSQL.html)
- [Amazon RDS backup and restore](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithAutomatedBackups.html)
- [Point-in-time recovery](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PIT.html)
- [Monitoring Amazon RDS](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/MonitoringOverview.html)
- [PostgreSQL read replicas](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_PostgreSQL.Replication.ReadReplicas.html)
- [Amazon RDS Multi-AZ deployments](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZ.html)

## Project Summary

This repository documents PostgreSQL DBA implementation on Amazon RDS, from initial secure connectivity through relational data design, role-based access, backup and recovery, monitoring, and read replication.

The documentation separates completed tests from uncompleted configurations and reference workflows. It is intended to provide clear evidence of the database administration tasks performed and the results validated.

## Author & Contact

**Mohamed Hasim Badhusha S**  
PostgreSQL DBA | Database Reliability Engineer

- **LinkedIn:** [Mohamed Hasim Badhusha S.](https://www.linkedin.com/in/mohamed-hasim-badhusha-s-ab386a35a/)
- **GitHub:** [hasimmohamed10-art](https://github.com/hasimmohamed10-art)
- **Gmail:** [Contact by email](mailto:hasimmohamed10@gmail.com)

