# Lab 01 – Commands

This document contains the commands used to connect to and validate the Amazon RDS for PostgreSQL environment.

> **Repository safety:** The RDS endpoint and certificate path shown below are represented with placeholders. Replace them with the actual local values when executing the commands.

---

## 1. Open the PostgreSQL Client Tools Directory

```powershell
cd $HOME\Downloads
```

This places the PowerShell session in the directory containing the RDS CA certificate bundle.

---

## 2. Connect to Amazon RDS PostgreSQL Using SSL/TLS

```powershell
& "C:\Program Files\PostgreSQL\18\bin\psql.exe" "host=<RDS_ENDPOINT> port=5432 dbname=mydb user=postgres sslmode=verify-full sslrootcert=./global-bundle.pem"
```

### Connection parameters

| Parameter | Value | Purpose |
|---|---|---|
| `host` | `<RDS_ENDPOINT>` | RDS PostgreSQL endpoint |
| `port` | `5432` | PostgreSQL port |
| `dbname` | `mydb` | Initial database used for connection validation |
| `user` | `postgres` | RDS master user |
| `sslmode` | `verify-full` | Verifies SSL encryption and server identity |
| `sslrootcert` | `./global-bundle.pem` | AWS RDS CA certificate bundle |

---

## 3. Verify PostgreSQL Version and Connection Identity

```sql
SELECT
    version(),
    current_user,
    inet_server_addr(),
    inet_client_addr(),
    pg_is_in_recovery();
```

This verifies the PostgreSQL server version, authenticated user, server/client network addresses, and whether the instance is operating in recovery mode.

---

## 4. Verify Database, User, Port and Primary Status

```sql
SELECT
    current_database(),
    current_user,
    inet_server_port(),
    pg_is_in_recovery();
```

This provides a concise validation of the connected database, user, PostgreSQL port, and recovery status.

---

## 5. Exit the PostgreSQL Session

```sql
\q
```

Use this command when the current PostgreSQL session is no longer required.
