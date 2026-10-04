# RDS PostgreSQL Architecture, Connectivity & Security — Commands with Output

This document records the commands and actual outputs used to validate the Amazon RDS PostgreSQL connectivity, SSL/TLS configuration, and database status for this implementation.

> Sensitive infrastructure identifiers are intentionally masked in the public documentation.

---

## 1. Connect to Amazon RDS PostgreSQL Using SSL/TLS

### Command

```powershell
& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<RDS-ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

### Actual Output

```text
psql (18.4, server 18.3)
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
You are now connected to database "mydb" as user "postgres".
mydb=>
```

### Validation

- PostgreSQL client version: 18.4
- RDS PostgreSQL server version: 18.3
- SSL/TLS protocol: TLSv1.3
- SSL mode: `verify-full`
- Connected database: `mydb`
- Connected user: `postgres`

---

## 2. Verify Database, User, Port, and Recovery Status

### Command

```sql
SELECT
    current_database(),
    current_user,
    inet_server_port(),
    pg_is_in_recovery();
```

### Actual Output

```text
 current_database | current_user | inet_server_port | pg_is_in_recovery
------------------+--------------+------------------+-------------------
 mydb             | postgres     |             5432 | f
(1 row)
```

### Validation

- Current database: `mydb`
- Current user: `postgres`
- PostgreSQL server port: `5432`
- `pg_is_in_recovery()`: `false`
- The RDS instance was operating as the primary/writer database.

---

## Lab 01 Validation Summary

| Validation Item | Result |
|---|---|
| RDS PostgreSQL connectivity | Successful |
| SSL/TLS connection | Successful |
| TLS protocol | TLSv1.3 |
| SSL verification mode | `verify-full` |
| Database | `mydb` |
| Database user | `postgres` |
| PostgreSQL port | 5432 |
| Primary/writer status | Confirmed |

---

## Notes

- The commands and outputs above belong only to the RDS Architecture, Connectivity, SSL/TLS, and database-status validation scope of this implementation.
- The `airlinedb` database, schemas, tables, sample data, and relationship validation are documented separately under **Lab 02 — Database Design**.
