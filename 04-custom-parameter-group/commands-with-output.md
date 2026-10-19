# Lab 04 — Custom PostgreSQL DB Parameter Group: Commands with Output

## Environment

- Database service: Amazon RDS for PostgreSQL
- PostgreSQL engine version: 18.3
- DB instance: `database-1`
- AWS Region: Sydney (`ap-southeast-2`)
- Custom parameter group: `airline-postgres18-params`
- Parameter group family: PostgreSQL 18

## 1. Open the Parameter Groups Page

### Procedure

1. Open the AWS Management Console.
2. Navigate to **Amazon RDS → Parameter groups**.
3. Review the available parameter groups.

### Observed result

The RDS Parameter groups page was opened to create a custom PostgreSQL parameter group.

## 2. Create the Custom Parameter Group

### Configuration

| Setting | Value |
|---|---|
| Parameter group name | `airline-postgres18-params` |
| Parameter group family | PostgreSQL 18 |
| Database engine | PostgreSQL |

### Procedure

1. Select **Create parameter group**.
2. Choose the PostgreSQL 18 parameter group family.
3. Enter `airline-postgres18-params` as the parameter group name.
4. Enter a suitable description.
5. Review the configuration and create the parameter group.

### Observed result

The custom parameter group `airline-postgres18-params` was created.

The PostgreSQL 18 family was selected for compatibility with the database engine.

## 3. Associate the Parameter Group with the DB Instance

### Procedure

1. Navigate to **Amazon RDS → Databases**.
2. Select `database-1`.
3. Choose **Actions → Modify**.
4. Locate the DB parameter group setting.
5. Select `airline-postgres18-params`.
6. Review the modification summary.
7. Apply the modification.

### Observed result

The custom parameter group was associated with `database-1`.

The console initially displayed the status `pending-reboot` after the association.

This status indicates that a reboot may be required for applicable pending changes to take effect. The current status should be verified in the AWS Console before documenting whether a reboot is still required.

## 4. Verify the Parameter Group Association

### Procedure

1. Open **Amazon RDS → Databases**.
2. Select `database-1`.
3. Open the **Configuration** tab.
4. Locate the DB parameter group field.
5. Review the associated parameter group and its status.

### Recorded configuration

| Setting | Recorded value |
|---|---|
| DB instance | `database-1` |
| Associated parameter group | `airline-postgres18-params` |
| Initial association status | `pending-reboot` |

The initial status is recorded here because it was observed during the association process. Check the current console status before describing the final state.

## 5. Review Parameter Settings

### Procedure

1. Navigate to **Amazon RDS → Parameter groups**.
2. Open `airline-postgres18-params`.
3. Review the available parameter names and values.
4. Check whether parameters are dynamic or static before changing them.
5. Do not modify parameters unless the change has been reviewed and its impact is understood.

### Observed result

The parameter group was created and associated with the database instance.

No custom parameter value change was recorded in this lab. In particular, `log_min_duration_statement` was not changed.

## 6. Understand Parameter Apply Types

| Parameter type | General behaviour |
|---|---|
| Dynamic | Can generally take effect without a database restart, depending on the parameter and its context. |
| Static | Requires a database restart before the changed value takes effect. |

Always check the specific parameter's apply type and the current RDS status before deciding whether a reboot is required.

## 7. Validation Summary

| Validation item | Recorded result |
|---|---|
| Custom parameter group created | Confirmed |
| PostgreSQL 18 family selected | Confirmed |
| Parameter group associated with `database-1` | Confirmed |
| Initial association status | `pending-reboot` |
| Custom parameter values changed | No changes recorded |
| Database reboot completed for this lab | Not recorded |

## Important Notes

- The parameter group family must match the database engine family.
- A `pending-reboot` status should not be described as cleared unless the current configuration confirms it.
- A database reboot may interrupt database connections.
- No performance improvement is claimed because no performance comparison was conducted.
- Do not state that a reboot occurred unless it was verified.

## Related Files

- [`README.md`](README.md) — Lab overview and implementation steps.
- [`commands.md`](commands.md) — AWS Console procedure and configuration review steps.
- [`screenshots/`](screenshots/) — Supporting screenshots.

## Security Note

Do not publish AWS account identifiers, private database endpoints, passwords, or other sensitive connection details in screenshots or documentation.

## Official AWS References

- [Amazon RDS DB parameter groups](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/parameter-groups-overview.html)
- [Modifying parameters in a DB parameter group](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_WorkingWithParamGroups.Modifying.html)
