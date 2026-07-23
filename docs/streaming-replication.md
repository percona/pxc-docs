# Streaming replication for large transactions

Streaming replication optimizes replication of large or long-running transactions in Percona XtraDB Cluster (PXC). A node normally executes a transaction fully. The node replicates the complete [write-set](glossary.md#write-set) to other nodes at `COMMIT` time. Standard replication works well for most workloads. Very large or lengthy transactions can overwhelm standard replication.

With streaming replication, the initiating node divides the transaction into smaller fragments. Galera [certifies](glossary.md#certification) and replicates each fragment while the transaction remains open. Certified fragments lock their rows across the cluster. A later fragment in the same transaction can still fail certification. Streaming replication supports transaction totals larger than 2 GB when each fragment stays within global write-set limits.

!!! note

    Streaming replication is available in Galera Cluster 4.0 and later. PXC uses Galera 4 (`libgalera_smm.so` in the `galera4` directory).

## Prerequisites

The following PXC concepts appear throughout this page:

* [Write-set](glossary.md#write-set): the replicated payload that describes row changes in a transaction or fragment.

* [Certification](glossary.md#certification): the conflict-detection step that runs before Galera applies a write-set. See [Certification in Percona XtraDB Cluster](certification.md).

* [Flow control](glossary.md#flow-control): the mechanism that throttles replication when a node falls behind. See [Index of wsrep status variables](wsrep-status-index.md#flow-control-variables).

* Write-set caching: how Galera buffers transaction data before commit. See [Understand GCache and Record-Set cache](gcache-record-set-cache-difference.md).

## When should you use streaming replication?

Standard replication is sufficient for most workloads. Streaming replication targets specific scenarios. Enable streaming replication at the session level for transactions that require fragmentation.

### When do large data transactions need streaming replication?

Large data transactions are the primary use case. A massive `UPDATE`, `DELETE`, or `INSERT` forces the originating node to hold the full transaction locally. The node sends one large write-set at commit time. Large write-sets can cause these problems:

* Significant replication lag, because the cluster waits for transfer and application of the large write-set.

* [Flow control](glossary.md#flow-control) on peer nodes. Cluster nodes that apply a large transaction cannot commit other transactions. Busy peer nodes can trigger [flow control](wsrep-status-index.md#wsrep_flow_control_interval) and throttle the cluster.

Streaming replication sends data in fragments throughout the transaction lifetime. Fragmentation spreads network load across the transaction duration. Peer nodes can apply concurrent transactions between fragments. Cluster performance impact stays lower than with a single large write-set at commit.

### When do long-running transactions need streaming replication?

A long-running transaction conflicts more often with smaller transactions that commit first. Galera aborts the long-running transaction when a conflict occurs at commit time.

Streaming replication certifies the transaction in fragments. A certified fragment cannot abort because of a conflicting transaction on the rows that fragment already modified. A later fragment in the same streaming transaction is not protected. The later fragment must pass certification against the current cluster state, including transactions that committed after earlier fragments.

!!! warning

    Certification keys derive from record locks, not gap locks. A streaming transaction that holds a gap lock does not block conflicting write-sets in that gap. Another node can apply a write-set in the gap and abort the streaming transaction.

### When do high-contention records need streaming replication?

Applications that update the same row repeatedly include counters, job queues, and locking schemes. Streaming replication can force a critical update to replicate immediately. The update locks the contended record on all nodes. Other transactions cannot modify the record until certification completes. Critical transactions commit with a higher success rate.

## How does fragment certification affect rollbacks?

Certified fragments and streaming transaction rollbacks operate at different levels.

A certified fragment is final for the rows that fragment touched. Galera will not abort a certified fragment because a later transaction conflicts with those rows. The fragment remains applied on every node.

The streaming transaction as a whole is not final until `COMMIT`. A later fragment can fail certification when another transaction commits conflicting changes first. Galera then aborts the entire streaming transaction. Every node must roll back all fragments from that transaction, including fragments that already passed certification.

Frequent rollbacks of streaming transactions increase resource use on every node. Prefer shorter transactions when possible. Size fragments so each certification step has a lower conflict risk.

## How do global write-set limits apply to fragments?

[`wsrep_max_ws_size`](wsrep-system-index.md#wsrep_max_ws_size) and [`wsrep_max_ws_rows`](wsrep-system-index.md#wsrep_max_ws_rows) apply to each fragment write-set, not to the full streaming transaction total.

Galera evaluates limits when a fragment is replicated and certified. Each fragment must pass `wsrep_max_ws_size` and `wsrep_max_ws_rows` independently. A 5 GB transaction can succeed with streaming replication when fragmentation keeps every fragment under the configured limits.

You do not need to raise `wsrep_max_ws_size` or `wsrep_max_ws_rows` globally for a large streaming transaction. Instead, reduce fragment size until each fragment complies with the existing limits. Raising the global limits affects every transaction on the cluster, not only the large job.

When `wsrep_trx_fragment_unit` is `bytes`, compare the fragment threshold against `wsrep_max_ws_size`. When the unit is `rows`, compare against `wsrep_max_ws_rows`.

## How do you enable streaming replication?

Enable streaming replication at the session level for transactions that need fragmentation. Two session variables control fragmentation:

* [`wsrep_trx_fragment_unit`](wsrep-system-index.md#wsrep_trx_fragment_unit) defines the unit of replication.

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

The following example creates, certifies, and replicates one fragment after every 10 SQL statements in the transaction.

### Configuration variables

The following table summarizes the streaming replication variables:

| Variable Name | Default Value | Scope | Dynamic | Valid Values |
| ------------- | ------------- | ----- | ------- | ------------ |
| `wsrep_trx_fragment_unit` | `bytes` | Global, Session | Yes | `bytes`, `rows`, `statements` |
| `wsrep_trx_fragment_size` | `0` | Global, Session | Yes | `0` to `2147483647` (`0` disables streaming replication) |
| `wsrep_SR_store` | `table` | Global | No | `table`, `none` |

A value of `0` for `wsrep_trx_fragment_size` disables streaming replication.

### Fragment unit values

`wsrep_trx_fragment_unit` accepts these values:

| Value | Description |
| ----- | ----------- |
| `bytes` | Fragment size is measured in the transaction binlog events buffer size. Galera counts the internal write-set payload, not the SQL text or raw InnoDB page size. |
| `rows` | Fragment size is measured in affected rows. |
| `statements` | Fragment size is measured in executed SQL statements. |

When `wsrep_trx_fragment_unit` is `bytes`, set `wsrep_trx_fragment_size` from write-set size, not from SQL statement length. A 1 MB SQL statement can produce a much larger write-set.

#### How are bytes counted?

Galera measures the transaction binlog events buffer for the current streaming transaction. The counter includes the internal write-set payload Galera builds for certification. The counter excludes SQL text length, result set size, and raw InnoDB page size on disk.

Galera creates a new fragment when the accumulated buffer reaches `wsrep_trx_fragment_size` bytes since the previous fragment.

#### What is a safe byte threshold?

Use these starting points and tune from production observations:

| Setting | Guidance |
| ------- | -------- |
| Minimum practical size | `262144` (256 KB) or higher. Sizes below 256 KB often create fragment thrashing. |
| Common starting size | `1048576` (1 MB). Percona testing found 1 MB fragments balance cluster catch-up and origin-node overhead. |
| Upper bound | Each fragment must stay below [`wsrep_max_ws_size`](wsrep-system-index.md#wsrep_max_ws_size) (default 2 GB). |
| Tuning signal | Rising `mysql.wsrep_streaming_log` row counts with minimal row progress indicate fragments are too small. |

Compare byte thresholds against `wsrep_max_ws_size`, not against SQL or application payload estimates. Tune from observed `mysql.wsrep_streaming_log` growth and [`wsrep_replicated_bytes`](wsrep-status-index.md#wsrep_replicated_bytes) rates during test jobs.

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

Apply [optimizer `SET_VAR` hints](wsrep-system-index.md#wsrep_trx_fragment_size) to individual data manipulation language (DML) statements within a transaction. Hints help when a single statement dominates the transaction size.

### How do SET_VAR hints scope inside a transaction?

A `SET_VAR` hint applies only to the single statement that contains the hint. The hint does not change session variables for later statements in the same transaction.

Session variables set with `SET SESSION` remain in effect for all subsequent statements until you change them again. A `SET_VAR` hint overrides the session value only while the annotated statement runs.

Example within one transaction:

```sql
BEGIN;
SET SESSION wsrep_trx_fragment_size = 10;
INSERT /*+ SET_VAR(wsrep_trx_fragment_size = 500) */ INTO t1 SELECT * FROM t2;
INSERT INTO t1 SELECT * FROM t3;
COMMIT;
```

The first `INSERT` fragments at 500 units. The second `INSERT` uses the session value of 10.

Verify session state after a hinted statement:

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

You can apply hints on separate statements within one transaction. The [wsrep-system-index example](wsrep-system-index.md#wsrep_trx_fragment_size) uses `SET_VAR(wsrep_trx_fragment_size = 100)` on separate `INSERT`, `UPDATE`, and `DELETE` statements. Each statement gets its own fragment threshold. Other statements in the transaction keep the session defaults.

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

Galera does not expose a dedicated `wsrep_streaming_*` status variable or a cumulative fragment counter. Use `mysql.wsrep_streaming_log` for real-time fragment visibility. Use general `wsrep_*` status variables for cluster pressure during streaming jobs.

### Which wsrep status variables should you watch?

The following table lists variables useful for dashboards and alerts during streaming replication workloads:

| Variable | What it indicates | Alert when |
| -------- | ----------------- | ---------- |
| `mysql.wsrep_streaming_log` row count | Active certified fragments on this node (query with `SELECT COUNT(*)`) | Count stays above zero outside known maintenance windows |
| [`wsrep_local_recv_queue`](wsrep-status-index.md#wsrep_local_recv_queue) | Applier backlog on the local node | Sustained high values during streaming jobs |
| [`wsrep_local_recv_queue_avg`](wsrep-status-index.md#wsrep_local_recv_queue_avg) | Smoothed applier backlog | Sustained increase over baseline |
| [`wsrep_flow_control_sent`](wsrep-status-index.md#wsrep_flow_control_sent) | Flow control pauses this node initiated | Rate spikes during large streaming jobs |
| [`wsrep_flow_control_recv`](wsrep-status-index.md#wsrep_flow_control_recv) | Flow control pauses this node received | Rate spikes during large streaming jobs |
| [`wsrep_flow_control_paused`](wsrep-status-index.md#wsrep_flow_control_paused) | Time spent paused by flow control | Sustained non-zero growth |
| [`wsrep_replicated`](wsrep-status-index.md#wsrep_replicated) | Write-sets this node sent to the cluster (includes fragment write-sets) | Unusual rate increase on the originating node |
| [`wsrep_received`](wsrep-status-index.md#wsrep_received) | Write-sets this node received (includes fragment write-sets) | Unusual rate increase on peer nodes |
| [`wsrep_replicated_bytes`](wsrep-status-index.md#wsrep_replicated_bytes) | Bytes this node sent in write-sets | Sudden sustained increase during streaming |
| [`wsrep_received_bytes`](wsrep-status-index.md#wsrep_received_bytes) | Bytes this node received in write-sets | Sudden sustained increase during streaming |
| [`wsrep_local_cert_failures`](wsrep-status-index.md#wsrep_local_cert_failures) | Certification failures, including streaming aborts | Any sustained increase |
| [`wsrep_local_bf_aborts`](wsrep-status-index.md#wsrep_local_bf_aborts) | Local transactions aborted by conflicting replicated transactions | Increase during hot-row streaming |

Galera counts each certified fragment as a separate write-set in `wsrep_replicated` and `wsrep_received`. A single large streaming transaction increases these counters incrementally as fragments certify, not only at final `COMMIT`.

Query the recommended variables on every node:

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

Combine the query above with `mysql.wsrep_streaming_log` on each node for a complete streaming replication dashboard. See [Monitor the cluster](monitoring.md) for general PXC alerting guidance.

### Check active fragments in mysql.wsrep_streaming_log

Each row in `mysql.wsrep_streaming_log` represents one certified fragment from an in-progress streaming transaction. Query metadata columns only. The `frag` column contains full binary log events and can exceed [`max_allowed_packet`](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_max_allowed_packet).

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

List active streaming transactions during a large job:

```sql
SELECT node_uuid, trx_id, seqno, flags
FROM mysql.wsrep_streaming_log;
```

The following output shows the latest certified fragment (`seqno = 4`) for one in-progress transaction:

??? example "Expected output"

    ```{.text .no-copy}
    +--------------------------------------+--------+-------+-------+
    | node_uuid                            | trx_id | seqno | flags |
    +--------------------------------------+--------+-------+-------+
    | a006244a-7ed8-11e9-bf00-867215999c7c |     26 |     4 |     1 |
    +--------------------------------------+--------+-------+-------+
    1 row in set (0.00 sec)
    ```

The `seqno` value increases as Galera certifies additional fragments for the same `trx_id`. The `node_uuid` column identifies the cluster node that originated the streaming transaction.

### Check related cluster status variables

The variables in the table above cover cluster pressure during streaming jobs. Query all wsrep status variables when you need broader context:

```sql
SHOW GLOBAL STATUS LIKE 'wsrep_%';
```

For broader cluster monitoring guidance, see [Monitor the cluster](monitoring.md).

## What does wsrep_SR_store control?

[`wsrep_SR_store`](wsrep-system-index.md#wsrep_sr_store) defines where Galera persists streaming replication fragments. The variable is global and not dynamic. Change the value only at startup.

### When wsrep_SR_store is table

The default value is `table`. Galera writes each fragment to `mysql.wsrep_streaming_log` on every node. The table provides durable storage for partial streaming transactions.

Disk writes add overhead during active streaming replication. The overhead is the trade-off for crash recovery. After a node crash, Galera can use the log to recover or roll back in-progress streaming transactions.

### When wsrep_SR_store is none

The value `none` disables persistence to `mysql.wsrep_streaming_log`. Galera still replicates and certifies fragments in memory during normal operation. Fragments are not double-written to the log table.

The `none` setting reduces log-table I/O. The setting also removes the durable recovery path for partial streaming transactions. After a crash, Galera cannot reconstruct in-progress streaming fragments from the log. Recovery may require manual intervention or transaction rollback.

The `none` setting does not eliminate memory use. Each node still buffers fragment write-sets during replication and certification. Very small fragment sizes increase fragment count and can raise memory pressure across the cluster.

Use `wsrep_SR_store = 'none'` only when you accept the crash-recovery trade-off and you have tested fragment sizing under peak load. Keep the default `table` value for production workloads unless profiling shows a clear benefit.

### What is the lifecycle of mysql.wsrep_streaming_log?

Galera manages `mysql.wsrep_streaming_log` automatically. Do not manually `DELETE` from or `TRUNCATE` the table.

The table holds rows only while a streaming transaction is open and has certified fragments. Each row stores one fragment's metadata and, when `wsrep_SR_store` is `table`, the fragment payload in the `frag` column.

Galera removes fragment rows from every node when the streaming transaction completes normally:

* `COMMIT`: Galera removes all fragment rows after the final fragment certifies.

* `ROLLBACK`: Galera removes fragment rows and rolls back applied fragment data on every node.

The table is empty when no streaming transaction is active on the cluster. A non-zero `COUNT(*)` during idle periods indicates an open streaming transaction somewhere in the cluster.

An abandoned client session can leave fragment rows in place until the server ends the transaction. The server rolls back the transaction when the client disconnects or when [`wait_timeout`](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_wait_timeout) or [`interactive_timeout`](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_interactive_timeout) closes the idle connection. Until that rollback completes, fragment rows and `frag` data remain on disk.

A long-running streaming transaction with an open connection can grow `mysql.wsrep_streaming_log` on every node. Monitor fragment counts during large jobs. Size fragments to balance replication progress against log-table growth.

### How do you force-kill a runaway streaming transaction?

Do not manually delete rows from `mysql.wsrep_streaming_log`. End the open transaction on the originating node instead. A connection-level `KILL` rolls back the streaming transaction through Galera on every node. The operation does not cause a cluster split-brain event. Split-brain results from network partitions and quorum loss, not from terminating a single client session.

Use this emergency playbook when log-table growth threatens disk space:
{.power-number}

1. Confirm active streaming fragments on each node:

    ```sql
    SELECT COUNT(*) AS active_fragments FROM mysql.wsrep_streaming_log;
    ```

2. Identify the originating node and session. Query fragment metadata to find the origin node:

    ```sql
    SELECT DISTINCT node_uuid, trx_id FROM mysql.wsrep_streaming_log;
    ```

    The `node_uuid` value identifies the cluster node where the streaming transaction started. Query [`mysql.wsrep_cluster_members`](https://mariadb.com/docs/galera-cluster/reference/galera-cluster-system-tables) on any node to map the UUID to a host name or address. Run `SHOW FULL PROCESSLIST` on that node:

    ```sql
    SHOW FULL PROCESSLIST;
    ```

    Look for a long-running DML statement or an open transaction in the `State` column. Note the connection `Id`. The streaming log does not expose the connection `Id` directly.

3. Terminate the connection on the node where the session runs:

    ```sql
    KILL <connection_id>;
    ```

    Replace `<connection_id>` with the `Id` from `SHOW FULL PROCESSLIST`. `KILL` is shorthand for `KILL CONNECTION`.

4. Wait for Galera rollback to finish on all nodes. Rollback duration depends on the number of certified fragments.

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

    Expect `Primary` cluster status and `Synced` local state on healthy nodes.

`KILL QUERY <connection_id>` ends only the current statement. Use `KILL CONNECTION` when the session sits idle inside an open transaction.

Avoid waiting for [`wait_timeout`](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_wait_timeout) or [`interactive_timeout`](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_interactive_timeout) when disk use is already critical. A forced disconnect triggers immediate rollback and log cleanup across the cluster.

## Do tables need a primary key for streaming replication?

Streaming replication uses the same [certification](glossary.md#certification) and write-set machinery as standard Galera replication. Tables involved in streaming transactions must have a primary key.

Galera uses primary-key values to identify rows for certification and cluster-wide locking. Without a primary key, Galera cannot reliably order or lock individual rows across nodes during fragment certification.

Under [`pxc_strict_mode=ENFORCING`](strict-mode.md), PXC denies `DELETE` and other undesirable writes to tables without an explicit primary key. Streaming replication does not relax that validation. A streaming `DELETE` against a table without a primary key fails with an error when strict mode is `ENFORCING`.

All tables in the cluster should define a primary key. See [Percona XtraDB Cluster limitations](limitation.md) and [Tables without primary keys](strict-mode.md).

## How does streaming replication interact with pxc_strict_mode?

Streaming replication does not require a dedicated [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) setting. The default value `ENFORCING` is supported and recommended.

Streaming replication is a supported Galera 4 feature. [`PXC Strict Mode`](strict-mode.md) does not treat streaming replication as a tech preview capability.

Strict mode validations still apply inside streaming transactions. Operations that strict mode blocks on a standard transaction are also blocked when streaming replication is enabled for the session. Examples include unsupported storage engines and conflicting plugins listed in [PXC Strict Mode validations](strict-mode.md#validations).

Keep `pxc_strict_mode=ENFORCING` on all cluster nodes. Streaming replication does not bypass strict mode checks on peer nodes.

## How does streaming replication affect binary logging and async replicas?

Streaming replication affects Galera cluster nodes and downstream async replicas differently.

### Galera cluster replication (between PXC nodes)

Galera replicates certified fragments to every cluster node while the transaction remains open. Fragments travel through the Galera write-set path. Fragments do not appear in the MySQL binary log during the open transaction. Partial fragment data is stored in `mysql.wsrep_streaming_log` when [`wsrep_SR_store`](wsrep-system-index.md#wsrep_sr_store) is `table`.

Other cluster nodes apply certified fragments before final `COMMIT`. Applied fragments hold cluster-wide row locks, but they remain part of an in-progress transaction on every peer node. Under normal isolation levels, other sessions do not read the uncommitted fragment data. Instead, sessions that try to modify the affected rows block on the fragment locks until the originating session commits or rolls back. Only the full transaction boundary is atomic from the application perspective.

### Binary log on the originating PXC node

The MySQL binary log records the client transaction when the originating session issues `COMMIT`. Galera does not write streaming fragments to `log_bin` as separate binlog events during the open transaction.

A streaming transaction therefore produces one binlog transaction entry at commit time on the node where the client connected. The binlog event size reflects the full committed change set, not individual fragment sizes.

Galera builds row-format write-sets internally. Set `binlog_format=ROW` on nodes that serve as async replication sources or that require row-based binary logging.

### Downstream asynchronous MySQL replicas

Downstream async replicas do not receive streaming fragments in real time. The replica SQL thread applies the transaction when the source node commits and writes the binlog event.

A large streaming transaction can still produce a sudden downstream replication lag spike at commit time. The async replica receives one large binlog transaction, even though PXC cluster nodes applied fragments incrementally during the open transaction.

Configure the PXC source node for async replication as described in [Does Percona XtraDB Cluster work with regular MySQL replication?](faq.md#does-percona-xtradb-cluster-work-with-regular-mysql-replication):

* Unique non-zero `server_id` on each PXC node.

* `log_bin` enabled on the node that acts as the replication source. Client writes that originate on the source node are written to the binary log at commit time.

* [`log_replica_updates`](https://dev.mysql.com/doc/refman/{{vers}}/en/replication-options-replica.html#sysvar_log_replica_updates) (or deprecated `log_slave_updates`) enabled when the source node must relay writes that executed on other PXC nodes and were received through Galera. Without this setting, only locally originated writes appear in the source binary log.

Streaming replication reduces lag and flow control between PXC nodes during the open transaction. Streaming replication does not spread downstream async apply load across the transaction lifetime. Plan async replica capacity for peak commit-time binlog volume.

## How do you manage a hot record?

A work order queue must assign each user a unique queue position. Streaming replication can protect the critical update that increments the queue position.

### Why not use SELECT ... FOR UPDATE?

`SELECT ... FOR UPDATE` locks rows on the local node only. Galera uses optimistic replication by default. Other nodes do not see those locks until the transaction commits and the write-set replicates. A concurrent transaction on another node can modify the same row and win certification first.

Streaming replication certifies and applies the critical fragment on all nodes before the transaction continues. The fragment propagates cluster-wide locks for the rows the fragment modified. Other nodes block conflicting changes on those rows during certification. The queue update reaches every node before non-critical work in the same transaction runs.

`SELECT ... FOR UPDATE` still helps for local read consistency within one node. Streaming replication addresses cluster-wide ordering under Galera's certification model.

Configure streaming replication for the queue update with these steps:
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

2. Read the data the transaction requires. Enable streaming replication for the next statement:

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

3. Run the critical update. Galera fragments and replicates the statement immediately:

    ```sql
    UPDATE work_orders
       SET queue_position = queue_position + 1;
    ```

    The following output confirms that the queue position was updated:

    ??? example "Expected output"

        ```{.text .no-copy}
        Query OK, 1 row affected (0.01 sec)
        Rows matched: 1  Changed: 1  Warnings: 0
        ```

4. Disable streaming replication for the remainder of the transaction:

    ```sql
    SET SESSION wsrep_trx_fragment_size = 0;
    ```

    The following output confirms that streaming replication is disabled:

    ??? example "Expected output"

        ```{.text .no-copy}
        Query OK, 0 rows affected (0.00 sec)
        ```

5. Complete non-critical work for the work order. Commit the transaction:

    ```sql
    COMMIT;
    ```

    The following output confirms that the transaction committed:

    ??? example "Expected output"

        ```{.text .no-copy}
        Query OK, 0 rows affected (0.01 sec)
        ```

The `queue_position` update replicates and certifies across the cluster before the transaction continues. Race conditions on queue assignment are less likely.

## What happens when the initiating node crashes mid-transaction?

A streaming transaction is incomplete until the originating session issues `COMMIT`. Certified fragments on surviving nodes are not final committed work from the application perspective until the full transaction completes.

### When the originating node fails

If the initiating node crashes or is evicted from the cluster, the open transaction on that node is lost. Surviving nodes retain the certified fragments they already applied.

Galera treats the incomplete streaming transaction as aborted. The rollbacker thread rolls back applied fragment data on every surviving node. Galera also removes the corresponding rows from `mysql.wsrep_streaming_log` on those nodes. Surviving nodes do not keep partial streaming changes as committed data.

There is no separate administrator timeout for orphaned streaming fragments on healthy nodes. Cleanup runs as part of Galera's transaction abort handling when the originator disappears or the transaction fails.

### When the crashed node rejoins

The rejoining node was behind the cluster state at the moment of failure. The node typically receives an [IST](glossary.md#ist) or [SST](glossary.md#sst) from a donor to resynchronize. See [Crash recovery](crash-recovery.md).

When `wsrep_SR_store` is `table`, Galera can use `mysql.wsrep_streaming_log` on the restarted node to recover or roll back that node's own partial streaming state after a crash. When `wsrep_SR_store` is `none`, that local recovery path is not available.

### When the entire cluster stops during streaming

A power loss or full cluster outage may leave nodes with `seqno: -1` in `grastate.dat`. Follow [Crash recovery](crash-recovery.md) to identify the most advanced node and bootstrap the cluster. Streaming log rows from incomplete transactions are resolved during Galera startup and transaction recovery on each node.

## What are the limitations?

Review these limitations before you enable streaming replication.

### What is the performance overhead?

Galera records write-sets to `mysql.wsrep_streaming_log` on every node when streaming replication is enabled and [`wsrep_SR_store`](wsrep-system-index.md#wsrep_sr_store) is `table`. The log provides crash recovery for partial transactions. Log writes add overhead. Use streaming replication only when fragmentation is required.

### What is the rollback cost?

Galera must roll back every certified fragment on all nodes when a streaming transaction aborts after partial certification. A failure in any fragment aborts the full streaming transaction. Frequent rollbacks of streaming transactions increase resource use on every node.

Prefer shorter, smaller transactions in application design. When large transactions are unavoidable, use streaming replication with appropriately sized fragments instead of raising [`wsrep_max_ws_size`](wsrep-system-index.md#wsrep_max_ws_size) and [`wsrep_max_ws_rows`](wsrep-system-index.md#wsrep_max_ws_rows) globally.

### What happens with SAVEPOINTs inside a streaming transaction?

`SAVEPOINT`, `ROLLBACK TO SAVEPOINT`, and `RELEASE SAVEPOINT` are discouraged inside streaming transactions. Galera does not support partial savepoint rollback across the cluster.

!!! warning

    Galera does not send compensation write-sets for savepoint rollbacks. Once fragments are certified on peer nodes, `ROLLBACK TO SAVEPOINT` escalates to a full transaction rollback on every node.

#### Explicit savepoints

An explicit savepoint records a rollback point in the local transaction on the originating node only. Peer nodes do not receive savepoint events. Peer nodes have already certified and applied fragments from statements that ran after the savepoint was created.

Consider a transaction that runs five statements and streams a fragment after each statement. The session then executes `ROLLBACK TO SAVEPOINT sp1` after statement five. Peer nodes still hold certified fragments from statements two through five. Galera cannot undo only those fragments on peer nodes.

Galera does not send compensation write-sets to reverse savepoint rollbacks on peer nodes. Partial savepoint rollback does not produce cluster-wide consistency.

Once any statement in the streaming transaction has replicated one or more fragments, Galera treats `ROLLBACK TO SAVEPOINT` as unsafe. Galera escalates the savepoint rollback to a full transaction rollback on every node. All certified fragments from the streaming transaction are undone cluster-wide. The `mysql.wsrep_streaming_log` table becomes empty after rollback completes.

`SAVEPOINT` before any fragment is replicated may succeed locally. Any later `ROLLBACK TO SAVEPOINT` after fragments were replicated still triggers the full cluster-wide rollback described above.

#### Implicit statement rollbacks

An implicit statement rollback occurs when a single statement fails inside an open transaction. MySQL rolls back that statement's changes on the local node. Examples include duplicate-key errors and constraint violations on a multi-row `INSERT`.

Galera applies the same safety rule as explicit savepoint rollbacks. Once the failing statement has replicated one or more fragments, Galera cannot perform a statement-only rollback on peer nodes. Galera aborts the entire streaming transaction and rolls back all certified fragments on every node.

#### Application guidance

Prefer plain `BEGIN`, `COMMIT`, and `ROLLBACK` boundaries for streaming transactions. Split complex logic into separate transactions instead of nested savepoints. For general cluster limitations on transaction types, see [Percona XtraDB Cluster limitations](limitation.md).
