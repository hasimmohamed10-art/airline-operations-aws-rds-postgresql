# Lab 03 — PostgreSQL Users, Roles and Security: Commands with Output

## Environment

- Database service: Amazon RDS for PostgreSQL
- PostgreSQL server version: 18.3
- PostgreSQL client: `psql` 18.4
- Database: `airlinedb`
- Schemas: `airline`, `booking`
- Connection security: SSL/TLS

Passwords and private connection details are excluded.

## 1. Connect to the Database

### Command

```powershell
cd $HOME\Downloads

& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

### Recorded output

```text
psql (18.4, server 18.3)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.

mydb=>
```

Switch to the Airline database:

```sql
\c airlinedb
```

### Recorded output

```text
You are now connected to database "airlinedb" as user "postgres".
```

## 2. Create the Read-Only Role

### Command

```sql
CREATE ROLE airline_readonly NOLOGIN;
```

### Output

```text
CREATE ROLE
```

Grant schema usage:

```sql
GRANT USAGE ON SCHEMA airline, booking TO airline_readonly;
```

### Output

```text
GRANT
```

Grant SELECT access to existing tables:

```sql
GRANT SELECT ON ALL TABLES IN SCHEMA airline, booking
TO airline_readonly;
```

### Output

```text
GRANT
```

Configure default SELECT privileges for future tables created by the role executing this command:

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA airline, booking
GRANT SELECT ON TABLES TO airline_readonly;
```

### Output

```text
ALTER DEFAULT PRIVILEGES
```

These commands establish the read-only role and grant the specified schema and table privileges.

## 3. Create the Application Login Role

### Command

```sql
CREATE ROLE airline_app_user LOGIN;
```

### Output

```text
CREATE ROLE
```

Set the password interactively:

```text
\password airline_app_user
```

The password was set interactively and is intentionally not included in this document.

Grant membership in the read-only role:

```sql
GRANT airline_readonly TO airline_app_user;
```

### Output

```text
GRANT ROLE
```

The application login role can inherit privileges through its membership in `airline_readonly`.

## 4. Validate Role Membership

### Query

```sql
SELECT
    member.rolname AS member_role,
    parent.rolname AS granted_role
FROM pg_auth_members m
JOIN pg_roles parent ON parent.oid = m.roleid
JOIN pg_roles member ON member.oid = m.member
WHERE member.rolname = 'airline_app_user';
```

### Recorded output

```text
    member_role    |   granted_role
-------------------+------------------
 airline_app_user  | airline_readonly
(1 row)
```

**Result:** `airline_app_user` was a member of `airline_readonly`.

## 5. Validate Read Privileges

### Query

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

### Recorded result

```text
 flights_select | bookings_select | tickets_select
----------------+-----------------+---------------
 t              | t               | t
(1 row)
```

**Result:** All three SELECT privilege checks returned `true`.

## 6. Validate Write Privileges

### Query

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

### Recorded result

```text
 insert_privilege | update_privilege | delete_privilege
------------------+------------------+-----------------
 f                | f                | f
(1 row)
```

**Result:** All three write privilege checks returned `false`.

## 7. Validate Rejected Write Operations

Write attempts were performed as `airline_app_user`.

### Recorded error messages

Attempts to modify the flights table returned a permission-denied error:

```text
ERROR: permission denied for table flights
```

An attempt to create a role as the application user returned:

```text
ERROR: permission denied to create role
```

**Result:** The application role was unable to perform the tested write operations or create additional roles.

These errors confirm the access restrictions observed during the lab.

## 8. Create the Administrative Role

Reconnect as the RDS master user before running the administrative role commands.

### Command

```sql
CREATE ROLE airline_admin NOLOGIN;
```

### Output

```text
CREATE ROLE
```

Grant privileges on both schemas:

```sql
GRANT ALL PRIVILEGES ON SCHEMA airline, booking
TO airline_admin;
```

### Output

```text
GRANT
```

Grant privileges on existing tables:

```sql
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA airline, booking
TO airline_admin;
```

### Output

```text
GRANT
```

Grant privileges on existing sequences:

```sql
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA airline, booking
TO airline_admin;
```

### Output

```text
GRANT
```

Configure default table privileges for future tables created by the role executing this command:

```sql
ALTER DEFAULT PRIVILEGES IN SCHEMA airline, booking
GRANT ALL PRIVILEGES ON TABLES TO airline_admin;
```

### Output

```text
ALTER DEFAULT PRIVILEGES
```

## 9. Validate Administrative Table Privileges

### Query

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

### Recorded result

```text
 select_privilege | insert_privilege | update_privilege | delete_privilege
------------------+------------------+------------------+-----------------
 t                | t                | t                | t
(1 row)
```

**Result:** All four tested table privilege checks returned `true` for `airline_admin`.

## 10. Inspect Role Attributes

### Query

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

### Recorded output

```text
     rolname      | rolcanlogin | rolsuper | rolcreaterole
------------------+-------------+----------+---------------
 airline_admin    | f           | f        | f
 airline_app_user | t           | f        | f
 airline_readonly | f           | f        | f
(3 rows)
```

**Result:**

- `airline_readonly` cannot log in directly.
- `airline_app_user` can log in.
- `airline_admin` cannot log in directly.
- None of these roles is a superuser or has permission to create roles.

## Final Validation Summary

| Validation | Recorded result |
|---|---|
| Read-only role | Created |
| Application login role | Created |
| Role membership | Validated |
| SELECT privileges | `true` for all three tested tables |
| INSERT, UPDATE, DELETE privileges | `false` for the application role on `airline.flights` |
| Rejected write attempts | Permission-denied errors recorded |
| Administrative table privileges | `true` for all four tested operations |
| Role login and administrative attributes | Validated |

## Technical Notes

- `NOLOGIN` prevents direct login through a role.
- Role membership can provide inherited privileges to a login role.
- `has_table_privilege()` checks effective table privileges, which can include privileges received through role membership.
- PostgreSQL default privileges apply to future objects created by the role whose defaults are being changed. They do not automatically apply to future objects created by every owner.
- The administrative role has the privileges granted in these commands; it is not a PostgreSQL superuser.

## Related Files

- [`README.md`](README.md) — Lab overview and implementation steps.
- [`commands.md`](commands.md) — Role creation, grants, and validation commands.
- [`screenshots/`](screenshots/) — Supporting evidence for the lab.

## Security Note

Passwords, live RDS endpoints, AWS account identifiers, and other private connection details must not be published. Review screenshots before uploading them to GitHub.
