# Index of wsrep system variables

Percona XtraDB Cluster introduces a number of MySQL system variables
related to write-set replication.

### `pxc_encrypt_cluster_traffic`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--pxc-encrypt-cluster-traffic`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | `ON` |

Enables automatic configuration of SSL encryption.
When disabled, you need to configure SSL manually to encrypt Percona XtraDB Cluster traffic.

Possible values:

* `ON`, `1`, `true`: Enabled (default)

* `OFF`, `0`, `false`: Disabled 

For more information, see [SSL Automatic Configuration](security/encrypt-traffic.md#ssl-auto-conf).

### `pxc_maint_mode`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--pxc-maint-mode`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``DISABLED`` |

Specifies the maintenance mode for taking a node down
without adjusting settings in ProxySQL.

The following values are available:

* `DISABLED`: This is the default state
that tells ProxySQL to route traffic to the node as usual.

* `SHUTDOWN`: This state is set automatically
when you initiate node shutdown.

* `MAINTENANCE`: You can manually change to this state
if you need to perform maintenance on a node without shutting it down.

For more information, see [Assisted Maintenance Mode](howtos/proxysql.md#pxc-maint-mode).

### `pxc_maint_transition_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--pxc-maint-transition-period`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``10`` (ten seconds) |

Defines the transition period when you change [`pxc_maint_mode`](wsrep-system-index.md#pxc_maint_mode) to `SHUTDOWN` or ``MAINTENANCE``.
By default, the period is set to 10 seconds,
which should be enough for most transactions to finish.
You can increase the value to accommodate for longer-running transactions.

For more information, see [Assisted Maintenance Mode](howtos/proxysql.md#pxc-maint-mode).

### `pxc_strict_mode`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--pxc-strict-mode`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``ENFORCING`` or ``DISABLED`` |

Controls [PXC Strict Mode](features/pxc-strict-mode.md#pxc-strict-mode), which runs validations to avoid the use of experimental and unsupported features in Percona XtraDB Cluster.

Depending on the actual mode you select,
upon encountering a failed validation,
the server will either throw an error
(halting startup or denying the operation),
or log a warning and continue running as normal.
The following modes are available:

* `DISABLED`: Do not perform strict mode validations
and run as normal.

* `PERMISSIVE`: If a validation fails, log a warning and continue running
as normal.

* `ENFORCING`: If a validation fails during startup,
halt the server and throw an error.
If a validation fails during runtime,
deny the operation and throw an error.

* `MASTER`: The same as `ENFORCING` except that the validation of
[explicit table locking](features/pxc-strict-mode.md#explicit-table-locking) is not performed.
This mode can be used with clusters
in which write operations are isolated to a single node.

By default, [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) is set to `ENFORCING`,
except if the node is acting as a standalone server
or the node is bootstrapping, then [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) defaults to `DISABLED`.

!!! note

    When changing the value of `pxc_strict_mode`
    from `DISABLED` or `PERMISSIVE` to `ENFORCING` or `MASTER`,
    ensure that the following configuration is used:
     
     * `wsrep_replicate_myisam=OFF`
     
     * `binlog_format=ROW`
     
     * `log_output=FILE` or `log_output=NONE` or `log_output=FILE,NONE`
     
    The `SERIALIZABLE` method of isolation is not allowed in `ENFORCING` mode.

For more information, see [PXC Strict Mode](features/pxc-strict-mode.md#pxc-strict-mode).

### `wsrep_applier_FK_checks`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-applier-FK-checks`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |
| Default Value: | ``ON``             |

As of *Percona XtraDB Cluster* 8.0.26-16, the ``wsrep_slave_FK_checks`` variable is deprecated in favor of this variable.

Defines whether foreign key checking is done for applier threads.
This is enabled by default.

!!! admonition "See also"

    [`MySQL wsrep option: wsrep_applier_FK_checks`](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-applier-fk-checks)

### `wsrep_applier_threads`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-applier-threads`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |
| Default Value: | ``1``              |

As of *Percona XtraDB Cluster* 8.0.26-16, the ``wsrep_slave_threads`` variable is deprecated and may be removed in a later version. Use the ``wsrep_applier_threads`` variable.

Specifies the number of threads
that can apply replication transactions in parallel.
Galera supports true parallel replication
that applies transactions in parallel only when it is safe to do so.
This variable is dynamic.
You can increase/decrease it at any time.

!!! note 

    When you decrease the number of threads,
    it won't kill the threads immediately,
    but stop them after they are done applying current transaction
    (the effect with an increase is immediate though).

If any replication consistency problems are encountered,
it's recommended to set this back to ``1`` to see if that resolves the issue.
The default value can be increased for better throughput.

You may want to increase it as suggested in [`Codership documentation for flow control`](https://galeracluster.com/documentation-webpages/nodestates.html#flow-control): when the node is in ``JOINED`` state, increasing the number of replica threads can speed up the catchup to ``SYNCED``.

You can also estimate the optimal value for this from [`wsrep_cert_deps_distance`](wsrep-status-index.md#wsrep_cert_deps_distance) as suggested [`on this page`](https://galeracluster.com/documentation-webpages/monitoringthecluster.html#checking-the-replication-health).

For more configuration tips, see [`this document`](https://galeracluster.com/documentation-webpages/configurationtips.html#setting-parallel-slave-threads).

!!! admonition "See also"

    [`MySQL wsrep option: wsrep_applier_threads`](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-applier-threads)

### `wsrep_applier_UK_checks`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-applier-UK-checks`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |
| Default Value: | ``OFF``            |

As of *Percona XtraDB Cluster* 8.0.26-16, the ``wsrep_slave_UK_checks`` variable is deprecated and may be removed in a later version. Use the ``wsrep_applier_UK_checks`` variable.

Defines whether unique key checking is done for applier threads.
This is disabled by default.

!!! admonition "See also"

    [`MySQL wsrep option: wsrep_applier_UK_checks`](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-applier-uk-checks)

### `wsrep_auto_increment_control`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-auto-increment-control`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``ON`` |

Enables automatic adjustment of auto-increment system variables
depending on the size of the cluster:

* `auto_increment_increment` controls the interval
between successive `AUTO_INCREMENT` column values

* `auto_increment_offset` determines the starting point
for the `AUTO_INCREMENT` column value

This helps prevent auto-increment replication conflicts across the cluster
by giving each node its own range of auto-increment values.
It is enabled by default.

Automatic adjustment may not be desirable depending on application’s use
and assumptions of auto-increments.
It can be disabled in source-replica clusters.

!!! admonition "See also"

    [`MySQL wsrep option: wsrep_auto_increment_control`](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-auto-increment-control)

### `wsrep_causal_reads`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-causal-reads`` |
| Config File:   | Yes                |
| Scope:         | Global, Session             |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

In some cases, the source may apply events faster than a replica,
which can cause source and replica to become out of sync for a brief moment.
When this variable is set to `ON`, the replica will wait
until that event is applied before doing any other queries.
Enabling this variable will result in larger latencies.

!!! note

    This variable was deprecated because enabling it is the equivalent of setting
    [`wsrep_sync_wait`](wsrep-system-index.md#wsrep_sync_wait) to `1`.

!!! admonition "See also"

    [`MySQL wsrep option: wsrep_causal_reads`](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-causal-reads)

### `wsrep_certification_rules`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-certification-rules`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |
| Values:        | STRICT, OPTIMIZED  |
| Default Value: | STRICT |

This variable controls how certification is done in the cluster, in particular
this affects how foreign keys are handled.

**STRICT**
   Two INSERTs that happen at about the same time on two different nodes in a
   child table, that insert different (non conflicting rows), but both rows
   point to the same row in the parent table **may result** in the certification
   failure.

**OPTIMIZED**
   Two INSERTs that happen at about the same time on two different nodes in a
   child table, that insert different (non conflicting rows), but both rows
   point to the same row in the parent table **will not result** in the
   certification failure.

!!! admonition "See also"

    [Galera Cluster Documentation: MySQL wsrep options](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-certification-rules)

### `wsrep_certify_nonPK`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-certify-nonpk`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | ``ON`` |

Enables automatic generation of primary keys for rows that don’t have them.
Write set replication requires primary keys on all tables
to allow for parallel applying of transactions.
This variable is enabled by default.
As a rule, make sure that all tables have primary keys.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_certify_nonPK](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-certify-nonpk)

### `wsrep_cluster_address`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-cluster-address`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |

Defines the back-end schema, IP addresses, ports, and options
that the node uses when connecting to the cluster.
This variable needs to specify at least one other node’s address,
which is alive and a member of the cluster.
In practice, it is best (but not necessary) to provide a complete list
of all possible cluster nodes.
The value should be of the following format:

```text
<schema>://<address>[?<option1>=<value1>[&<option2>=<value2>]],...
```

The only back-end schema currently supported is `gcomm`.
The IP address can contain a port number after a colon.
Options are specified after `?` and separated by `&`.
You can specify multiple addresses separated by commas.

For example:

```text
wsrep_cluster_address="gcomm://192.168.0.1:4567?gmcast.listen_addr=0.0.0.0:5678"
```

If an empty `gcomm://` is provided, the node will bootstrap itself
(that is, form a new cluster).
It is not recommended to have empty cluster address in production config
after the cluster has been bootstrapped initially.
If you want to bootstrap a new cluster with a node,
you should pass the `--wsrep-new-cluster` option when starting.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_cluster_address](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-cluster-address)

### `wsrep_cluster_name`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-cluster-name`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | ``my_wsrep_cluster`` |

Specifies the name of the cluster and should be identical on all nodes.

!!! note

    It should not exceed 32 characters. A node cannot join the cluster if the cluster names do not match. You must re-bootstrap the cluster after a name change.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_cluster_name](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-cluster-name)

### `wsrep_data_home_dir`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | No |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | ``/var/lib/mysql`` (or whatever path is specified by `datadir`) |

Specifies the path to the directory where the wsrep provider stores its files
(such as `grastate.dat`).

!!! admonition "See also"

    [MySQL wsrep option: wsrep_data_home_dir](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-data-home-dir)

### `wsrep_dbug_option`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-dbug-option`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |

Defines `DBUG` options to pass to the wsrep provider.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_dbug_option](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-dbug-option)

### `wsrep_debug`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-debug`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``NONE`` |

Enables debug level logging for the database server and [`wsrep-lib`]
(https://github.com/codership/wsrep-lib) - an integration library for WSREP
API with additional convenience for transaction processing. By default,
`--wsrep-debug` variable is disabled.

This variable can be used when trying to diagnose problems or when submitting a
bug.

You can set `wsrep_debug` in the following `my.cnf` groups:

* Under `[mysqld]` it enables debug logging for `mysqld` and the SST script.

* Under `[sst]` it enables debug logging for the SST script only.

This variable may be set to one of the following values:

**NONE**

No debug-level messages.

**SERVER**

``wsrep-lib`` general debug-level messages and detailed debug-level messages
from the *server_state* part are printed out.  Galera debug-level logs are
printed out.

**TRANSACTION**

Same as SERVER + wsrep-lib *transaction* part

**STREAMING**

Same as TRANSACTION + wsrep-lib *streaming* part

**CLIENT**

Same as STREAMING + wsrep-lib *client_service* part

!!! note

    Do not enable debugging in production environments, because it logs authentication info (that is, passwords).

!!! admonition "See also"

    [MySQL wsrep option: wsrep_debug](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-debug)

### `wsrep_desync`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | No |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

Defines whether the node should participate in Flow Control.
By default, this variable is disabled,
meaning that if the receive queue becomes too big,
the node engages in Flow Control:
it works through the receive queue until it reaches a more manageable size.
For more information, see [`wsrep_local_recv_queue`](wsrep-status-index.md#wsrep_local_recv_queue) and [`wsrep_flow_control_interval`](wsrep-status-index.md#wsrep_flow_control_interval).

Enabling this variable will disable Flow Control for the node.
It will continue to receive write-sets that it is not able to apply,
the receive queue will keep growing,
and the node will keep falling behind the cluster indefinitely.

Toggling this back to `OFF` will require an IST or an SST,
depending on how long it was desynchronized.
This is similar to cluster desynchronization, which occurs during RSU TOI.
Because of this, it’s not a good idea to enable `wsrep_desync`
for a long period of time or for several nodes at once.

!!! note

    You can also desync a node using the `/\*! WSREP_DESYNC \*/` query comment.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_desync](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-desync)

### `wsrep_dirty_reads`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-dirty-reads`` |
| Config File:   | Yes                |
| Scope:         | Session, Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

Defines whether the node accepts read queries when in a non-operational state,
that is, when it loses connection to the Primary Component.
By default, this variable is disabled and the node rejects all queries,
because there is no way to tell if the data is correct.

If you enable this variable, the node will permit read queries
(`USE`, `SELECT`, `LOCK TABLE`, and `UNLOCK TABLES`),
but any command that modifies or updates the database
on a non-operational node will still be rejected
(including DDL and DML statements,
such as `INSERT`, `DELETE`, and `UPDATE`).

To avoid deadlock errors,
set the [`wsrep_sync_wait`](wsrep-system-index.md#wsrep_sync_wait) variable to `0`
if you enable `wsrep_dirty_reads`.

As of Percona XtraDB Cluster 8.0.26-16, you can update the variable with a [`set_var hint`](https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html#optimizer-hints-set-var).

```sql
mysql> SELECT @@wsrep_dirty_reads;
```

The example of the output:

```text
+-----------------------+
| @@wsrep_dirty_reads   |
+=======================+
| OFF                   |
+-----------------------+
```

```sql
mysql> SELECT /*+ SET_VAR(wsrep_dirty_reads=ON) */ @@wsrep_dirty_reads;
```

The example of the output:

```text
+-----------------------+
| @@wsrep_dirty_reads   |
+=======================+
| ON                    |
+-----------------------+
```

!!! admonition "See also"

    [MySQL wsrep option: wsrep_dirty_reads](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-dirty-reads)

### `wsrep_drupal_282555_workaround`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-drupal-282555-workaround`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

Enables a workaround for MySQL InnoDB bug that affects Drupal
([Drupal bug #282555](http://drupal.org/node/282555)
and [MySQL bug #41984](http://bugs.mysql.com/bug.php?id=41984)).
In some cases, duplicate key errors would occur
when inserting the `DEFAULT` value into an `AUTO_INCREMENT` column.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_drupal_282555_workaround](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-drupal-282555-workaround)

### `wsrep_forced_binlog_format`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-forced-binlog-format`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``NONE`` |

Defines a binary log format that will always be effective,
regardless of the client session [`binlog_format`](https://dev.mysql.com/doc/refman/8.0/en/binary-log-setting.html) variable value.

Possible values for this variable are:

* `ROW`: Force row-based logging format

* `STATEMENT`: Force statement-based logging format

* `MIXED`: Force mixed logging format

* `NONE`: Do not force the binary log format and use whatever is set by the `binlog_format` variable (default)

!!! admonition "See also"

    [MySQL wsrep option: wsrep_forced_binlog_format](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-forced-binlog-format)

### `wsrep_ignore_apply_errors`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-ignore-apply-errors`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | 0 |

Defines the rules of wsrep applier behavior on errors. You can change the settings by editing the ``my.cnf`` file under ``[mysqld]`` or at runtime.

!!! note
 
    In Percona XtraDB Cluster version 8.0.19-10, the default value has changed from ``7`` to ``0``. If you have been working with an earlier version of the PXC 8.0 series, you may see different behavior when upgrading to this version or later.

The variable has the following options:

| Value| Description|
| ----- | ----------|
|WSREP_IGNORE_ERRORS_NONE|All replication errors are treated as errors and will shutdown the node (default behavior)|
|WSREP_IGNORE_ERRORS_ON_RECONCILING_DDL| DROP DATABASE, DROP TABLE, DROP INDEX, ALTER TABLE are converted to a warning if they result in ER_DB_DROP_EXISTS, ER_BAD_TABLE_ERROR OR ER_CANT_DROP_FIELD_OR_KEY errors|
|WSREP_IGNORE_ERRORS_ON_RECONCILING_DML| DELETE events are treated as warnings if they failed because the deleted row was not found (ER_KEY_NOT_FOUND)|
|WSREP_IGNORE_ERRORS_ON_DDL| All DDL errors will be treated as a warning|
|WSREP_IGNORE_ERRORS_MAX| Infers WSREP_IGNORE_ERRORS_ON_RECONCILING_DDL, WSREP_IGNORE_ERRORS_ON_RECONCILING_DML and WSREP_IGNORE_ERRORS_ON_DDL|

Setting the variable between ``0`` and ``7`` has the following behavior:

| Setting| Behavior|
| ----- | ----------|
|0|WSREP_IGNORE_ERRORS_NONE|
|1| WSREP_IGNORE_ERRORS_ON_RECONCILING_DDL|
|2| WSREP_IGNORE_ERRORS_ON_RECONCILING_DML|
|3| WSREP_IGNORE_ERRORS_ON_RECONCILING_DDL, WSREP_IGNORE_ERRORS_ON_RECONCILING_DML|
|4| WSREP_IGNORE_ERRORS_ON_DDL|
|5| WSREP_IGNORE_ERRORS_ON_DDL, WSREP_IGNORE_ERRORS_ON_RECONCILING_DDL|
|6| WSREP_IGNORE_ERRORS_ON_DDL, WSREP_IGNORE_ERRORS_ON_RECONCILING_DML|
|7| WSREP_IGNORE_ERRORS_ON_DDL, WSREP_IGNORE_ERRORS_ON_RECONCILING_DML, WSREP_IGNORE_ERRORS_ON_RECONCILING_DDL|

### `wsrep_min_log_verbosity`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-min-log-verbosity`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | 3 |

This variable defines the *minimum* logging verbosity of wsrep/Galera and acts
in conjunction with the ``log_error_verbosity`` variable. The
[`wsrep_min_log_verbosity`]() has the same values as
``log_error_verbosity``.

The actual log verbosity of wsrep/Galera can be greater than the value of
[`wsrep_min_log_verbosity`]() if ``log_error_verbosity`` is greater than
[`wsrep_min_log_verbosity`]().

A few examples:

|log_error_verbosity| wsrep_min_log_verbosity| MySQL Logs Verbosity| wsrep Logs Verbosity|
| ----------------- | ---------------------- |-------------------- |-------------------- |
|2|3|system error, warning|system error, warning, info|
|1|3|system error|system error, warning, info|
|1|2|system error|system error, warning|
|3|1|system error, warning, info|system error, warning, info|

Note the case where ``log_error_verbosity=3`` and ``wsrep_min_log_verbosity=1``. The actual log verbosity of wsrep/Galera is *3* (system error, warning, info) because ``log_error_verbosity`` is greater.

!!! admonition "See also"

    [MySQL Documentation: log_error_verbosity](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_log_error_verbosity)

    [Galera Cluster Documentation: Database Server Logs](https://galeracluster.com/library/documentation/log.html)

### `wsrep_load_data_splitting`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-load-data-splitting`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``ON`` |

Defines whether the node should split large `LOAD DATA` transactions.
This variable is enabled by default, meaning that `LOAD DATA` commands
are split into transactions of 10 000 rows or less.

If you disable this variable, then huge data loads may prevent the node
from completely rolling the operation back in the event of a conflict,
and whatever gets committed stays committed.

!!! note

    It doesn’t work as expected with `autocommit=0` when enabled.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_load_data_splitting](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-load-data-splitting)

### `wsrep_log_conflicts`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-log-conflicts`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | ``OFF`` |

Defines whether the node should log additional information about conflicts.
By default, this variable is disabled
and Percona XtraDB Cluster uses standard logging features in MySQL.

If you enable this variable, it will also log table and schema
where the conflict occurred, as well as the actual values for keys
that produced the conflict.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_log_conflicts](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-log-conflicts)

### `wsrep_max_ws_rows`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-max-ws-rows`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``0`` (no limit) |

Defines the maximum number of rows each write-set can contain.

By default, there is no limit for the maximum number of rows in a write-set.
The maximum allowed value is `1048576`.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_max_ws_rows](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-max-ws-rows)

### `wsrep_max_ws_size`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep_max_ws_size`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``2147483647`` (2 GB) |

Defines the maximum write-set size (in bytes).
Anything bigger than the specified value will be rejected.

You can set it to any value between `1024` and the default `2147483647`.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_max_ws_size](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-max-ws-size)

### `wsrep_node_address`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-node-address`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | IP of the first network interface (``eth0``) and default port (``4567``) |

Specifies the network address of the node.
By default, this variable is set to the IP address
of the first network interface (usually `eth0` or `enp2s0`)
and the default port (`4567`).

While default value should be correct in most cases,
there are situations when you need to specify it manually.
For example:

* Servers with multiple network interfaces

* Servers that run multiple nodes

* Network Address Translation (NAT)

* Clusters with nodes in more than one region

* Container deployments, such as Docker

* Cloud deployments, such as Amazon EC2
(use the global DNS name instead of the local IP address)

The value should be specified in the following format:

```text
<ip_address>[:port]
```

!!! note

    The value of this variable is also used as the default value
    for the [`wsrep_sst_receive_address`](wsrep-system-index.md#wsrep_sst_receive_address) variable and the [`ist.recv_addr`](wsrep-provider-index.md#ist.recv_addr) option.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_node_address](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-node-address)

### `wsrep_node_incoming_address`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-node-incoming-address`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | ``AUTO`` |

Specifies the network address from which the node expects client connections.
By default, it uses the IP address from [`wsrep_node_address`](wsrep-system-index.md#wsrep_node_address) and port number 3306.

This information is used for the [`wsrep_incoming_addresses`](wsrep-status-index.md#wsrep_incoming_addresses) variable
which shows all active cluster nodes.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_node_incoming_address](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-node-incoming-address)

### `wsrep_node_name`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-node-name`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | The node's host name |

Defines a unique name for the node. Defaults to the host name.

In many situations, you may use the value of this variable as a means to
identify the given node in the cluster as the alternative to using the node address
(the value of the [`wsrep_node_address`](wsrep-system-index.md#wsrep_node_address)).

!!! note

    The variable [`wsrep_sst_donor`](wsrep-system-index.md#wsrep_sst_donor) is an example where you may only use
    the value of [`wsrep_node_name`](wsrep-system-index.md#wsrep_node_name) and the node address is not permitted.

### `wsrep_notify_cmd`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-notify-cmd`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |

Specifies the [notification command](https://galeracluster.com/library/documentation/notification-cmd.html)
that the node should execute
whenever cluster membership or local node status changes.
This can be used for alerting or to reconfigure load balancers.

!!! note

    The node will block and wait
    until the command or script completes and returns before it can proceed.
    If the script performs any potentially blocking
    or long-running operations, such as network communication,
    you should consider initiating such operations in the background
    and have the script return immediately.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_notify_cmd](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-notify-cmd)

### `wsrep_on`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | No |
| Config File:   | No                |
| Scope:         | Session             |
| Dynamic:       | Yes                 |
| Default Value: | ``ON`` |

Defines if current session transaction changes for a node are replicated to the cluster.

If set to `OFF` for a session, no transaction changes are replicated in that session. The setting does not cause the node to leave the cluster, and the node communicates with other nodes.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_on](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-on)

### `wsrep_OSU_method`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-OSU-method`` |
| Config File:   | Yes                |
| Scope:         | Global, Session             |
| Dynamic:       | Yes                 |
| Default Value: | ``TOI`` |

Defines the method for Online Schema Upgrade
that the node uses to replicate DDL statements.

For information on the available methods, see [Online Schema upgrade](./features/online-schema-upgrade.md) and for information on Non-blocking operations, see [NBO](./features/nbo.md).


!!! admonition "See also"

    [MySQL wsrep option: wsrep_OSU_method](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-osu-method)

### `wsrep_provider`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-provider`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |

Specifies the path to the Galera library.
This is usually
`/usr/lib64/libgalera_smm.so` on *CentOS*/*RHEL* and
`/usr/lib/libgalera_smm.so` on *Debian*/*Ubuntu*.

If you do not specify a path or the value is not valid,
the node will behave as standalone instance of MySQL.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_provider](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-provider)

### `wsrep_provider_options`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-provider-options`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | No                 |

Specifies optional settings for the replication provider
documented in [Index of :variable:\`wsrep_provider\` options](wsrep-provider-index.md#wsrep-provider-index).
These options affect how various situations are handled during replication.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_provider_options](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-provider-options)

### `wsrep_recover`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-recover`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | No                 |
| Default Value: | ``OFF`` |
| Location:      | mysqld_safe` |

Recovers database state after crash by parsing GTID from the log.
If the GTID is found, it will be assigned as the initial position for server.

### `wsrep_reject_queries`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | No |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | ``NONE`` |

Defines whether the node should reject queries from clients.
Rejecting queries can be useful during upgrades,
when you want to keep the node up and apply write-sets
without accepting queries.

When a query is rejected, the following error is returned:

```text
Error 1047: Unknown command
```

The following values are available:

* `NONE`: Accept all queries from clients (default)

* `ALL`: Reject all new queries from clients, but maintain existing client connections

* `ALL_KILL`: Reject all new queries from clients and kill existing client connections

!!! note

    This variable doesn’t affect Galera replication in any way,
    only the applications that connect to the database are affected.
    If you want to desync a node, use [`wsrep_desync`](wsrep-system-index.md#wsrep_desync).

!!! admonition "See also"

    [MySQL wsrep option: wsrep_reject_queries](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-reject-queries)

### `wsrep_replicate_myisam`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-replicate-myisam`` |
| Config File:   | Yes                |
| Scope:         | Session, Global            |
| Dynamic:       | No                 |
| Default Value: | ``OFF`` |

Defines whether DML statements for MyISAM tables should be replicated.
It is disabled by default, because MyISAM replication is still experimental.

On the global level, `wsrep_replicate_myisam` can be set only during startup.
On session level, you can change it during runtime as well.

For older nodes in the cluster, `wsrep_replicate_myisam` should work
since the TOI decision (for MyISAM DDL) is done on origin node.
Mixing of non-MyISAM and MyISAM tables in the same DDL statement
is not recommended when `wsrep_replicate_myisam` is disabled,
since if any table in the list is MyISAM,
the whole DDL statement is not put under TOI.

!!! note

    You should keep in mind the following when using MyISAM replication:

     * DDL (CREATE/DROP/TRUNCATE) statements on MyISAM will be replicated irrespective of [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) value

     * DML (INSERT/UPDATE/DELETE) statements on MyISAM will be replicated only if[`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) is enabled

     * SST will get full transfer irrespective of [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam)value (it will get MyISAM tables from donor)

     * Difference in configuration of `pxc-cluster` node on [enforce_storage_engine](https://www.percona.com/doc/percona-server/8.0/management/enforce_engine.html) front may result in picking up different engine for the same table on different nodes

     * `CREATE TABLE AS SELECT` (CTAS) statements use TOI replication. MyISAM tables are created and loaded even if `wsrep_replicate_myisam` is set to **ON**.

### `wsrep_restart_replica`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-restart-replica`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

As of *Percona XtraDB Cluster* 8.0.26-16, the ``wsrep_restart_slave`` variable is deprecated in favor of this variable. 

Defines whether replication replica should be restarted
when the node joins back to the cluster.
Enabling this can be useful because asynchronous replication replica thread
is stopped when the node tries to apply the next replication event
while the node is in non-primary state.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_restart_slave](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-restart-replica)

### `wsrep_restart_slave`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-restart-slave`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

As of *Percona XtraDB Cluster* 8.0.26-16, the ``wsrep_restart_slave`` variable is deprecated and may be removed in later versions. Use ``wsrep_restart_replica``.

Defines whether replication replica should be restarted
when the node joins back to the cluster.
Enabling this can be useful because asynchronous replication replica thread
is stopped when the node tries to apply the next replication event
while the node is in non-primary state.

### `wsrep_retry_autocommit`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-retry-autocommit`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | No                 |
| Default Value: | ``1`` |

Specifies the number of times autocommit transactions will be retried
in the cluster if it encounters certification errors.
In case there is a conflict, it should be safe for the cluster node
to simply retry the statement without returning an error to the client,
hoping that it will pass next time.

This can be useful to help an application using autocommit
to avoid deadlock errors that can be triggered by replication conflicts.

If this variable is set to `0`,
autocommit transactions won’t be retried.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_retry_autocommit](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-retry-autocommit)

### `wsrep_RSU_commit_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-RSU-commit-timeout`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | ``5000`` |
| Range:         | From ``5000`` (5 milliseconds) to ``31536000000000`` (365 days) |

Specifies the timeout in microseconds to allow active connection to complete
COMMIT action before starting RSU.

While running RSU it is expected that user has isolated the node and there is
no active traffic executing on the node. RSU has a check to ensure this, and
waits for any active connection in `COMMIT` state before starting RSU.

By default this check has timeout of 5 milliseconds, but in some cases
COMMIT is taking longer. This variable sets the timeout, and has allowed values
from the range of (5 milliseconds, 365 days). The value is to be set in
microseconds. Unit of variable is in micro-secs so set accordingly.

!!! note

    RSU operation will not auto-stop node from receiving active traffic.
    So there could be a continuous flow of active traffic while RSU continues to
    wait, and that can result in RSU starvation. User is expected to block
    active RSU traffic while performing operation.

### `wsrep_slave_FK_checks`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-slave-FK-checks`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | ``ON`` |

As of *Percona XtraDB Cluster* 8.0.26-16, this variable is deprecated and may be removed in a later version. Use the ``wsrep_applier_FK_checks`` variable.

Defines whether foreign key checking is done for applier threads.
This is enabled by default.

### `wsrep_slave_threads`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-slave-threads`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | ``1`` |

As of *Percona XtraDB Cluster* 8.0.26-16, this variable is deprecated and may be removed in a later version. Use the ``wsrep_applier_threads`` variable.

Specifies the number of threads
that can apply replication transactions in parallel.
Galera supports true parallel replication
that applies transactions in parallel only when it is safe to do so.
This variable is dynamic.
You can increase/decrease it at any time.

!!! note

    When you decrease the number of threads,
    it won’t kill the threads immediately,
    but stop them after they are done applying current transaction
    (the effect with an increase is immediate though).

If any replication consistency problems are encountered,
it’s recommended to set this back to `1` to see if that resolves the issue.
The default value can be increased for better throughput.

You may want to increase it as suggested
in [`Codership documentation for flow control`](http://galeracluster.com/documentation-webpages/nodestates.html#flow-control):
when the node is in ``JOINED`` state,
increasing the number of replica threads can speed up the catchup to ``SYNCED``.

You can also estimate the optimal value for this from [`wsrep_cert_deps_distance`](wsrep-status-index.md#wsrep_cert_deps_distance) as suggested [in the Galera Cluster documentation](https://galeracluster.com/library/training/tutorials/galera-monitoring.html).

For more configuration tips, see [this document](https://galeracluster.com/library/kb/parallel-slave-threads.html).

### `wsrep_slave_UK_checks`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-slave-UK-checks`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

As of *Percona XtraDB Cluster* 8.0.26-16, this variable is deprecated and may be removed in a later version. Use the ``wsrep_applier_UK_checks`` variable.

Defines whether unique key checking is done for applier threads.
This is disabled by default.

### `wsrep_sst_donor`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |

Specifies a list of nodes (using their `wsrep_node_name` values)
that the current node should prefer as donors for [SST](glossary.md#sst) and [IST](glossary.md#ist).

!!! warning

    Using IP addresses of nodes instead of node names (the value of  [`wsrep_node_name`](wsrep-system-index.md#wsrep_node_name)) as values of [`wsrep_sst_donor`](wsrep-system-index.md#wsrep_sst_donor) results in an error.
    
    ```text
    ERROR] WSREP: State transfer request failed unrecoverably: 113 (No route
    to host). Most likely it is due to inability to communicate with the
    cluster primary component. Restart required.
    ```

If the value is empty, the first node in SYNCED state in the index
becomes the donor and will not be able to serve requests during the state transfer.

To consider other nodes if the listed nodes are not available,
add a comma at the end of the list, for example:

```text
wsrep_sst_donor=node1,node2,
```

If you remove the trailing comma from the previous example,
then the joining node will consider *only* `node1` and `node2`.

!!! note

    By default, the joiner node does not wait for more than 100 seconds
    to receive the first packet from a donor.
    This is implemented via the [`sst-initial-timeout`](manual/xtrabackup_sst.md#cmdoption-arg-sst-initial-timeout) option.
    If you set the list of preferred donors without the trailing comma
    or believe that all nodes in the cluster can often be unavailable for SST
    (this is common for small clusters),
    then you may want to increase the initial timeout
    (or disable it completely
    if you don’t mind the joiner node waiting for the state transfer indefinitely).

!!! admonition "See also"

    [MySQL wsrep option: wsrep_sst_donor](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-sst-donor)

### `wsrep_sst_method`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-sst-method`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | xtrabackup-v2 |

Defines the method or script for [State Snapshot Transfer](manual/state_snapshot_transfer.md#state-snapshot-transfer) (SST).

Available values are:

* `xtrabackup-v2`: Uses *Percona XtraBackup* to perform SST.
This method requires [`wsrep_sst_auth`](wsrep-system-index.md#wsrep_sst_auth) to be set up with credentials (`<user>:<password>`) on the donor node.
Privileges and permissions for running *Percona XtraBackup*
can be found in [Percona XtraBackup documentation](https://www.percona.com/doc/percona-xtrabackup/2.4/using_xtrabackup/privileges.html). 

For more information, see [Percona XtraBackup SST Configuration](manual/xtrabackup_sst.md#xtrabackup-sst).

* `<custom_script_name>`: Galera supports [Scriptable State Snapshot Transfer](https://galeracluster.com/library/documentation/scriptable-sst.html).
This enables users to create their own custom scripts for performing SST.
For example, you can create a script `/usr/bin/wsrep_MySST.sh`
and specify `MySST` for this variable to run your custom SST script.

* `skip`: Use this to skip SST.
This can be used when initially starting the cluster
and manually restoring the same data to all nodes.
It shouldn’t be used permanently
because it could lead to data inconsistency across the nodes.

!!! note

    ``xtrabackup-v2`` provides support for clusters with GTIDs and async replicas.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_sst_method](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-sst-method)

### `wsrep_sst_receive_address`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-sst-receive-address`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | ``AUTO`` |

Specifies the network address where donor node should send state transfers.
By default, this variable is set to `AUTO`,
meaning that the IP address from [`wsrep_node_address`](wsrep-system-index.md#wsrep_node_address) is used.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_sst_receive_address](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-sst-receive-address)

### `wsrep_start_position`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-start-position`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | ``00000000-0000-0000-0000-00000000000000:-1`` |

Specifies the node’s start position as `UUID:seqno`.
By setting all the nodes to have the same value for this variable,
the cluster can be set up without the state transfer.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_start_position](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-start-position)

### `wsrep_sync_wait`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-sync-wait`` |
| Config File:   | Yes                |
| Scope:         | Session           |
| Dynamic:       | Yes                 |
| Default Value: | ``0`` |

Controls cluster-wide causality checks on certain statements.
Checks ensure that the statement is executed on a node
that is fully synced with the cluster.

As of Percona XtraDB Cluster 8.0.26-16, you are able to update the variable with a [set_var hint](https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html#optimizer-hints-set-var).

```sql
   mysql> SELECT @@wsrep_sync_wait;
```
The example of the output is the following:

```text
   +---------------------+
   | @@wsrep_sync_wait   |
   +=====================+
   | 3                   |
   +---------------------+
```

```sql
   mysql> SELECT /*+ SET_VAR(wsrep_sync_wait=7) */ @@wsrep_sync_wait;
```

The example of the output is the following:

```text
   +---------------------+
   | @@wsrep_sync_wait   |
   +=====================+
   | 7                   |
   +---------------------+
```

!!! note

    Causality checks of any type can result in increased latency.

The type of statements to undergo checks
is determined by bitmask:

* `0`: Do not run causality checks for any statements. This is the default.

* `1`: Perform checks for `READ` statements (including `SELECT`, `SHOW`, and `BEGIN` or `START TRANSACTION`).
 
* `2`: Perform checks for `UPDATE` and `DELETE` statements.

* `3`: Perform checks for `READ`, `UPDATE`, and `DELETE` statements.

* `4`: Perform checks for `INSERT` and `REPLACE` statements.

* `5`: Perform checks for `READ`, `INSERT`, and `REPLACE` statements.

* `6`: Perform checks for `UPDATE`, `DELETE`, `INSERT`,
and `REPLACE` statements.

* `7`: Perform checks for `READ`, `UPDATE`, `DELETE`, `INSERT`, and `REPLACE` statements.

!!! note

    Setting [`wsrep_sync_wait`](wsrep-system-index.md#wsrep_sync_wait) to `1` is the equivalent of setting the deprecated [`wsrep_causal_reads`](wsrep-system-index.md#wsrep_causal_reads) to `ON`.

!!! admonition "See also"

    [MySQL wsrep option: wsrep_sync_wait](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-sync-wait)

### `wsrep_trx_fragment_size`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-trx-fragment-size`` |
| Config File:   | Yes                |
| Scope:         | Global, Session          |
| Dynamic:       | Yes                 |
| Default Value: | 0 |

Defines the the streaming replication fragment size. This variable is measured in the value defined by ``wsrep_trx_fragment_unit``. The minimum value is 0 and the maximum value is 2147483647.

As of *Percona XtraDB Cluster for MySQL* 8.0.26-16, you can update the variable with a [set_var hint](https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html#optimizer-hints-set-var).

```sql
mysql> SELECT @@@wsrep_trx_fragment_unit; SELECT @@wsrep_trx_fragment_size;
```

The example of the output is the following:

```text
+------------------------------+
| @@wsrep_trx_fragment_unit    |
+==============================+
| statements                   |
+------------------------------+
| @@wsrep_trx_fragment_size    |
+------------------------------+
| 3                            |
+------------------------------+
```

```sql
mysql> SELECT /*+ SET_VAR(wsrep_trx_fragment_size=5) */ @@wsrep_trx_fragment_size;
```

The example of the output is the following:

```text
+------------------------------+
| @@wsrep_trx_fragment_size    |
+==============================+
| 5                            |
+------------------------------+
```

You can also use set_var() in a data manipulation language (DML) statement. This ability is useful when streaming large statements within a transaction.

```text
node1> BEGIN;
Query OK, 0 rows affected (0.00 sec)

node1> INSERT /*+SET_VAR(wsrep_trx_fragment_size = 100)*/ INTO t1 SELECT * FROM t1; 
Query OK, 65536 rows affected (15.15 sec)
Records: 65536 Duplicates: 0 Warnings: 0

node1> UPDATE /*+SET_VAR(wsrep_trx_fragment_size = 100)*/ t1 SET i=2;
Query OK, 131072 rows affected (1 min 35.93 sec)
Rows matched: 131072 Changed: 131072 Warnings: 0

node2> SET SESSION TRANSACTION_ISOLATION = 'READ-UNCOMMITTED';
Query OK, 0 rows affected (0.00 sec)

node2> SELECT * FROM t1 LIMIT 5;
+---+
| i |
+===+
| 2 |
+---+
| 2 |
+---+
| 2 |
+---+
| 2 |
+---+
| 2 |
+---+
node1> DELETE  /*+SET_VAR(wsrep_trx_fragment_size = 10000)*/ FROM t1;
Query OK, 131072 rows affected (15.09 sec)
```

### `wsrep_trx_fragment_unit`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-trx-fragment-unit`` |
| Config File:   | Yes                |
| Scope:         | Global, Session          |
| Dynamic:       | Yes                 |
| Default Value: | "bytes" |

Defines the type of measure for the ``wsrep_trx_fragment_size``. The possible values are: bytes, rows, statements. 

As of *Percona XtraDB Cluster for MySQL* 8.0.26-16, you can update the variable with a [set_var hint](https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html#optimizer-hints-set-var).

```sql
mysql> SELECT @@wsrep_trx_fragment_unit; SELECT @@wsrep_trx_fragment_size;
```

The example of the output is the following:

```text
+------------------------------+
| @@wsrep_trx_fragment_unit    |
+==============================+
| statements                   |
+------------------------------+
| @@wsrep_trx_fragment_size    |
+------------------------------+
| 3                            |
+------------------------------+
```

```sql
mysql> SELECT /*+ SET_VAR(wsrep_trx_fragment_unit=rows) */ @@wsrep_trx_fragment_unit;
```

The example of the output is the following:

```text
+------------------------------+
| @@wsrep_trx_fragment_unit    |
+==============================+
| rows                         |
+------------------------------+
```