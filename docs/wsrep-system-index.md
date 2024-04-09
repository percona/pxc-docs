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
| Default Value: | `OFF` |

This variable has been implemented in [`5.7.16`](release-notes/Percona-XtraDB-Cluster-5.7.16-27.19.md). Enables automatic configuration of SSL encryption.
When disabled, you need to configure SSL manually to encrypt Percona XtraDB Cluster traffic.

Possible values:

* `OFF`, `0`, `false`: Disabled (default)

* `ON`, `1`, `true`: Enabled

For more information, see [SSL Automatic Configuration](security/encrypt-traffic.md#ssl-auto-conf).

### `pxc_maint_mode`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--pxc-maint-mode`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``DISABLED`` |

This variable has been implemented in [`5.7.16`](release-notes/Percona-XtraDB-Cluster-5.7.16-27.19.md). Specifies the maintenance mode for taking a node down
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

This variable has been implemented in [`5.7.16`](release-notes/Percona-XtraDB-Cluster-5.7.16-27.19.md). Defines the transition period
when you change [`pxc_maint_mode`](wsrep-system-index.md#pxc_maint_mode) to `SHUTDOWN`.
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

This variable has been implemented in `5.7`.Controls [PXC Strict Mode](features/pxc-strict-mode.md#pxc-strict-mode), which runs validations
to avoid the use of experimental and unsupported features in Percona XtraDB Cluster.

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

### `wsrep_causal_reads`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-causal-reads`` |
| Config File:   | Yes                |
| Scope:         | Global, Session             |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

This variable has been implemented in `5.6.20-25.7`. In some cases, the source may apply events faster than a replica,
which can cause source and replica to become out of sync for a brief moment.
When this variable is set to `ON`, the replica will wait
until that event is applied before doing any other queries.
Enabling this variable will result in larger latencies.

!!! note

    This variable was deprecated because enabling it is the equivalent of setting
    [`wsrep_sync_wait`](wsrep-system-index.md#wsrep_sync_wait) to `1`.

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

### `wsrep_cluster_name`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-cluster-name`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | ``my_wsrep_cluster`` |

Specifies the name of the cluster and must be identical on all nodes. A node checks the value when attempting to connect to the cluster. If the names match, the node connects. 

Edit the value in the `my.cnf` in the [galera] section.

```text

[galera]

    wsrep_cluster_name=simple-cluster
```

Execute `SHOW VARIABLES` with the LIKE operator to view the variable:

```sql

mysql> SHOW VARIABLES LIKE 'wsrep_cluster_name';
```

??? example "Expected output"

    ```text
    +--------------------+----------------+
    | Variable_name      | Value          |
    +--------------------+----------------+
    | wsrep_cluster_name | simple-cluster |
    +--------------------+----------------+
    ```

!!! note

    It should not exceed 32 characters. A node cannot join the cluster if the cluster names do not match. You must re-bootstrap the cluster after a name change.

### `wsrep_convert_lock_to_trx`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-convert-lock-to-trx`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

This variable has been deprecated in `5.7.23-31.31`. Defines whether locking sessions should be converted into transactions.
By default, this is disabled.

Enabling this variable can help older applications to work
in a multi-source setup by converting `LOCK/UNLOCK TABLES` statements
into `BEGIN/COMMIT` statements.
It is not the same as support for locking sessions,
but it does prevent the database from ending up
in a logically inconsistent state.
Enabling this variable can also result in having huge write-sets.

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

### `wsrep_dbug_option`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-dbug-option`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |

Defines `DBUG` options to pass to the wsrep provider.

### `wsrep_debug`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-debug`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

Enables additional debugging output for the database server error log.
By default, it is disabled.
This variable can be used when trying to diagnose problems
or when submitting a bug.

You can set `wsrep_debug` in the following `my.cnf` groups:

* Under `[mysqld]` it enables debug logging for `mysqld` and the SST script

* Under `[sst]` it enables debug logging for the SST script only

!!! note

    Do not enable debugging in production environments, because it logs authentication info (that is, passwords).

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

### `wsrep_drupal_282555_workaround`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-drupal-282555-workaround`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

This variable has been announced as deprecated in `5.7.24-31.33`. Enables a workaround for MySQL InnoDB bug that affects Drupal
([Drupal bug #282555](https://drupal.org/node/282555)
and [MySQL bug #41984](https://bugs.mysql.com/bug.php?id=41984)).
In some cases, duplicate key errors would occur
when inserting the `DEFAULT` value into an `AUTO_INCREMENT` column.

### `wsrep_forced_binlog_format`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-forced-binlog-format`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | ``NONE`` |

This variable has been announced as deprecated in `5.7.22-29.26`. Defines a binary log format that will always be effective,
regardless of the client session [`binlog_format`](https://dev.mysql.com/doc/refman/5.7/en/binary-log-setting.html) variable value.

Possible values for this variable are:

* `ROW`: Force row-based logging format

* `STATEMENT`: Force statement-based logging format

* `MIXED`: Force mixed logging format

* `NONE`: Do not force the binary log format and use whatever is set by the `binlog_format` variable (default)

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
| Dynamic:       | No                 |

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
The following methods are available:

`TOI`:
When the *Total Order Isolation* method is selected,
data definition language (DDL) statements are processed in the same order
with regards to other transactions in each node.
This guarantees data consistency.

In the case of DDL statements,
the cluster will have parts of the database locked
and it will behave like a single server.
In some cases (like big `ALTER TABLE`)
this could have impact on cluster’s performance and availability,
but it could be fine for quick changes that happen almost instantly
(like fast index changes).

When DDL statements are processed under TOI,
the DDL statement will be replicated up front to the cluster.
That is, the cluster will assign global transaction ID
for the DDL statement before DDL processing begins.
Then every node in the cluster has the responsibility
to execute the DDL statement in the given slot
in the sequence of incoming transactions,
and this DDL execution has to happen with high priority.

!!! important

    Under the `TOI` method, when DDL operations are performed, `MDL` is ignored. If `MDL` is important, use the `RSU` method.

`RSU`: 
When the *Rolling Schema Upgrade* method is selected,
DDL statements won’t be replicated across the cluster.
Instead, it’s up to the user to run them on each node separately.

The node applying the changes will desynchronize from the cluster briefly,
while normal work happens on all the other nodes.
When a DDL statement is processed,
the node will apply delayed replication events.

The schema changes must be backwards compatible for this method to work,
otherwise, the node that receives the change
will likely break Galera replication.
If replication breaks, SST will be triggered
when the node tries to join again but the change will be undone.

!!! note

    This variable’s behavior is consistent with MySQL behavior for variables that have both global and session scope. This means if you want to change the variable in current session, you need to do it with `SET wsrep_OSU_method` (without the `GLOBAL` keyword). Setting the variable with `SET GLOBAL wsrep_OSU_method` will change the variable globally but it won’t have effect on the current session.

### `wsrep_preordered`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-preordered`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

This variable has been announced as deprecated in `5.7.24-31.33`. Defines whether the node should use transparent handling
of preordered replication events (like replication from traditional source).
By default, this is disabled.

If you enable this variable, such events will be applied locally first
before being replicated to other nodes in the cluster.
This could increase the rate at which they can be processed,
which would be otherwise limited by the latency
between the nodes in the cluster.

Preordered events should not interfere with events that originate on the local
node. Therefore, you should not run local update queries on a table that is
also being updated through asynchronous replication.

### `wsrep_provider`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-provider`` |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |

Specifies the path to the Galera library.
This is usually
`/usr/lib64/libgalera_smm.so` on *CentOS*/*RHEL* and
`/usr/lib/libgalera_smm.so` on *Debian*/*Ubuntu*.

If you do not specify a path or the value is not valid,
the node will behave as standalone instance of MySQL.

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

* `ALL`: Reject all new queries from clients,
but maintain existing client connections

* `ALL_KILL`: Reject all new queries from clients
and kill existing client connections

!!! note

    This variable doesn’t affect Galera replication in any way,
    only the applications that connect to the database are affected.
    If you want to desync a node, use [`wsrep_desync`](wsrep-system-index.md#wsrep_desync).

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

     * Difference in configuration of `pxc-cluster` node
     on [enforce_storage_engine](https://www.percona.com/doc/percona-server/5.7/management/enforce_engine.html)
     front may result in picking up different engine for the same table
     on different nodes

     * `CREATE TABLE AS SELECT` (CTAS) statements use non-TOI replication
     and are replicated only if there is involvement of InnoDB table
     that needs transactions
     (in case of MyISAM table, CTAS statements will not be replicated).

### `wsrep_restart_slave`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-restart-slave`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

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

Review the [Galera Cluster documentation for flow control](https://galeracluster.com/library/documentation/node-states.html) for suggested settings.

You can also estimate the optimal value for this from [`wsrep_cert_deps_distance`](wsrep-status-index.md#wsrep_cert_deps_distance) as suggested [in the Galera Cluster documentation](https://galeracluster.com/library/training/tutorials/galera-monitoring.html).

For more configuration tips, see [this document](https://galeracluster.com/library/kb/parallel-applier-threads.html).

### `wsrep_slave_UK_checks`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-slave-UK-checks`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | ``OFF`` |

Defines whether unique key checking is done for applier threads.
This is disabled by default.

### `wsrep_sst_auth`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | <username>:<password> |

Specifies authentication information for State Snapshot Transfer (SST).
Required information depends on the method
specified in the [`wsrep_sst_method`](wsrep-status-index.md##wsrep_sst_method) variable.

For more information about SST authentication,
see [State Snapshot Transfer](manual/state_snapshot_transfer.md#state-snapshot-transfer).

!!! note

    Value of this variable is masked in the log and in the `SHOW VARIABLES` query output.

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

### `wsrep_sst_donor_rejects_queries`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | ``--wsrep-sst-donor-rejects-queries`` |
| Config File:   | Yes                |
| Scope:         | Global            |
| Dynamic:       | Yes                 |
| Default Value: | OFF |

Defines whether the node should reject blocking client sessions
when it is serving as a donor during a blocking state transfer method
(when [`wsrep_sst_method`](wsrep-system-index.md#wsrep_sst_method) is set to `mysqldump` or `rsync`).
This is disabled by default, meaning that the node accepts such queries.

If you enable this variable, queries will return the `Unknown command` error.
This can be used to signal load-balancer that the node isn’t available.

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
can be found [in Percona XtraBackup documentation](https://www.percona.com/doc/percona-xtrabackup/2.4/using_xtrabackup/privileges.html). This is the **recommended** and default method for Percona XtraDB Cluster.
For more information, see [Percona XtraBackup SST Configuration](manual/xtrabackup_sst.md#xtrabackup-sst).

* `rsync`: Uses `rsync` to perform SST.
This method doesn’t use the [`wsrep_sst_auth`](wsrep-system-index.md#wsrep_sst_auth) variable.

* `mysqldump`: Uses `mysqldump` to perform SST
This method requires superuser credentials for the donor node
to be specified in the [`wsrep_sst_auth`](wsrep-system-index.md#wsrep_sst_auth) variable.

!!! note

    This method is deprecated as of [`5.7.22-29.26`](release-notes/Percona-XtraDB-Cluster-5.7.22-29.26.md) and not recommended unless it is required for specific reasons.
    Also, it is not compatible with `bind_address` set to `127.0.0.1`
    or `localhost`, and will cause startup to fail in this case.

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

    Only `xtrabackup-v2` and `rsync` provide support
    for clusters with GTIDs and async replicas.

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

### `wsrep_sync_wait`
| Option         | Description               |
| -------------- | ------------------------- |
| Command Line:  | ``--wsrep-sync-wait``     |
| Config File:   | Yes                       |
| Scope:         | Session, Global           |
| Dynamic:       | Yes                       |
| Default Value: | ``0``                     |

This variable has been implemented in `5.6.20-25.7`. Controls cluster-wide causality checks on certain statements.
Checks ensure that the statement is executed on a node
that is fully synced with the cluster.

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




