# Lab 03 — PostgreSQL Users, Roles and Security

## Overview

This lab covers role-based access control (RBAC) in PostgreSQL on Amazon RDS. It includes creating read-only, application login, and administrative roles, assigning privileges, and validating access restrictions.

## Objectives

- Create reusable PostgreSQL roles.
- Configure an application login role.
- Grant read-only access to the Airline Operations schemas.
- Create an administrative role with table and schema privileges.
- Validate role membership and access permissions.
- Verify that unauthorized write operations are rejected.

## Environment

| Component | Details |
|---|---|
| Database service | Amazon RDS for PostgreSQL |
| PostgreSQL server | 18.3 |
| Database | `airlinedb` |
| Schemas | `airline`, `booking` |
| SQL client | `psql` 18.4 |
| Connection security | SSL/TLS |

## Roles Implemented

### 1. `airline_readonly`

- Created as a `NOLOGIN` role.
- Granted `USAGE` on the `airline` and `booking` schemas.
- Granted `SELECT` privileges on existing tables in both schemas.
- Configured default table privileges for future objects created by the role owner.

### 2. `airline_app_user`

- Created as a login role.
- Assigned membership in `airline_readonly`.
- Configured to access the database with read-only privileges inherited from the granted role.

### 3. `airline_admin`

- Created as a `NOLOGIN` administrative role.
- Granted privileges on the `airline` and `booking` schemas.
- Granted privileges on existing tables and sequences.
- Configured default table privileges for future tables created by the role owner.

## Implementation Steps

### Step 1 — Create the read-only role

Created `airline_readonly` without login permission and granted schema usage and table read privileges.

### Step 2 — Create the application login

Created `airline_app_user`, set its password interactively, and granted membership in `airline_readonly`.

### Step 3 — Validate role membership

Queried PostgreSQL system catalogs to verify that `airline_app_user` was a member of `airline_readonly`.

### Step 4 — Validate read-only privileges

Used `has_table_privilege()` to verify that the application user could read the required tables.

### Step 5 — Test denied operations

Tested `INSERT`, `UPDATE`, and `DELETE` operations as the application user. The attempts were rejected with permission errors.

The application user was also unable to create additional roles.

### Step 6 — Create the administrative role

Created `airline_admin` and granted schema, table, and sequence privileges.

### Step 7 — Validate administrative privileges

Used privilege-checking queries to confirm the recorded permissions on `airline.flights`.

## Validation Results

- The application role belonged to `airline_readonly`.
- The application role had `SELECT` access to the validated tables.
- The application role did not have direct `INSERT`, `UPDATE`, or `DELETE` privileges on `airline.flights`.
- Attempts to modify data were rejected.
- The administrative role's tested table privileges returned `true`.
- The roles were not configured as PostgreSQL superusers.

## Security Considerations

- Assign only the permissions required by each role.
- Do not publish passwords or private connection details.
- Use a login role with limited privileges for application access.
- Review role memberships and effective privileges when permissions change.
- PostgreSQL default privileges apply to future objects created by the role whose defaults are being changed; they do not automatically cover objects created by every owner.

## Screenshots

Supporting evidence is available in the `screenshots/` folder.

## Related Documentation

- [`commands.md`](commands.md) — Role creation, grants, and validation commands.
- [`commands-with-output.md`](commands-with-output.md) — Recorded commands and outputs.
- [`screenshots/`](screenshots/) — Supporting screenshots.

## Outcome

Role-based access control was implemented and validated for the Airline Operations database. Read-only restrictions and administrative table privileges were checked using PostgreSQL privilege queries and access tests.
