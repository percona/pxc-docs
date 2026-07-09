# Percona XtraDB Cluster strict mode

Percona XtraDB Cluster (PXC) Strict Mode blocks tech preview and unsupported features in PXC.
Strict Mode runs validations at startup and during runtime.

When a validation fails, the server takes one of the following actions:

* Log a warning and continue running.

* Throw an error and halt startup or deny the operation.

The action depends on the selected mode. The following modes are available:

* `DISABLED`: skip strict mode validations and run normally.

* `ENFORCING`: throw an error on validation failure. Halt the server at startup. Deny the operation at runtime.

* `MASTER`: behave as `ENFORCING` but skip the [explicit table locking](#explicit-table-locking) validation. Use this mode with clusters in which write operations are isolated to a single node.

* `PERMISSIVE`: log a warning when a validation fails, then continue running.

By default, PXC Strict Mode is set to `ENFORCING`.
The mode defaults to `DISABLED` when the node acts as a standalone server or during bootstrapping.

Keep PXC Strict Mode at `ENFORCING` for production clusters.
In `ENFORCING`, Percona XtraDB Cluster denies tech preview features and unsupported operations.
The denial forces a configuration review before any operation can compromise data integrity.

Review the limitations and impact on data integrity before setting PXC Strict Mode to a value other than `ENFORCING`.
For more information, see [Validations](#validations).

Set the mode through the [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) variable in the configuration file.
Alternatively, pass `--pxc-strict-mode` to `mysqld` at startup.

!!! note

    Start the server with the target mode.
    `ENFORCING` is the default and recommended setting.

    Change the value at runtime with the following statement:

    ```shell
    SET GLOBAL pxc_strict_mode=PERMISSIVE;
    ```

!!! note

    All nodes in the cluster must run with the same configuration to maintain data consistency.
    The configuration includes [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode).

## Validations

PXC Strict Mode validations enforce a stable configuration for common cluster deployments.
The validations block tech preview features and operations that Percona XtraDB Cluster does not support.

!!! warning

    Strict mode validates an operation only on the originating node.
    The destination nodes do not revalidate the operation, even when their [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) is `ENFORCING`.
    Set [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) to `ENFORCING` on every writer node to prevent unsupported operations from replicating.

This section describes the purpose and consequences of each validation.

### Group replication

<!-- TODO:

Provide steps for migrating from group replication

describing why (a completely different
clustering product, and we only support migration from/to, not
actively running them together), and how (disabled - allowed,
permission - warnings, enforcing/master - can't be turned on) -->

Group replication is a MySQL feature that [provides distributed state machine replication with strong coordination between servers :octicons-link-external-16:](https://dev.mysql.com/doc/refman/{{vers}}/en/group-replication.html).
The active group replication plugin conflicts with Percona XtraDB Cluster.
Run only one of the two products at a time.

You can migrate to Percona XtraDB Cluster from a group replication environment.

Disable the group replication plugin before strict mode runs.
With [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) set to `ENFORCING` or `MASTER`, an active plugin causes the server to fail with the following error:

??? example "The error message"

    ```text
    Group replication cannot be used with PXC in strict mode.
    ```

With [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) set to `DISABLED`, group replication runs without validation.
With `pxc_strict_mode` set to `PERMISSIVE`, the server logs the following warning:

??? example "Warning message"

    ```text
    Using group replication with PXC is only supported for migration. Please
    make sure that group replication is turned off once all data is migrated to PXC.
    ```

### Storage engine

Percona XtraDB Cluster supports replication only for tables that use a transactional storage engine such as XtraDB or InnoDB.
The following statements compromise data consistency on tables that use a non-transactional storage engine such as MyISAM, MEMORY, or CSV:

* `ALTER TABLE` and `TRUNCATE TABLE`

* Administrative statements: `ANALYZE`, `CHECK`, `OPTIMIZE`, and `REPAIR`

* Data manipulation statements that write to a table, such as `INSERT`, `UPDATE`, and `DELETE`

The following table summarizes startup and runtime behavior by mode:

| Mode                   | Startup behavior      | Runtime behavior                                                                                                  |
|------------------------|-----------------------|-------------------------------------------------------------------------------------------------------------------|
| `DISABLED`             | No validation runs.   | All operations are permitted.                                                                                      |
| `ENFORCING` / `MASTER` | No validation runs.   | The server denies undesirable operations on unsupported tables and logs an error.                                  |
| `PERMISSIVE`           | No validation runs.   | All operations are permitted. The server logs a warning when an undesirable operation runs on an unsupported table. |

!!! note

    Convert unsupported tables to a supported storage engine.

### MyISAM replication

Percona XtraDB Cluster replicates tables that use the MyISAM storage engine.
MyISAM is non-transactional, so the storage engine receives only partial support in Percona XtraDB Cluster.
Use MyISAM in a cluster at your own risk.

The [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) variable controls MyISAM replication.
The default value is `OFF`.
Leave MyISAM replication disabled to maintain data consistency.

The following table summarizes startup and runtime behavior by mode:

| Mode                   | Startup behavior                                                                                                                  | Runtime behavior                                                                                                                                              |
|------------------------|-----------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DISABLED`             | No validation runs.                                                                                                               | `wsrep_replicate_myisam` accepts any value.                                                                                                                    |
| `ENFORCING` / `MASTER` | If [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) is `ON`, the server logs an error and aborts startup. | Any attempt to set [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) to `ON` fails and the server logs an error.                        |
| `PERMISSIVE`           | If `wsrep_replicate_myisam` is `ON`, the server logs a warning and continues startup.                                             | `wsrep_replicate_myisam` accepts any value. The server logs a warning when the value changes to `ON`.                                                          |

!!! note

    The [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) variable controls replication for MyISAM tables.
    This validation only checks whether replication is allowed.
    The Storage engine validation restricts undesirable operations on MyISAM tables.

### Primary key requirement

Percona XtraDB Cluster requires every replicated table to have an explicit primary key.
A missing primary key prevents write-set replication from guaranteeing identical row order across nodes.
The `DELETE` statement also fails on tables without a primary key.
For related guidance, see [Limitations](limitation.md) and [`wsrep_certify_nonPK`](wsrep-system-index.md#wsrep_certify_nonpk).

The [`sql_require_primary_key` :octicons-link-external-16:](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_sql_require_primary_key) variable rejects `CREATE TABLE` and `ALTER TABLE` statements that would leave a table without a primary key.
When [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) is `ENFORCING` or `MASTER`, Percona XtraDB Cluster manages `sql_require_primary_key` to align both policies.

The following rules apply while [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) is `ENFORCING` or `MASTER`:

* Switching to `ENFORCING` or `MASTER` sets the global `sql_require_primary_key` to `ON` when the previous value was `OFF`. The server log records the change:

    ```{.text .no-copy}
    Setting sql_require_primary_key=ON because pxc_strict_mode is being changed to ENFORCING.
    ```

* Setting `sql_require_primary_key=OFF` is rejected. Both `SET GLOBAL` and `SET SESSION` return the following error:

    ```{.text .no-copy}
    ERROR 42000: Variable 'sql_require_primary_key' can't be set to the value of 'OFF'
    ```

    The error log records the reason:

    ```{.text .no-copy}
    Cannot set sql_require_primary_key=OFF while pxc_strict_mode is ENFORCING.
    ```

* Existing sessions retain their session value of `sql_require_primary_key`. Reconnect or update the session value explicitly to inherit the updated global default.

* Lowering [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) to `DISABLED` or `PERMISSIVE` does not reset `sql_require_primary_key`. Set the variable explicitly to allow tables without a primary key.

The following table summarizes startup and runtime behavior by mode:

| Mode                   | Startup behavior                                                                                                  | Runtime behavior                                                                                                                                                                                                |
|------------------------|-------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DISABLED`             | No validation runs. `sql_require_primary_key` keeps the configured value.                                         | All operations are permitted. `sql_require_primary_key` accepts `ON` or `OFF`.                                                                                                                                  |
| `ENFORCING` / `MASTER` | If Galera is active and `sql_require_primary_key` is `OFF`, Percona XtraDB Cluster forces the value to `ON`.      | Switching to this mode sets `sql_require_primary_key` to `ON` when the previous value was `OFF`. Setting `sql_require_primary_key=OFF` is rejected. Creating or altering a table without a primary key fails.   |
| `PERMISSIVE`           | No validation runs. `sql_require_primary_key` keeps the configured value.                                         | All operations are permitted. A warning is logged for any undesirable operation on a table without an explicit primary key. `sql_require_primary_key` accepts `ON` or `OFF`.                                    |

??? example "Error message when creating or altering a table without a primary key"

    ```{.text .no-copy}
    ERROR HY000: Unable to create or change a table without a primary key, when the system variable 'sql_require_primary_key' is set. Add a primary key to the table or unset this variable to avoid this message. Note that tables without a primary key can cause performance problems in row-based replication, so please consult your DBA before changing this setting.
    ```

!!! note

    Replication applier threads use replicated session metadata for each event.
    Events that originated on a non-strict source continue to apply on a strict-mode node.
    The events apply even when the source had `sql_require_primary_key` set to `OFF`.
    Use [`wsrep_certify_nonPK`](wsrep-system-index.md#wsrep_certify_nonpk) only as a fallback for existing tables without a primary key.

### Log output

Percona XtraDB Cluster does not support log output to tables in the `mysql` system database.
The server writes log entries to file by default.
This validation checks the value of the [`log_output` :octicons-link-external-16:](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_log_output) variable.

The following table summarizes startup and runtime behavior by mode:

| Mode                   | Startup behavior                                                                          | Runtime behavior                                                                                          |
|------------------------|-------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| `DISABLED`             | No validation runs.                                                                       | `log_output` accepts any value.                                                                            |
| `ENFORCING` / `MASTER` | If `log_output` is set only to `TABLE`, the server logs an error and aborts startup.      | Any attempt to set `log_output` only to `TABLE` fails and the server logs an error.                        |
| `PERMISSIVE`           | If `log_output` is set only to `TABLE`, the server logs a warning and continues startup.  | `log_output` accepts any value. The server logs a warning when the value changes to only `TABLE`.          |

### Explicit table locking

Percona XtraDB Cluster supports explicit table locking only at the tech preview level.
This validation covers the following operations that trigger explicit table locking:

* `FLUSH TABLES <tables> WITH READ LOCK`

* `GET_LOCK()` and `RELEASE_LOCK()`

* `LOCK TABLES`

* Setting the `SERIALIZABLE` transaction level

The following table summarizes startup and runtime behavior by mode:

| Mode                  | Startup behavior      | Runtime behavior                                                                                  |
|-----------------------|-----------------------|---------------------------------------------------------------------------------------------------|
| `DISABLED` / `MASTER` | No validation runs.   | All operations are permitted.                                                                      |
| `ENFORCING`           | No validation runs.   | The server denies undesirable operations and logs an error.                                        |
| `PERMISSIVE`          | No validation runs.   | All operations are permitted. The server logs a warning for undesirable operations.                |

### Auto-increment lock mode

The auto-increment lock mode must be `interleaved` so each node generates a unique but non-sequential identifier.

This validation checks the value of the [`innodb_autoinc_lock_mode` :octicons-link-external-16:](https://dev.mysql.com/doc/refman/{{vers}}/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) variable.
The default value is `1` for consecutive lock mode.
Set the value to `2` for interleaved lock mode.

The following table summarizes startup behavior by mode. The variable cannot change at runtime.

| Mode                   | Startup behavior                                                                                |
|------------------------|-------------------------------------------------------------------------------------------------|
| `DISABLED`             | No validation runs.                                                                              |
| `ENFORCING` / `MASTER` | If `innodb_autoinc_lock_mode` is not `2`, the server logs an error and aborts startup.           |
| `PERMISSIVE`           | If `innodb_autoinc_lock_mode` is not `2`, the server logs a warning and continues startup.       |

!!! note

    The `innodb_autoinc_lock_mode` variable cannot change at runtime, so the validation runs only at startup.

### Combine schema and data changes in a single statement

`CREATE TABLE ... AS SELECT` (CTAS) statements combine schema and data changes in a single operation.
With strict mode set to `ENFORCING`, Percona XtraDB Cluster rejects CTAS statements.
Every replication node must contain the tables referenced in the `SELECT` clause.

With strict mode set to `PERMISSIVE` or `DISABLED`, Percona XtraDB Cluster replicates CTAS statements through the [Non-Blocking Operations (NBO)](nbo.md) method.

!!! important

    Percona XtraDB Cluster creates and loads MyISAM tables even when `wsrep_replicate_myisam` equals `1`.
    Percona XtraDB Cluster does not recommend the MyISAM storage engine.
    Support for MyISAM may be removed in a future release.

!!! admonition "See also"

    [MySQL Bug System: XID inconsistency on master-slave with CTAS :octicons-link-external-16:](https://bugs.mysql.com/bug.php?id=93948)

The following table summarizes startup and runtime behavior by mode:

| Mode         | Startup behavior     | Runtime behavior                                                                                                  |
|--------------|----------------------|-------------------------------------------------------------------------------------------------------------------|
| `DISABLED`   | No validation runs.  | All operations are permitted.                                                                                      |
| `ENFORCING`  | No validation runs.  | The server denies any CTAS operation and logs an error.                                                            |
| `PERMISSIVE` | No validation runs.  | All operations are permitted. The server logs a warning when a CTAS operation runs.                                |

!!! important

    CTAS operations on temporary tables are permitted even in `STRICT` mode.
    However, do not use temporary tables as source tables in CTAS operations.
    Temporary tables are not present on every node.

    Consider a `node-1` that has a temporary and a non-temporary table with the same name.
    A CTAS statement on `node-1` uses the temporary table.
    The same CTAS statement on `node-2` uses the non-temporary table.
    The mismatch produces data-level inconsistency.

### Discard and import tablespaces

Percona XtraDB Cluster does not replicate `DISCARD TABLESPACE` or `IMPORT TABLESPACE` through Total Order Isolation (TOI).
Running either statement on a single node produces data inconsistency across the cluster.

The following table summarizes startup and runtime behavior by mode:

| Mode         | Startup behavior      | Runtime behavior                                                                                                          |
|--------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------|
| `DISABLED`   | No validation runs.   | All operations are permitted.                                                                                              |
| `ENFORCING`  | No validation runs.   | The server denies `DISCARD TABLESPACE` and `IMPORT TABLESPACE` and logs an error.                                          |
| `PERMISSIVE` | No validation runs.   | All operations are permitted. The server logs a warning when you discard or import a tablespace.                           |

### Major version check

This validation checks that the protocol version is the same as the server major version. This validation protects the cluster against writes attempted on already upgraded nodes.

??? example "Expected output"

    ```sql
    ERROR 1105 (HY000): Percona-XtraDB-Cluster prohibits use of multiple major versions while accepting write workload with pxc_strict_mode = ENFORCING or MASTER
    ```