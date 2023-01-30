# PXC strict mode

PXC Strict Mode is designed to avoid the use of
tech preview features and unsupported features in Percona XtraDB Cluster.
It performs a number of validations at startup and during runtime.

Depending on the actual mode you select,
upon encountering a failed validation,
the server will either throw an error
(halting startup or denying the operation),
or log a warning and continue running as normal.
The following modes are available:

* `DISABLED`: Do not perform strict mode validations
and run as normal.

* `PERMISSIVE`: If a vaidation fails, log a warning and continue running
as normal.

* `ENFORCING`: If a validation fails during startup,
halt the server and throw an error.
If a validation fails during runtime,
deny the operation and throw an error.

* `MASTER`: The same as `ENFORCING` except that the validation of
[explicit table locking](#explicit-table-locking) is not performed.
This mode can be used with clusters
in which write operations are isolated to a single node.

By default, PXC Strict Mode is set to `ENFORCING`,
except if the node is acting as a standalone server
or the node is bootstrapping, then PXC Strict Mode defaults to `DISABLED`.

It is recommended to keep PXC Strict Mode set to `ENFORCING`,
because in this case whenever Percona XtraDB Cluster encounters a tech preview feature
or an unsupported operation, the server will deny it.
This will force you to re-evaluate your Percona XtraDB Cluster configuration
without risking the consistency of your data.

If you are planning to set PXC Strict Mode to anything else than `ENFORCING`,
you should be aware of the limitations and effects
that this may have on data integrity.
For more information, see [Validations](#validations).

To set the mode, use the [`pxc_strict_mode`](../wsrep-system-index.md#pxc_strict_mode) variable in the configuration file
or the `--pxc-strict-mode` option during `mysqld` startup.

!!! note

    It is better to start the server with the necessary mode
    (the default `ENFORCING` is highly recommended).
    However, you can dynamically change it during runtime.
    For example, to set PXC Strict Mode to `PERMISSIVE`,
    run the following command:

    ```{.bash data-prompt="mysql>"}
    mysql> SET GLOBAL pxc_strict_mode=PERMISSIVE;
    ```

!!! note

    To further ensure data consistency,
    it is important to have all nodes in the cluster
    running with the same configuration,
    including the value of [`pxc_strict_mode`](../wsrep-system-index.md#pxc_strict_mode) variable.

## Validations

PXC Strict Mode validations are designed to ensure optimal operation
for common cluster setups that do not require tech preview features
and do not rely on operations not supported by Percona XtraDB Cluster.

!!! warning

    If an unsupported operation is performed on a node with [`pxc_strict_mode`](../wsrep-system-index.md#pxc_strict_mode) set to `DISABLED` or `PERMISSIVE`, it will not be validated on nodes where it is replicated to, even if the destination node has [`pxc_strict_mode`](../wsrep-system-index.md#pxc_strict_mode) set to `ENFORCING`.

This section describes the purpose and consequences of each validation.

### Group replication

<!-- TODO:

Provide steps for migrating from group replication

describing why (e.g. it is a completely different
clustering product, and we only support migration from/to, not
actively running them together), and how (disabled - allowed,
permission - warnings, enforcing/master - can't be turned on) -->
*Group replication* is a feature of MySQL that [provides distributed state
machine replication with strong coordination between servers](https://dev.mysql.com/doc/refman/8.0/en/group-replication.html). It is
implemented as a plugin which, if activated, may conflict with PXC. Group
replication cannot be activated to run alongside PXC. However, you can migrate
to PXC from the environment that uses group replication.

For the strict mode to work correctly, make sure that the group replication
plugin is *not active*. In fact, if [`pxc_strict_mode`](../wsrep-system-index.md#pxc_strict_mode) is set to
ENFORCING or MASTER, the server will stop with an error:

**Error message with [`pxc_strict_mode`](../wsrep-system-index.md#pxc_strict_mode) set to `ENFORCING` or `MASTER`**

??? example "The error message"

    ```{.text .no-copy}
    Group replication cannot be used with PXC in strict mode.
    ```

If `pxc_strict_mode` is set to `DISABLED` you can use group
replication at your own risk. Setting [`pxc_strict_mode`](../wsrep-system-index.md#pxc_strict_mode) to
`PERMISSIVE` will result in a warning.

**Warning message with `pxc_strict_mode` set to `PERMISSIVE`**

??? example "Warning message"

    ```{.text .no-copy}
    Using group replication with PXC is only supported for migration. Please
    make sure that group replication is turned off once all data is migrated to PXC.
    ```

### Storage engine

Percona XtraDB Cluster currently supports replication only for tables
that use a transactional storage engine (XtraDB or InnoDB).
To ensure data consistency,
the following statements should not be allowed for tables
that use a non-transactional storage engine (MyISAM, MEMORY, CSV, etc.):

* Data manipulation statements that perform writing to table
(for example, `INSERT`, `UPDATE`, `DELETE`, etc.)

* The following administrative statements:
`CHECK`, `OPTIMIZE`, `REPAIR`, and `ANALYZE`

* `TRUNCATE TABLE` and `ALTER TABLE`

Depending on the selected mode, the following happens:

`DISABLED`

At startup, no validation is performed.

At runtime, all operations are permitted.

`PERMISSIVE`

At startup, no validation is perfromed.

At runtime, all operations are permitted, but a warning is logged when an undesirable operation is performed on an unsupported table.

`ENFORCING` or `MASTER`

At startup, no validation is performed.

At runtime, any undesirable operation performed on an unsupported table is denied and an error is logged.

!!! note

    Unsupported tables can be converted to use a supported storage engine.

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

`DISABLED`

At startup, no validation is performed.

At runtime, you can set `wsrep_replicate_myisam` to any value.

`PERMISSIVE`

At startup, if `wsrep_replicate_myisam` is set to `ON`, a warning is logged and startup continues.

At runtime, it is permitted to change `wsrep_replicate_myisam` to any value, but if you set it to `ON`, a warning is logged.

`ENFORCING` or `MASTER`

At startup, if [`wsrep_replicate_myisam`](../wsrep-system-index.md#wsrep_replicate_myisam) is set to `ON`, an error is logged and startup is aborted.

At runtime, any attempt to change [`wsrep_replicate_myisam`](../wsrep-system-index.md#wsrep_replicate_myisam) to `ON` fails and an error is logged.

!!! note

    The [`wsrep_replicate_myisam`](../wsrep-system-index.md#wsrep_replicate_myisam) variable controls *replication* for MyISAM tables, and this validation only checks whether it is allowed. Undesirable operations for MyISAM tables are restricted using the Storage engine validation.

### Binary log format

Percona XtraDB Cluster supports only the default row-based binary logging format.  In
8.0, setting the [binlog_format](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_binlog_format) variable to anything but
`ROW` at startup or runtime is not allowed regardless of the value of the `pxc_strict_mode` variable.

### Tables without primary keys

Percona XtraDB Cluster cannot properly propagate certain write operations
to tables that do not have primary keys defined.
Undesirable operations include data manipulation statements
that perform writing to table (especially `DELETE`).

Depending on the selected mode, the following happens:

`DISABLED`

At startup, no validation is performed.

At runtime, all operations are permitted.

`PERMISSIVE`

At startup, no validation is perfromed.

At runtime, all operations are permitted,
but a warning is logged when an undesirable operation
is performed on a table without an explicit primary key defined.

`ENFORCING` or `MASTER`

At startup, no validation is performed.

At runtime, any undesirable operation
performed on a table without an explicit primary key
is denied and an error is logged.

### Log output

Percona XtraDB Cluster does not support tables in the MySQL database
as the destination for log output.
By default, log entries are written to file.
This validation checks the value of the [log_output](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_log_output) variable.

Depending on the selected mode, the following happens:

`DISABLED`

At startup, no validation is performed.

At runtime, you can set `log_output` to any value.

`PERMISSIVE`

At startup, if `log_output` is set only to `TABLE`,
a warning is logged and startup continues.

At runtime, it is permitted to change `log_output`
to any value, but if you set it only to `TABLE`,
a warning is logged.

`ENFORCING` or `MASTER`

At startup, if `log_output` is set only to `TABLE`,
an error is logged and startup is aborted.

At runtime, any attempt to change `log_output` only to `TABLE` fails
and an error is logged.

### Explicit table locking

Percona XtraDB Cluster provides only the tech-preview-level of support for explicit table locking operations,
The following undesirable operations lead to explicit table locking
and are covered by this validation:

* `LOCK TABLES`

* `GET_LOCK()` and `RELEASE_LOCK()`

* `FLUSH TABLES <tables> WITH READ LOCK`

* Setting the `SERIALIZABLE` transaction level

Depending on the selected mode, the following happens:

`DISABLED` or `MASTER`

At startup, no validation is performed.

At runtime, all operations are permitted.

`PERMISSIVE`

At startup, no validation is performed.

At runtime, all operations are permitted,
but a warning is logged when an undesirable operation is performed.

`ENFORCING`

At startup, no validation is performed.

At runtime, any undesirable operation is denied and an error is logged.

### Auto-increment lock mode

The lock mode for generating auto-increment values must be *interleaved*
to ensure that each node generates a unique (but non-sequential) identifier.

This validation checks the value of the [innodb_autoinc_lock_mode](https://dev.mysql.com/doc/refman/8.0/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode) variable.
By default, the variable is set to `1` (*consecutive* lock mode),
but it should be set to `2` (*interleaved* lock mode).

Depending on the strict mode selected,
the following happens:

`DISABLED`

At startup, no validation is performed.

`PERMISSIVE`

At startup, if `innodb_autoinc_lock_mode` is not set to `2`,
a warning is logged and startup continues.

`ENFORCING` or `MASTER`

At startup, if `innodb_autoinc_lock_mode` is not set to `2`, an error is logged and startup is aborted.

!!! note

    This validation is not performed during runtime, because the `innodb_autoinc_lock_mode` variable cannot be set dynamically.

### Combine schema and data changes in a single statement

With strict mode set to `ENFORCING`, Percona XtraDB Cluster does not support  statements, because they combine both schema and
data changes. Note that tables in the SELECT clause should be present on all
replication nodes.

With strict mode set to `PERMISSIVE` or `DISABLED`, CREATE TABLE … AS SELECT (CTAS) statements are
replicated using the  method to ensure
consistency. In Percona XtraDB Cluster 5.7, CREATE TABLE … AS SELECT (CTAS) statements were replicated using DML
write-sets when strict mode was set to `PERMISSIVE` or `DISABLED`.

!!! important

    MyISAM tables are created and loaded even if `wsrep_replicate_myisam` equals to 1.  Percona XtraDB Cluster does not recommend using the MyISAM storage engine. The support for MyISAM may be removed in a future release.

!!! admonition "See also"

    MySQL Bug System: XID inconsistency on master-slave with CTAS https://bugs.mysql.com/bug.php?id=93948  

Depending on the strict mode selected, the following happens:

| Mode| Behavior|
| ------- | ---- | 
| DISABLED| At startup, no validation is performed. At runtime, all operations are permitted.  |
| PERMISSIVE| At startup, no validation is perfromed. At runtime, all operations are permitted, but a warning is logged when a CREATE TABLE … AS SELECT (CTAS) operation is performed.|
| ENFORCING| At startup, no validation is performed. At runtime, any CTAS operation is denied and an error is logged.|

!!! important

    Although CREATE TABLE ... AS SELECT (CTAS) operations for temporary tables are permitted even in ``STRICT`` mode, temporary tables should not be used as *source* tables in CREATE TABLE ... AS SELECT (CTAS) operations due to the fact that temporary tables are not present on all nodes.

    If ``node-1`` has a temporary and a non-temporary table with the same name,
    CREATE TABLE ... AS SELECT (CTAS) on ``node-1`` will use temporary and CREATE TABLE ... AS SELECT (CTAS) on ``node-2`` will use the non-temporary table resulting in a data level inconsistency.

### Discard and import tablespaces

`DISCARD TABLESPACE` and `IMPORT TABLESPACE` are not replicated using TOI.
This can lead to data inconsistency if executed on only one node.

Depending on the strict mode selected, the following happens:

`DISABLED`

At startup, no validation is performed.

At runtime, all operations are permitted.

`PERMISSIVE`

At startup, no validation is perfromed.

At runtime, all operations are permitted, but a warning is logged when you discard or import a tablespace.

`ENFORCING`

At startup, no validation is performed.

At runtime, discarding or importing a tablespace is denied and an error is logged.

### Major version check

This validation checks that the protocol version is the same as the server major version. This validation protects the cluster against writes attempted on already upgraded nodes.

??? example "Expected output"

    ```{.mysql no-copy}

    ERROR 1105 (HY000): Percona-XtraDB-Cluster prohibits use of multiple major versions while accepting write workload with pxc_strict_mode = ENFORCING or MASTER

    ```

