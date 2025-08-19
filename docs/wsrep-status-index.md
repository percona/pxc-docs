# Index of wsrep status variables

This document provides detailed information about the cluster status variables. These variables help you monitor and troubleshoot your cluster's performance, health, and behavior.

## Quick reference table

### Application and parallelization variables

| Variable              | Description                                                      | Link                                          |
| --------------------- | ---------------------------------------------------------------- | --------------------------------------------- |
| `wsrep_apply_oooe`    | Out of Order of Execution - measures parallelization efficiency  | [`wsrep_apply_oooe`](#wsrep_apply_oooe)       |
| `wsrep_apply_oool`    | Out of Order of Latest - measures parallelization inefficiency   | [`wsrep_apply_oool`](#wsrep_apply_oool)       |
| `wsrep_apply_window`  | Average distance between concurrently applied sequence numbers   | [`wsrep_apply_window`](#wsrep_apply_window)   |
| `wsrep_commit_oooe`   | Commit phase parallelization efficiency                          | [`wsrep_commit_oooe`](#wsrep_commit_oooe)     |
| `wsrep_commit_window` | Average distance between concurrently committed sequence numbers | [`wsrep_commit_window`](#wsrep_commit_window) |

### Certification and conflict detection variables

| Variable                    | Description                                              | Link                                                      |
| --------------------------- | -------------------------------------------------------- | --------------------------------------------------------- |
| `wsrep_cert_bucket_count`   | Number of buckets in certification index hash table      | [`wsrep_cert_bucket_count`](#wsrep_cert_bucket_count)     |
| `wsrep_cert_deps_distance`  | Average distance between parallelizable sequence numbers | [`wsrep_cert_deps_distance`](#wsrep_cert_deps_distance)   |
| `wsrep_cert_index_size`     | Number of entries in certification index                 | [`wsrep_cert_index_size`](#wsrep_cert_index_size)         |
| `wsrep_cert_interval`       | Average time between certification events                | [`wsrep_cert_interval`](#wsrep_cert_interval)             |
| `wsrep_local_cert_failures` | Number of writesets that failed certification            | [`wsrep_local_cert_failures`](#wsrep_local_cert_failures) |

### Cluster membership and state variables

| Variable                   | Description                                                   | Link                                                    |
| -------------------------- | ------------------------------------------------------------- | ------------------------------------------------------- |
| `wsrep_cluster_conf_id`    | Cluster configuration ID (increments with membership changes) | [`wsrep_cluster_conf_id`](#wsrep_cluster_conf_id)       |
| `wsrep_cluster_size`       | Number of nodes in primary component                          | [`wsrep_cluster_size`](#wsrep_cluster_size)             |
| `wsrep_cluster_state_uuid` | UUID of current cluster state                                 | [`wsrep_cluster_state_uuid`](#wsrep_cluster_state_uuid) |
| `wsrep_cluster_status`     | Status of primary component                                   | [`wsrep_cluster_status`](#wsrep_cluster_status)         |
| `wsrep_connected`          | Whether node is connected to cluster                          | [`wsrep_connected`](#wsrep_connected)                   |
| `wsrep_ready`              | Whether node is ready to accept queries                       | [`wsrep_ready`](#wsrep_ready)                           |

### EVS (Extended Virtual Synchrony) protocol variables

| Variable                 | Description                                        | Link                                                |
| ------------------------ | -------------------------------------------------- | --------------------------------------------------- |
| `wsrep_evs_delayed`      | List of nodes considered delayed in cluster        | [`wsrep_evs_delayed`](#wsrep_evs_delayed)           |
| `wsrep_evs_evict_list`   | List of nodes evicted from cluster                 | [`wsrep_evs_evict_list`](#wsrep_evs_evict_list)     |
| `wsrep_evs_repl_latency` | Group communication replication latency statistics | [`wsrep_evs_repl_latency`](#wsrep_evs_repl_latency) |
| `wsrep_evs_state`        | Internal EVS protocol state                        | [`wsrep_evs_state`](#wsrep_evs_state)               |

### Flow control variables

| Variable                           | Description                                   | Link                                                                    |
| ---------------------------------- | --------------------------------------------- | ----------------------------------------------------------------------- |
| `wsrep_flow_control_interval`      | Lower and upper limits for flow control       | [`wsrep_flow_control_interval`](#wsrep_flow_control_interval)           |
| `wsrep_flow_control_interval_high` | Upper limit for flow control to trigger       | [`wsrep_flow_control_interval_high`](#wsrep_flow_control_interval_high) |
| `wsrep_flow_control_interval_low`  | Lower limit for flow control to stop          | [`wsrep_flow_control_interval_low`](#wsrep_flow_control_interval_low)   |
| `wsrep_flow_control_paused`        | Time paused due to flow control               | [`wsrep_flow_control_paused`](#wsrep_flow_control_paused)               |
| `wsrep_flow_control_paused_ns`     | Time paused due to flow control (nanoseconds) | [`wsrep_flow_control_paused_ns`](#wsrep_flow_control_paused_ns)         |
| `wsrep_flow_control_recv`          | Number of flow control pause events received  | [`wsrep_flow_control_recv`](#wsrep_flow_control_recv)                   |
| `wsrep_flow_control_requested`     | Whether node has requested flow control pause | [`wsrep_flow_control_requested`](#wsrep_flow_control_requested)         |
| `wsrep_flow_control_sent`          | Number of flow control pause events sent      | [`wsrep_flow_control_sent`](#wsrep_flow_control_sent)                   |
| `wsrep_flow_control_status`        | Whether flow control is enabled               | [`wsrep_flow_control_status`](#wsrep_flow_control_status)               |

### Local node performance variables

| Variable                     | Description                                                  | Link                                                        |
| ---------------------------- | ------------------------------------------------------------ | ----------------------------------------------------------- |
| `wsrep_local_bf_aborts`      | Number of local transactions aborted by replica transactions | [`wsrep_local_bf_aborts`](#wsrep_local_bf_aborts)           |
| `wsrep_local_cached_downto`  | Lowest sequence number in GCache                             | [`wsrep_local_cached_downto`](#wsrep_local_cached_downto)   |
| `wsrep_local_commits`        | Number of writesets committed on node                        | [`wsrep_local_commits`](#wsrep_local_commits)               |
| `wsrep_local_index`          | Node's position in cluster                                   | [`wsrep_local_index`](#wsrep_local_index)                   |
| `wsrep_local_recv_queue`     | Current length of receive queue                              | [`wsrep_local_recv_queue`](#wsrep_local_recv_queue)         |
| `wsrep_local_recv_queue_avg` | Average length of receive queue                              | [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg) |
| `wsrep_local_replays`        | Number of transaction replays                                | [`wsrep_local_replays`](#wsrep_local_replays)               |
| `wsrep_local_send_queue`     | Current length of send queue                                 | [`wsrep_local_send_queue`](#wsrep_local_send_queue)         |
| `wsrep_local_send_queue_avg` | Average length of send queue                                 | [`wsrep_local_send_queue_avg`](#wsrep_local_send_queue_avg) |
| `wsrep_local_state`          | Local node state                                             | [`wsrep_local_state`](#wsrep_local_state)                   |
| `wsrep_local_state_comment`  | Human-readable local state description                       | [`wsrep_local_state_comment`](#wsrep_local_state_comment)   |
| `wsrep_local_state_uuid`     | UUID of local node state                                     | [`wsrep_local_state_uuid`](#wsrep_local_state_uuid)         |

### Replication and transfer variables

| Variable                 | Description                                         | Link                                                |
| ------------------------ | --------------------------------------------------- | --------------------------------------------------- |
| `wsrep_received`         | Total number of writesets received from other nodes | [`wsrep_received`](#wsrep_received)                 |
| `wsrep_received_bytes`   | Total size of writesets received from other nodes   | [`wsrep_received_bytes`](#wsrep_received_bytes)     |
| `wsrep_replicated`       | Total number of writesets sent to other nodes       | [`wsrep_replicated`](#wsrep_replicated)             |
| `wsrep_replicated_bytes` | Total size of writesets sent to other nodes         | [`wsrep_replicated_bytes`](#wsrep_replicated_bytes) |
| `wsrep_repl_data_bytes`  | Total size of data replicated                       | [`wsrep_repl_data_bytes`](#wsrep_repl_data_bytes)   |
| `wsrep_repl_keys`        | Total number of keys replicated                     | [`wsrep_repl_keys`](#wsrep_repl_keys)               |
| `wsrep_repl_keys_bytes`  | Total size of keys replicated                       | [`wsrep_repl_keys_bytes`](#wsrep_repl_keys_bytes)   |
| `wsrep_repl_other_bytes` | Total size of other bits replicated                 | [`wsrep_repl_other_bytes`](#wsrep_repl_other_bytes) |

### Sequence number and state variables

| Variable                          | Description                                   | Link                                                                  |
| --------------------------------- | --------------------------------------------- | --------------------------------------------------------------------- |
| `wsrep_last_applied`              | Sequence number of last applied transaction   | [`wsrep_last_applied`](#wsrep_last_applied)                           |
| `wsrep_last_committed`            | Sequence number of last committed transaction | [`wsrep_last_committed`](#wsrep_last_committed)                       |
| `wsrep_ist_receive_seqno_current` | Sequence number of current transaction in IST | [`wsrep_ist_receive_seqno_current`](#wsrep_ist_receive_seqno_current) |
| `wsrep_ist_receive_seqno_end`     | Sequence number of last transaction in IST    | [`wsrep_ist_receive_seqno_end`](#wsrep_ist_receive_seqno_end)         |
| `wsrep_ist_receive_seqno_start`   | Sequence number of first transaction in IST   | [`wsrep_ist_receive_seqno_start`](#wsrep_ist_receive_seqno_start)     |

### System and provider variables

| Variable                   | Description                                     | Link                                                    |
| -------------------------- | ----------------------------------------------- | ------------------------------------------------------- |
| `wsrep_causal_reads`       | Number of causal reads performed                | [`wsrep_causal_reads`](#wsrep_causal_reads)             |
| `wsrep_gcache_pool_size`   | Size of GCache page pool and dynamic memory     | [`wsrep_gcache_pool_size`](#wsrep_gcache_pool_size)     |
| `wsrep_gcomm_uuid`         | cluster view ID from gvwstate.dat               | [`wsrep_gcomm_uuid`](#wsrep_gcomm_uuid)                 |
| `wsrep_incoming_addresses` | Comma-separated list of incoming node addresses | [`wsrep_incoming_addresses`](#wsrep_incoming_addresses) |
| `wsrep_ist_receive_status` | Progress of IST for joiner node                 | [`wsrep_ist_receive_status`](#wsrep_ist_receive_status) |
| `wsrep_monitor_status`     | Cluster monitoring status                       | [`wsrep_monitor_status`](#wsrep_monitor_status)         |
| `wsrep_protocol_version`   | Version of wsrep protocol used                  | [`wsrep_protocol_version`](#wsrep_protocol_version)     |
| `wsrep_provider_name`      | Name of wsrep provider                          | [`wsrep_provider_name`](#wsrep_provider_name)           |
| `wsrep_provider_vendor`    | Name of wsrep provider vendor                   | [`wsrep_provider_vendor`](#wsrep_provider_vendor)       |
| `wsrep_provider_version`   | Current version of wsrep provider               | [`wsrep_provider_version`](#wsrep_provider_version)     |

---

## Detailed variable descriptions

### `wsrep_apply_oooe`

#### What the variable measures

The `wsrep_apply_oooe` status variable measures the parallelization efficiency of the applier threads. The acronym stands for "Out of Order of Execution." The variable tracks the frequency with which a node applies incoming writesets out of their original sequence number order. This out-of-order application indicates that multiple applier threads are successfully applying transactions simultaneously.

#### Expected results

A high value, close to `1`, indicates excellent parallelization. This means the cluster is efficiently using its multiple applier threads to apply replicated writesets. A value of `0` suggests no out-of-order application, which points to a lack of parallelization and a potential performance bottleneck. A value greater than `0.5` is generally a good sign, while values below this may warrant further investigation into applier thread configuration.

##### Example of use

You can run this command on any node in the cluster; it requires only the `PROCESS` privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_apply_oooe';
```

??? example "Expected output"

    ```{.text .no-copy}
    +------------------+----------+
    | Variable_name    | Value    |
    +------------------+----------+
    | wsrep_apply_oooe | 0.985654 |
    +------------------+----------+
    ```

##### Related status variables

When you analyze parallelization performance, consider these related status variables together:

###### [`wsrep_apply_window`](#wsrep_apply_window)

This variable measures the average distance between the highest and lowest concurrently applied sequence numbers. A higher value indicates better parallelization potential. Use this variable with `wsrep_apply_oooe` to understand both the potential for parallelization and how effectively you are utilizing parallelization.

#### [`wsrep_apply_oool`](#wsrep_apply_oool)

This variable tracks "Out of Order of Latest" - when older transactions apply after newer ones. A low value (close to 0) is ideal. High values may indicate slow transactions causing bottlenecks, which could explain low `wsrep_apply_oooe` values.

#### [`wsrep_cert_deps_distance`](#wsrep_cert_deps_distance)

This variable shows the average distance between sequence numbers that can be applied in parallel. This indicates the potential for parallelization based on transaction dependencies. Compare this variable with `wsrep_apply_oooe` to see if your cluster is achieving its parallelization potential.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable shows the average length of the receive queue. If this value is consistently high (> 0) while `wsrep_apply_oooe` is low, the node cannot apply writesets fast enough, suggesting you need more applier threads or better resource allocation.

#### [`wsrep_commit_oooe`](#wsrep_commit_oooe)

This variable is similar to `wsrep_apply_oooe` but for the commit phase. While transactions can be applied in parallel, they must be committed in order. A high value here indicates good commit parallelization.

#### Monitoring parallelization performance

To get a comprehensive view of your cluster's parallelization efficiency, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_apply_oooe|wsrep_apply_window|wsrep_apply_oool|wsrep_cert_deps_distance|wsrep_local_recv_queue_avg|wsrep_commit_oooe';
```

This query helps you identify whether parallelization issues are due to the following:

* Insufficient applier threads

* Transaction conflicts limiting parallelization

* Resource constraints

* Network or replication bottlenecks

### `wsrep_apply_oool`

#### What the variable measures

The `wsrep_apply_oool` status variable measures a specific type of parallelization inefficiency within the applier threads. The acronym stands for "Out of Order of Latest." This variable tracks when a writeset (a collection of changes from a database transaction) with a lower sequence number (an "older" transaction) applies after a writeset with a higher sequence number (a "newer" transaction). This condition occurs when a smaller transaction finishes first while a larger, earlier transaction is still processing.

#### Expected results

You expect this variable to have a value close to `0`. A low value indicates that writesets are generally applying in the correct sequence order, which is ideal. A high value suggests that "slow" transactions are causing a bottleneck, forcing the applier threads to skip ahead and apply newer writesets first. A high value indicates you need to investigate the performance of slow-running transactions or adjust applier thread settings.

### Example of use

You can run this command on any node in the cluster; it requires only the `PROCESS` privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_apply_oool';
```

??? example "Expected output"

    ```{.text .no-copy}
    +------------------+----------+
    | Variable_name    | Value    |
    +------------------+----------+
    | wsrep_apply_oool | 0.000132 |
    +------------------+----------+
    ```

### Related status variables

When you analyze out-of-order application issues, consider these related status variables together:

#### [`wsrep_apply_oooe`](#wsrep_apply_oooe)

This variable measures the parallelization efficiency of applier threads. A high value indicates good parallelization, while a low value may explain why you see high `wsrep_apply_oool` values - limited parallelization can cause older transactions to lag behind newer ones.

#### [`wsrep_apply_window`](#wsrep_apply_window)

This variable shows the average distance between concurrently applied sequence numbers. A narrow window combined with high `wsrep_apply_oool` values suggests the applier threads cannot process transactions fast enough, causing older transactions to fall behind.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable indicates the average length of the receive queue. High queue values with high `wsrep_apply_oool` values confirm that the node cannot keep up with incoming writesets, causing older transactions to be delayed.

#### [`wsrep_cert_deps_distance`](#wsrep_cert_deps_distance)

This variable shows the potential for parallelization based on transaction dependencies. A low value combined with high `wsrep_apply_oool` values indicates that transaction conflicts are preventing efficient parallelization.

#### [`wsrep_commit_oooe`](#wsrep_commit_oooe)

This variable measures commit phase parallelization. While this variable focuses on commits rather than applies, high values here with low `wsrep_apply_oool` values suggest the bottleneck is in the apply phase, not the commit phase.

### Monitoring out-of-order application

To get a comprehensive view of your cluster's out-of-order application patterns, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_apply_oool|wsrep_apply_oooe|wsrep_apply_window|wsrep_local_recv_queue_avg|wsrep_cert_deps_distance|wsrep_commit_oooe';
```

This query helps you identify whether out-of-order application issues are due to the following:

* Insufficient applier threads

* Transaction conflicts limiting parallelization

* Resource constraints causing processing delays

* Network or replication bottlenecks

* Inefficient transaction ordering

### `wsrep_apply_window`

#### What the variable measures

The `wsrep_apply_window` status variable measures the average distance between the highest and lowest concurrently applied sequence numbers (`seqno`). This variable gives you insight into the potential degree of parallelization your cluster achieves. A "wider" window, or a higher value, means the applier threads can process a broader range of transactions at the same time.

### Expected results

A higher value indicates a healthy, efficient cluster with strong parallelization. A higher value suggests your node has sufficient resources and is keeping up with the replication flow. A low value suggests a bottleneck in the applier threads, which may be due to resource constraints or a high number of large, conflicting transactions.

### Example of use

To retrieve the current value of this variable, run this standard `SHOW GLOBAL STATUS` command:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_apply_window';
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------+----------+
    | Variable_name       | Value    |
    +---------------------+----------+
    | wsrep_apply_window  | 5.163966 |
    +---------------------+----------+
    ```

### Related status variables

When you analyze parallelization window performance, consider these related status variables together:

#### [`wsrep_apply_oooe`](#wsrep_apply_oooe)

This variable measures the parallelization efficiency of applier threads. A high value combined with a wide `wsrep_apply_window` indicates excellent parallelization performance. A low value with a narrow window suggests limited parallelization capability.

#### [`wsrep_apply_oool`](#wsrep_apply_oool)

This variable tracks out-of-order application of older transactions. A narrow `wsrep_apply_window` combined with high `wsrep_apply_oool` values suggests the applier threads cannot process transactions fast enough, causing older transactions to fall behind.

#### [`wsrep_cert_deps_distance`](#wsrep_cert_deps_distance)

This variable shows the potential for parallelization based on transaction dependencies. A wide `wsrep_apply_window` with a high `wsrep_cert_deps_distance` indicates your cluster is achieving its full parallelization potential.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable indicates the average length of the receive queue. A narrow `wsrep_apply_window` with high queue values confirms that the node cannot keep up with incoming writesets, limiting parallelization opportunities.

#### [`wsrep_commit_window`](#wsrep_commit_window)

This variable measures the average distance between concurrently committed sequence numbers. Compare this with `wsrep_apply_window` to see if bottlenecks occur in the apply phase or commit phase.

### Monitoring parallelization window performance

To get a comprehensive view of your cluster's parallelization window performance, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_apply_window|wsrep_apply_oooe|wsrep_apply_oool|wsrep_cert_deps_distance|wsrep_local_recv_queue_avg|wsrep_commit_window';
```

This query helps you identify whether window performance issues are due to the following:

* Insufficient applier threads

* Transaction conflicts limiting parallelization

* Resource constraints reducing processing capacity

* Network or replication bottlenecks

* Inefficient transaction dependency patterns

### `wsrep_causal_reads`

#### What the variable measures

The `wsrep_causal_reads` status variable tracks the number of writesets (collections of changes from database transactions) a node processes while the `wsrep_causal_reads` system variable is enabled. This variable shows you the number of transactions that trigger a "causal read" or "causal wait" on the node. A causal wait ensures that your node applies all replicated transactions up to a certain point before a client can perform a read.

### Expected results

This value is a counter, so it only increases. The number on your node reflects how often an application or user enables causal reads for their sessions. A high number suggests that applications are frequently using this feature to ensure reads return the latest data. A value of `0` is normal if you do not use this feature.

### Example of use

First, you set the session variable to enable causal reads. This is often done for critical reads in an application where you need to guarantee the freshest data:

```{.bash data-prompt="$"}
$ SET SESSION wsrep_causal_reads = ON;
$ SELECT * FROM my_important_table;
$ SET SESSION wsrep_causal_reads = OFF;
```

Then, you can check the `wsrep_causal_reads` status variable to see how many writesets were processed during the causal read period:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_causal_reads';
```

??? example "Expected output"

    ```{.text .no-copy}
    +------------------------+-------+
    | Variable_name          | Value |
    +------------------------+-------+
    | wsrep_causal_reads     | 13    |
    +------------------------+-------+
    ```

### Related status variables

When you analyze causal read performance, consider these related status variables together:

#### [`wsrep_local_commits`](#wsrep_local_commits)

This variable shows the number of writesets committed on your node. Compare this with `wsrep_causal_reads` to understand the ratio of causal reads to total commits, which indicates how frequently your applications use causal consistency.

#### [`wsrep_received`](#wsrep_received)

This variable tracks the total number of writesets received from other nodes. A high `wsrep_causal_reads` value with a high `wsrep_received` value suggests your applications frequently need to wait for replicated data before reading.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable indicates the average length of the receive queue. High queue values with high `wsrep_causal_reads` values suggest that causal reads are waiting for queued writesets to be applied, potentially causing read delays.

#### [`wsrep_apply_window`](#wsrep_apply_window)

This variable measures the parallelization window for applied transactions. A narrow window with high `wsrep_causal_reads` values indicates that causal reads may be waiting for slower transaction processing.

#### [`wsrep_ready`](#wsrep_ready)

This variable shows if your node is ready to accept queries. When `wsrep_ready` is `OFF`, causal reads will fail, so you should monitor this variable alongside `wsrep_causal_reads`.

### Monitoring causal read performance

To get a comprehensive view of your cluster's causal read performance, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_causal_reads|wsrep_local_commits|wsrep_received|wsrep_local_recv_queue_avg|wsrep_apply_window|wsrep_ready';
```

This query helps you identify whether causal read issues are due to the following:

* High application demand for causal consistency

* Slow replication is causing read delays

* Queue backlogs affecting read performance

* Node readiness issues preventing causal reads

* Insufficient parallelization is slowing transaction processing

### `wsrep_cert_bucket_count`

#### What the variable measures

The `wsrep_cert_bucket_count` status variable shows the number of buckets in the hash table of the certification index. The certification index is a critical component that helps the cluster detect conflicts between transactions by checking for conflicting writesets (collections of changes from database transactions) before a transaction is committed. A bucket is a slot in this hash table, and the count directly relates to the size of the index.

The certification index uses a hash table for fast conflict detection. Think of it as a filing cabinet where each bucket (drawer) can hold multiple active transactions. More buckets mean a larger hash table, which reduces hash collisions and improves conflict detection performance. Fewer buckets can lead to performance issues in busy clusters where many transactions compete for the same hash table slots.

### Expected results

This value is a counter and should be greater than zero. This value reflects the configured size of the certification index hash table. A larger number indicates a larger hash table, which can help reduce hash collisions and improve the efficiency of conflict checking in a busy cluster. A low value might suggest that the certification index is too small for your workload, potentially leading to increased conflicts.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cert_bucket_count';
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------------+-------+
    | Variable_name             | Value |
    +---------------------------+-------+
    | wsrep_cert_bucket_count   | 1533  |
    +---------------------------+-------+
    ```

### Related status variables

When you analyze certification index performance, consider these related status variables together:

#### [`wsrep_cert_index_size`](#wsrep_cert_index_size)

This variable shows the number of entries in the certification index. Compare this with `wsrep_cert_bucket_count` to understand the utilization ratio of your certification index hash table. A high ratio suggests potential hash collisions.

#### [`wsrep_local_cert_failures`](#wsrep_local_cert_failures)

This variable tracks the number of writesets that failed the certification test. High failure rates with a low `wsrep_cert_bucket_count` value suggest that your certification index is too small for your workload.

#### [`wsrep_cert_deps_distance`](#wsrep_cert_deps_distance)

This variable measures the potential for parallelization based on transaction dependencies. A low `wsrep_cert_bucket_count` combined with a low `wsrep_cert_deps_distance` suggests that your certification index size may be limiting parallelization.

#### [`wsrep_cert_interval`](#wsrep_cert_interval)

This variable shows the average number of writesets received during local transaction processing. A high value with a low `wsrep_cert_bucket_count` indicates that your certification index may be undersized for your cluster's concurrency level.

#### [`wsrep_local_commits`](#wsrep_local_commits)

This variable shows the number of writesets committed on your node. A high commit rate with a low `wsrep_cert_bucket_count` suggests that your certification index may need to be larger to handle your transaction volume efficiently.

### Monitoring certification index performance

To get a comprehensive view of your cluster's certification index performance, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cert_bucket_count|wsrep_cert_index_size|wsrep_local_cert_failures|wsrep_cert_deps_distance|wsrep_cert_interval|wsrep_local_commits';
```

This query helps you identify whether certification index issues are due to the following:

* Insufficient hash table size for your workload

* High transaction volume overwhelming the index

* Hash collisions reducing conflict detection efficiency

* Inadequate parallelization due to index constraints

* Configuration mismatches with cluster concurrency

### `wsrep_cert_deps_distance`

#### What the variable measures

The `wsrep_cert_deps_distance` status variable measures the average distance between the highest and lowest sequence numbers that can be applied in parallel. This value represents the potential degree of parallelization your node can achieve based on the current workload. A larger distance signifies more writesets (collections of changes from database transactions) that are independent of each other, allowing the applier threads to work on them concurrently.

### Expected results

A higher value indicates greater potential for parallel application, which is a good sign for cluster performance. This means the transactions are not highly dependent on each other, allowing the applier to process them with multiple threads. A low value suggests that a high number of transactions are conflicting or are serialized, which limits parallelization and could lead to performance issues.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cert_deps_distance';
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------------------+-----------+
    | Variable_name            | Value     |
    +--------------------------+-----------+
    | wsrep_cert_deps_distance | 25.123456 |
    +--------------------------+-----------+
    ```

### Related status variables

When you analyze transaction dependency patterns, consider these related status variables together:

#### [`wsrep_apply_oooe`](#wsrep_apply_oooe)

This variable measures the parallelization efficiency of applier threads. A high `wsrep_cert_deps_distance` value with a high `wsrep_apply_oooe` value indicates your cluster is achieving excellent parallelization based on independent transactions.

#### [`wsrep_apply_window`](#wsrep_apply_window)

This variable shows the average distance between concurrently applied sequence numbers. A wide window with a high `wsrep_cert_deps_distance` suggests your applier threads are effectively utilizing the available parallelization potential.

#### [`wsrep_cert_bucket_count`](#wsrep_cert_bucket_count)

This variable shows the number of buckets in the certification index hash table. A low `wsrep_cert_deps_distance` with a low bucket count suggests that your certification index size may be limiting parallelization opportunities.

#### [`wsrep_cert_index_size`](#wsrep_cert_index_size)

This variable shows the number of entries in the certification index. A high index size with a low `wsrep_cert_deps_distance` indicates that many transactions are conflicting, reducing parallelization potential.

#### [`wsrep_local_cert_failures`](#wsrep_local_cert_failures)

This variable tracks certification failures. High failure rates with a low `wsrep_cert_deps_distance` suggest that transaction conflicts are preventing effective parallelization.

### Monitoring transaction dependency performance

To get a comprehensive view of your cluster's transaction dependency patterns, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cert_deps_distance|wsrep_apply_oooe|wsrep_apply_window|wsrep_cert_bucket_count|wsrep_cert_index_size|wsrep_local_cert_failures';
```

This query helps you identify whether dependency-related performance issues are due to the following:

* High transaction conflicts limiting parallelization

* Insufficient certification index capacity

* Inefficient applier thread utilization

* Configuration mismatches affecting parallelization

* Workload patterns that create transaction dependencies

### `wsrep_cert_index_size`

#### What the variable measures

The `wsrep_cert_index_size` status variable shows the number of entries in the certification index. This index is an in-memory hash table that stores metadata about active, uncommitted writesets (collections of changes from database transactions). The cluster uses this index to quickly detect conflicts between transactions as they are replicated across the cluster. Each entry in the index represents a writeset that is currently in the certification process.

### Expected results

This value is a counter and should be greater than zero. The number of entries reflects your node's current workload. A consistently increasing value or a high value indicates a large number of concurrent transactions being processed, which is typical for a busy cluster. A high value that corresponds with performance issues may suggest that the applier threads cannot keep up with the incoming writeset rate. A value of zero means no transactions are in the certification phase.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cert_index_size';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-----------------------+-------+
    | Variable_name         | Value |
    +-----------------------+-------+
    | wsrep_cert_index_size | 2244  |
    +-----------------------+-------+
    ```

### Related status variables

When you analyze certification index utilization, consider these related status variables together:

#### [`wsrep_cert_bucket_count`](#wsrep_cert_bucket_count)

This variable shows the number of buckets in the certification index hash table. Compare this with `wsrep_cert_index_size` to understand the utilization ratio of your certification index. A high ratio suggests potential hash collisions and may indicate you need more buckets.

#### [`wsrep_local_cert_failures`](#wsrep_local_cert_failures)

This variable tracks the number of writesets that failed the certification test. High failure rates with a high `wsrep_cert_index_size` suggest that your certification index is overwhelmed and may need configuration adjustments.

#### [`wsrep_cert_deps_distance`](#wsrep_cert_deps_distance)

This variable measures the potential for parallelization based on transaction dependencies. A high `wsrep_cert_index_size` with a low `wsrep_cert_deps_distance` indicates that many transactions are conflicting, reducing parallelization potential.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable indicates the average length of the receive queue. High queue values with a high `wsrep_cert_index_size` suggest that your node cannot process incoming writesets fast enough, causing certification index buildup.

#### [`wsrep_apply_window`](#wsrep_apply_window)

This variable measures the parallelization window for applied transactions. A narrow window with a high `wsrep_cert_index_size` indicates that slow transaction processing is causing certification index accumulation.

### Monitoring certification index utilization

To get a comprehensive view of your cluster's certification index utilization, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cert_index_size|wsrep_cert_bucket_count|wsrep_local_cert_failures|wsrep_cert_deps_distance|wsrep_local_recv_queue_avg|wsrep_apply_window';
```

This query helps you identify whether certification index issues are due to:

* High transaction volume overwhelming the index

* Insufficient hash table capacity causing collisions

* Slow processing causing index buildup

* Transaction conflicts reducing parallelization

* Configuration mismatches with workload demands

### `wsrep_cert_interval`

#### What the variable measures

The `wsrep_cert_interval` status variable measures the average number of writesets (collections of changes from database transactions) your node receives while a local transaction is in the replication process. This value helps you understand your cluster's concurrency, specifically how many other transactions are active in the cluster while a transaction on your node is being prepared.

### Expected results

A value of `1.0` indicates a fully synchronous cluster where only one writeset is being replicated at a time. This is typical in low-concurrency environments. In a busy, highly concurrent cluster, the value will be greater than `1.0`. A higher value suggests a higher degree of concurrency.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cert_interval';
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------+-------+
    | Variable_name       | Value |
    +---------------------+-------+
    | wsrep_cert_interval | 2.50  |
    +---------------------+-------+
    ```

### Related status variables

When you analyze cluster concurrency patterns, consider these related status variables together:

#### [`wsrep_cert_index_size`](#wsrep_cert_index_size)

This variable shows the number of entries in the certification index. A high `wsrep_cert_interval` value with a high `wsrep_cert_index_size` indicates your cluster has high concurrency with many active transactions in the certification process.

#### [`wsrep_cert_bucket_count`](#wsrep_cert_bucket_count)

This variable shows the number of buckets in the certification index hash table. A high `wsrep_cert_interval` with a low bucket count suggests your certification index may be undersized for your cluster's concurrency level.

#### [`wsrep_local_commits`](#wsrep_local_commits)

This variable shows the number of writesets committed on your node. A high commit rate with a high `wsrep_cert_interval` indicates your cluster is processing many concurrent transactions efficiently.

#### [`wsrep_received`](#wsrep_received)

This variable tracks the total number of writesets received from other nodes. A high `wsrep_cert_interval` with a high `wsrep_received` value suggests your cluster has high inter-node communication and concurrency.

#### [`wsrep_cert_deps_distance`](#wsrep_cert_deps_distance)

This variable measures the potential for parallelization based on transaction dependencies. A high `wsrep_cert_interval` with a high `wsrep_cert_deps_distance` indicates your cluster is achieving good parallelization despite high concurrency.

### Monitoring cluster concurrency performance

To get a comprehensive view of your cluster's concurrency performance, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cert_interval|wsrep_cert_index_size|wsrep_cert_bucket_count|wsrep_local_commits|wsrep_received|wsrep_cert_deps_distance';
```

This query helps you identify whether concurrency-related performance issues are due to:

* High cluster concurrency overwhelming resources

* Insufficient certification index capacity for concurrency level

* Inefficient transaction processing under high load

* Network or replication bottlenecks limiting concurrency

* Configuration mismatches with cluster workload demands

### `wsrep_cluster_conf_id`

#### What the variable measures

The `wsrep_cluster_conf_id` status variable is a counter that tracks the number of cluster membership changes. This value increments every time a node joins or leaves the primary component of your cluster, either gracefully or due to a failure. The number is the same on all nodes in your cluster that belong to the same primary component.

### Expected results

This value should be consistent across all nodes in a healthy cluster. If your node's `wsrep_cluster_conf_id` value is different from the other nodes, your node is not part of the current primary component. This state often occurs after a network split or a node failure.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cluster_conf_id';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-----------------------+-------+
    | Variable_name         | Value |
    +-----------------------+-------+
    | wsrep_cluster_conf_id | 34    |
    +-----------------------+-------+
    ```

### Related status variables

When you analyze cluster membership stability, consider these related status variables together:

#### [`wsrep_cluster_size`](#wsrep_cluster_size)

This variable shows the number of nodes in the primary component. A changing `wsrep_cluster_conf_id` value with a decreasing `wsrep_cluster_size` indicates nodes are leaving your cluster, potentially due to failures or network issues.

#### [`wsrep_cluster_status`](#wsrep_cluster_status)

This variable shows the status of your cluster component. Monitor this alongside `wsrep_cluster_conf_id` to understand whether membership changes are due to normal operations or cluster issues.

#### [`wsrep_cluster_state_uuid`](#wsrep_cluster_state_uuid)

This variable contains the UUID state of your cluster. When this value changes along with `wsrep_cluster_conf_id`, it indicates a significant cluster state change.

#### [`wsrep_local_state_comment`](#wsrep_local_state_comment)

This variable shows your node's current state. A changing `wsrep_cluster_conf_id` with state changes in `wsrep_local_state_comment` helps you understand how your node is responding to cluster membership changes.

#### [`wsrep_connected`](#wsrep_connected)

This variable shows if your node is connected to the cluster. A changing `wsrep_cluster_conf_id` with `wsrep_connected` showing `OFF` indicates your node has lost connection to the cluster.

### Monitoring cluster membership stability

To get a comprehensive view of your cluster's membership stability, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cluster_conf_id|wsrep_cluster_size|wsrep_cluster_status|wsrep_cluster_state_uuid|wsrep_local_state_comment|wsrep_connected';
```

This query helps you identify whether cluster membership issues are due to:

* Network connectivity problems causing node disconnections

* Node failures requiring cluster reconfiguration

* Planned maintenance or node restarts

* Cluster quorum issues affecting membership

* Configuration problems preventing stable cluster formation

### `wsrep_cluster_size`

#### What the variable measures

The `wsrep_cluster_size` status variable shows the number of nodes that are currently part of the primary component of your cluster. The primary component is the group of nodes that have established a quorum and are capable of accepting writes. This value is a crucial health indicator for your cluster.

### Expected results

This value should be consistent across all nodes in a healthy cluster and match the total number of nodes you expect to be online. If this value is lower than the actual number of nodes, a network partition or node failure has likely occurred. The nodes that have a `wsrep_cluster_size` less than the expected number are in a non-primary state and cannot accept writes.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size';
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------------+-------+
    | Variable_name      | Value |
    +--------------------+-------+
    | wsrep_cluster_size | 3     |
    +--------------------+-------+
    ```

### Related status variables

When you analyze cluster quorum and membership, consider these related status variables together:

#### [`wsrep_cluster_conf_id`](#wsrep_cluster_conf_id)

This variable tracks the number of cluster membership changes. A decreasing `wsrep_cluster_size` with an increasing `wsrep_cluster_conf_id` indicates nodes are leaving your cluster, requiring membership reconfiguration.

#### [`wsrep_cluster_status`](#wsrep_cluster_status)

This variable shows the status of your cluster component. Monitor this alongside `wsrep_cluster_size` to understand whether your cluster is in a healthy primary state or experiencing issues.

#### [`wsrep_cluster_state_uuid`](#wsrep_cluster_state_uuid)

This variable contains the UUID state of your cluster. When this value changes along with `wsrep_cluster_size`, it indicates a significant cluster state change affecting quorum.

#### [`wsrep_local_state_comment`](#wsrep_local_state_comment)

This variable shows your node's current state. A decreasing `wsrep_cluster_size` with state changes in `wsrep_local_state_comment` helps you understand how your node is responding to cluster membership changes.

#### [`wsrep_connected`](#wsrep_connected)

This variable shows if your node is connected to the cluster. A decreasing `wsrep_cluster_size` with `wsrep_connected` showing `OFF` indicates your node has lost connection to the cluster.

### Monitoring cluster quorum health

To get a comprehensive view of your cluster's quorum health, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size|wsrep_cluster_conf_id|wsrep_cluster_status|wsrep_cluster_state_uuid|wsrep_local_state_comment|wsrep_connected';
```

This query helps you identify whether cluster quorum issues are due to:

* Network partitions causing node disconnections

* Node failures reducing cluster size

* Quorum loss preventing write operations

* Configuration problems affecting cluster formation

* Planned maintenance or node restarts

### `wsrep_cluster_state_uuid`

#### What the variable measures

The `wsrep_cluster_state_uuid` status variable contains the [UUID](glossary.md#uuid) state of your cluster. This value represents the current state identifier for your cluster and is used to track cluster state changes. When this value is the same as the one in [`wsrep_local_state_uuid`](#wsrep_local_state_uuid), your node is synced with the cluster.

### Expected results

This value should be consistent across all nodes in a healthy cluster. A changing `wsrep_cluster_state_uuid` value indicates that your cluster has undergone a significant state change, such as membership changes, quorum loss, or cluster reconfiguration. If your node's `wsrep_local_state_uuid` differs from `wsrep_cluster_state_uuid`, your node is not fully synced with the cluster.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cluster_state_uuid';
```

??? example "Expected output"

    ```{.text .no-copy}
    +------------------------+----------------------------------------+
    | Variable_name          | Value                                  |
    +------------------------+----------------------------------------+
    | wsrep_cluster_state_uuid | 550e8400-e29b-41d4-a716-446655440000 |
    +------------------------+----------------------------------------+
    ```

### Related status variables

When you analyze cluster state synchronization, consider these related status variables together:

#### [`wsrep_local_state_uuid`](#wsrep_local_state_uuid)

This variable contains the UUID state stored on your node. Compare this with `wsrep_cluster_state_uuid` to determine if your node is synced with the cluster. Different values indicate your node is not fully synchronized.

#### [`wsrep_cluster_conf_id`](#wsrep_cluster_conf_id)

This variable tracks cluster membership changes. A changing `wsrep_cluster_state_uuid` with an increasing `wsrep_cluster_conf_id` indicates that membership changes have triggered a cluster state change.

#### [`wsrep_cluster_size`](#wsrep_cluster_size)

This variable shows the number of nodes in the primary component. A changing `wsrep_cluster_state_uuid` with a decreasing `wsrep_cluster_size` suggests that node failures or network issues have caused a cluster state change.

#### [`wsrep_cluster_status`](#wsrep_cluster_status)

This variable shows the status of your cluster component. Monitor this alongside `wsrep_cluster_state_uuid` to understand whether state changes are due to normal operations or cluster issues.

#### [`wsrep_local_state_comment`](#wsrep_local_state_comment)

This variable shows your node's current state. A changing `wsrep_cluster_state_uuid` with state changes in `wsrep_local_state_comment` helps you understand how your node is responding to cluster state changes.

### Monitoring cluster state synchronization

To get a comprehensive view of your cluster's state synchronization, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cluster_state_uuid|wsrep_local_state_uuid|wsrep_cluster_conf_id|wsrep_cluster_size|wsrep_cluster_status|wsrep_local_state_comment';
```

This query helps you identify whether cluster state synchronization issues are due to:

* Node synchronization problems requiring resync

* Cluster membership changes affecting state

* Network partitions causing state divergence

* Node failures triggering state changes

* Configuration issues preventing proper synchronization

### `wsrep_cluster_status`

#### What the variable measures

The `wsrep_cluster_status` status variable shows the status of your cluster component. This variable indicates whether your cluster is in a healthy state and capable of accepting write operations. The cluster component status is a crucial indicator of your cluster's operational health.

### Expected results

This variable has three possible values:

- `Primary` - Your cluster is in a healthy state with quorum established and capable of accepting writes. This is the normal operational state.

- `Non-Primary` - Your cluster has lost quorum or is in a degraded state. Write operations are not accepted in this state, but the cluster may still be able to read data.

- `Disconnected` - Your cluster component is not connected to any cluster members. A `Disconnected` status indicates a complete loss of cluster connectivity.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cluster_status';
```

??? example "Expected output"

    ```{.text .no-copy}
    +----------------------+----------+
    | Variable_name        | Value    |
    +----------------------+----------+
    | wsrep_cluster_status | Primary  |
    +----------------------+----------+
    ```

### Related status variables

When you analyze cluster component status, consider these related status variables together:

#### [`wsrep_cluster_size`](#wsrep_cluster_size)

This variable shows the number of nodes in the primary component. A `Non-Primary` status with a decreasing `wsrep_cluster_size` indicates quorum loss due to node failures or network issues.

#### [`wsrep_cluster_conf_id`](#wsrep_cluster_conf_id)

This variable tracks cluster membership changes. A status change with an increasing `wsrep_cluster_conf_id` indicates that membership changes have affected your cluster's operational state.

#### [`wsrep_cluster_state_uuid`](#wsrep_cluster_state_uuid)

This variable contains the UUID state of your cluster. A status change with a changing `wsrep_cluster_state_uuid` indicates a significant cluster state change affecting operational status.

#### [`wsrep_connected`](#wsrep_connected)

This variable shows if your node is connected to the cluster. A `Disconnected` status with `wsrep_connected` showing `OFF` confirms that your node has lost cluster connectivity.

#### [`wsrep_local_state_comment`](#wsrep_local_state_comment)

This variable shows your node's current state. Monitor this alongside `wsrep_cluster_status` to understand how your node is responding to cluster status changes.

### Monitoring cluster component status

To get a comprehensive view of your cluster's component status, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_cluster_status|wsrep_cluster_size|wsrep_cluster_conf_id|wsrep_cluster_state_uuid|wsrep_connected|wsrep_local_state_comment';
```

This query helps you identify whether cluster status issues are due to:

* Network connectivity problems causing disconnections

* Node failures resulting in quorum loss

* Configuration issues preventing cluster formation

* Planned maintenance affecting cluster state

* Cluster reconfiguration events

### `wsrep_commit_oooe`

#### What the variable measures

The `wsrep_commit_oooe` status variable shows how often transactions are committed out of order. The acronym stands for "Out of Order of Execution" for commits. This variable measures the parallelization efficiency of the commit phase, where transactions can be applied in parallel but must be committed in sequence order.

### Expected results

A high value, close to `1`, indicates excellent commit parallelization. This means your cluster is efficiently committing transactions despite the requirement for sequential commit ordering. A value of `0` suggests no out-of-order commits, which may indicate limited parallelization in the commit phase. A value greater than `0.5` is generally a good sign for commit performance.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_commit_oooe';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------+----------+
    | Variable_name     | Value    |
    +-------------------+----------+
    | wsrep_commit_oooe | 0.875432 |
    +-------------------+----------+
    ```

### Related status variables

When you analyze commit parallelization performance, consider these related status variables together:

#### [`wsrep_apply_oooe`](#wsrep_apply_oooe)

This variable measures the parallelization efficiency of the apply phase. Compare this with `wsrep_commit_oooe` to understand whether bottlenecks occur in the apply phase or commit phase.

#### [`wsrep_commit_window`](#wsrep_commit_window)

This variable shows the average distance between concurrently committed sequence numbers. A wide commit window with a high `wsrep_commit_oooe` value indicates excellent commit parallelization.

#### [`wsrep_apply_window`](#wsrep_apply_window)

This variable measures the parallelization window for applied transactions. Compare this with `wsrep_commit_window` to see if the bottleneck is in applying or committing transactions.

#### [`wsrep_cert_deps_distance`](#wsrep_cert_deps_distance)

This variable shows the potential for parallelization based on transaction dependencies. A high value with a low `wsrep_commit_oooe` suggests that commit phase parallelization is limited despite available dependency distance.

#### [`wsrep_local_commits`](#wsrep_local_commits)

This variable shows the number of writesets committed on your node. A high commit rate with a low `wsrep_commit_oooe` suggests that commit phase parallelization could be improved.

### Monitoring commit parallelization performance

To get a comprehensive view of your cluster's commit parallelization performance, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_commit_oooe|wsrep_apply_oooe|wsrep_commit_window|wsrep_apply_window|wsrep_cert_deps_distance|wsrep_local_commits';
```

This query helps you identify whether commit parallelization issues are due to:

* Limited commit phase parallelization efficiency

* Transaction dependency constraints affecting commits

* Apply phase bottlenecks limiting commit performance

* Configuration issues affecting commit ordering

* Workload patterns that reduce commit parallelization

### `wsrep_commit_window`

#### What the variable measures

The `wsrep_commit_window` status variable measures the average distance between the highest and lowest concurrently committed sequence numbers. This variable shows the parallelization window for the commit phase, indicating how many transactions can be committed within the same time window despite the requirement for sequential commit ordering.

### Expected results

A higher value indicates better commit parallelization potential. This suggests your cluster is efficiently committing transactions across a wider range of sequence numbers. A low value suggests limited commit parallelization, which may indicate that transactions are being committed more sequentially. A value greater than `1.0` generally indicates good commit parallelization performance.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_commit_window';
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------+----------+
    | Variable_name       | Value    |
    +---------------------+----------+
    | wsrep_commit_window | 3.456789 |
    +---------------------+----------+
    ```

### Related status variables

When you analyze commit window performance, consider these related status variables together:

#### [`wsrep_commit_oooe`](#wsrep_commit_oooe)

This variable measures the parallelization efficiency of the commit phase. A wide `wsrep_commit_window` with a high `wsrep_commit_oooe` value indicates excellent commit parallelization.

#### [`wsrep_apply_window`](#wsrep_apply_window)

This variable shows the parallelization window for applied transactions. Compare this with `wsrep_commit_window` to see if bottlenecks occur in the apply phase or commit phase.

#### [`wsrep_apply_oooe`](#wsrep_apply_oooe)

This variable measures the parallelization efficiency of the apply phase. A narrow `wsrep_commit_window` with a high `wsrep_apply_oooe` suggests that the commit phase is the bottleneck.

#### [`wsrep_cert_deps_distance`](#wsrep_cert_deps_distance)

This variable shows the potential for parallelization based on transaction dependencies. A low `wsrep_commit_window` with a high `wsrep_cert_deps_distance` suggests that commit phase parallelization is limited despite available dependency distance.

#### [`wsrep_local_commits`](#wsrep_local_commits)

This variable shows the number of writesets committed on your node. A high commit rate with a narrow `wsrep_commit_window` suggests that commit phase parallelization could be improved.

### Monitoring commit window performance

To get a comprehensive view of your cluster's commit window performance, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_commit_window|wsrep_commit_oooe|wsrep_apply_window|wsrep_apply_oooe|wsrep_cert_deps_distance|wsrep_local_commits';
```

This query helps you identify whether commit window issues are due to:

* Limited commit phase parallelization efficiency

* Apply phase bottlenecks affecting commit performance

* Transaction dependency constraints limiting commits

* Configuration issues affecting commit ordering

* Workload patterns that reduce commit parallelization

### `wsrep_connected`

#### What the variable measures

The `wsrep_connected` status variable shows if your node is connected to the cluster. This variable is a simple binary indicator that tells you whether your node has established a connection to any cluster components. This is a fundamental connectivity status that affects all cluster operations.

### Expected results

This variable has two possible values:

- `ON` - Your node is connected to at least one cluster component and can participate in cluster operations. This is the normal operational state.

- `OFF` - Your node is not connected to any cluster components. This indicates a connectivity issue that may be due to network problems, misconfiguration, or cluster component unavailability.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_connected';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-----------------+-------+
    | Variable_name   | Value |
    +-----------------+-------+
    | wsrep_connected | ON    |
    +-----------------+-------+
    ```

### Related status variables

When you analyze node connectivity, consider these related status variables together:

#### [`wsrep_cluster_status`](#wsrep_cluster_status)

This variable shows the status of your cluster component. A `Disconnected` status with `wsrep_connected` showing `OFF` confirms that your node has lost cluster connectivity.

#### [`wsrep_cluster_size`](#wsrep_cluster_size)

This variable shows the number of nodes in the primary component. A `wsrep_connected` value of `OFF` with a decreasing `wsrep_cluster_size` suggests that connectivity issues are affecting cluster membership.

#### [`wsrep_cluster_conf_id`](#wsrep_cluster_conf_id)

This variable tracks cluster membership changes. A `wsrep_connected` value of `OFF` with an increasing `wsrep_cluster_conf_id` indicates that connectivity loss has triggered membership reconfiguration.

#### [`wsrep_local_state_comment`](#wsrep_local_state_comment)

This variable shows your node's current state. A `wsrep_connected` value of `OFF` with state changes in `wsrep_local_state_comment` helps you understand how your node is responding to connectivity loss.

#### [`wsrep_ready`](#wsrep_ready)

This variable shows if your node is ready to accept queries. A `wsrep_connected` value of `OFF` will typically result in `wsrep_ready` being `OFF` as well.

### Monitoring node connectivity

To get a comprehensive view of your node's connectivity status, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_connected|wsrep_cluster_status|wsrep_cluster_size|wsrep_cluster_conf_id|wsrep_local_state_comment|wsrep_ready';
```

This query helps you identify whether connectivity issues are due to:

* Network connectivity problems preventing cluster access

* Cluster component unavailability or failures

* Configuration issues affecting cluster connection

* Node startup or shutdown processes

* Cluster membership changes affecting connectivity

### `wsrep_evs_delayed`

#### What the variable measures

The `wsrep_evs_delayed` status variable shows a list of nodes that are running slowly in your cluster. Think of it like a traffic report that tells you which cars are falling behind on the highway. A "delayed" node is one that cannot keep up with the speed of data changes happening in the cluster. It's falling behind in processing and applying database changes from other nodes. This variable is part of the EVS (Extended Virtual Synchrony) protocol. It helps you identify nodes that are experiencing performance problems. The node format is `<uuid>:<address>:<count>`. The `<count>` shows how many times this node has been marked as delayed.

### Expected results

In a healthy cluster, this variable should typically be empty or show very low counts. A node appearing in this list with a high count indicates that the node is experiencing replication delays or performance issues. The presence of delayed nodes can affect cluster performance. It may indicate network problems, resource constraints, or node-specific issues.

#### What the values mean

| Value            | Meaning                    | Action Needed                                                                                                                                                                                |
| ---------------- | -------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Empty            | No delayed nodes           | Cluster is healthy                                                                                                                                                                           |
| Low count (1-5)  | Minor delays               | Monitor the situation: Check the delayed node's system resources (CPU, memory, disk I/O) and network connectivity. Review recent workload patterns and consider if the delays are temporary. |
| High count (>10) | Serious performance issues | Investigate immediately: Check the delayed node's error logs, system resources, and network connectivity. Consider restarting the node or reducing workload. Monitor for potential eviction. |

#### Real-world examples:

Example 1: Healthy Cluster

```text
wsrep_evs_delayed: (empty)
```

This means all nodes are keeping up with the cluster.

Example 2: Minor Issues

```text
wsrep_evs_delayed: 550e8400-e29b-41d4-a716-446655440000:192.168.1.10:3306:2
```

This shows one node has been delayed 2 times. Monitor this node.

Example 3: Serious Problems

```text
wsrep_evs_delayed: 550e8400-e29b-41d4-a716-446655440000:192.168.1.10:3306:15
```

This node has been delayed 15 times. This needs immediate attention.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_evs_delayed';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------+----------------------------------------------------------+
    | Variable_name     | Value                                                    |
    +-------------------+----------------------------------------------------------+
    | wsrep_evs_delayed | 550e8400-e29b-41d4-a716-446655440000:192.168.1.10:3306:5 |
    +-------------------+----------------------------------------------------------+
    ```

### Background: Understanding EVS Protocol

The EVS (Extended Virtual Synchrony) protocol is like a traffic control system for your database cluster. It ensures all nodes work together smoothly and identifies when some nodes are having trouble keeping up. Think of it as a supervisor that watches all workers and reports when someone is falling behind.

### Related status variables

When you analyze node delay patterns, consider these related status variables together:

#### [`wsrep_evs_evict_list`](#wsrep_evs_evict_list)

This variable shows the list of evicted nodes. A node appearing in both `wsrep_evs_delayed` and `wsrep_evs_evict_list` indicates severe performance issues that may lead to eviction.

#### [`wsrep_evs_repl_latency`](#wsrep_evs_repl_latency)

This variable shows replication latency statistics. High latency values with nodes in `wsrep_evs_delayed` suggest network or performance issues affecting replication.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable shows the average length of the receive queue. High queue values with delayed nodes suggest that replication processing is backing up.

#### [`wsrep_cluster_size`](#wsrep_cluster_size)

This variable shows the number of nodes in the primary component. A decreasing cluster size with delayed nodes may indicate that delayed nodes are being evicted.

#### [`wsrep_evs_state`](#wsrep_evs_state)

This variable shows the internal EVS protocol state. Monitor this alongside `wsrep_evs_delayed` to understand how the EVS protocol is responding to node delays.

### Monitoring node delay patterns

To get a comprehensive view of your cluster's node delay patterns, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_evs_delayed|wsrep_evs_evict_list|wsrep_evs_repl_latency|wsrep_local_recv_queue_avg|wsrep_cluster_size|wsrep_evs_state';
```

This query helps you identify whether node delay issues are due to:

* Network connectivity problems causing replication delays

* Node resource constraints affecting performance

* Replication processing bottlenecks

* EVS protocol eviction events

* Cluster configuration issues affecting node performance

### `wsrep_evs_evict_list`

#### What the variable measures

The `wsrep_evs_evict_list` status variable shows a list of nodes that have been kicked out of your cluster. Think of it like a list of employees who were fired for not doing their job properly. This variable is part of the EVS (Extended Virtual Synchrony) protocol. It shows nodes that have been removed from the cluster because they had performance problems, network issues, or other failures. These failures made it impossible for them to stay in sync with the rest of the cluster.

### Expected results

In a healthy cluster, this variable should typically be empty. The presence of UUIDs in this list indicates that nodes have been evicted from your cluster. Evicted nodes are no longer part of the cluster and cannot participate in replication or accept client connections. You should investigate the reasons for eviction and take appropriate action to restore cluster health.

#### What the values mean

| Value          | Meaning                    | Action Needed                                                                                                                                                                                                                 |
| -------------- | -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Empty          | No evicted nodes           | Cluster is healthy                                                                                                                                                                                                            |
| Contains UUIDs | Nodes have been kicked out | Investigate and fix issues: Check the evicted node's error logs, system resources, and network connectivity. Determine why the node was evicted and fix the underlying problem before attempting to rejoin it to the cluster. |

#### Real-world examples:

Example 1: Healthy Cluster

```text
wsrep_evs_evict_list: (empty)
```

This means no nodes have been kicked out of the cluster.

Example 2: Problem Cluster

```text
wsrep_evs_evict_list: 550e8400-e29b-41d4-a716-446655440000
```

This shows one node has been kicked out. You need to find out why and fix the problem.

Example 3: Multiple Problems

```text
wsrep_evs_evict_list: 550e8400-e29b-41d4-a716-446655440000, 660e8400-e29b-41d4-a716-446655440001
```

This shows two nodes have been kicked out. This indicates serious cluster problems.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_evs_evict_list';
```

??? example "Expected output"

    ```{.text .no-copy}
    +----------------------+----------------------------------------+
    | Variable_name        | Value                                  |
    +----------------------+----------------------------------------+
    | wsrep_evs_evict_list | 550e8400-e29b-41d4-a716-446655440000   |
    +----------------------+----------------------------------------+
    ```

### Related status variables

When you analyze node eviction patterns, consider these related status variables together:

#### [`wsrep_evs_delayed`](#wsrep_evs_delayed)

This variable shows nodes that are considered delayed. A node appearing in both `wsrep_evs_delayed` and `wsrep_evs_evict_list` indicates that the node was evicted after experiencing severe performance issues.

#### [`wsrep_cluster_size`](#wsrep_cluster_size)

This variable shows the number of nodes in the primary component. A decreasing cluster size with evicted nodes confirms that evictions have reduced your cluster's membership.

#### [`wsrep_cluster_conf_id`](#wsrep_cluster_conf_id)

This variable tracks cluster membership changes. An increasing `wsrep_cluster_conf_id` with evicted nodes indicates that evictions have triggered cluster reconfiguration.

#### [`wsrep_evs_repl_latency`](#wsrep_evs_repl_latency)

This variable shows replication latency statistics. High latency values with evicted nodes suggest that network or performance issues led to the evictions.

#### [`wsrep_evs_state`](#wsrep_evs_state)

This variable shows the internal EVS protocol state. Monitor this alongside `wsrep_evs_evict_list` to understand how the EVS protocol is managing cluster membership.

### Monitoring node eviction patterns

To get a comprehensive view of your cluster's node eviction patterns, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_evs_evict_list|wsrep_evs_delayed|wsrep_cluster_size|wsrep_cluster_conf_id|wsrep_evs_repl_latency|wsrep_evs_state';
```

This query helps you identify whether node eviction issues are due to:

* Network connectivity problems causing node failures

* Node resource constraints leading to performance issues

* Replication latency problems affecting cluster synchronization

* EVS protocol decisions to maintain cluster stability

* Configuration issues affecting node performance

### `wsrep_evs_repl_latency`

#### What the variable measures

The `wsrep_evs_repl_latency` status variable provides information regarding group communication replication latency in your cluster. This variable measures the time it takes for replication messages to travel between nodes in your cluster. The latency is measured in seconds from when a message is sent out to when a message is received, and it helps you understand the network performance characteristics of your cluster.

### Expected results

This variable provides latency statistics in the format `<min>/<avg>/<max>/<std_dev>/<sample_size>`. Lower values indicate better network performance and faster replication. High latency values may indicate network congestion, bandwidth limitations, or connectivity issues between nodes. You should monitor this variable to ensure your cluster's replication performance meets your application requirements.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_evs_repl_latency';
```

??? example "Expected output"

    ```{.text .no-copy}
    +------------------------+------------------------------+
    | Variable_name          | Value                        |
    +------------------------+------------------------------+
    | wsrep_evs_repl_latency | 0.001/0.005/0.020/0.003/1000 |
    +------------------------+------------------------------+
    ```

### Related status variables

When you analyze replication latency performance, consider these related status variables together:

#### [`wsrep_evs_delayed`](#wsrep_evs_delayed)

This variable shows nodes that are considered delayed. High latency values with delayed nodes suggest that network performance issues are affecting cluster synchronization.

#### [`wsrep_evs_evict_list`](#wsrep_evs_evict_list)

This variable shows evicted nodes. High latency values with evicted nodes suggest that network issues may have contributed to the evictions.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable shows the average length of the receive queue. High latency with high queue values suggests that replication processing is backing up due to network delays.

#### [`wsrep_local_send_queue_avg`](#wsrep_local_send_queue_avg)

This variable shows the average length of the send queue. High latency with high send queue values suggests that network issues are preventing efficient message transmission.

#### [`wsrep_evs_state`](#wsrep_evs_state)

This variable shows the internal EVS protocol state. Monitor this alongside `wsrep_evs_repl_latency` to understand how the EVS protocol is responding to latency issues.

### Monitoring replication latency performance

To get a comprehensive view of your cluster's replication latency performance, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_evs_repl_latency|wsrep_evs_delayed|wsrep_evs_evict_list|wsrep_local_recv_queue_avg|wsrep_local_send_queue_avg|wsrep_evs_state';
```

This query helps you identify whether replication latency issues are due to:

* Network connectivity problems affecting message transmission

* Bandwidth limitations causing message delays

* Node resource constraints affecting processing speed

* EVS protocol state changes affecting latency

* Configuration issues impacting network performance

### `wsrep_evs_state`

#### What the variable measures

The `wsrep_evs_state` status variable shows the internal Extended Virtual Synchrony (EVS) protocol state of your node. This variable provides insight into the current state of the EVS protocol. The EVS protocol manages cluster membership, message ordering, and failure detection in your cluster. The EVS protocol state indicates how your node is participating in the cluster's group communication system.

### Expected results

This variable shows internal EVS protocol states that are typically used for debugging and advanced troubleshooting. The specific values and their meanings are internal to the EVS protocol implementation. In normal operation, you should see consistent state values across your cluster nodes. Significant changes in this variable may indicate EVS protocol state transitions. These transitions can be due to cluster membership changes, network issues, or protocol reconfiguration events.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_evs_state';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-----------------+--------+
    | Variable_name   | Value  |
    +-----------------+--------+
    | wsrep_evs_state | 4      |
    +-----------------+--------+
    ```

### Related status variables

When you analyze EVS protocol state, consider these related status variables together:

#### [`wsrep_evs_delayed`](#wsrep_evs_delayed)

This variable shows nodes that are considered delayed. EVS state changes with delayed nodes may indicate protocol responses to performance issues.

#### [`wsrep_evs_evict_list`](#wsrep_evs_evict_list)

This variable shows evicted nodes. EVS state changes with evicted nodes may indicate protocol decisions to maintain cluster stability.

#### [`wsrep_evs_repl_latency`](#wsrep_evs_repl_latency)

This variable shows replication latency statistics. EVS state changes with high latency may indicate protocol responses to network issues.

#### [`wsrep_cluster_conf_id`](#wsrep_cluster_conf_id)

This variable tracks cluster membership changes. EVS state changes with membership changes may indicate protocol state transitions.

#### [`wsrep_cluster_status`](#wsrep_cluster_status)

This variable shows the cluster component status. EVS state changes with status changes may indicate protocol responses to cluster events.

### Monitoring EVS protocol state

To get a comprehensive view of your cluster's EVS protocol state, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_evs_state|wsrep_evs_delayed|wsrep_evs_evict_list|wsrep_evs_repl_latency|wsrep_cluster_conf_id|wsrep_cluster_status';
```

This query helps you identify whether EVS protocol state changes are due to:

* Cluster membership changes affecting protocol state

* Network issues triggering protocol responses

* Node performance problems causing protocol decisions

* Protocol reconfiguration events

* Cluster stability maintenance actions

### `wsrep_flow_control_interval`

#### What the variable measures

The `wsrep_flow_control_interval` status variable shows the safety limits for your cluster's data processing. Think of it like a traffic light system that controls how much data can flow through your cluster at once. Flow control is a safety mechanism that prevents your cluster from being overwhelmed by too many database changes. The upper limit is the maximum number of requests allowed in the queue, and the lower limit is the point where normal processing starts again.

#### Expected results

This variable shows the flow control limits in the format `<lower_limit>/<upper_limit>`. When the queue reaches the upper limit, new requests are denied to prevent cluster overload. As existing requests get processed, the queue decreases, and once it reaches the lower limit, new requests are allowed again. You should monitor this variable to understand your cluster's flow control behavior and ensure the limits are appropriate for your workload.

#### What the values mean

| Format         | Meaning                           | Example  | Action                                                                                         |
| -------------- | --------------------------------- | -------- | ---------------------------------------------------------------------------------------------- |
| `<low>/<high>` | Safety limits for data processing | `16/32`  | Monitor cluster load and adjust if needed                                                      |
| Low values     | Conservative limits               | `8/16`   | May cause frequent pauses: Consider increasing limits if your cluster has sufficient resources |
| High values    | Aggressive limits                 | `64/128` | May allow overload: Monitor for performance issues and consider reducing limits if needed      |

#### Real-world examples:

Example 1: Normal Limits

```text
wsrep_flow_control_interval: 16/32
```

This means flow control starts when 32 requests are waiting and stops when 16 remain.

Example 2: Conservative Limits

```text
wsrep_flow_control_interval: 8/16
```

This means flow control activates sooner, which is safer but may slow down processing.

Example 3: Aggressive Limits

```text
wsrep_flow_control_interval: 64/128
```

This means flow control activates later, which is faster but may allow overload.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_interval';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-----------------------------+--------------+
    | Variable_name               | Value        |
    +-----------------------------+--------------+
    | wsrep_flow_control_interval | 16/32        |
    +-----------------------------+--------------+
    ```

### Background: Understanding Flow Control

Flow control is like a traffic management system for your database cluster. When too many cars (database requests) try to enter a road at once, traffic lights (flow control) temporarily stop new cars from entering. This prevents traffic jams and keeps everything moving smoothly. In database terms, when too many database changes arrive at once, flow control temporarily stops new changes until the cluster can catch up.

### Related status variables

When you analyze flow control behavior, consider these related status variables together:

#### [`wsrep_flow_control_interval_high`](#wsrep_flow_control_interval_high)

This variable shows the upper limit for flow control to trigger. Compare this with the upper limit in `wsrep_flow_control_interval` to verify your flow control configuration.

#### [`wsrep_flow_control_interval_low`](#wsrep_flow_control_interval_low)

This variable shows the lower limit for flow control to stop. Compare this with the lower limit in `wsrep_flow_control_interval` to verify your flow control configuration.

#### [`wsrep_flow_control_paused`](#wsrep_flow_control_paused)

This variable shows the time since the last status query that was paused due to flow control. High pause times with flow control limits suggest that your cluster is experiencing high load.

#### [`wsrep_flow_control_status`](#wsrep_flow_control_status)

This variable shows whether flow control is enabled for normal traffic. Monitor this alongside `wsrep_flow_control_interval` to understand your cluster's flow control state.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable shows the average length of the receive queue. High queue values with flow control limits suggest that your cluster is processing requests at capacity.

### Monitoring flow control behavior

To get a comprehensive view of your cluster's flow control behavior, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_interval|wsrep_flow_control_interval_high|wsrep_flow_control_interval_low|wsrep_flow_control_paused|wsrep_flow_control_status|wsrep_local_recv_queue_avg';
```

This query helps you identify whether flow control issues are due to:

* High cluster load exceeding processing capacity

* Inappropriate flow control limit configuration

* Network or replication bottlenecks causing queue buildup

* Node resource constraints affecting processing speed

* Configuration issues impacting flow control behavior

### `wsrep_flow_control_interval_high`

#### What the variable measures

The `wsrep_flow_control_interval_high` status variable shows the upper limit for flow control to trigger in your cluster. This is the threshold at which the flow control activates to prevent your cluster from being overwhelmed by too many replication requests. When the queue reaches this limit, new requests are denied until the queue size decreases.

### Expected results

This value should match the upper limit shown in `wsrep_flow_control_interval`. A higher value allows more requests to queue before flow control activates, while a lower value triggers flow control more aggressively. You should configure this value based on your cluster's processing capacity and workload characteristics.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_interval_high';
```

??? example "Expected output"

    ```{.text .no-copy}
    +----------------------------------+-------+
    | Variable_name                    | Value |
    +----------------------------------+-------+
    | wsrep_flow_control_interval_high | 32    |
    +----------------------------------+-------+
    ```

### Related status variables

When you analyze flow control upper limits, consider these related status variables together:

#### [`wsrep_flow_control_interval`](#wsrep_flow_control_interval)

This variable shows both the lower and upper flow control limits. Compare the upper limit with `wsrep_flow_control_interval_high` to verify your configuration.

#### [`wsrep_flow_control_interval_low`](#wsrep_flow_control_interval_low)

This variable shows the lower limit for flow control. Monitor this alongside the upper limit to understand your flow control range.

#### [`wsrep_flow_control_paused`](#wsrep_flow_control_paused)

This variable shows pause times due to flow control. High pause times with a low upper limit suggest that your limit may be too restrictive.

#### [`wsrep_flow_control_status`](#wsrep_flow_control_status)

This variable shows whether flow control is enabled. Monitor this to understand when flow control is active.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable shows the average receive queue length. High queue values approaching the upper limit suggest that flow control may activate soon.

### `wsrep_flow_control_interval_low`

#### What the variable measures

The `wsrep_flow_control_interval_low` status variable shows the lower limit for flow control to stop in your cluster. This is the threshold at which the flow control deactivates and allows new requests to be processed again. When the queue size decreases to this limit, normal processing resumes.

### Expected results

This value should match the lower limit shown in `wsrep_flow_control_interval`. A higher value means flow control stops sooner, allowing more requests to be processed, while a lower value means flow control remains active longer. You should configure this value to provide adequate buffer for your cluster's processing capacity.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_interval_low';
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------------------+-------+
    | Variable_name                   | Value |
    +---------------------------------+-------+
    | wsrep_flow_control_interval_low | 16    |
    +---------------------------------+-------+
    ```

### Related status variables

When you analyze flow control lower limits, consider these related status variables together:

#### [`wsrep_flow_control_interval`](#wsrep_flow_control_interval)

This variable shows both the lower and upper flow control limits. Compare the lower limit with `wsrep_flow_control_interval_low` to verify your configuration.

#### [`wsrep_flow_control_interval_high`](#wsrep_flow_control_interval_high)

This variable shows the upper limit for flow control. Monitor this alongside the lower limit to understand your flow control range.

#### [`wsrep_flow_control_paused`](#wsrep_flow_control_paused)

This variable shows pause times due to flow control. High pause times with a high lower limit suggest that your limit may be too restrictive.

#### [`wsrep_flow_control_status`](#wsrep_flow_control_status)

This variable shows whether flow control is enabled. Monitor this to understand when flow control is active.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable shows the average receive queue length. Queue values near the lower limit suggest that flow control may deactivate soon.

### Monitoring flow control limits

To get a comprehensive view of your cluster's flow control limits, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_interval_high|wsrep_flow_control_interval_low|wsrep_flow_control_interval|wsrep_flow_control_paused|wsrep_flow_control_status|wsrep_local_recv_queue_avg';
```

This query helps you identify whether flow control limit issues are due to:

* Inappropriate upper limit configuration causing frequent flow control activation

* Inappropriate lower limit configuration preventing flow control deactivation

* High cluster load requiring limit adjustments

* Configuration mismatches between related flow control variables

* Queue processing bottlenecks affecting flow control behavior

### `wsrep_flow_control_paused`

#### What the variable measures

The `wsrep_flow_control_paused` status variable shows the time since the last status query that was paused due to flow control in your cluster. This variable helps you understand how recently your cluster experienced flow control activation. It also shows how long the pause lasted. Flow control pauses occur when the replication queue reaches the upper limit. New requests are temporarily denied during these pauses.

### Expected results

In a healthy cluster, this value should typically be low or zero. This indicates that flow control is not frequently activated. A high value or frequently changing values suggest that your cluster is experiencing high load or processing bottlenecks. These bottlenecks trigger flow control. You should monitor this variable to understand your cluster's flow control behavior. It helps you identify potential performance issues.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_paused';
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------------+-------+
    | Variable_name             | Value |
    +---------------------------+-------+
    | wsrep_flow_control_paused | 0.5   |
    +---------------------------+-------+
    ```

### Related status variables

When you analyze flow control pause patterns, consider these related status variables together:

#### [`wsrep_flow_control_interval`](#wsrep_flow_control_interval)

This variable shows the flow control limits. High pause times with flow control limits suggest that your cluster is experiencing high load.

#### [`wsrep_flow_control_interval_high`](#wsrep_flow_control_interval_high)

This variable shows the upper limit for flow control. Frequent pauses with a low upper limit suggest that your limit may be too restrictive.

#### [`wsrep_flow_control_interval_low`](#wsrep_flow_control_interval_low)

This variable shows the lower limit for flow control. Long pauses with a high lower limit suggest that your limit may be too restrictive.

#### [`wsrep_flow_control_status`](#wsrep_flow_control_status)

This variable shows whether flow control is enabled. Monitor this alongside pause times to understand your cluster's flow control state.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable shows the average length of the receive queue. High queue values with pause times suggest that replication processing is backing up.

### Monitoring flow control pause patterns

To get a comprehensive view of your cluster's flow control pause patterns, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_paused|wsrep_flow_control_interval|wsrep_flow_control_interval_high|wsrep_flow_control_interval_low|wsrep_flow_control_status|wsrep_local_recv_queue_avg';
```

This query helps you identify whether flow control pause issues are due to:

* High cluster load exceeding processing capacity

* Inappropriate flow control limit configuration

* Network or replication bottlenecks causing queue buildup

* Node resource constraints affecting processing speed

* Configuration issues impacting flow control behavior

### `wsrep_flow_control_paused_ns`

#### What the variable measures

The `wsrep_flow_control_paused_ns` status variable shows the total time spent in a paused state due to flow control, measured in nanoseconds. This variable provides a high-precision measurement of how much time your cluster has spent in flow control pauses. Unlike `wsrep_flow_control_paused`, which shows the time since the last pause, this variable accumulates the total pause time over the server's lifetime.

### Expected results

In a healthy cluster, this value should be relatively low, indicating minimal flow control pauses. A high value suggests that your cluster has experienced significant flow control activity, which may indicate performance issues, high load, or inappropriate flow control configuration. You should monitor this variable to understand the cumulative impact of flow control on your cluster's performance.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_paused_ns';
```

??? example "Expected output"

    ```{.text .no-copy}
    +------------------------------+------------+
    | Variable_name                | Value      |
    +------------------------------+------------+
    | wsrep_flow_control_paused_ns | 1500000000 |
    +------------------------------+------------+
    ```

### Related status variables

When you analyze cumulative flow control pause time, consider these related status variables together:

#### [`wsrep_flow_control_paused`](#wsrep_flow_control_paused)

This variable shows the time since the last pause. Compare this with the cumulative pause time to understand recent vs. total flow control activity.

#### [`wsrep_flow_control_interval`](#wsrep_flow_control_interval)

This variable shows the flow control limits. High cumulative pause times with flow control limits suggest that your cluster frequently experiences high load.

#### [`wsrep_flow_control_recv`](#wsrep_flow_control_recv)

This variable shows the number of flow control pause events received. High pause counts with high cumulative pause time suggest frequent flow control activation.

#### [`wsrep_flow_control_sent`](#wsrep_flow_control_sent)

This variable shows the number of flow control pause events sent. High sent counts with high cumulative pause time suggest that your node frequently requests flow control.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable shows the average length of the receive queue. High queue values with high cumulative pause time suggest that replication processing bottlenecks contribute to flow control.

### Monitoring cumulative flow control impact

To get a comprehensive view of your cluster's cumulative flow control impact, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_paused_ns|wsrep_flow_control_paused|wsrep_flow_control_interval|wsrep_flow_control_recv|wsrep_flow_control_sent|wsrep_local_recv_queue_avg';
```

This query helps you identify whether cumulative flow control issues are due to:

* Frequent high load periods causing repeated flow control activation

* Inappropriate flow control limit configuration leading to excessive pauses

* Network or replication bottlenecks causing persistent queue buildup

* Node resource constraints affecting processing speed over time

* Configuration issues resulting in excessive flow control activity

### `wsrep_flow_control_recv`

#### What the variable measures

The `wsrep_flow_control_recv` status variable shows the number of `FC_PAUSE` events received since the last status query. This variable tracks how many times your node has received flow control pause requests from other nodes. Unlike most status variables, this counter does not reset each time you run the query. It only resets when the server restarts.

### Expected results

In a healthy cluster, this value should be relatively low. This indicates that flow control pause requests are infrequent. A high value suggests that your cluster is experiencing frequent flow control activity. This may indicate performance issues, high load, or inappropriate flow control configuration. You should monitor this variable to understand how often your node receives flow control requests from other cluster members.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_recv';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------------+-------+
    | Variable_name           | Value |
    +-------------------------+-------+
    | wsrep_flow_control_recv | 15    |
    +-------------------------+-------+
    ```

### Related status variables

When you analyze flow control pause events received, consider these related status variables together:

#### [`wsrep_flow_control_sent`](#wsrep_flow_control_sent)

This variable shows the number of flow control pause events sent. Compare this with received events to understand flow control activity patterns in your cluster.

#### [`wsrep_flow_control_paused`](#wsrep_flow_control_paused)

This variable shows the time since the last pause. High received counts with recent pause times suggest active flow control in your cluster.

#### [`wsrep_flow_control_paused_ns`](#wsrep_flow_control_paused_ns)

This variable shows the total time spent in paused state. High received counts with high cumulative pause time suggest frequent flow control activity.

#### [`wsrep_flow_control_interval`](#wsrep_flow_control_interval)

This variable shows the flow control limits. High received counts with flow control limits suggest that your cluster frequently reaches flow control thresholds.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable shows the average length of the receive queue. High queue values with high received counts suggest that replication processing bottlenecks contribute to flow control.

### Monitoring flow control pause events received

To get a comprehensive view of your cluster's flow control pause events received, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_recv|wsrep_flow_control_sent|wsrep_flow_control_paused|wsrep_flow_control_paused_ns|wsrep_flow_control_interval|wsrep_local_recv_queue_avg';
```

This query helps you identify whether flow control pause event issues are due to:

* Frequent high load periods causing repeated flow control requests

* Network or replication bottlenecks leading to queue buildup

* Inappropriate flow control limit configuration causing excessive pauses

* Node resource constraints affecting processing speed

* Cluster-wide performance issues requiring flow control activation

### `wsrep_flow_control_requested`

#### What the variable measures

The `wsrep_flow_control_requested` status variable shows whether your node has asked other nodes to slow down their data sending. Think of it like a worker asking their colleagues to stop sending more work because they're already too busy. This variable indicates when your node's work queue has reached the safety limit. Your node then asks other cluster members to pause their data sending activity. This prevents your node from being overwhelmed with too much work.

### Expected results

This variable typically returns `0` (no pause requested) or `1` (pause requested). In a healthy cluster, this value should usually be `0`. This indicates that your node is keeping up with replication and not requesting pauses. A value of `1` suggests that your node is experiencing high load or processing bottlenecks. These bottlenecks require flow control intervention. You should monitor this variable to understand when your node is requesting flow control assistance.

#### What the values mean

| Value | Meaning            | Status             | Action Needed                                                                                                                                                                        |
| ----- | ------------------ | ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `0`   | No pause requested | Normal operation   | Continue monitoring for changes                                                                                                                                                      |
| `1`   | Pause requested    | Node is overloaded | Investigate performance: Check system resources (CPU, memory, disk I/O), review replication queue length, and consider adjusting flow control limits or adding more applier threads. |

#### Real-world examples:

Example 1: Normal Operation

```text
wsrep_flow_control_requested: 0
```

Your node is handling the workload normally.

Example 2: Overloaded Node

```text
wsrep_flow_control_requested: 1
```

Your node is struggling and has asked other nodes to slow down.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_requested';
```

??? example "Expected output"

    ```{.text .no-copy}
    +------------------------------+-------+
    | Variable_name                | Value |
    +------------------------------+-------+
    | wsrep_flow_control_requested | 0     |
    +------------------------------+-------+
    ```

### Related status variables

When you analyze flow control requests, consider these related status variables together:

#### [`wsrep_flow_control_sent`](#wsrep_flow_control_sent)

This variable shows the number of flow control pause events sent. A request value of `1` with increasing sent counts indicates active flow control requests.

#### [`wsrep_flow_control_recv`](#wsrep_flow_control_recv)

This variable shows the number of flow control pause events received. Compare this with requests to understand flow control activity patterns.

#### [`wsrep_flow_control_interval`](#wsrep_flow_control_interval)

This variable shows the flow control limits. A request value of `1` with flow control limits suggests that your node has reached the flow control threshold.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable shows the average length of the receive queue. A request value of `1` with high queue values suggests that replication processing bottlenecks are causing flow control requests.

#### [`wsrep_flow_control_status`](#wsrep_flow_control_status)

This variable shows whether flow control is enabled. A request value of `1` with enabled status confirms that flow control is active.

### Monitoring flow control requests

To get a comprehensive view of your cluster's flow control requests, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_requested|wsrep_flow_control_sent|wsrep_flow_control_recv|wsrep_flow_control_interval|wsrep_local_recv_queue_avg|wsrep_flow_control_status';
```

This query helps you identify whether flow control request issues are due to:

* High replication load exceeding your node's processing capacity

* Network or replication bottlenecks causing queue buildup

* Inappropriate flow control limit configuration

* Node resource constraints affecting processing speed

* Cluster-wide performance issues requiring flow control intervention

### `wsrep_flow_control_sent`

#### What the variable measures

The `wsrep_flow_control_sent` status variable counts how many "slow down" messages your node has sent to other nodes. Think of it like counting how many times you've asked your colleagues to stop sending you work because you're too busy. This variable tracks the number of `FC_PAUSE` events (flow control pause events) that your node has sent since the server started. Unlike most status variables, this counter does not reset each time you run the query. It only resets when the server restarts.

### Expected results

In a healthy cluster, this value should be low or zero. A high value indicates that your node has frequently asked other nodes to slow down their data sending. This suggests that your node is struggling to keep up with the replication workload. You should monitor this variable to understand how often your node needs flow control assistance.

#### What the values mean

| Value Range | Meaning                        | Status                       | Action Needed                                                                                                                                                       |
| ----------- | ------------------------------ | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0-10        | Rare flow control requests     | Normal operation             | Continue monitoring for changes                                                                                                                                     |
| 11-50       | Moderate flow control activity | Some performance issues      | Investigate workload: Check system resources, review replication patterns, and consider optimizing queries or adjusting flow control settings.                      |
| 50+         | Frequent flow control requests | Serious performance problems | Immediate attention needed: Check system resources, review error logs, consider reducing workload, and investigate potential bottlenecks in replication processing. |

#### Real-world examples:

Example 1: Healthy Node

```
wsrep_flow_control_sent: 3
```

Your node has only asked others to slow down 3 times since startup.

Example 2: Struggling Node

```
wsrep_flow_control_sent: 45
```

Your node has asked others to slow down 45 times, indicating performance problems.

Example 3: Very Problematic Node

```
wsrep_flow_control_sent: 127
```

Your node is constantly asking others to slow down, indicating serious issues.

### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_sent';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------------+-------+
    | Variable_name           | Value |
    +-------------------------+-------+
    | wsrep_flow_control_sent | 12    |
    +-------------------------+-------+
    ```

### Related status variables

When you analyze flow control sending patterns, consider these related status variables together:

#### [`wsrep_flow_control_requested`](#wsrep_flow_control_requested)

This variable shows whether your node is currently requesting flow control. A high sent count with a request value of `1` indicates active flow control.

#### [`wsrep_flow_control_recv`](#wsrep_flow_control_recv)

This variable shows how many flow control messages your node has received. Compare sent vs received to understand flow control patterns.

#### [`wsrep_flow_control_interval`](#wsrep_flow_control_interval)

This variable shows the flow control limits. High sent counts with specific limits help you understand when flow control activates.

#### [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg)

This variable shows the average queue length. High sent counts with high queue values indicate processing bottlenecks.

#### [`wsrep_flow_control_status`](#wsrep_flow_control_status)

This variable shows whether flow control is enabled. High sent counts with enabled status confirm flow control is working.

### Monitoring flow control sending

To get a comprehensive view of your cluster's flow control sending activity, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_sent|wsrep_flow_control_requested|wsrep_flow_control_recv|wsrep_flow_control_interval|wsrep_local_recv_queue_avg|wsrep_flow_control_status';
```

This query helps you identify whether flow control sending issues are due to:

* Your node's processing capacity being exceeded by replication load

* Network or replication bottlenecks causing queue buildup

* Inappropriate flow control limit configuration

* Node resource constraints affecting processing speed

* Cluster-wide performance issues requiring flow control intervention

### `wsrep_flow_control_status`

#### What the variable measures

The `wsrep_flow_control_status` status variable shows whether your node has the safety system (flow control) turned on for normal database traffic. Think of it like checking if the traffic lights are working on a road. This variable tells you whether your node can ask other nodes to slow down when it gets too busy. It does not show the status of flow control during SST (State Snapshot Transfer) operations.

### Expected results

This variable typically returns `ON` or `OFF`. A value of `ON` means flow control is enabled and your node can request other nodes to slow down when needed. A value of `OFF` means flow control is disabled and your node cannot request slowdowns. In most cases, you want this to be `ON` to protect your node from being overwhelmed.

#### What the values mean

| Value | Meaning               | Status                  | Action Needed                                                                                                                                               |
| ----- | --------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ON`  | Flow control enabled  | Safety system active    | Continue monitoring for changes                                                                                                                             |
| `OFF` | Flow control disabled | No protection available | Consider enabling: Review your flow control configuration and enable it if disabled. This protects your node from being overwhelmed by replication traffic. |

#### Real-world examples:

Example 1: Protected Node

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_status';
```

Your node can ask others to slow down when it gets too busy.

Example 2: Unprotected Node

```{.bash data-prompt="$"}
$ wsrep_flow_control_status: OFF
```

Your node cannot ask others to slow down, which may cause overload.

### `wsrep_gcache_pool_size`

#### What the variable measures

Use the `wsrep_gcache_pool_size` status variable to check how much memory your node allocates for the cluster cache pool. This pool stores writesets (collections of changes from database transactions) that your node receives from other nodes. The cluster uses this pool to support state transfers and reduce disk I/O during replication.

The cluster cache pool works like a short-term memory buffer. Your node uses this buffer to serve writesets to joining nodes or to recover from temporary delays.

#### Expected results

Expect this value to reflect the total memory allocated for the cache pool. The value appears in bytes. A larger value allows your node to store more writesets in memory, which improves performance during state transfers.

If the value is too small, your node may fall back to disk-based recovery or fail to serve state transfers efficiently.

#### What the values mean

| Value range        | Meaning                      | Action needed                      | What you should do                                                                                            |
| ------------------ | ---------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| Above 134217728    | Sufficient cache pool size   | Monitor during state transfers     | Run `SHOW STATUS LIKE 'wsrep_cluster_status'` and confirm `Synced`. Check logs for `State transfer complete`. |
| 67108864–134217728 | Limited cache pool size      | Monitor and adjust if needed       | Review `mysqld.log` for transfer delays. Increase pool size if nodes fall behind.                             |
| Below 67108864     | Insufficient for most setups | Investigate and increase pool size | Increase `wsrep_gcache_pool_size` in your config file. Restart the node to apply.                             |

#### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_gcache_pool_size';
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------------------+-----------+
    | Variable_name            | Value     |
    +--------------------------+-----------+
    | wsrep_gcache_pool_size   | 1073741824|
    +--------------------------+-----------+
    ```
    
#### Related status variables

* `wsrep_gcache_size`: Current amount of gcache data stored in memory

* [`wsrep_local_cached_downto`](#wsreplocalcacheddownto): Sequence number of the oldest writeset available in the gcache
  • wsrep_flow_control_recv: Number of flow control signals received from other nodes
  • wsrep_flow_control_sent: Number of flow control signals your node has sent to others
  • wsrep_local_recv_queue_avg: Average length of the replication receive queue

#### Monitoring flow control and cache

```{.bash data-prompt="$"}
SHOW GLOBAL STATUS LIKE 'wsrep_gcache_pool_size|wsrep_gcache_size|wsrep_local_cached_downto|wsrep_flow_control_recv|wsrep_flow_control_sent|wsrep_local_recv_queue_avg';
```

•	Check if wsrep_gcache_pool_size is large enough to retain writesets during heavy workloads

•	Investigate repeated SST events even when network connectivity and system resources are stable

•	Review node error logs when the gcache pool reaches its limit and the node cannot apply incremental replication.

### `wsrep_gcomm_uuid`

#### What the variable measures

The `wsrep_gcomm_uuid` status variable measures the unique identifier for the group communication layer in the Galera cluster. This variable tracks the UUID (Universally Unique Identifier) assigned to the group communication system. Each node in the cluster has a unique UUID that helps identify its participation in the cluster.

#### Expected results

The value of `wsrep_gcomm_uuid` is a string representing the unique identifier for the node's group communication. You should expect to see a UUID format, which typically looks like `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

#### What the values mean

| Value        | Meaning                                           | Action Needed                                                  |
| ------------ | ------------------------------------------------- | -------------------------------------------------------------- |
| Valid UUID   | Node is correctly identified in the cluster       | Continue monitoring for changes                                |
| Invalid UUID | Node may not be properly connected to the cluster | Investigate: Check network connectivity and node configuration |

#### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
SHOW GLOBAL STATUS LIKE 'wsrep_gcomm_uuid';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------+--------------------------------------+
    | Variable_name     | Value                                |
    +-------------------+--------------------------------------+
    | wsrep_gcomm_uuid  | 123e4567-e89b-12d3-a456-426614174000 |
    +-------------------+--------------------------------------+
    ```

#### Related status variables

- [`wsrep_cluster_state_uuid`](#wsrep_cluster_state_uuid): UUID of the current cluster state

- [`wsrep_cluster_size`](#wsrep_cluster_size): Number of nodes in the cluster

- [`wsrep_connected`](#wsrep_connected): Indicates if the node is connected to the cluster

### `wsrep_incoming_addresses`

#### What the variable measures

The `wsrep_incoming_addresses` status variable lists the IP addresses from which a node receives incoming replication traffic. This variable shows you the sources of replication data in your cluster. It acts like a list of approved nodes that can send updates to your local node, ensuring that only recognized sources are used for replication.

#### Expected results

You should expect to see a comma-separated list of IP addresses. Each address represents a node in the cluster that sends replication data. If the list is empty, it indicates that the node is not receiving any replication traffic, which may suggest a configuration issue or network problem.

#### What the values mean

| Value          | Meaning                                 | Action Needed                                                  |
| -------------- | --------------------------------------- | -------------------------------------------------------------- |
| Non-empty list | Active replication from specified nodes | Continue monitoring for changes                                |
| Empty list     | No incoming replication traffic         | Investigate: Check node configuration and network connectivity |

##### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege and reflects the status of the node where you execute it.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_incoming_addresses';
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------------------+---------------------------+
    | Variable_name            | Value                     |
    +--------------------------+---------------------------+
    | wsrep_incoming_addresses | 192.168.1.10,192.168.1.11 |
    +--------------------------+---------------------------+
    ```

#### Related status variables

- [`wsrep_local_state`](#wsrep_local_state): Indicates the local state of the node in the cluster.

- [`wsrep_cluster_size`](#wsrep_cluster_size): Shows the total number of nodes in the cluster.

- [`wsrep_connected`](#wsrep_connected): Indicates whether the node is connected to the cluster.

- [`wsrep_cluster_state_uuid`](#wsrep_cluster_state_uuid): Provides the unique identifier for the current cluster state.

- [`wsrep_ready`](#wsrep_ready): Indicates whether the node is ready to process replication events.

#### Monitoring incoming replication traffic

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_incoming_addresses|wsrep_local_state|wsrep_cluster_size|wsrep_connected|wsrep_cluster_state_uuid|wsrep_ready';
```

This query helps you monitor the status of incoming replication traffic and related variables. If you notice an empty list for `wsrep_incoming_addresses`, check the node's configuration and network settings to ensure proper connectivity with other nodes in the cluster.

### `wsrep_ist_receive_status`

#### What the variable measures

The `wsrep_ist_receive_status` status variable indicates the status of the Incremental State Transfer (IST) process for a node. This variable tracks whether the node is currently receiving data during the IST process, which occurs when a node joins the cluster and needs to catch up with the current state of the other nodes.

#### Expected results

You should expect to see values that indicate the current status of the IST process. A value of `1` means the node is actively receiving data, while a value of `0` indicates that the node is not receiving data. Understanding this variable helps you monitor the synchronization process of nodes joining the cluster.

#### What the values mean

| Value | Meaning                   | Action Needed                                                  |
| ----- | ------------------------- | -------------------------------------------------------------- |
| 1     | Receiving data during IST | Continue monitoring for completion                             |
| 0     | Not receiving data        | Investigate: Check network connectivity and node configuration |

##### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege and reflects the status of the node where you execute it.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_ist_receive_status';
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------------------+-------+
    | Variable_name            | Value |
    +--------------------------+-------+
    | wsrep_ist_receive_status | 1     |
    +--------------------------+-------+
    ```

#### Related status variables

- [`wsrep_ist_receive_queue`](#wsrep_ist_receive_queue): Number of writesets waiting to be received during IST.

- [`wsrep_local_state`](#wsrep_local_state): Indicates the local state of the node in the cluster.

- [`wsrep_cluster_size`](#wsrep_cluster_size): Shows the total number of nodes in the cluster.

- [`wsrep_connected`](#wsrep_connected): Indicates whether the node is connected to the cluster.

- [`wsrep_ready`](#wsrep_ready): Indicates whether the node is ready to process replication events.

#### Monitoring IST process

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_ist_receive_status|wsrep_ist_receive_queue|wsrep_local_state|wsrep_cluster_size|wsrep_connected|wsrep_ready';
```

This query helps you monitor the status of the IST process and related variables. If you see a value of `0` for `wsrep_ist_receive_status`, check the node's network connectivity and configuration to ensure proper synchronization with the cluster.

### `wsrep_ist_receive_seqno_end`

#### What the variable measures

The `wsrep_ist_receive_seqno_end` status variable measures the sequence number of the last writeset received during the Incremental State Transfer (IST) process. This variable helps you track the progress of data synchronization when a node joins the cluster and needs to catch up with the current state of other nodes.

#### Expected results

You should expect to see a numeric value that represents the sequence number of the last writeset received. A higher value indicates that the node has successfully received more data during the IST process. If the value does not increase as expected, it may indicate issues with data transfer or network connectivity.

#### What the values mean

| Value            | Meaning                           | Action Needed                                                  |
| ---------------- | --------------------------------- | -------------------------------------------------------------- |
| Positive integer | Last writeset received during IST | Continue monitoring for completion                             |
| 0                | No writesets received             | Investigate: Check network connectivity and node configuration |

##### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege and reflects the status of the node where you execute it.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_ist_receive_seqno_end';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------------------+-------+
    | Variable_name                 | Value |
    +-------------------------------+-------+
    | wsrep_ist_receive_seqno_end   | 12345 |
    +-------------------------------+-------+
    ```

#### Related status variables

- [`wsrep_ist_receive_queue`](#wsrep_ist_receive_queue): Number of writesets waiting to be received during IST.

- [`wsrep_local_state`](#wsrep_local_state): Indicates the local state of the node in the cluster.

- [`wsrep_cluster_size`](#wsrep_cluster_size): Shows the total number of nodes in the cluster.

- [`wsrep_connected`](#wsrep_connected): Indicates whether the node is connected to the cluster.

- [`wsrep_ready`](#wsrep_ready): Indicates whether the node is ready to process replication events.

#### Monitoring IST process

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_ist_receive_seqno_end|wsrep_ist_receive_queue|wsrep_local_state|wsrep_cluster_size|wsrep_connected|wsrep_ready';
```

This query helps you monitor the status of the IST process and related variables. If you see a value of `0` for `wsrep_ist_receive_seqno_end`, check the node's network connectivity and configuration to ensure proper synchronization with the cluster.

### `wsrep_ist_receive_seqno_current`

#### What the variable measures

The `wsrep_ist_receive_seqno_current` status variable measures the current sequence number of the writeset being received during the Incremental State Transfer (IST) process. This variable helps you track the progress of data synchronization when a node joins the cluster and needs to catch up with the current state of other nodes.

#### Expected results

You should expect to see a numeric value that represents the current sequence number of the writeset being received. A higher value indicates that the node is actively receiving data during the IST process. If the value does not increase as expected, it may indicate issues with data transfer or network connectivity.

#### What the values mean

| Value            | Meaning                                    | Action Needed                                                  |
| ---------------- | ------------------------------------------ | -------------------------------------------------------------- |
| Positive integer | Current writeset being received during IST | Continue monitoring for completion                             |
| 0                | No writeset currently being received       | Investigate: Check network connectivity and node configuration |

##### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege and reflects the status of the node where you execute it.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_ist_receive_seqno_current';
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------------------+-------+
    | Variable_name                   | Value |
    +---------------------------------+-------+
    | wsrep_ist_receive_seqno_current | 12345 |
    +---------------------------------+-------+
    ```

#### Related status variables

- [`wsrep_ist_receive_queue`](#wsrep_ist_receive_queue): Number of writesets waiting to be received during IST.

- [`wsrep_local_state`](#wsrep_local_state): Indicates the local state of the node in the cluster.

- [`wsrep_cluster_size`](#wsrep_cluster_size): Shows the total number of nodes in the cluster.

- [`wsrep_connected`](#wsrep_connected): Indicates whether the node is connected to the cluster.

- [`wsrep_ready`](#wsrep_ready): Indicates whether the node is ready to process replication events.

#### Monitoring IST process

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_ist_receive_seqno_current|wsrep_ist_receive_queue|wsrep_local_state|wsrep_cluster_size|wsrep_connected|wsrep_ready';
```

This query helps you monitor the status of the IST process and related variables. If you see a value of `0` for `wsrep_ist_receive_seqno_current`, check the node's network connectivity and configuration to ensure proper synchronization with the cluster.

### `wsrep_ist_receive_seqno_start`

#### What the variable measures

The `wsrep_ist_receive_seqno_start` status variable measures the sequence number of the first writeset received by the node during the IST (Incremental State Transfer) process. This variable helps you understand the starting point of the writesets that the node receives when synchronizing with other nodes.

#### Expected results

You should expect to see a value that indicates the sequence number of the first writeset received during the IST process. A higher value suggests that the node is receiving more recent writesets. If the value remains static for an extended period, it may indicate issues with the IST process or connectivity with other nodes.

#### What the values mean

| Value   | Meaning           | Action Needed                                                                |
| ------- | ----------------- | ---------------------------------------------------------------------------- |
| 0-100   | Normal operation  | Continue monitoring for changes                                              |
| 101-500 | Moderate activity | Monitor for performance trends                                               |
| 501+    | High activity     | Investigate: Check for potential delays in the IST process or network issues |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_ist_receive_seqno_start';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------------------+-------+
    | Variable_name                 | Value |
    +-------------------------------+-------+
    | wsrep_ist_receive_seqno_start | 200   |
    +-------------------------------+-------+
    ```

#### Related status variables

* [`wsrep_ist_receive_seqno_end`](#wsrep_ist_receive_seqno_end): Sequence number of the last writeset received during the IST process.

* [`wsrep_local_index`](#wsrep_local_index): Local index of the last applied writeset on the node.

* [`wsrep_local_commits`](#wsrep_local_commits): Number of local transactions that have been successfully committed.

#### Monitoring IST performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_ist_receive_seqno_start|wsrep_ist_receive_seqno_end|wsrep_local_index|wsrep_local_commits|wsrep_local_recv_queue';
```

This query shows the current values of several global status variables that provide a snapshot of a Percona XtraDB Cluster node's replication and synchronization state. The variables are particularly useful for monitoring the performance and health of the node.

* `wsrep_ist_receive_seqno_start`: This variable indicates the starting sequence number (LSN) for an Incremental State Transfer (IST). An IST is a process where a node that has fallen slightly behind the cluster receives missing transactions without having to perform a full State Transfer (SST).

* `wsrep_ist_receive_seqno_end`: This is the ending sequence number (LSN) for the IST. A node will catch up to this LSN to become fully synchronized with the cluster.

* `wsrep_local_index`: This shows the local index of the last transaction applied on the node.

* `wsrep_local_commits`: This variable tracks the total number of transactions successfully committed locally on the node since the MySQL server started.

* `wsrep_local_recv_queue`: This is a crucial metric that shows the current size of the replication receive queue. A non-zero value indicates that the node is receiving transactions faster than it can apply them, which can be a sign of a slow node or a very busy cluster. A constantly growing queue can lead to the node falling behind and potentially being evicted from the cluster.


### `wsrep_last_applied`

#### What the variable measures

The `wsrep_last_applied` status variable measures the sequence number of the last writeset that has been successfully applied on a node. This variable helps you track the most recent changes that the node has processed in the replication process.

#### Expected results

You should expect to see a steadily increasing value for `wsrep_last_applied`. A high value indicates that the node is applying writesets efficiently. If the value does not increase over time, it may suggest issues with replication or transaction processing.

#### What the values mean

| Value   | Meaning           | Action Needed                                                          |
| ------- | ----------------- | ---------------------------------------------------------------------- |
| 0-100   | Normal operation  | Continue monitoring for changes                                        |
| 101-500 | Moderate activity | Monitor for performance trends                                         |
| 501+    | High activity     | Investigate: Check for potential replication delays or resource issues |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_last_applied';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------+-------+
    | Variable_name     | Value |
    +-------------------+-------+
    | wsrep_last_applied| 300   |
    +-------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_index`](#wsrep_local_index): Local index of the last applied writeset on the node.

- [`wsrep_ist_receive_seqno_end`](#wsrep_ist_receive_seqno_end): Sequence number of the last writeset received during the IST process.

- [`wsrep_local_commits`](#wsrep_local_commits): Number of local transactions that have been successfully committed.

#### Monitoring replication performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_last_applied|wsrep_local_index|wsrep_ist_receive_seqno_end|wsrep_local_commits|wsrep_local_recv_queue';
```

This query retrieves the current values for several global status variables that provide a snapshot of a Percona XtraDB Cluster node's replication and synchronization state. These variables are useful for monitoring the performance and health of the node.

* `wsrep_last_applied`: This variable shows the sequence number (LSN) of the last transaction that was successfully applied by the node. This is a critical metric for tracking a node's progress in catching up with the cluster.

* `wsrep_local_index`: This shows the local index of the last transaction applied on the node. The local index provides a more granular view of the applied sequence within the node's own commit log.

* `wsrep_ist_receive_seqno_end`: This variable indicates the ending sequence number (LSN) for a running Incremental State Transfer (IST). A node will catch up to this LSN to become fully synchronized.

* `wsrep_local_commits`: This variable tracks the total number of transactions successfully committed locally on the node since the MySQL server started. It's a useful counter for monitoring the node's workload and throughput.

* `wsrep_local_recv_queue`: This is a crucial metric that shows the current size of the replication receive queue. A non-zero value means the node is receiving transactions faster than it can apply them. A constantly growing queue can indicate a bottleneck or a slow node that is falling behind the cluster.


### `wsrep_last_committed`

#### What the variable measures

The `wsrep_last_committed` status variable measures the sequence number of the last committed writeset on a node. This variable helps you track the most recent writeset that has been confirmed and committed in the replication process.

#### Expected results

You should expect to see a steadily increasing value for `wsrep_last_committed`. A high value indicates that the node is successfully committing writesets. If the value remains static for an extended period, it may suggest issues with transaction processing or replication.

#### What the values mean

| Value   | Meaning           | Action Needed                                                                         |
| ------- | ----------------- | ------------------------------------------------------------------------------------- |
| 0-100   | Normal operation  | Continue monitoring for changes                                                       |
| 101-500 | Moderate activity | Monitor for performance trends                                                        |
| 501+    | High activity     | Investigate: Check for potential delays in committing transactions or resource issues |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_last_committed';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-----------------------+-------+
    | Variable_name         | Value |
    +-----------------------+-------+
    | wsrep_last_committed  | 400   |
    +-----------------------+-------+
    ```

#### Related status variables

- [`wsrep_last_applied`](#wsrep_last_applied): Sequence number of the last writeset that has been successfully applied on the node.

- [`wsrep_local_index`](#wsrep_local_index): Local index of the last applied writeset on the node.

- [`wsrep_local_commits`](#wsrep_local_commits): Number of local transactions that have been successfully committed.

#### Monitoring commit performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_last_committed|wsrep_last_applied|wsrep_local_index|wsrep_local_commits|wsrep_local_recv_queue';
```

This query shows a series of global status variables that provide a snapshot of a Percona XtraDB Cluster node's replication and synchronization state. The variables are useful for monitoring the performance and health of the node.

* `wsrep_last_committed`: This variable shows the sequence number (LSN) of the last transaction that was committed on the node. This number is identical across all synchronized nodes in the cluster.

* `wsrep_last_applied`: This variable shows the LSN of the last transaction that was successfully applied by the node. This is a critical metric for tracking a node's progress in catching up with the cluster.

* `wsrep_local_index`: This shows the local index of the last transaction applied on the node. The local index provides a more granular view of the applied sequence within the node's own commit log.

* `wsrep_local_commits`: This variable tracks the total number of transactions successfully committed locally on the node since the MySQL server started.

* `wsrep_local_recv_queue`: This is a crucial metric that shows the current size of the replication receive queue. A non-zero value means the node is receiving transactions faster than it can apply them. A constantly growing queue can indicate a bottleneck or a slow node that is falling behind the cluster.

These variables are key for a database administrator to monitor a cluster's health and troubleshoot potential performance issues. For example, by checking wsrep_local_recv_queue, an administrator can identify a potential bottleneck before it impacts the entire cluster's performance.

### `wsrep_local_bf_aborts`

#### What the variable measures

The `wsrep_local_bf_aborts` status variable measures the number of local transactions that have been aborted due to a conflict with other transactions. The acronym "bf" stands for "before," indicating that these transactions were not able to complete because they conflicted with changes made by other transactions that were committed earlier.

#### Expected results

You should expect to see a low value for `wsrep_local_bf_aborts`. A high value indicates frequent transaction conflicts, which can lead to performance issues. Monitoring this variable helps you understand how often transactions are aborted and whether you need to adjust your workload or transaction management.

#### What the values mean

| Value | Meaning            | Action Needed                                                                      |
| ----- | ------------------ | ---------------------------------------------------------------------------------- |
| 0-10  | Normal operation   | Continue monitoring for changes                                                    |
| 11-50 | Moderate conflicts | Investigate: Review transaction patterns and consider optimizing queries           |
| 51+   | High conflicts     | Investigate immediately: Check for long-running transactions and optimize workload |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_bf_aborts';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-----------------------+-------+
    | Variable_name         | Value |
    +-----------------------+-------+
    | wsrep_local_bf_aborts | 5     |
    +-----------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_commits`](#wsrep_local_commits): Number of local transactions that have been successfully committed.

- [`wsrep_local_replays`](#wsrep_local_replays): Number of local transactions that have been replayed due to conflicts.

- [`wsrep_local_recv_queue`](#wsrep_local_recv_queue): Number of writesets waiting to be applied on the local node.

#### Monitoring transaction conflicts

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_bf_aborts|wsrep_local_commits|wsrep_local_replays|wsrep_local_recv_queue|wsrep_commit_oooe';
```

### `wsrep_local_cached_downto`

#### What the variable measures

The `wsrep_local_cached_downto` status variable measures the lowest sequence number of writesets that are currently cached on the local node. This variable helps you understand how far back the node can apply changes without needing to fetch additional data from other nodes in the cluster.

#### Expected results

You should expect to see a value that indicates the sequence number of the oldest cached writeset. A lower value suggests that the node has a larger cache of writesets, which can improve performance. A higher value may indicate that the node is relying more on fetching data from other nodes, which can slow down operations.

#### What the values mean:

| Value                          | Meaning         | Action Needed                                                                       |
| ------------------------------ | --------------- | ----------------------------------------------------------------------------------- |
| Low value (e.g., 0-100)        | Good caching    | Continue monitoring for changes                                                     |
| Moderate value (e.g., 101-500) | Average caching | Investigate: Check for potential delays in applying writesets                       |
| High value (e.g., 501+)        | Poor caching    | Investigate immediately: Review node performance and consider increasing cache size |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_cached_downto';
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------------------+-------+
    | Variable_name            | Value |
    +--------------------------+-------+
    | wsrep_local_cached_downto| 50    |
    +--------------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_cached_up_to`](#wsrep_local_cached_up_to): Highest sequence number of writesets currently cached on the local node.

- [`wsrep_local_recv_queue`](#wsrep_local_recv_queue): Number of writesets waiting to be applied on the local node.

- [`wsrep_local_commits`](#wsrep_local_commits): Number of local transactions that have been successfully committed.

#### Monitoring caching performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_cached_downto|wsrep_local_cached_up_to|wsrep_local_recv_queue|wsrep_local_commits|wsrep_commit_oooe';
```

### `wsrep_local_cert_failures`

#### What the variable measures

The `wsrep_local_cert_failures` status variable measures the number of local transactions that fail certification. Certification is the process that ensures transactions do not conflict with each other before they are committed. When a transaction fails certification, it means that another transaction has already modified the data that the failing transaction intended to change.

#### Expected results

You should expect to see a low value for `wsrep_local_cert_failures`. A high value indicates frequent certification failures, which can lead to performance issues and increased transaction retries. Monitoring this variable helps you understand how often transactions are failing certification and whether you need to adjust your transaction management.

#### What the values mean:

| Value | Meaning           | Action Needed                                                                      |
| ----- | ----------------- | ---------------------------------------------------------------------------------- |
| 0-5   | Normal operation  | Continue monitoring for changes                                                    |
| 6-20  | Moderate failures | Investigate: Review transaction patterns and consider optimizing queries           |
| 21+   | High failures     | Investigate immediately: Check for long-running transactions and optimize workload |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_cert_failures';
```

??? example "Expected output"

    ```{.text .no-copy}
    +----------------------------+-------+
    | Variable_name              | Value |
    +----------------------------+-------+
    | wsrep_local_cert_failures  | 3     |
    +----------------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_commits`](#wsrep_local_commits): Number of local transactions that have been successfully committed.

- [`wsrep_local_bf_aborts`](#wsrep_local_bf_aborts): Number of local transactions that have been aborted due to conflicts.

- [`wsrep_local_replays`](#wsrep_local_replays): Number of local transactions that have been replayed due to conflicts.

#### Monitoring certification failures

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_cert_failures|wsrep_local_commits|wsrep_local_bf_aborts|wsrep_local_replays|wsrep_local_recv_queue';
```

### `wsrep_local_commits`

#### What the variable measures

The `wsrep_local_commits` status variable measures the number of local transactions that have been successfully committed on a node. This variable helps you understand how many transactions have been completed without conflicts or errors.

#### Expected results

You should expect to see a steadily increasing value for `wsrep_local_commits`. A high value indicates that the node is processing transactions efficiently. A sudden drop in this value may suggest issues with transaction processing or conflicts.

#### What the values mean

| Value   | Meaning           | Action Needed                                                   |
| ------- | ----------------- | --------------------------------------------------------------- |
| 0-100   | Normal operation  | Continue monitoring for changes                                 |
| 101-500 | Moderate activity | Monitor for performance trends                                  |
| 501+    | High activity     | Investigate: Check for potential bottlenecks or resource issues |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_commits';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-----------------------+-------+
    | Variable_name         | Value |
    +-----------------------+-------+
    | wsrep_local_commits   | 250   |
    +-----------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_bf_aborts`](#wsrep_local_bf_aborts): Number of local transactions that have been aborted due to conflicts.

- [`wsrep_local_replays`](#wsrep_local_replays): Number of local transactions that have been replayed due to conflicts.

- [`wsrep_local_recv_queue`](#wsrep_local_recv_queue): Number of writesets waiting to be applied on the local node.

#### Monitoring transaction performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_commits|wsrep_local_bf_aborts|wsrep_local_replays|wsrep_local_recv_queue|wsrep_commit_oooe';
```

### `wsrep_local_index`

#### What the variable measures

The `wsrep_local_index` status variable measures the local index of the last applied writeset on a node. This variable helps you track the position of the last successfully applied writeset in the replication process.

#### Expected results

You should expect to see a steadily increasing value for `wsrep_local_index`. A high value indicates that the node is applying writesets efficiently. A sudden drop or stagnation in this value may suggest issues with replication or transaction processing.

#### What the values mean

| Value   | Meaning           | Action Needed                                                          |
| ------- | ----------------- | ---------------------------------------------------------------------- |
| 0-100   | Normal operation  | Continue monitoring for changes                                        |
| 101-500 | Moderate activity | Monitor for performance trends                                         |
| 501+    | High activity     | Investigate: Check for potential replication delays or resource issues |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_index';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------+-------+
    | Variable_name     | Value |
    +-------------------+-------+
    | wsrep_local_index | 150   |
    +-------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_commits`](#wsrep_local_commits): Number of local transactions that have been successfully committed.

- [`wsrep_local_bf_aborts`](#wsrep_local_bf_aborts): Number of local transactions that have been aborted due to conflicts.

- [`wsrep_local_recv_queue`](#wsrep_local_recv_queue): Number of writesets waiting to be applied on the local node.

#### Monitoring replication performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_index|wsrep_local_commits|wsrep_local_bf_aborts|wsrep_local_recv_queue|wsrep_commit_oooe';
```

### `wsrep_local_recv_queue`

#### What the variable measures

The `wsrep_local_recv_queue` status variable measures the current length of the local receive queue for incoming writesets. This variable tracks how many writesets are waiting to be processed by the applier threads on a node. A longer queue may indicate that the node is experiencing delays in processing incoming data.

#### Expected results

A low value indicates that the node is efficiently processing incoming writesets. A high value suggests that the node may be overwhelmed and unable to keep up with incoming data.

#### What the values mean

| Value | Meaning               | Action Needed                                    |
| ----- | --------------------- | ------------------------------------------------ |
| 0-5   | Efficient processing  | Continue monitoring for changes                  |
| 6-15  | Moderate queue length | Monitor for performance trends                   |
| 16+   | High queue length     | Investigate: Check system resources and workload |

#### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_recv_queue';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------------------+-------+
    | Variable                      | Value |
    +-------------------------------+-------+
    | wsrep_local_recv_queue        | 3     |
    +-------------------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg): Average length of the local receive queue for incoming writesets

- [`wsrep_local_send_queue`](#wsrep_local_send_queue): Current length of the local send queue for outgoing writesets

- [`wsrep_flow_control_paused`](#wsrep_flow_control_paused): Indicates if flow control is currently paused

### Monitoring the receive queue average

To get a comprehensive view of your cluster’s receive queue activity, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_recv_queue_avg|wsrep_local_recv_queue_max|wsrep_local_recv_queue_min|wsrep_flow_control_paused|wsrep_local_send_queue_avg';
```

This query helps you determine whether replication apply lag is being introduced due to:

- The node’s replication apply rate falling behind incoming transactions

- Temporary spikes in replication load causing queue buildup

- Flow control being triggered frequently (wsrep_flow_control_paused > 0)

- Imbalances between send queue and receive queue, indicating network or cluster bottlenecks

- Resource limitations on the node (CPU or disk I/O saturation) slowing down replication apply

### `wsrep_local_recv_queue_avg`

#### What the variable measures

The `wsrep_local_recv_queue_avg` status variable measures the average length of the local receive queue for incoming writesets. This variable tracks how many writesets are waiting to be processed by the applier threads on a node. A longer queue may indicate that the node is experiencing delays in processing incoming data.

#### Expected results

A low value indicates that the node is efficiently processing incoming writesets. A high value suggests that the node may be overwhelmed and unable to keep up with incoming data.

#### What the values mean

| Value | Meaning               | Action Needed                                    |
| ----- | --------------------- | ------------------------------------------------ |
| 0-5   | Efficient processing  | Continue monitoring for changes                  |
| 6-15  | Moderate queue length | Monitor for performance trends                   |
| 16+   | High queue length     | Investigate: Check system resources and workload |

#### Example of use

You can run this command on any node in the cluster; it requires only the PROCESS privilege (no special role or node restrictions), and results reflect the status of the node where it is executed, not the whole cluster.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_recv_queue_avg';
```

??? example "Expected output"

    ```{.text .no-copy}
    +----------------------------+-------+
    | Variable_name              | Value |
    +----------------------------+-------+
    | wsrep_local_recv_queue_avg | 0.002 |
    +----------------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_recv_queue`](#wsrep_local_recv_queue): Current length of the local receive queue

- [`wsrep_local_send_queue_avg`](#wsrep_local_send_queue_avg): Average length of the local send queue for outgoing writesets

- [`wsrep_flow_control_paused`](#wsrep_flow_control_paused): Indicates if flow control is currently paused

#### Monitoring receive queue performance

```{.bash data-prompt="$"}
SHOW GLOBAL STATUS LIKE 'wsrep_local_recv_queue_avg|wsrep_local_recv_queue|wsrep_local_send_queue_avg|wsrep_flow_control_paused|wsrep_commit_oooe';
```

### `wsrep_local_replays`

#### What the variable measures

The `wsrep_local_replays` status variable measures the number of local transactions that have been replayed due to conflicts with other transactions. This variable helps you understand how often the node has to reapply transactions because of replication conflicts.

#### Expected results

You should expect to see a low value for `wsrep_local_replays`. A high value indicates frequent conflicts, which can lead to performance issues. Monitoring this variable helps you identify potential problems in transaction management.

#### What the values mean

| Value | Meaning            | Action Needed                                                                      |
| ----- | ------------------ | ---------------------------------------------------------------------------------- |
| 0-10  | Normal operation   | Continue monitoring for changes                                                    |
| 11-50 | Moderate conflicts | Investigate: Review transaction patterns and consider optimizing queries           |
| 51+   | High conflicts     | Investigate immediately: Check for long-running transactions and optimize workload |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_replays';
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------+-------+
    | Variable_name       | Value |
    +---------------------+-------+
    | wsrep_local_replays | 5     |
    +---------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_commits`](#wsrep_local_commits): Number of local transactions that have been successfully committed.

- [`wsrep_local_bf_aborts`](#wsrep_local_bf_aborts): Number of local transactions that have been aborted due to conflicts.

- [`wsrep_local_recv_queue`](#wsrep_local_recv_queue): Number of writesets waiting to be applied on the local node.

#### Monitoring replay performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_replays|wsrep_local_commits|wsrep_local_bf_aborts|wsrep_local_recv_queue|wsrep_commit_oooe';
```

### `wsrep_local_send_queue`

#### What the variable measures

The `wsrep_local_send_queue` status variable measures the number of writesets that are waiting to be sent from the local node to other nodes in the cluster. This variable helps you understand the backlog of writesets that have not yet been transmitted.

#### Expected results

You should expect to see a low value for `wsrep_local_send_queue`. A high value indicates that there is a backlog of writesets waiting to be sent, which may suggest network issues or high load on the node. Monitoring this variable helps you identify potential delays in replication.

#### What the values mean

| Value | Meaning          | Action Needed                                                                 |
| ----- | ---------------- | ----------------------------------------------------------------------------- |
| 0-10  | Normal operation | Continue monitoring for changes                                               |
| 11-50 | Moderate backlog | Investigate: Check network connectivity and node performance                  |
| 51+   | High backlog     | Investigate immediately: Assess network issues and consider reducing workload |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_send_queue';
```

??? example "Expected output"

    ```{.text .no-copy}
    +------------------------+-------+
    | Variable_name          | Value |
    +------------------------+-------+
    | wsrep_local_send_queue | 3     |
    +------------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_recv_queue`](#wsrep_local_recv_queue): Number of writesets waiting to be applied on the local node.

- [`wsrep_local_commits`](#wsrep_local_commits): Number of local transactions that have been successfully committed.

- [`wsrep_local_replays`](#wsrep_local_replays): Number of local transactions that have been replayed due to conflicts.

#### Monitoring send queue performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_send_queue|wsrep_local_recv_queue|wsrep_local_commits|wsrep_local_replays|wsrep_commit_oooe';
```

### `wsrep_local_send_queue_avg`

#### What the variable measures

The `wsrep_local_send_queue_avg` status variable measures the average number of writesets in the local send queue over a specified time period. This variable helps you understand the typical backlog of writesets waiting to be sent from the local node to other nodes in the cluster.

#### Expected results

You should expect to see a low average value for `wsrep_local_send_queue_avg`. A high average indicates that there is a consistent backlog of writesets waiting to be sent, which may suggest network issues or high load on the node. Monitoring this variable helps you identify trends in replication delays.

#### What the values mean

| Value | Meaning          | Action Needed                                                                 |
| ----- | ---------------- | ----------------------------------------------------------------------------- |
| 0-10  | Normal operation | Continue monitoring for changes                                               |
| 11-50 | Moderate backlog | Investigate: Check network connectivity and node performance                  |
| 51+   | High backlog     | Investigate immediately: Assess network issues and consider reducing workload |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_send_queue_avg';
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------------+-------+
    | Variable_name             | Value |
    +---------------------------+-------+
    | wsrep_local_send_queue_avg| 4     |
    +---------------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_send_queue`](#wsrep_local_send_queue): Number of writesets waiting to be sent from the local node.

- [`wsrep_local_recv_queue`](#wsrep_local_recv_queue): Number of writesets waiting to be applied on the local node.

- [`wsrep_local_commits`](#wsrep_local_commits): Number of local transactions that have been successfully committed.

#### Monitoring send queue average performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_send_queue_avg|wsrep_local_send_queue|wsrep_local_recv_queue|wsrep_local_commits|wsrep_local_replays';
```

### `wsrep_local_state`

#### What the variable measures

The `wsrep_local_state` status variable measures the current state of the local node in the Galera cluster. This variable indicates whether the node is in a state of synchronization, replication, or other operational modes.

#### Expected results

You should expect to see a value that corresponds to the current operational state of the node. Common states include "Synced," "Donor," "Joining," and "Disconnected." Monitoring this variable helps you understand the node's status within the cluster.

#### What the values mean

| Value | Meaning      | Action Needed                                                    |
| ----- | ------------ | ---------------------------------------------------------------- |
| 0     | Disconnected | Investigate: Check network connectivity and node status          |
| 1     | Joining      | Continue monitoring for completion                               |
| 2     | Donor        | Continue monitoring for changes                                  |
| 3     | Synced       | Normal operation, continue monitoring                            |
| 4     | Error        | Investigate immediately: Check error logs and node configuration |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_state';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------+-------+
    | Variable_name     | Value |
    +-------------------+-------+
    | wsrep_local_state | 3     |
    +-------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_state_comment`](#wsrep_local_state_comment): A textual description of the current state of the local node.

- [`wsrep_cluster_size`](#wsrep_cluster_size): The total number of nodes in the cluster.

- [`wsrep_cluster_state_uuid`](#wsrep_cluster_state_uuid): The unique identifier for the current state of the cluster.

#### Monitoring local state performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_state|wsrep_local_state_comment|wsrep_cluster_size|wsrep_cluster_state_uuid';
```

### `wsrep_local_state_comment`

#### What the variable measures

The `wsrep_local_state_comment` status variable provides a textual description of the current state of the local node in the Galera cluster. This variable helps you understand the operational status of the node in a more human-readable format.

#### Expected results

You should expect to see a descriptive string that indicates the current state of the node, such as "Synced," "Donor," "Joining," or "Disconnected." Monitoring this variable helps you quickly assess the node's status within the cluster.

#### What the values mean

| Value          | Meaning                                           | Action Needed                                                    |
| -------------- | ------------------------------------------------- | ---------------------------------------------------------------- |
| "Disconnected" | The node is not connected to the cluster          | Investigate: Check network connectivity and node status          |
| "Joining"      | The node is in the process of joining the cluster | Continue monitoring for completion                               |
| "Donor"        | The node is providing data to another node        | Continue monitoring for changes                                  |
| "Synced"       | The node is fully synchronized with the cluster   | Normal operation, continue monitoring                            |
| "Error"        | The node has encountered an error                 | Investigate immediately: Check error logs and node configuration |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_state_comment';
```

??? example "Expected output"

    ```{.text .no-copy}
    +----------------------------+------------------+
    | Variable_name              | Value            |
    +----------------------------+------------------+
    | wsrep_local_state_comment  | Synced           |
    +----------------------------+------------------+
    ```

#### Related status variables

- [`wsrep_local_state`](#wsrep_local_state): The numeric representation of the current state of the local node.

- [`wsrep_cluster_size`](#wsrep_cluster_size): The total number of nodes in the cluster.

- [`wsrep_cluster_state_uuid`](#wsrep_cluster_state_uuid): The unique identifier for the current state of the cluster.

#### Monitoring local state comment performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_state_comment|wsrep_local_state|wsrep_cluster_size|wsrep_cluster_state_uuid';
```

### `wsrep_local_state_uuid`

#### What the variable measures

The `wsrep_local_state_uuid` status variable measures the unique identifier for the current state of the local node in the Galera cluster. This variable helps you track the specific state of the node and can be useful for debugging and monitoring purposes.

#### Expected results

You should expect to see a UUID (Universally Unique Identifier) string that represents the current state of the node. This value changes whenever the state of the node changes, allowing you to identify the specific state at any given time.

#### What the values mean

| Value                           | Meaning                             | Action Needed                                              |
| ------------------------------- | ----------------------------------- | ---------------------------------------------------------- |
| A valid UUID string             | The node is in a specific state     | Continue monitoring for changes                            |
| An empty string or invalid UUID | The node may be experiencing issues | Investigate immediately: Check node configuration and logs |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_state_uuid';
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------------+--------------------------------------+
    | Variable_name             | Value                                |
    +---------------------------+--------------------------------------+
    | wsrep_local_state_uuid    | 123e4567-e89b-12d3-a456-426614174000 |
    +---------------------------+--------------------------------------+
    ```

#### Related status variables

- [`wsrep_local_state`](#wsrep_local_state): The numeric representation of the current state of the local node.

- [`wsrep_local_state_comment`](#wsrep_local_state_comment): A textual description of the current state of the local node.

- [`wsrep_cluster_state_uuid`](#wsrep_cluster_state_uuid): The unique identifier for the current state of the entire cluster.

#### Monitoring local state UUID performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_local_state_uuid|wsrep_local_state|wsrep_local_state_comment|wsrep_cluster_state_uuid';
```

### `wsrep_monitor_status`

The status of the local monitor (local and replicating actions), apply monitor
(apply actions of write-set), and commit monitor (commit actions of write
sets). In the value of this variable, each monitor (L: Local, A: Apply, C:
Commit) is represented as a _last_entered_, and _last_left_ pair:

```{.text .no-copy}
wsrep_monitor_status (L/A/C)	[ ( 7, 5), (2, 2), ( 2, 2) ]
```

last_entered

Shows which transaction or write-set has recently entered the queue.

last_left

Shows which last transaction or write-set has been executed and left the queue.

According to the Galera protocol, transactions can be applied in parallel but
must be committed in a given order. This rule implies that there can be multiple
transactions in the _apply_ state at a given point of time but transactions are
_committed_ sequentially.

!!! admonition "See also"

    [`Galera Documentation: Database replication`](https://galeracluster.com/library/documentation/tech-desc-introduction.html)

### `wsrep_protocol_version`

#### What the variable measures

The `wsrep_protocol_version` status variable measures the version of the Galera replication protocol that the node is currently using. This variable helps you understand the compatibility and features available in the replication process.

#### Expected results

You should expect to see a numeric value that indicates the current protocol version. This value is important for ensuring that all nodes in the cluster are using compatible versions of the Galera protocol.

#### What the values mean

| Value | Meaning                          | Action Needed                                         |
| ----- | -------------------------------- | ----------------------------------------------------- |
| 1     | Basic protocol version           | Continue monitoring for changes                       |
| 2     | Enhanced features available      | Ensure all nodes are compatible                       |
| 3+    | Latest features and improvements | Verify that all nodes are updated to the same version |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_protocol_version';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------------+-------+
    | Variable_name           | Value |
    +-------------------------+-------+
    | wsrep_protocol_version   | 3     |
    +-------------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_state`](#wsrep_local_state): The numeric representation of the current state of the local node.

- [`wsrep_local_state_comment`](#wsrep_local_state_comment): A textual description of the current state of the local node.

- [`wsrep_cluster_size`](#wsrep_cluster_size): The total number of nodes in the cluster.

#### Monitoring protocol version performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_protocol_version|wsrep_local_state|wsrep_local_state_comment|wsrep_cluster_size';
```

### `wsrep_provider_name`

#### What the variable measures

The `wsrep_provider_name` status variable measures the name of the Galera replication provider that the node is currently using. This variable helps you identify the specific implementation of the Galera library that is being utilized for replication.

#### Expected results

You should expect to see a string value that indicates the name of the provider. Common values include "Galera" or other specific implementations. Monitoring this variable helps you ensure that the correct provider is in use for your cluster.

#### What the values mean

| Value    | Meaning                             | Action Needed                           |
| -------- | ----------------------------------- | --------------------------------------- |
| "Galera" | Standard Galera provider            | Continue monitoring for changes         |
| "Other"  | A different provider implementation | Verify compatibility with cluster nodes |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_provider_name';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-----------------------+-------+
    | Variable_name         | Value |
    +-----------------------+-------+
    | wsrep_provider_name    | Galera|
    +-----------------------+-------+
    ```

#### Related status variables

- [`wsrep_provider_version`](#wsrep_provider_version): The version of the Galera provider currently in use.

- [`wsrep_local_state`](#wsrep_local_state): The numeric representation of the current state of the local node.

- [`wsrep_cluster_size`](#wsrep_cluster_size): The total number of nodes in the cluster.

#### Monitoring provider name performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_provider_name|wsrep_provider_version|wsrep_local_state|wsrep_cluster_size';
```

### `wsrep_provider_vendor`

#### What the variable measures

The `wsrep_provider_vendor` status variable measures the name of the vendor that provides the Galera replication library being used by the node. This variable helps you identify the organization or company responsible for the implementation of the Galera provider.

#### Expected results

You should expect to see a string value that indicates the name of the vendor. Common values include "Codership" for the original Galera provider. Monitoring this variable helps you ensure that you are aware of the source of the replication library in use.

#### What the values mean

| Value       | Meaning                           | Action Needed                           |
| ----------- | --------------------------------- | --------------------------------------- |
| "Codership" | The original vendor of Galera     | Continue monitoring for changes         |
| "Other"     | A different vendor implementation | Verify compatibility with cluster nodes |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_provider_vendor';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-----------------------+----------+
    | Variable_name         | Value    |
    +-----------------------+----------+
    | wsrep_provider_vendor   | Codership|
    +-----------------------+----------+
    ```

#### Related status variables

- [`wsrep_provider_name`](#wsrep_provider_name): The name of the Galera provider currently in use.

- [`wsrep_provider_version`](#wsrep_provider_version): The version of the Galera provider currently in use.

- [`wsrep_local_state`](#wsrep_local_state): The numeric representation of the current state of the local node.

#### Monitoring provider vendor performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_provider_vendor|wsrep_provider_name|wsrep_provider_version|wsrep_local_state';
```

### `wsrep_provider_version`

#### What the variable measures

The `wsrep_provider_version` status variable measures the version of the Galera replication provider that the node is currently using. This variable helps you identify the specific version of the Galera library, which is important for ensuring compatibility and accessing the latest features.

#### Expected results

You should expect to see a string value that indicates the current version of the provider. This value is crucial for verifying that all nodes in the cluster are running compatible versions of the Galera protocol.

#### What the values mean

| Value              | Meaning                                        | Action Needed                                                |
| ------------------ | ---------------------------------------------- | ------------------------------------------------------------ |
| "3.0.0" or similar | Specific version of the Galera provider        | Continue monitoring for changes                              |
| "Incompatible"     | The version is not compatible with other nodes | Investigate immediately: Update nodes to compatible versions |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_provider_version';
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------------+----------+
    | Variable_name             | Value    |
    +---------------------------+----------+
    | wsrep_provider_version    | 3.1.0    |
    +---------------------------+----------+
    ```

#### Related status variables

- [`wsrep_provider_name`](#wsrep_provider_name): The name of the Galera provider currently in use.

- [`wsrep_provider_vendor`](#wsrep_provider_vendor): The name of the vendor that provides the Galera replication library.

- [`wsrep_local_state`](#wsrep_local_state): The numeric representation of the current state of the local node.

#### Monitoring provider version performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_provider_version|wsrep_provider_name|wsrep_provider_vendor|wsrep_local_state';
```

### `wsrep_ready`

#### What the variable measures

The `wsrep_ready` status variable measures whether the Galera node is ready to process transactions. This variable indicates the operational status of the node in relation to its ability to participate in the cluster's replication process.

#### Expected results

You should expect to see a boolean value, either "ON" or "OFF." A value of "ON" indicates that the node is ready to accept and process transactions, while "OFF" indicates that the node is not ready, possibly due to synchronization or other issues.

#### What the values mean

| Value | Meaning                                       | Action Needed                                                         |
| ----- | --------------------------------------------- | --------------------------------------------------------------------- |
| "ON"  | The node is ready to process transactions     | Continue monitoring for changes                                       |
| "OFF" | The node is not ready to process transactions | Investigate immediately: Check synchronization status and node health |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_ready';
```

??? example "Expected output"

    ```{.text .no-copy}
    +-------------------+-------+
    | Variable_name     | Value |
    +-------------------+-------+
    | wsrep_ready       | ON    |
    +-------------------+-------+
    ```

#### Related status variables

- [`wsrep_local_state`](#wsrep_local_state): The numeric representation of the current state of the local node.

- [`wsrep_local_state_comment`](#wsrep_local_state_comment): A textual description of the current state of the local node.

- [`wsrep_cluster_size`](#wsrep_cluster_size): The total number of nodes in the cluster.

#### Monitoring readiness performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_ready|wsrep_local_state|wsrep_local_state_comment|wsrep_cluster_size';
```

### `wsrep_received`

#### What the variable measures

The `wsrep_received` status variable measures the total number of writesets that have been received by the local node from other nodes in the Galera cluster. This variable helps you track the volume of incoming replication traffic.

#### Expected results

You should expect to see a steadily increasing value for `wsrep_received`. A high value indicates that the node is actively receiving writesets from other nodes. If the value stagnates, it may suggest issues with replication or network connectivity.

#### What the values mean

| Value   | Meaning           | Action Needed                                                |
| ------- | ----------------- | ------------------------------------------------------------ |
| 0-100   | Normal operation  | Continue monitoring for changes                              |
| 101-500 | Moderate activity | Monitor for performance trends                               |
| 501+    | High activity     | Investigate: Check network connectivity and node performance |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_received';
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------------+-------+
    | Variable_name      | Value |
    +--------------------+-------+
    | wsrep_received     | 250   |
    +--------------------+-------+
    ```

#### Related status variables

- [`wsrep_received_bytes`](#wsrep_received_bytes): The total number of bytes received from other nodes.

- [`wsrep_local_commits`](#wsrep_local_commits): The number of local transactions that have been successfully committed.

- [`wsrep_local_replays`](#wsrep_local_replays): The number of local transactions that have been replayed due to conflicts.

#### Monitoring received writesets performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_received|wsrep_received_bytes|wsrep_local_commits|wsrep_local_replays';
```

### `wsrep_received_bytes`

Total size (in bytes) of writesets received from other nodes.

### `wsrep_repl_data_bytes`

Total size (in bytes) of data replicated.

### `wsrep_repl_keys`

Total number of keys replicated.

### `wsrep_repl_keys_bytes`

Total size (in bytes) of keys replicated.

### `wsrep_repl_other_bytes`

Total size of other bits replicated.

### `wsrep_replicated`

#### What the variable measures

The `wsrep_replicated` status variable measures the total number of writesets that have been replicated from the local node to other nodes in the Galera cluster. This variable helps you track the volume of outgoing replication traffic.

#### Expected results

You should expect to see a steadily increasing value for `wsrep_replicated`. A high value indicates that the node is actively sending writesets to other nodes. If the value stagnates, it may suggest issues with replication or network connectivity.

#### What the values mean

| Value   | Meaning           | Action Needed                                                |
| ------- | ----------------- | ------------------------------------------------------------ |
| 0-100   | Normal operation  | Continue monitoring for changes                              |
| 101-500 | Moderate activity | Monitor for performance trends                               |
| 501+    | High activity     | Investigate: Check network connectivity and node performance |

#### Example of use

You can run this command on any node in the cluster. It requires only the PROCESS privilege, and the results reflect the status of the node where you execute the command.

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_replicated';
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------------+-------+
    | Variable_name      | Value |
    +--------------------+-------+
    | wsrep_replicated   | 300   |
    +--------------------+-------+
    ```

#### Related status variables

- [`wsrep_replicated_bytes`](#wsrep_replicated_bytes): The total number of bytes replicated to other nodes.

- [`wsrep_local_commits`](#wsrep_local_commits): The number of local transactions that have been successfully committed.

- [`wsrep_local_replays`](#wsrep_local_replays): The number of local transactions that have been replayed due to conflicts.

#### Monitoring replicated writesets performance

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_replicated|wsrep_replicated_bytes|wsrep_local_commits|wsrep_local_replays';
```

### `wsrep_replicated_bytes`

Total size of replicated writesets. To compute the actual size of bytes sent
over network to cluster peers, multiply the value of this variable by the number
of cluster peers in the given [`network segment`](wsrep-provider-index.md#gmcastsegment).

!!! admonition "See also"

    [`Galera status variable: wsrep_replicated_bytes`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-replicated-bytes)
