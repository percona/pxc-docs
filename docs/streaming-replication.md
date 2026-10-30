# Streaming replication for large transactions

Streaming replication optimizes replication of large or long-running transactions in Percona XtraDB Cluster (PXC). In standard replication, the originating node executes the full transaction before `COMMIT`. The originating node replicates the complete [write-set](glossary.md#write-set) to other nodes at `COMMIT`. Standard replication works for most workloads. A very large or lengthy transaction can overwhelm standard replication.

The originating node divides the transaction into smaller fragments. Galera [certifies](glossary.md#certification) and replicates each fragment while the transaction stays open. A certified fragment locks the fragment rows across the cluster. A later fragment in the same transaction can still fail certification. Streaming replication supports a transaction total larger than 2 GB. Each fragment must stay within the global write-set limits.

!!! note

    Streaming replication is available in Galera Cluster 4.0 and later. PXC uses Galera 4 (`libgalera_smm.so` in the `galera4` directory).

## Prerequisites

The following PXC concepts apply to streaming replication:

* [Write-set](glossary.md#write-set): the replicated payload that describes row changes in a transaction or fragment.

* [Certification](glossary.md#certification): the conflict check that runs before Galera applies a write-set. See [Certification in Percona XtraDB Cluster](certification.md).

* [Flow control](glossary.md#flow-control): the mechanism that throttles replication when a node lags. See [Index of wsrep status variables](wsrep-status-index.md#flow-control-variables).

* Write-set cache: Galera buffers transaction data before commit. See [Understand GCache and Record-Set cache](gcache-record-set-cache-difference.md).

## When should you use streaming replication?

Standard replication is sufficient for most workloads. Streaming replication targets specific scenarios. Enable streaming replication at the session level for a transaction that requires fragmentation.

### When do large data transactions need streaming replication?

A large data transaction is the primary use case. A large `UPDATE`, `DELETE`, or `INSERT` forces the originating node to hold the full transaction locally. The node sends one large write-set at commit time. A large write-set can cause the following problems:

* Replication lag grows while the cluster transfers and applies the large write-set.

* [Flow control](glossary.md#flow-control) on peer nodes. A peer node that applies a large transaction cannot commit other transactions. A busy peer node can trigger [flow control](wsrep-status-index.md#wsrep_flow_control_interval) and throttle the cluster.

Streaming replication sends fragments for the full transaction. Fragmentation spreads network load across the transaction. A peer node can apply other transactions between fragments. The cluster impact stays lower than one large write-set at commit.

### When do long-running transactions need streaming replication?

A long-running transaction conflicts more often with smaller transactions that commit first. Galera aborts the long-running transaction when a conflict occurs at commit time.

Streaming replication certifies the transaction in fragments. A later transaction cannot abort a certified fragment. A later fragment in the same streaming transaction can still fail certification. The later fragment must pass certification against the current cluster state. The state includes transactions that committed after earlier fragments.

!!! warning

    Certification keys use record locks. Certification keys do not use gap locks. A streaming transaction that holds a gap lock does not block a write-set in the same gap. Another node can apply a write-set in the gap and abort the streaming transaction.

### When do high-contention records need streaming replication?

Some applications update the same row many times. Examples include counters, job queues, and lock schemes. Streaming replication can replicate a critical update before the transaction continues. The update locks the contended record on all nodes. Other transactions cannot modify the record until certification completes. A critical transaction then commits with a higher success rate.

## How does fragment certification affect rollbacks?

Certified fragments and streaming transaction rollbacks operate at different levels.

Galera does not abort a certified fragment after a later transaction conflicts on the fragment rows. The fragment stays applied on every node.

The full streaming transaction stays open until `COMMIT`. A later fragment can fail certification when another transaction commits a conflict first. Galera aborts the full streaming transaction. Every node runs a rollback of all fragments from the aborted streaming transaction. The rollback includes fragments that already passed certification.

Frequent rollbacks of a streaming transaction raise resource use on every node. Prefer a short transaction. Size each fragment to lower the conflict risk at certification.

## How do global write-set limits apply to fragments?

[`wsrep_max_ws_size`](wsrep-system-index.md#wsrep_max_ws_size) and [`wsrep_max_ws_rows`](wsrep-system-index.md#wsrep_max_ws_rows) apply to each fragment write-set. The limits do not apply to the full streaming transaction total.

Galera checks each fragment at replication and at certification. Each fragment must pass `wsrep_max_ws_size` and `wsrep_max_ws_rows` on its own. A 5 GB transaction can succeed with streaming replication. Every fragment must stay under the configured limits.

Leave `wsrep_max_ws_size` and `wsrep_max_ws_rows` unchanged for a large streaming transaction. Reduce the fragment size until each fragment meets the current limits. A higher global limit affects every transaction on the cluster. The change applies beyond the large job.

When `wsrep_trx_fragment_unit` is `bytes`, compare the fragment threshold with [`wsrep_max_ws_size`](wsrep-system-index.md#wsrep_max_ws_size). When the unit is `rows`, compare the fragment threshold with [`wsrep_max_ws_rows`](wsrep-system-index.md#wsrep_max_ws_rows).

## How do you enable streaming replication?

Enable streaming replication at the session level for a transaction that needs fragmentation. Two session variables control fragmentation:

* [`wsrep_trx_fragment_unit`](wsrep-system-index.md#wsrep_trx_fragment_unit) defines the replication unit.

* [`wsrep_trx_fragment_size`](wsrep-system-index.md#wsrep_trx_fragment_size) defines how many units form one fragment.

Set both variables to enable streaming replication:

```sql
SET SESSION wsrep_trx_fragment_unit = 'statements';
SET SESSION wsrep_trx_fragment_size = 10;
SELECT @@wsrep_trx_fragment_unit, @@wsrep_trx_fragment_size;
```

The following output confirms that streaming replication is enabled for the session:

??? example "Expected output"

    ```{.text .no-copy}
    Query OK, 0 rows affected (0.00 sec)

    Query OK, 0 rows affected (0.00 sec)

    +--------------------------+---------------------------+
    | @@wsrep_trx_fragment_unit | @@wsrep_trx_fragment_size |
    +--------------------------+---------------------------+
    | statements               | 10                        |
    +--------------------------+---------------------------+
    1 row in set (0.00 sec)
    ```

Galera creates, certifies, and replicates one fragment after every 10 SQL statements in the transaction.

### Configuration variables

The following table lists the streaming replication variables:

| Variable Name | Default Value | Scope | Dynamic | Valid Values |
| ------------- | ------------- | ----- | ------- | ------------ |
| `wsrep_trx_fragment_unit` | `bytes` | Global, Session | Yes | `bytes`, `rows`, `statements` |
| `wsrep_trx_fragment_size` | `0` | Global, Session | Yes | `0` to `2147483647` (`0` disables streaming replication) |
| `wsrep_SR_store` | `table` | Global | No | `table`, `none` |

A value of `0` for `wsrep_trx_fragment_size` disables streaming replication.

### Fragment unit values

`wsrep_trx_fragment_unit` accepts the following values:

| Value | Description |
| ----- | ----------- |
| `bytes` | Galera measures fragment size from the transaction binlog events buffer. Galera counts the internal write-set payload. Galera does not count SQL text or raw InnoDB page size. |
| `rows` | Galera measures fragment size in affected rows. |
| `statements` | Galera measures fragment size in executed SQL statements. |

When `wsrep_trx_fragment_unit` is `bytes`, set `wsrep_trx_fragment_size` from the write-set size. Do not set the value from the SQL statement length. A 1 MB SQL statement can produce a much larger write-set.

#### How are bytes counted?

Galera measures the transaction binlog events buffer for the current streaming transaction. The counter includes the internal write-set payload Galera builds for certification. The counter excludes SQL text length. The counter excludes result set size. The counter excludes raw InnoDB page size on disk.

Galera creates a fragment when the buffer reaches `wsrep_trx_fragment_size` bytes after the previous fragment.

#### What is a safe byte threshold?

Use the following initial values. Tune the values from production results:

| Option | Guidance |
| ------ | -------- |
| Minimum practical size | `262144` (256 KB) or higher. A size under 256 KB often creates too many fragments. |
| Common initial size | `1048576` (1 MB). Percona tests show that a 1 MB fragment balances cluster recovery and cost on the originating node. |
| Upper bound | Each fragment must stay under [`wsrep_max_ws_size`](wsrep-system-index.md#wsrep_max_ws_size) (default 2 GB). |
| Size signal | A high `mysql.wsrep_streaming_log` row count with little row progress means the fragments are too small. |

Compare byte thresholds with `wsrep_max_ws_size`. Do not compare byte thresholds with SQL length or application payload estimates. Tune from `mysql.wsrep_streaming_log` growth and [`wsrep_replicated_bytes`](wsrep-status-index.md#wsrep_replicated_bytes) rates on test jobs.

Set [`wsrep_trx_fragment_size`](wsrep-system-index.md#wsrep_trx_fragment_size) to `0` to disable streaming replication for the session:

```sql
SET SESSION wsrep_trx_fragment_size = 0;
SELECT @@wsrep_trx_fragment_size;
```

The following output confirms that streaming replication is disabled for the session:

??? example "Expected output"

    ```{.text .no-copy}
    Query OK, 0 rows affected (0.00 sec)

    +---------------------------+
    | @@wsrep_trx_fragment_size |
    +---------------------------+
    | 0                         |
    +---------------------------+
    1 row in set (0.00 sec)
    ```

Apply an optimizer [`SET_VAR` hint](wsrep-system-index.md#wsrep_trx_fragment_size) to one data manipulation language (DML) statement in a transaction. A hint helps when one statement dominates the transaction size.

### How do SET_VAR hints scope inside a transaction?

A `SET_VAR` hint applies only to the statement that contains the hint. The hint does not change session variables for later statements in the same transaction.

A session variable from `SET SESSION` applies to later statements. The session value stays until you change the session value. A `SET_VAR` hint overrides the session value only for the hinted statement.

Example for one transaction:

```sql
BEGIN;
SET SESSION wsrep_trx_fragment_size = 10;
INSERT /*+ SET_VAR(wsrep_trx_fragment_size = 500) */ INTO t1 SELECT * FROM t2;
INSERT INTO t1 SELECT * FROM t3;
COMMIT;
```

The first `INSERT` uses a fragment size of 500 units. The second `INSERT` uses the session value of 10.

Verify the session value after the hinted statement:

```sql
SELECT @@wsrep_trx_fragment_size;
```

The following output confirms that the session value is unchanged:

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------------+
    | @@wsrep_trx_fragment_size |
    +---------------------------+
    | 10                        |
    +---------------------------+
    1 row in set (0.00 sec)
    ```

Apply a hint on each statement in one transaction. The [wsrep-system-index example](wsrep-system-index.md#wsrep_trx_fragment_size) uses `SET_VAR(wsrep_trx_fragment_size = 100)` on separate `INSERT`, `UPDATE`, and `DELETE` statements. Each statement gets a separate fragment threshold. Other statements in the transaction keep the session default.

```sql
UPDATE /*+ SET_VAR(wsrep_trx_fragment_size = 100) */ work_orders SET status = 'active';
```

The following output confirms that the optimizer hint applied the fragment size for the statement:

??? example "Expected output"

    ```{.text .no-copy}
    Query OK, 150 rows affected (2.34 sec)
    Rows matched: 150  Changed: 150  Warnings: 0
    ```

## How do you monitor active streaming replication?

Galera does not expose a `wsrep_streaming_*` status variable. Galera does not expose a cumulative fragment counter. Use `mysql.wsrep_streaming_log` to view fragments on an active transaction. Use `wsrep_*` status variables for cluster pressure on a streaming job.

### Which wsrep status variables should you watch?

The following table lists variables for dashboards and alerts on a streaming replication workload:

| Variable | Meaning | Alert when |
| -------- | ----------------- | ---------- |
| `mysql.wsrep_streaming_log` row count | Active certified fragments on this node (query with `SELECT COUNT(*)`) | Count stays greater than zero outside a known maintenance window |
| [`wsrep_local_recv_queue`](wsrep-status-index.md#wsrep_local_recv_queue) | Applier backlog on the local node | Sustained high values on a streaming job |
| [`wsrep_local_recv_queue_avg`](wsrep-status-index.md#wsrep_local_recv_queue_avg) | Smoothed applier backlog | Sustained increase over the baseline |
| [`wsrep_flow_control_sent`](wsrep-status-index.md#wsrep_flow_control_sent) | Flow control pauses sent by this node | Rate spikes on a large streaming job |
| [`wsrep_flow_control_recv`](wsrep-status-index.md#wsrep_flow_control_recv) | Flow control pauses received by this node | Rate spikes on a large streaming job |
| [`wsrep_flow_control_paused`](wsrep-status-index.md#wsrep_flow_control_paused) | Time spent paused by flow control | Sustained growth greater than zero |
| [`wsrep_replicated`](wsrep-status-index.md#wsrep_replicated) | Write-sets this node sent to the cluster, and fragment write-sets | Unusual rate increase on the originating node |
| [`wsrep_received`](wsrep-status-index.md#wsrep_received) | Write-sets this node received, and fragment write-sets | Unusual rate increase on a peer node |
| [`wsrep_replicated_bytes`](wsrep-status-index.md#wsrep_replicated_bytes) | Bytes this node sent in write-sets | Sudden sustained increase on a streaming job |
| [`wsrep_received_bytes`](wsrep-status-index.md#wsrep_received_bytes) | Bytes this node received in write-sets | Sudden sustained increase on a streaming job |
| [`wsrep_local_cert_failures`](wsrep-status-index.md#wsrep_local_cert_failures) | Certification failures and streaming aborts | Any sustained increase |
| [`wsrep_local_bf_aborts`](wsrep-status-index.md#wsrep_local_bf_aborts) | Local transactions aborted by a replicated transaction on the same rows | Increase on a hot-row streaming job |

Galera counts each certified fragment as a separate write-set in `wsrep_replicated` and `wsrep_received`. A large streaming transaction increases `wsrep_replicated` and `wsrep_received` as each fragment certifies. The increase is not limited to the final `COMMIT`.

Query the variables on every node:

```sql
SHOW GLOBAL STATUS WHERE Variable_name IN (
  'wsrep_local_recv_queue',
  'wsrep_local_recv_queue_avg',
  'wsrep_flow_control_sent',
  'wsrep_flow_control_recv',
  'wsrep_flow_control_paused',
  'wsrep_replicated',
  'wsrep_received',
  'wsrep_replicated_bytes',
  'wsrep_received_bytes',
  'wsrep_local_cert_failures',
  'wsrep_local_bf_aborts'
);
```

Combine the status query with `mysql.wsrep_streaming_log` on each node. See [Monitor the cluster](monitoring.md) for general PXC alert guidance.

### Check active fragments in mysql.wsrep_streaming_log

Each row in `mysql.wsrep_streaming_log` represents one certified fragment from an open streaming transaction. Query metadata columns only. The `frag` column contains full binary log events. The column can exceed [`max_allowed_packet`](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_max_allowed_packet).

Count active fragments on a node:

```sql
SELECT COUNT(*) AS active_fragments FROM mysql.wsrep_streaming_log;
```

The following output shows no active streaming transactions:

??? example "Expected output"

    ```{.text .no-copy}
    +------------------+
    | active_fragments |
    +------------------+
    |                0 |
    +------------------+
    1 row in set (0.00 sec)
    ```

List active streaming transactions on a large job:

```sql
SELECT node_uuid, trx_id, seqno, flags
FROM mysql.wsrep_streaming_log;
```

The following output shows the latest certified fragment (`seqno = 4`) for one open transaction:

??? example "Expected output"

    ```{.text .no-copy}
    +--------------------------------------+--------+-------+-------+
    | node_uuid                            | trx_id | seqno | flags |
    +--------------------------------------+--------+-------+-------+
    | a006244a-7ed8-11e9-bf00-867215999c7c |     26 |     4 |     1 |
    +--------------------------------------+--------+-------+-------+
    1 row in set (0.00 sec)
    ```

The `seqno` value increases as Galera certifies more fragments for the same `trx_id`. The `node_uuid` column identifies the originating node for the streaming transaction.

### Check related cluster status variables

The variables in the monitoring table cover cluster pressure on a streaming job. Query all wsrep status variables for wider context:

```sql
SHOW GLOBAL STATUS LIKE 'wsrep_%';
```

For wider cluster monitoring guidance, see [Monitor the cluster](monitoring.md).

## What does wsrep_SR_store control?

[`wsrep_SR_store`](wsrep-system-index.md#wsrep_sr_store) defines where Galera stores streaming replication fragments. The variable is global. You cannot change the variable at runtime. Change the value only at server startup.

### When wsrep_SR_store is table

The default value is `table`. Galera writes each fragment to `mysql.wsrep_streaming_log` on every node. The table stores a partial streaming transaction on disk.

Disk writes add overhead for an active streaming transaction. The disk writes support crash recovery. After a node crash, Galera can use the log to recover an open streaming transaction. Galera can also run a rollback of an open streaming transaction from the log.

### When wsrep_SR_store is none

The value `none` disables storage in `mysql.wsrep_streaming_log`. Galera still replicates and certifies fragments in memory. Galera does not write fragments to the log table.

The `none` value reduces log-table I/O. The `none` value removes durable recovery for a partial streaming transaction. After a crash, Galera cannot rebuild open streaming fragments from the log. Recovery can require manual action or a transaction rollback.

The `none` value does not remove memory use. Each node still buffers fragment write-sets for replication and certification. A very small fragment size raises the fragment count. A higher fragment count can raise memory pressure on the cluster.

Use `wsrep_SR_store = 'none'` only when you accept the crash-recovery cost. Test fragment size under peak load before you use `none`. Keep the default `table` value for a production workload. Change the value when a profile shows a clear benefit.

### What is the lifecycle of mysql.wsrep_streaming_log?

Galera manages `mysql.wsrep_streaming_log`. Do not run `DELETE` on the table. Do not run `TRUNCATE` on the table.

The table holds rows only while a streaming transaction stays open. The transaction must have certified fragments. Each row stores metadata for one fragment. When `wsrep_SR_store` is `table`, the `frag` column stores the fragment payload.

Galera removes fragment rows from every node after the transaction ends:

* `COMMIT`: Galera removes all fragment rows after the final fragment certifies.

* `ROLLBACK`: Galera removes fragment rows. Galera runs a rollback of applied fragment data on every node.

The table is empty when no streaming transaction is active on the cluster. A `COUNT(*)` greater than zero in an idle period shows an open streaming transaction on the cluster.

An abandoned client session can leave fragment rows until the server ends the transaction. The server runs a rollback when the client disconnects. The server also runs a rollback when [`wait_timeout`](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_wait_timeout) closes the idle connection. The server also runs a rollback when [`interactive_timeout`](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_interactive_timeout) closes the idle connection. Fragment rows and `frag` data stay on disk until the rollback completes.

A long-running streaming transaction with an open connection can grow `mysql.wsrep_streaming_log` on every node. Monitor fragment counts on a large job. Size fragments to balance replication progress and log-table growth.

### How do you force-kill a runaway streaming transaction?

Do not delete rows from `mysql.wsrep_streaming_log`. End the open transaction on the originating node. A connection-level `KILL` runs a rollback of the streaming transaction through Galera on every node. The `KILL` does not cause a cluster split-brain event. A network partition and quorum loss cause split-brain. Termination of one client session does not cause split-brain.

Use the following steps when log-table growth puts disk space at risk:
{.power-number}

1. Confirm active streaming fragments on each node:

    ```sql
    SELECT COUNT(*) AS active_fragments FROM mysql.wsrep_streaming_log;
    ```

2. Identify the originating node and session. Query fragment metadata to find the originating node:

    ```sql
    SELECT DISTINCT node_uuid, trx_id FROM mysql.wsrep_streaming_log;
    ```

    The `node_uuid` value identifies the cluster node where the streaming transaction started. Query [`mysql.wsrep_cluster_members`](https://mariadb.com/docs/galera-cluster/reference/galera-cluster-system-tables) on any node to map the UUID to a host name or address. Run `SHOW FULL PROCESSLIST` on the originating node:

    ```sql
    SHOW FULL PROCESSLIST;
    ```

    Look for a long DML statement or an open transaction in the `State` column. Note the connection `Id`. The streaming log does not expose the connection `Id`.

3. End the connection on the node where the session runs:

    ```sql
    KILL <connection_id>;
    ```

    Replace `<connection_id>` with the `Id` from `SHOW FULL PROCESSLIST`. `KILL` is shorthand for `KILL CONNECTION`.

4. Allow the Galera rollback to finish on all nodes. Rollback duration depends on the number of certified fragments.

5. Verify cleanup on every node:

    ```sql
    SELECT COUNT(*) AS active_fragments FROM mysql.wsrep_streaming_log;
    ```

    The following output confirms that no fragments remain:

    ??? example "Expected output"

        ```{.text .no-copy}
        +------------------+
        | active_fragments |
        +------------------+
        |                0 |
        +------------------+
        1 row in set (0.00 sec)
        ```

6. Confirm cluster health:

    ```sql
    SHOW GLOBAL STATUS LIKE 'wsrep_cluster_status';
    SHOW GLOBAL STATUS LIKE 'wsrep_local_state_comment';
    ```

    Expect `Primary` cluster status and `Synced` local state on a healthy node.

`KILL QUERY <connection_id>` ends only the current statement. Use `KILL CONNECTION` when the session is idle in an open transaction.

Do not rely on [`wait_timeout`](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_wait_timeout) or [`interactive_timeout`](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_interactive_timeout) when disk use is critical. A forced disconnect starts a rollback and log cleanup on every node.

## Do tables need a primary key for streaming replication?

Streaming replication uses the same [certification](glossary.md#certification) and write-set path as standard Galera replication. A table in a streaming transaction must have a primary key.

Galera uses primary key values to identify rows for certification. Galera uses primary key values for a cluster-wide lock. Galera cannot order individual rows across nodes without a primary key. Galera cannot lock individual rows for fragment certification without a primary key.

Under [`pxc_strict_mode=ENFORCING`](strict-mode.md#primary-key-requirement), PXC denies `DELETE` and other undesirable writes on a table without an explicit primary key. Streaming replication does not relax the primary key check. A streaming `DELETE` on a table without a primary key fails when strict mode is `ENFORCING`.

Define a primary key on every table in the cluster. See [Percona XtraDB Cluster limitations](limitation.md) and [Primary key requirement](strict-mode.md#primary-key-requirement).

## How does streaming replication interact with pxc_strict_mode?

Streaming replication does not require a dedicated [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) value. The default value `ENFORCING` is supported. Keep the default value `ENFORCING`.

Streaming replication is a supported Galera 4 feature. [PXC Strict Mode](strict-mode.md) does not treat streaming replication as a tech preview feature.

A strict mode check still applies in a streaming transaction. Strict mode blocks the same operations in a streaming transaction and in a standard transaction. Examples include an unsupported storage engine. Examples include a plugin that strict mode blocks. See [PXC Strict Mode validations](strict-mode.md#validations).

Keep `pxc_strict_mode=ENFORCING` on all cluster nodes. Streaming replication does not bypass strict mode checks on peer nodes.

## How does streaming replication affect binary logging and async replicas?

Streaming replication affects Galera cluster nodes and downstream async replicas in different ways.

### Galera cluster replication (between PXC nodes)

Galera replicates certified fragments to every cluster node while the transaction stays open. Fragments travel through the Galera write-set path. Fragments do not appear in the MySQL binary log while the transaction stays open. Galera stores partial fragment data in `mysql.wsrep_streaming_log` when [`wsrep_SR_store`](wsrep-system-index.md#wsrep_sr_store) is `table`.

Other cluster nodes apply certified fragments before the final `COMMIT`. Applied fragments hold cluster-wide row locks. The fragments stay part of an open transaction on every peer node. Under a normal isolation level, other sessions cannot read the uncommitted fragment data. A session that modifies the affected rows waits on the fragment locks. The wait ends when the originating session runs `COMMIT` or `ROLLBACK`. Only the full transaction boundary is atomic for the application.

### Binary log on the originating PXC node

The MySQL binary log records the client transaction when the originating session issues `COMMIT`. Galera does not write streaming fragments to `log_bin` as separate binlog events while the transaction stays open.

A streaming transaction produces one binlog transaction entry at commit time on the client node. The binlog event size matches the full committed change set. The binlog event size does not match individual fragment sizes.

Galera builds row-format write-sets in the server. Set `binlog_format=ROW` on a node that serves as an async replication source. Set `binlog_format=ROW` on a node that requires row-based binary logging.

### Downstream asynchronous MySQL replicas

A downstream async replica does not receive streaming fragments before commit. The replica SQL thread applies the transaction when the source node commits. The source node writes the binlog event at commit.

A large streaming transaction can still cause a sudden downstream replication lag spike at commit time. The async replica receives one large binlog transaction. PXC nodes apply fragments in steps while the transaction stays open.

Configure the PXC source node for async replication as described in [Does Percona XtraDB Cluster work with regular MySQL replication?](faq.md#does-percona-xtradb-cluster-work-with-regular-mysql-replication):

* Set a unique non-zero `server_id` on each PXC node.

* Enable `log_bin` on the node that acts as the replication source. The source node writes local client changes to the binary log at commit time.

* Enable [`log_replica_updates`](https://dev.mysql.com/doc/refman/{{vers}}/en/replication-options-replica.html#sysvar_log_replica_updates) when the source node must relay writes from other PXC nodes. `log_slave_updates` is the deprecated name for `log_replica_updates`. Galera delivers the relayed writes to the source node. Without `log_replica_updates`, only local writes appear in the source binary log.

Streaming replication reduces lag and flow control between PXC nodes while the transaction stays open. Streaming replication does not spread downstream async apply load across the transaction. Plan async replica capacity for the peak binlog volume at commit time.

## How do you manage a hot record?

A work-order queue must assign each user a unique queue position. Streaming replication can protect the critical update that increments the queue position.

### Why not use SELECT ... FOR UPDATE?

`SELECT ... FOR UPDATE` locks rows on the local node only. Galera uses optimistic replication by default. Other nodes do not see the local locks until the transaction commits. Other nodes do not see the local locks until the write-set replicates. A concurrent transaction on another node can modify the same row and win certification first.

Streaming replication certifies and applies the critical fragment on all nodes before the transaction continues. The fragment creates cluster-wide locks for the rows in the fragment. Other nodes block a change to the locked rows at certification. The queue update reaches every node before other work in the same transaction runs.

`SELECT ... FOR UPDATE` still helps local read consistency on one node. Streaming replication addresses cluster-wide order under the Galera certification model.

Configure streaming replication for the queue update with the following steps:
{.power-number}

1. Begin the transaction:

    ```sql
    START TRANSACTION;
    ```

    The following output confirms that the transaction has started:

    ??? example "Expected output"

        ```{.text .no-copy}
        Query OK, 0 rows affected (0.00 sec)
        ```

2. Read the work-order row that you are about to update:

    ```sql
    SELECT id, queue_position
      FROM work_orders
     WHERE id = 1001;
    ```

    The following output shows the current queue position:

    ??? example "Expected output"

        ```{.text .no-copy}
        +------+----------------+
        | id   | queue_position |
        +------+----------------+
        | 1001 |              5 |
        +------+----------------+
        1 row in set (0.00 sec)
        ```

3. Enable streaming replication for the next statement:

    ```sql
    SET SESSION wsrep_trx_fragment_unit = 'statements';
    SET SESSION wsrep_trx_fragment_size = 1;
    ```

    The following output confirms that streaming replication is enabled for the next statement:

    ??? example "Expected output"

        ```{.text .no-copy}
        Query OK, 0 rows affected (0.00 sec)

        Query OK, 0 rows affected (0.00 sec)
        ```

4. Run the critical update. Galera fragments and replicates the statement before the next statement:

    ```sql
    UPDATE work_orders
       SET queue_position = queue_position + 1
     WHERE id = 1001;
    ```

    The following output confirms that the queue position was updated:

    ??? example "Expected output"

        ```{.text .no-copy}
        Query OK, 1 row affected (0.01 sec)
        Rows matched: 1  Changed: 1  Warnings: 0
        ```

5. Disable streaming replication for the rest of the transaction:

    ```sql
    SET SESSION wsrep_trx_fragment_size = 0;
    ```

    The following output confirms that streaming replication is disabled:

    ??? example "Expected output"

        ```{.text .no-copy}
        Query OK, 0 rows affected (0.00 sec)
        ```

6. Run any remaining statements for the work order. Streaming replication is now off, so Galera replicates these statements at commit, as the transaction normally does. 

7. Commit the transaction as soon as the remaining work is done. The fragments replicated in step 4 keep the hot record protected until the transaction ends, so a long-running transaction blocks other writers:


    ```sql
    COMMIT;
    ```

    The following output confirms that the transaction committed:

    ??? example "Expected output"

        ```{.text .no-copy}
        Query OK, 0 rows affected (0.01 sec)
        ```

The `queue_position` update replicates and certifies on the cluster before the transaction continues. A race on queue assignment is less likely.

## What happens when the initiating node crashes mid-transaction?

A streaming transaction stays incomplete until the originating session issues `COMMIT`. Certified fragments on a node that remains in the cluster are not committed application data. The fragments stay uncommitted until the full transaction completes.

### When the originating node fails

The open transaction is lost when the originating node crashes. The open transaction is lost when the cluster evicts the originating node. A node that remains in the cluster keeps the certified fragments. The node keeps fragments that the node already applied.

Galera treats the incomplete streaming transaction as aborted. The rollbacker thread runs a rollback of applied fragment data on every other node. Galera also removes the related rows from `mysql.wsrep_streaming_log` on the other nodes. Other nodes do not keep partial streaming changes as committed data.

Galera has no separate administrator timeout for an orphaned streaming fragment on a healthy node. Cleanup runs in the Galera abort process when the originating node leaves the cluster. Cleanup also runs when the transaction fails.

### When the crashed node rejoins

The node that returns was behind the cluster state at the failure. The node usually receives an [IST](glossary.md#ist) or [SST](glossary.md#sst) from a donor. See [Crash recovery](crash-recovery.md).

When `wsrep_SR_store` is `table`, Galera can use `mysql.wsrep_streaming_log` on the restarted node. Galera can recover the partial streaming state of the restarted node after a crash. Galera can run a rollback of the partial streaming state after a crash. When `wsrep_SR_store` is `none`, the local recovery path is not available.

### When the entire cluster stops during streaming

A power loss or a full cluster outage can leave a node with `seqno: -1` in `grastate.dat`. Follow [Crash recovery](crash-recovery.md) to identify the most advanced node and bootstrap the cluster. Galera resolves streaming log rows from an incomplete transaction at startup. Galera resolves the rows in transaction recovery on each node.

## What are the limitations?

Review the following limitations before you enable streaming replication.

### What is the performance overhead?

Galera records write-sets in `mysql.wsrep_streaming_log` on every node when streaming replication is enabled. The log write also requires [`wsrep_SR_store`](wsrep-system-index.md#wsrep_sr_store) set to `table`. The log provides crash recovery for a partial transaction. Log writes add overhead. Use streaming replication only when a transaction requires fragmentation.

### What is the rollback cost?

Galera runs a rollback of every certified fragment on all nodes when a streaming transaction aborts after partial certification. A failure in any fragment aborts the full streaming transaction. Frequent rollbacks of a streaming transaction raise resource use on every node.

Prefer a short transaction in the application. For a large transaction, use streaming replication and size the fragments to meet the current limits. Leave [`wsrep_max_ws_size`](wsrep-system-index.md#wsrep_max_ws_size) and [`wsrep_max_ws_rows`](wsrep-system-index.md#wsrep_max_ws_rows) unchanged.

### What happens with SAVEPOINTs inside a streaming transaction?

Avoid `SAVEPOINT`, `ROLLBACK TO SAVEPOINT`, and `RELEASE SAVEPOINT` in a streaming transaction. Galera does not support a partial savepoint rollback across the cluster.

!!! warning

    Galera does not send a compensation write-set for a savepoint rollback. After Galera certifies fragments on peer nodes, `ROLLBACK TO SAVEPOINT` becomes a full transaction rollback on every node.

#### Explicit savepoints

An explicit savepoint records a rollback point only on the originating node. Peer nodes do not receive savepoint events. Peer nodes already certified fragments from statements after the savepoint. Peer nodes already applied the later fragments.

Consider a transaction that runs five statements. The transaction streams a fragment after each statement. The session runs `ROLLBACK TO SAVEPOINT sp1` after statement five. Peer nodes still hold certified fragments from statements two through five. Galera cannot undo only the later fragments on peer nodes.

Galera does not send a compensation write-set to reverse a savepoint rollback on peer nodes. A partial savepoint rollback does not produce cluster-wide consistency.

After any statement replicates one or more fragments, Galera treats `ROLLBACK TO SAVEPOINT` as unsafe. Galera turns the savepoint rollback into a full transaction rollback on every node. Galera undoes all certified fragments from the streaming transaction on the cluster. The `mysql.wsrep_streaming_log` table is empty after the rollback completes.

A `SAVEPOINT` can succeed on the originating node before any fragment replicates. A later `ROLLBACK TO SAVEPOINT` after fragment replication still triggers the full cluster-wide rollback.

#### Implicit statement rollbacks

An implicit statement rollback occurs when one statement fails in an open transaction. MySQL reverses the failed statement on the local node. Examples include a duplicate-key error. Examples include a constraint violation on a multi-row `INSERT`.

Galera applies the same safety rule as an explicit savepoint rollback. After the failed statement replicates one or more fragments, Galera cannot reverse only the failed statement on peer nodes. Galera aborts the full streaming transaction. Galera runs a rollback of all certified fragments on every node.

#### Application guidance

Prefer plain `BEGIN`, `COMMIT`, and `ROLLBACK` boundaries for a streaming transaction. Split complex logic into separate transactions. Do not use a nested savepoint. For general cluster limits on transaction types, see [Percona XtraDB Cluster limitations](limitation.md).
