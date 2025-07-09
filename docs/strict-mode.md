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


By default, PXC Strict Mode operates in `ENFORCING` mode. When a node acts
as a standalone server or during bootstrapping, the mode defaults to
`DISABLED`.

Selecting `ENFORCING` as the default mode is strongly recommended. Using
this mode ensures that unsupported operations and tech preview features are
denied, thereby protecting data consistency and prompting a review of cluster
configuration.

Choosing any mode other than `ENFORCING` requires a clear understanding of
the potential risks to data integrity. Refer to the [Validations](#validations)
section for detailed information.

To specify the mode, define the `pxc_strict_mode` variable in the configuration
file or use the `--pxc-strict-mode` option during `mysqld` startup.

!!! note

    It is better to start the server with the necessary mode
    (the default `ENFORCING` is highly recommended).
    However, you can dynamically change the mode during runtime.
    For example, to set PXC Strict Mode to `PERMISSIVE`,
    run the following command:

    ```{.bash data-prompt="mysql>"}
    mysql> SET GLOBAL pxc_strict_mode=PERMISSIVE;
    ```

    To ensure data consistency, all nodes in the cluster should use the same
    configuration, including the value of the
    [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) variable.


## Validations

PXC Strict Mode validations are implemented to ensure the smooth and reliable operation of cluster environments. These validations are particularly effective for standard cluster setups that do not require tech preview features or operations that are not supported by Percona XtraDB Cluster.

If [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) is set to `DISABLED` or `PERMISSIVE` on the originating node, unsupported operations can bypass strict mode validations. When these operations are replicated, they are not validated on destination nodes, even if those nodes have [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) set to `ENFORCING`. This behavior underscores the importance of carefully managing cluster configurations to avoid inconsistencies.

By adhering to strict mode recommendations, users can ensure smoother cluster operations while minimizing risks associated with unsupported actions.

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

With the strict mode set to `ENFORCING`, Percona XtraDB Cluster does not support  statements, because they combine both schema and
data changes. Note that tables in the SELECT clause should be present on all
replication nodes.

With the strict mode set to `PERMISSIVE` or `DISABLED`, CREATE TABLE … AS SELECT (CTAS) statements are
replicated using the  method to ensure
consistency. 

MyISAM tables are created and loaded even if `wsrep_replicate_myisam` equals `1`. Percona XtraDB Cluster does not recommend using the MyISAM storage engine. The support for MyISAM may be removed in a future release.


Depending on the strict mode selected, the following happens:

| Mode| Behavior|
| ------- | ---- | 
| DISABLED| At startup, no validation is performed. At runtime, all operations are permitted.  |
| PERMISSIVE| At startup, no validation is performed. At runtime, all operations are permitted, but a warning is logged when a CREATE TABLE … AS SELECT (CTAS) operation is performed.|
| ENFORCING| At startup, no validation is performed. At runtime, any CTAS operation is denied and an error is logged.|



Although `CREATE TABLE ... AS SELECT` (CTAS) operations for temporary tables are permitted even in ``STRICT`` mode, temporary tables should not be used as *source* tables in `CREATE TABLE ... AS SELECT` (CTAS) operations due to the fact that temporary tables are not present on all nodes.

If ``node-1`` has a temporary and a non-temporary table with the same name, CREATE TABLE ... AS SELECT (CTAS) on ``node-1`` will use temporary and CREATE TABLE ... AS SELECT (CTAS) on ``node-2`` will use the non-temporary table resulting in a data level inconsistency.
    

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

Percona XtraDB Cluster provides only the tech-preview-level of support for explicit table locking operations,
The following undesirable operations lead to explicit table locking
and are covered by this validation:

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

*Group replication* is a feature of MySQL that [provides distributed state
machine replication with strong coordination between servers](https://dev.mysql.com/doc/refman/8.0/en/group-replication.html). It is
implemented as a plugin which, if activated, may conflict with PXC. Group
replication cannot be activated to run alongside PXC. However, you can migrate
to PXC from the environment that uses group replication.

For the strict mode to work correctly, make sure that the group replication
plugin is *not active*. In fact, if [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) is set to
ENFORCING or MASTER, the server will stop with an error:

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

Percona XtraDB Cluster does not support tables in the MySQL database as the destination for log output. By default, log entries are written to file. This validation checks the value of the log_output variable.

Depending on the selected mode, the following happens:



### Major version check

This validation checks that the protocol version is the same as the server major version. This validation protects the cluster against writes attempted on already upgraded nodes.

??? example "Expected output"

    ```{.mysql no-copy}

    ERROR 1105 (HY000): Percona-XtraDB-Cluster prohibits use of multiple major versions while accepting write workload with pxc_strict_mode = ENFORCING or MASTER

    ```


### MyISAM replication

Percona XtraDB Cluster provides support for replication of tables
that use the MyISAM storage engine. The use of the MyISAM storage engine
in a cluster is not recommended and if you use the storage engine, this is
your own risk.
Due to the non-transactional nature of MyISAM, the storage
engine is not fully-supported in Percona XtraDB Cluster.

MyISAM replication is controlled using the `wsrep_replicate_myisam` variable,
which is set to `OFF` by default.
Due to its unreliability, MyISAM replication should not be enabled
if you want to ensure data consistency.

Depending on the selected mode, the following happens:

| Mode                 | Startup Behavior                                                                                          | Runtime Behavior                                                                                               |
|----------------------|------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| `DISABLED`           | No validation is performed.                                                                                | You can set `wsrep_replicate_myisam` to any value.                                                              |
| `PERMISSIVE`         | If `wsrep_replicate_myisam` is set to `ON`, a warning is logged and startup continues.                    | Changing `wsrep_replicate_myisam` to any value is permitted, but setting it to `ON` logs a warning.             |
| `ENFORCING` / `MASTER` | If [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) is set to `ON`, an error is logged and startup is aborted. | Any attempt to change [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) to `ON` fails and an error is logged. |

The [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) variable controls *replication* for MyISAM tables, and this validation only checks whether it is allowed. Undesirable operations for MyISAM tables are restricted using the Storage engine validation.

### Storage engine

Percona XtraDB Cluster currently supports replication only for tables
that use a transactional storage engine (XtraDB or InnoDB).
To ensure data consistency,
the following statements should not be allowed for tables
that use a non-transactional storage engine (MyISAM, MEMORY, CSV, and others):

* Data manipulation statements that perform writing to table
(for example, `INSERT`, `UPDATE`, `DELETE`, etc.)

* The following administrative statements:
`CHECK`, `OPTIMIZE`, `REPAIR`, and `ANALYZE`

* `TRUNCATE TABLE` and `ALTER TABLE`

Depending on the selected mode, the following happens:

| Mode                 | Startup Behavior                | Runtime Behavior                                                                                   |
|----------------------|----------------------------------|------------------------------------------------------------------------------------------------------|
| `DISABLED`           | No validation is performed.      | All operations are permitted.                                                                        |
| `PERMISSIVE`         | No validation is performed.      | All operations are permitted, but a warning is logged when an undesirable operation is performed on an unsupported table. |
| `ENFORCING` / `MASTER` | No validation is performed.      | Any undesirable operation on an unsupported table is denied and an error is logged.                  |

Unsupported tables can be converted to use a supported storage engine.
