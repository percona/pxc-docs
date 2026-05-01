# Percona XtraDB Cluster strict mode

The Percona XtraDB Cluster (PXC) Strict Mode prevents the use of tech preview
features and unsupported operations in Percona XtraDB Cluster. This mode
performs several validations during startup and runtime to ensure compliance.

The server’s response to a failed validation depends on the selected mode. The
server can halt with an error, stopping startup or denying the operation, or
log a warning and continue running. The available modes include:

| Mode        | Description                                                                                   |
|-------------|-----------------------------------------------------------------------------------------------|
| `DISABLED`  | Do not perform strict mode validations and run as normal.                                     |
| `PERMISSIVE`| If a validation fails, log a warning and continue running as normal.                          |
| `ENFORCING` | If a validation fails during startup, halt the server and throw an error.                     |
|             | If a validation fails during runtime, deny the operation and throw an error.                  |
| `MASTER`    | The same as `ENFORCING` except that the validation of [explicit table locking](#explicit-table-locking) is not performed. This mode can be used with clusters in which write operations are isolated to a single node.                                                                             |


By default, PXC Strict Mode operates in `ENFORCING` mode.
When a node acts as a standalone server or during bootstrapping, the mode defaults to `DISABLED`.

Percona recommends `ENFORCING` as the default mode.
The `ENFORCING` mode denies unsupported operations and tech preview features, which protects data consistency and prompts a review of cluster configuration.

Any mode other than `ENFORCING` requires a clear understanding of the risks to data integrity.
For detailed information, see the [Validations](#validations) section.

To select the mode, set the `pxc_strict_mode` variable in the configuration file or pass `--pxc-strict-mode` to `mysqld` at startup.

!!! note

    Start the server with the required mode.
    The default `ENFORCING` mode is recommended.
    You can also change the mode at runtime.
    For example, to set PXC Strict Mode to `PERMISSIVE`, run the following command:

    ```{.bash data-prompt="mysql>"}
    mysql> SET GLOBAL pxc_strict_mode=PERMISSIVE;
    ```

    Use the same value of [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) on every node in the cluster to preserve data consistency.


## Validations

PXC Strict Mode runs the validations described in the following sections to block tech preview features and unsupported operations.
Use these validations on standard cluster deployments that do not require tech preview features.

[`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) validates operations only on the node where the operation runs.
When the originating node uses `DISABLED` or `PERMISSIVE`, unsupported operations can bypass validation.
Replicated copies of these operations are not revalidated on destination nodes, even when destination nodes use `ENFORCING`.
Apply the same [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) value on every node to prevent inconsistencies.

### Auto-increment lock mode

The lock mode for generating auto-increment values must be *interleaved*
to ensure that each node generates a unique (but non-sequential) identifier.

This validation checks the value of the [innodb_autoinc_lock_mode](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) variable.
By default, the variable is set to `1` (*consecutive* lock mode),
but it should be set to `2` (*interleaved* lock mode). This validation is not performed during runtime, because the `innodb_autoinc_lock_mode` variable cannot be set dynamically.

Depending on the strict mode selected,
the following happens:

| Mode               | Description                                                                                 |
|--------------------|---------------------------------------------------------------------------------------------|
| `DISABLED`         | At startup, no validation is performed.                                                    |
| `PERMISSIVE`       | At startup, if `innodb_autoinc_lock_mode` is not set to `2`, a warning is logged and startup continues. |
| `ENFORCING` or `MASTER` | At startup, if `innodb_autoinc_lock_mode` is not set to `2`, an error is logged and startup is aborted. |


### Binary log format

Percona XtraDB Cluster supports only the default row-based binary logging format. 

In 8.0, setting the [binlog_format](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_binlog_format) variable to anything but `ROW` at startup or runtime is not allowed regardless of the value of the `pxc_strict_mode` variable.

### Combine schema and data changes in a single statement

With the strict mode set to `ENFORCING`, Percona XtraDB Cluster does not support `CREATE TABLE ... AS SELECT` (CTAS) statements because they combine schema and data changes.
Tables referenced in the `SELECT` clause must exist on every replication node.

With the strict mode set to `PERMISSIVE` or `DISABLED`, Percona XtraDB Cluster replicates CTAS statements through Total Order Isolation (TOI) to preserve consistency.

MyISAM tables are created and loaded even when `wsrep_replicate_myisam` equals `1`.
Percona XtraDB Cluster does not recommend the MyISAM storage engine.
Support for MyISAM may be removed in a future release.

The following table summarizes runtime behavior by mode:

| Mode         | Behavior                                                                                                                   |
|--------------|----------------------------------------------------------------------------------------------------------------------------|
| `DISABLED`   | At startup, no validation is performed. At runtime, all operations are permitted.                                          |
| `PERMISSIVE` | At startup, no validation is performed. At runtime, all operations are permitted, and a warning is logged for each CTAS.   |
| `ENFORCING`  | At startup, no validation is performed. At runtime, any CTAS operation is denied and an error is logged.                   |

CTAS operations on temporary tables are permitted in strict mode.
Do not use temporary tables as source tables for CTAS operations because temporary tables do not exist on every node.

For example, when `node-1` has both a temporary table and a non-temporary table with the same name, CTAS on `node-1` reads from the temporary table.
CTAS on `node-2` reads from the non-temporary table, which produces data-level inconsistency.


### Discard and import tablespaces

`DISCARD TABLESPACE` and `IMPORT TABLESPACE` are not replicated using TOI.
This can lead to data inconsistency if executed on only one node.

At startup no validation is performed.

Depending on the strict mode selected, the following happens:

| Mode        | Runtime Behavior |
|------------|------------------|
| `DISABLED`  | All operations are permitted. |
| `PERMISSIVE` | All operations are permitted, but a warning is logged when you discard or import a tablespace. |
| `ENFORCING`  | Discarding or importing a tablespace is denied and an error is logged. |

### Explicit table locking

Percona XtraDB Cluster provides only tech-preview support for explicit table locking operations.
This validation covers the following operations that introduce explicit table locking:

* `LOCK TABLES`

* `GET_LOCK()` and `RELEASE_LOCK()`

* `FLUSH TABLES <tables> WITH READ LOCK`

* Setting the `SERIALIZABLE` transaction level

At startup, no validation is performed. Depending on the selected mode, the following happens:

| Mode               | Runtime Behavior |
|--------------------|------------------|
| `DISABLED` / `MASTER`  | All operations are permitted. |
| `PERMISSIVE`      | All operations are permitted, but a warning is logged when an undesirable operation is performed. |
| `ENFORCING`       | Any undesirable operation is denied and an error is logged. |

### Group replication

*Group replication* is a MySQL feature that [provides distributed state machine replication with strong coordination between servers](https://dev.mysql.com/doc/refman/8.0/en/group-replication.html).
MySQL implements group replication as a plugin, which conflicts with PXC when activated.
Group replication cannot run alongside PXC.
You can, however, migrate from a group-replication environment to PXC.

Disable the group replication plugin before strict mode runs.
When [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) is `ENFORCING` or `MASTER`, the server stops with the following error:

**Error message with [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) set to `ENFORCING` or `MASTER`**

??? example "The error message"

    ```{.text .no-copy}
    Group replication cannot be used with PXC in strict mode.
    ```

If `pxc_strict_mode` is set to `DISABLED` you can use group
replication at your own risk. Setting [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) to
`PERMISSIVE` results in a warning.

**Warning message with `pxc_strict_mode` set to `PERMISSIVE`**

??? example "Warning message"

    ```{.text .no-copy}
    Using group replication with PXC is only supported for migration. Please
    make sure that group replication is turned off once all data is migrated to PXC.
    ```

### Log output

Percona XtraDB Cluster does not support tables in the MySQL database as the destination for log output.
By default, the server writes log entries to a file.
This validation checks the value of the [`log_output`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_log_output) variable.

The following table summarizes startup and runtime behavior by mode:

| Mode                   | Startup behavior                                                                              | Runtime behavior                                                                                                |
|------------------------|-----------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `DISABLED`             | No validation is performed.                                                                   | You can set `log_output` to any value.                                                                           |
| `PERMISSIVE`           | If `log_output` is set only to `TABLE`, a warning is logged and startup continues.            | You can change `log_output` to any value. Setting `log_output` only to `TABLE` logs a warning.                   |
| `ENFORCING` / `MASTER` | If `log_output` is set only to `TABLE`, an error is logged and startup is aborted.            | Any attempt to change `log_output` only to `TABLE` fails and an error is logged.                                 |

### Major version check

This validation checks that the protocol version is the same as the server major version. This validation protects the cluster against writes attempted on already upgraded nodes.

??? example "Expected output"

    ```{.mysql no-copy}

    ERROR 1105 (HY000): Percona-XtraDB-Cluster prohibits use of multiple major versions while accepting write workload with pxc_strict_mode = ENFORCING or MASTER

    ```


### MyISAM replication

Percona XtraDB Cluster supports replication of tables that use the MyISAM storage engine.
Percona does not recommend MyISAM in a cluster, and any such use is at your own risk.
MyISAM is non-transactional, so Percona XtraDB Cluster does not fully support the engine.

The `wsrep_replicate_myisam` variable controls MyISAM replication.
The default value is `OFF`.
Keep MyISAM replication disabled to preserve data consistency.

Depending on the selected mode, the following happens:

| Mode                 | Startup Behavior                                                                                          | Runtime Behavior                                                                                               |
|----------------------|------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `DISABLED`           | No validation is performed.                                                                                | You can set `wsrep_replicate_myisam` to any value.                                                              |
| `PERMISSIVE`         | If `wsrep_replicate_myisam` is set to `ON`, a warning is logged and startup continues.                    | Changing `wsrep_replicate_myisam` to any value is permitted, but setting it to `ON` logs a warning.             |
| `ENFORCING` / `MASTER` | If [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) is set to `ON`, an error is logged and startup is aborted. | Any attempt to change [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) to `ON` fails and an error is logged. |

The [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) variable controls *replication* for MyISAM tables, and this validation only checks whether it is allowed. Undesirable operations for MyISAM tables are restricted using the Storage engine validation.

### Primary key requirement

Percona XtraDB Cluster requires every replicated table to have an explicit primary key.
A missing primary key prevents write-set replication from guaranteeing identical row order across nodes.
The `DELETE` statement also fails on tables without a primary key.
For related guidance, see [Limitations](limitation.md) and [`wsrep_certify_nonPK`](wsrep-system-index.md#wsrep_certify_nonpk).

The [`sql_require_primary_key`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_sql_require_primary_key) variable rejects `CREATE TABLE` and `ALTER TABLE` statements that would leave a table without a primary key.
When [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) is `ENFORCING` or `MASTER`, Percona XtraDB Cluster manages `sql_require_primary_key` to keep both policies aligned.

The following rules apply while [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) is `ENFORCING` or `MASTER`:

- Switching to `ENFORCING` or `MASTER` sets the global `sql_require_primary_key` to `ON` when the previous value was `OFF`.
  The server log records the change:

    ```{.text .no-copy}
    Setting sql_require_primary_key=ON because pxc_strict_mode is being changed to ENFORCING.
    ```

- Setting `sql_require_primary_key=OFF` is rejected.
  Both `SET GLOBAL` and `SET SESSION` return the following error:

    ```{.text .no-copy}
    ERROR 42000: Variable 'sql_require_primary_key' can't be set to the value of 'OFF'
    ```

    The error log records the reason:

    ```{.text .no-copy}
    Cannot set sql_require_primary_key=OFF while pxc_strict_mode is ENFORCING.
    ```

- Existing sessions retain their session value of `sql_require_primary_key`.
  Reconnect or update the session value explicitly to inherit the updated global default.

- Lowering [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) to `DISABLED` or `PERMISSIVE` does not reset `sql_require_primary_key`.
  Set the variable explicitly to allow tables without a primary key.

The following table summarizes startup and runtime behavior by mode:

| Mode                   | Startup behavior                                                                                                                            | Runtime behavior                                                                                                                                                                                                  |
|------------------------|---------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `DISABLED`             | No validation is performed. `sql_require_primary_key` keeps the configured value.                                                           | All operations are permitted. `sql_require_primary_key` accepts `ON` or `OFF`.                                                                                                                                    |
| `PERMISSIVE`           | No validation is performed. `sql_require_primary_key` keeps the configured value.                                                           | All operations are permitted. `sql_require_primary_key` accepts `ON` or `OFF`.                                                                                                                                    |
| `ENFORCING` / `MASTER` | If Galera is active and `sql_require_primary_key` is `OFF`, startup forces the global value to `ON` to match strict mode.                   | Switching to this mode sets the global `sql_require_primary_key` to `ON` when previously `OFF`. Setting `sql_require_primary_key=OFF` is rejected. Creating or altering a table without a primary key fails.       |

??? example "Error message when creating or altering a table without a primary key"

    ```{.text .no-copy}
    ERROR HY000: Unable to create or change a table without a primary key, when the system variable 'sql_require_primary_key' is set. Add a primary key to the table or unset this variable to avoid this message. Note that tables without a primary key can cause performance problems in row-based replication, so please consult your DBA before changing this setting.
    ```

!!! note

    Replication applier threads use replicated session metadata for each event.
    Events that originated on a non-strict source continue to apply on a strict-mode node, even when `sql_require_primary_key` was `OFF` on the source.
    Use [`wsrep_certify_nonPK`](wsrep-system-index.md#wsrep_certify_nonpk) only as a fallback for existing tables without a primary key.

### Storage engine

Percona XtraDB Cluster supports replication only for tables that use a transactional storage engine, such as XtraDB or InnoDB.
To preserve data consistency, the following statements must not run on tables that use a non-transactional storage engine (MyISAM, MEMORY, CSV, and others):

- Data manipulation statements that write to the table, such as `INSERT`, `UPDATE`, and `DELETE`

- Administrative statements: `CHECK`, `OPTIMIZE`, `REPAIR`, and `ANALYZE`

- `TRUNCATE TABLE` and `ALTER TABLE`

Depending on the selected mode, the following happens:

| Mode                 | Startup Behavior                | Runtime Behavior                                                                                   |
|----------------------|----------------------------------|------------------------------------------------------------------------------------------------------|
| `DISABLED`           | No validation is performed.      | All operations are permitted.                                                                        |
| `PERMISSIVE`         | No validation is performed.      | All operations are permitted, but a warning is logged when an undesirable operation is performed on an unsupported table. |
| `ENFORCING` / `MASTER` | No validation is performed.      | Any undesirable operation on an unsupported table is denied and an error is logged.                  |

Unsupported tables can be converted to use a supported storage engine.
