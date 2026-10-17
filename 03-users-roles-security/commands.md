# Lab 03 — PostgreSQL Users, Roles and Security Commands

## Overview

This document records the SQL commands used to create PostgreSQL roles, assign privileges, configure role membership, and validate access controls in the Airline Operations database.

**Environment:** Amazon RDS for PostgreSQL 18.3, accessed using `psql` 18.4 over SSL/TLS.

## Step 1 — Connect to the Database

Open Windows PowerShell and connect to the RDS instance.

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

Enter the database password when prompted. Do not store the password in this file.

Switch to the Airline database:

```sql
\c airlinedb
```

## Step 2 — Create the Read-Only Role

Create a role that cannot log in directly:

```sql
CREATE ROLE airline_readonly NOLOGIN;
```

Grant access to the two schemas:

```sql
GRANT USAGE ON SCHEMA airline, booking TO airline_readonly;
```

Grant read access to existing tables:

```sql
GRANT SELECT ON ALL TABLES IN SCHEMA airline, booking
TO airline_readonly;
```

Configure default table privileges for future tables created by the current role:

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA airline, booking
GRANT SELECT ON TABLES TO airline_readonly;
```

## Step 3 — Create the Application Login Role

Create the login role:

```sql
CREATE ROLE airline_app_user LOGIN;
```

Set its password interactively:

```text
\password airline_app_user
```

Grant membership in the read-only role:

```sql
GRANT airline_readonly TO airline_app_user;
```

The application user can inherit the read-only permissions granted through role membership.

## Step 4 — Validate Role Membership

Run the following query:

```sql
SELECT
    member.rolname AS member_role,
    parent.rolname AS granted_role
FROM pg_auth_members m
JOIN pg_roles parent ON parent.oid = m.roleid
JOIN pg_roles member ON member.oid = m.member
WHERE member.rolname = 'airline_app_user';
```

The recorded result confirmed that `airline_app_user` was a member of `airline_readonly`.

## Step 5 — Validate Read Privileges

Check access to representative tables:

```sql
SELECT
    has_table_privilege(
        'airline_app_user',
        'airline.flights',
        'SELECT'
    ) AS flights_select,
    has_table_privilege(
        'airline_app_user',
        'booking.bookings',
        'SELECT'
    ) AS bookings_select,
    has_table_privilege(
        'airline_app_user',
        'booking.tickets',
        'SELECT'
    ) AS tickets_select;
```

The recorded results were `true` for all three SELECT checks.

## Step 6 — Validate Write Privileges

Check whether the application role has direct INSERT, UPDATE, and DELETE privileges on the flights table:

```sql
SELECT
    has_table_privilege(
        'airline_app_user',
        'airline.flights',
        'INSERT'
    ) AS insert_privilege,
    has_table_privilege(
        'airline_app_user',
        'airline.flights',
        'UPDATE'
    ) AS update_privilege,
    has_table_privilege(
        'airline_app_user',
        'airline.flights',
        'DELETE'
    ) AS delete_privilege;
```

All three recorded results were `false`.

Write attempts made as `airline_app_user` were rejected with a permission-denied error.

## Step 7 — Create the Administrative Role

Reconnect as the RDS master user before creating the administrative role and granting its privileges.

Create the role without login permission:

```sql
CREATE ROLE airline_admin NOLOGIN;
```

Grant schema privileges:

```sql
GRANT ALL PRIVILEGES ON SCHEMA airline, booking
TO airline_admin;
```

Grant privileges on existing tables:

```sql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA airline, booking
TO airline_admin;
```

Grant privileges on existing sequences:

```sql
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA airline, booking
TO airline_admin;
```

Configure default table privileges:

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA airline, booking
GRANT ALL PRIVILEGES ON TABLES TO airline_admin;
```

## Step 8 — Validate Administrative Table Privileges

Check the privileges granted to `airline_admin`:

```sql
SELECT
    has_table_privilege(
        'airline_admin',
        'airline.flights',
        'SELECT'
    ) AS select_privilege,
    has_table_privilege(
        'airline_admin',
        'airline.flights',
        'INSERT'
    ) AS insert_privilege,
    has_table_privilege(
        'airline_admin',
        'airline.flights',
        'UPDATE'
    ) AS update_privilege,
    has_table_privilege(
        'airline_admin',
        'airline.flights',
        'DELETE'
    ) AS delete_privilege;
```

The recorded privilege checks returned `true` for all four operations.

## Step 9 — Inspect Role Attributes

Use `pg_roles` to inspect the login, superuser, and role-creation attributes:

```sql
SELECT
    rolname,
    rolcanlogin,
    rolsuper,
    rolcreaterole
FROM pg_roles
WHERE rolname IN (
    'airline_readonly',
    'airline_app_user',
    'airline_admin'
)
ORDER BY rolname;
```

The recorded results showed:

| Role | Can login | Superuser | Can create roles |
|---|---|---|---|
| `airline_admin` | false | false | false |
| `airline_app_user` | true | false | false |
| `airline_readonly` | false | false | false |

## Important Technical Notes

- `NOLOGIN` prevents direct login with a role. It does not prevent that role's privileges from being granted to other roles.
- Role membership allows an application login to inherit privileges from `airline_readonly`.
- `has_table_privilege()` checks effective table privileges, including applicable inherited privileges and grants.
- PostgreSQL default privileges apply to future objects created by the role whose defaults are changed. They do not automatically apply to objects created by every possible owner.
- The `airline_admin` role has the specific schema, table, and sequence privileges granted above. It is not a PostgreSQL superuser.

## Security Notes

- Set passwords interactively with `\password`.
- Never commit passwords, private RDS endpoints, or AWS account identifiers.
- Grant only the permissions required by each role.
- Review role membership and effective privileges when access requirements change.

## Related Files

- [`README.md`](README.md) — Lab overview and implementation steps.
- [`commands-with-output.md`](commands-with-output.md) — Commands and recorded outputs.
- [`screenshots/`](screenshots/) — Supporting evidence for the lab.
