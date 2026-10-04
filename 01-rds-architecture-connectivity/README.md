# RDS PostgreSQL Architecture, Connectivity & Security

## Overview

This implementation establishes and validates an Amazon RDS for PostgreSQL environment, covering network placement, endpoint connectivity, security group access, and SSL/TLS connectivity.

## Scope

- Amazon RDS for PostgreSQL architecture
- VPC and DB subnet placement
- Availability Zone
- Security Group access control
- Public accessibility and PostgreSQL port
- SSL/TLS client connectivity
- Primary database status validation

## Environment

| Component | Configuration |
|---|---|
| Cloud service | Amazon RDS for PostgreSQL |
| Engine | PostgreSQL 18.3-R2 |
| DB instance class | `db.t4g.micro` |
| Deployment | Single-AZ |
| Storage | 20 GiB General Purpose SSD |
| Database | `mydb` |
| Port | `5432` |
| Region | Asia Pacific (Sydney) |
| Availability Zone | `ap-southeast-2c` |
| Public access | Enabled |
| Network type | IPv4 |
| Encryption | Enabled |

> Resource identifiers, endpoints, IP addresses, credentials, and other account-specific values are intentionally omitted or masked from public documentation.

## Architecture

```text
+-----------------------------+
| Windows PostgreSQL Client   |
| psql 18.4                   |
+-------------+---------------+
              |
           SSL/TLS
              |
              v
+-----------------------------+
| AWS VPC                     |
|                             |
|  Security Group : TCP 5432  |
|  Client IP restricted /32   |
|              |              |
|              v              |
|  +-----------------------+  |
|  | Amazon RDS PostgreSQL |  |
|  | PostgreSQL 18.3-R2    |  |
|  | Single-AZ             |  |
|  +-----------------------+  |
+-----------------------------+
```

## Network & Security Configuration

### VPC and Subnet Placement

The RDS instance is deployed inside the default VPC using the default DB subnet group. The DB subnet group contains subnets across the configured Availability Zones, while this DB instance is currently placed in `ap-southeast-2c`.

### Security Group

The RDS Security Group controls network-level access to PostgreSQL.

- Protocol: TCP
- Port: `5432`
- Source: client public IP using `/32`
- Access model: restricted rather than open to the internet

Security Groups provide network-level traffic control. PostgreSQL roles and privileges provide database-level authentication and authorization.

## Connectivity

The database was successfully accessed from Windows PowerShell using the PostgreSQL `psql` client and the RDS endpoint over SSL/TLS.

Connection settings used:

```text
host=<RDS_ENDPOINT>
port=5432
dbname=mydb
user=postgres
sslmode=verify-full
sslrootcert=<GLOBAL_BUNDLE_PATH>
```

The AWS RDS root certificate bundle (`global-bundle.pem`) was used for certificate verification.

## SSL/TLS Validation

The connection was successfully established with:

- TLS protocol: `TLSv1.3`
- SSL mode: `verify-full`
- Certificate validation: successful

This verifies encrypted client-to-RDS communication and endpoint certificate validation.

## Database Validation

The following query was used to validate the active database session and primary/standby status:

```sql
SELECT
    current_database(),
    current_user,
    inet_server_port(),
    pg_is_in_recovery();
```

Observed result:

| Check | Result |
|---|---|
| Current database | `mydb` |
| Current user | `postgres` |
| Server port | `5432` |
| `pg_is_in_recovery()` | `false` |

`pg_is_in_recovery() = false` confirms that the connected RDS PostgreSQL instance is operating as the primary rather than a standby at the time of validation.

## Self-Managed PostgreSQL vs Amazon RDS

| Self-Managed PostgreSQL | Amazon RDS for PostgreSQL |
|---|---|
| OS and network configuration managed by DBA | AWS manages underlying infrastructure |
| `postgresql.conf` | Parameter groups |
| OS firewall rules | Security Groups |
| Filesystem and storage administration | Managed database storage |
| Manual maintenance and patching | AWS-managed maintenance capabilities |

RDS removes direct operating-system administration while retaining core database administration responsibilities such as SQL, roles, permissions, parameter management, backup/recovery configuration, monitoring, and performance troubleshooting.

## Validation Outcome

The environment was successfully validated for:

- RDS PostgreSQL availability
- Network placement
- Restricted TCP `5432` access
- SSL/TLS connectivity
- PostgreSQL authentication
- Primary database status

## Evidence

Screenshots captured from the implementation can be stored under:

```text
screenshots/
├── rds-configuration.png
├── ssl-connection.png
└── database-validation.png
```

Add the screenshots after capture and redact account identifiers, public IP addresses, endpoints, or any other sensitive values before publishing.

## Related Documentation

- [commands.md](commands.md) — commands used for the implementation
- [commands-with-output.md](commands-with-output.md) — execution evidence and observed output

## Cost & Security Considerations

- Single-AZ deployment was used for the current implementation.
- Unnecessary supporting services were not enabled.
- Network access to PostgreSQL was restricted to the required client IP.
- Public documentation should not expose credentials, full resource identifiers, account IDs, or unrestricted network details.
- Additional high-availability, replica, proxy, or monitoring resources should be introduced only when required by a specific implementation and reviewed for cost impact.

## Status

**Completed**

