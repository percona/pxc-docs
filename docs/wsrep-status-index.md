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
| `wsrep_sync_wait`     | Controls consistency level for client queries via synchronization | [`wsrep_sync_wait`](#wsrep_sync_wait)         |

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

## wsrep_apply_oooe

### What the variable measures
`wsrep_apply_oooe` (Out‑of‑Order‑of‑Execution) reports the proportion of writesets that a node applies out of their original sequence‑number order.  
Because writesets can be processed by multiple applier threads concurrently, a higher value indicates greater parallelism among those threads.

### Expected results
| Value | Interpretation |
|------|----------------|
| **≈ 1** | Excellent parallelism – most writesets are applied out of order. |
| **0.5 – 0.8** | Acceptable parallelism; some room for improvement. |
| **< 0.5** | Limited parallelism – investigate applier‑thread configuration or contention. |
| **0** | No out‑of‑order execution; applier threads are effectively serial. |

### Example usage
```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_apply_oooe';
```

??? example "Expected output"

    ```{.text .no-copy}
    +------------------+----------+
    | Variable_name    | Value    |
    +------------------+----------+
    | wsrep_apply_oooe | 0.985654 |
    +------------------+----------+
    ```

The value is a floating‑point number between 0 and 1, reported with up to six decimal places.

#### Related Status Variables

| Variable | Description | Ideal value / Interpretation |
|---|---|---|
| [wsrep_apply_window](#wsrep_apply_window) | Measures the average distance between the highest and lowest concurrently applied sequence numbers. | Higher values indicate greater potential for parallelism because a wider window means more writesets can be processed simultaneously. |
| [wsrep_apply_oool](#wsrep_apply_oool) | Tracks “Out of Order of Latest” – when older transactions are applied after newer ones. | Low values (close to 0) are ideal; high values suggest that later transactions are being delayed, possibly due to slow or blocking operations. |
| [wsrep_cert_deps_distance](#wsrep_cert_deps_distance) | Shows the average distance between sequence numbers that can be applied in parallel based on dependencies. | Higher values suggest better parallelization potential, as more distant (and thus independent) transactions can be executed concurrently. |
| [wsrep_local_recv_queue_avg](#wsrep_local_recv_queue_avg) | Displays the average length of the receive queue on the node. | Consistently high values (especially when paired with a low `wsrep_apply_oooe`) indicate the node cannot apply incoming writesets quickly enough and may need more resources or tuning. |
| [wsrep_commit_oooe](#wsrep_commit_oooe) | Similar to `wsrep_apply_oooe` but measured during the commit phase of replication. | High values denote good commit‑phase parallelism; low values may point to bottlenecks in committing transactions after they have been applied. |                          

#### Monitor parallelization performance

Run a single query to collect all relevant metrics:

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_apply_oooe|wsrep_apply_window|wsrep_apply_oool|wsrep_cert_deps_distance|wsrep_local_recv_queue_avg|wsrep_commit_oooe';
```

Interpretation tips

* Low `wsrep_apply_oooe` + high `wsrep_apply_window` – Parallelism potential exists but isn’t being realized (possible thread‑count or lock contention). 

* High `wsrep_local_recv_queue_avg` + low `wsrep_apply_oooe` – The node is overwhelmed by incoming writesets; consider adding resources or tuning applier threads.

* Elevated `wsrep_apply_oool` or `wsrep_commit_oooe` – May indicate transaction conflicts or latency in later stages of replication.

Address identified bottlenecks by adjusting the number of applier threads, optimizing transaction size, or scaling hardware/network capacity.


# wsrep_apply_oool

## What the variable measures

`wsrep_apply_oool` (Out of Order of Latest) records situations where a writeset with a lower sequence number—an older transaction—applies after a writeset with a higher sequence number—a newer transaction. This condition appears when a small transaction finishes before a larger, earlier transaction that is still being processed.

## Expected results

A value near `0` is ideal, showing that writesets generally apply in the correct order. Larger values signal that slow‑running transactions are creating a bottleneck, forcing applier threads to skip ahead and apply newer writesets first. When the metric rises, investigate the performance of long‑duration transactions or adjust applier‑thread settings.

## Example of use

Run the following command on any node in the cluster; only the `PROCESS` privilege is required, and the result reflects the status of the node where the command executes.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_apply_oool';
```

??? example "Expected output"

    ```{.text .no-copy}
    +------------------+----------+
    | Variable_name    | Value    |
    +------------------+----------+
    | wsrep_apply_oool | 0.000132 |
    +------------------+----------+
    ```

## Related status variables

| Variable | Description |
|---|---|
| [wsrep_apply_oooe](#wsrep_apply_oooe) | Measures overall parallelism of the applier threads. Low parallelism can cause older transactions to fall behind, leading to higher `wsrep_apply_oool` values. |
| [wsrep_apply_window](#wsrep_apply_window) | Shows the average distance between the highest and lowest sequence numbers being applied concurrently. A narrow window together with an elevated `wsrep_apply_oool` suggests the applier threads cannot keep up with the transaction workload. |
| [wsrep_local_recv_queue_avg](#wsrep_local_recv_queue_avg) | Indicates the average length of the receive queue on the node. High queue lengths combined with a high `wsrep_apply_oool` confirm that incoming writesets arrive faster than they can be applied. |
| [wsrep_cert_deps_distance](#wsrep_cert_deps_distance) | Reflects the potential for parallel execution based on transaction dependencies. |
| [wsrep_apply_oool](#wsrep_apply_oool) | Tracks out‑of‑order application of older transactions (older writeset applied after a newer one). |

### Monitoring out-of-order application

Execute the following query to obtain a comprehensive snapshot of the relevant metrics:

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_apply_oool|wsrep_apply_oooe|wsrep_apply_window|wsrep_local_recv_queue_avg|wsrep_cert_deps_distance|wsrep_commit_oooe';
```

Analyzing the results helps pinpoint the root cause of out‑of‑order application, which may stem from:

* Insufficient applier threads

* Transaction conflicts that restrict parallelism

* Resource constraints causing processing delays

* Network or replication bottlenecks

* Inefficient transaction ordering

## wsrep_apply_window

### What the variable measures

`wsrep_apply_window` records the average distance between the highest and lowest sequence numbers (`seqno`) that applier threads are applying concurrently. A wider window—i.e., a larger numeric value—means that a broader range of transactions can be processed at the same time, indicating greater parallelization potential.

### Expected results

Higher values signal a healthy, efficiently parallelized cluster that keeps pace with the replication flow. Lower values suggest a bottleneck in the applier threads, which may stem from resource constraints or a prevalence of large, conflicting transactions.

### Example of use

Run the following command on any node; only the `PROCESS` privilege is required, and the result reflects the status of the node where the command is executed.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_apply_window';
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

| Variable | Description | Interpretation |
|---|---|---|
| [wsrep_apply_oooe](#wsrep_apply_oooe) | Measures overall parallelism of the applier threads. | High values together with a wide `wsrep_apply_window` indicate strong parallelization; low values with a narrow window suggest limited parallel capability. |
| [wsrep_apply_oool](#wsrep_apply_oool) | Tracks out‑of‑order application of older transactions (older writeset applied after a newer one). | A narrow window combined with high `wsrep_apply_oool` values implies applier threads cannot keep up, causing older transactions to fall behind. |
| [wsrep_cert_deps_distance](#wsrep_cert_deps_distance) | Shows the potential for parallelism based on transaction dependencies. | Wide `wsrep_apply_window` paired with a high distance value demonstrates that the cluster is achieving its parallelization potential. |
| [wsrep_local_recv_queue_avg](#wsrep_local_recv_queue_avg) | Indicates the average length of the receive queue on the node. | Narrow window with high queue lengths confirms the node cannot process incoming writesets quickly enough. |
| [wsrep_commit_window](#wsrep_commit_window) | Records the average distance between concurrently committed sequence numbers. | Comparing this metric with `wsrep_apply_window` helps identify whether bottlenecks lie in the apply phase or the commit phase. |

### Monitoring parallelization window performance

Execute the following query to obtain a comprehensive snapshot of the relevant metrics:

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_apply_window|wsrep_apply_oooe|wsrep_apply_oool|wsrep_cert_deps_distance|wsrep_local_recv_queue_avg|wsrep_commit_window';
```

This query helps you identify whether window performance issues are due to the following:

* Insufficient applier threads

* Transaction conflicts limiting parallelization

* Resource constraints reducing processing capacity

* Network or replication bottlenecks

* Inefficient transaction dependency patterns

### `wsrep_cert_bucket_count`

This variable, shows the number of cells in the certification index
hash-table.

### `wsrep_cert_deps_distance`

Average distance between highest and lowest sequence number that can be
possibly applied in parallel.

!!! admonition "See also"

    [`Galera status variable: wsrep_cert_deps_distance`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-cert-deps-distance)

### `wsrep_cert_index_size`

Number of entries in the certification index.

!!! admonition "See also"

    [`Galera status variable: wsrep_cert_index_size`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-cert-index-size)

### `wsrep_cert_interval`

Average number of write-sets received while a transaction replicates.

!!! admonition "See also"

    [`Galera status variable: wsrep_cert_interval`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-cert-interval)

### `wsrep_cluster_conf_id`

Number of cluster membership changes that have taken place.

!!! admonition "See also"

    [`Galera status variable: wsrep_cluster_conf_id`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-cluster-conf-id)

### `wsrep_cluster_size`

Current number of nodes in the cluster.

!!! admonition "See also"

    [`Galera status variable: wsrep_cluster_size`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-cluster-size)

### `wsrep_cluster_state_uuid`

This variable contains [UUID](glossary.md#uuid) state of the cluster. When this value is
the same as the one in [`wsrep_local_state_uuid`](wsrep-status-index.md#wsrep_local_state_uuid), node is synced with
the cluster.

!!! admonition "See also"

    [`Galera status variable: wsrep_cluster_state_uuid`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-cluster-state-uuid)

### `wsrep_cluster_status`

Status of the cluster component. Possible values are:

* `Primary`

* `Non-Primary`

* `Disconnected`

!!! admonition "See also"

    [`Galera status variable: wsrep_cluster_status`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-cluster-status)

### `wsrep_commit_oooe`

This variable shows how often a transaction was committed out of order.

!!! admonition "See also"

    [`Galera status variable: wsrep_commit_oooe`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-commit-oooe)

### `wsrep_commit_oool`

This variable currently has no meaning.

!!! admonition "See also"

    [`Galera status variable: wsrep_commit_oool`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-commit-oool)

### `wsrep_commit_window`

Average distance between highest and lowest concurrently committed sequence
number.

!!! admonition "See also"

    [`Galera status variable: wsrep_commit_window`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-commit-window)

### `wsrep_connected`

This variable shows if the node is connected to the cluster. If the value is
`OFF`, the node has not yet connected to any of the cluster components. This
may be due to misconfiguration.

!!! admonition "See also"

    [`Galera status variable: wsrep_connected`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-connected)

### `wsrep_evs_delayed`

Comma separated list of nodes that are considered delayed. The node format is
`<uuid>:<address>:<count>`, where `<count>` is the number of entries on
delayed list for that node.

!!! admonition "See also"

    [`Galera status variable: wsrep_evs_delayed`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-evs-delayed)

### `wsrep_evs_evict_list`

List of UUIDs of the evicted nodes.

!!! admonition "See also"

    [`Galera status variable: wsrep_evs_evict_list`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-evs-evict-list)

### `wsrep_evs_repl_latency`

This status variable provides information regarding group communication
replication latency. This latency is measured in seconds from when a message is
sent out to when a message is received.

The format of the output is `<min>/<avg>/<max>/<std_dev>/<sample_size>`.

!!! admonition "See also"

    [`Galera status variable: wsrep_evs_repl_latency`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-evs-repl-latency)

### `wsrep_evs_state`

Internal EVS protocol state.

!!! admonition "See also"

    [`Galera status variable: wsrep_evs_state`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-evs-state)

### `wsrep_flow_control_interval`

This variable shows the lower and upper limits for Galera flow control.
The upper limit is the maximum allowed number of requests in the queue.
If the queue reaches the upper limit, new requests are denied.
As existing requests get processed, the queue decreases,
and once it reaches the lower limit, new requests will be allowed again.

### `wsrep_flow_control_interval_high`

Shows the upper limit for flow control to trigger.

### `wsrep_flow_control_interval_low`

Shows the lower limit for flow control to stop.

### `wsrep_flow_control_paused`

Time since the last status query that was paused due to flow control.

!!! admonition "See also"

    [`Galera status variable: wsrep_flow_control_paused`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-flow-control-paused)

### `wsrep_flow_control_paused_ns`

Total time spent in a paused state measured in nanoseconds.

!!! admonition "See also"

    [`Galera status variable: wsrep_flow_control_paused_ns`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-flow-control-paused-ns)

### `wsrep_flow_control_recv`

The number of `FC_PAUSE` events received since the last status query. Unlike most status variables, this counter does not reset each time you run the query. This counter is reset when the server restarts. 

!!! admonition "See also"

    [`Galera status variable: wsrep_flow_control_recv`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-flow-control-recv)

### `wsrep_flow_control_requested`

This variable returns whether or not a node requested a replication pause.

### `wsrep_flow_control_sent`

The number of `FC_PAUSE` events sent since the last status query. Unlike most status variables, this counter does not reset each time you run the query. This counter is reset when the server restarts.

!!! admonition "See also"

    [`Galera status variable: wsrep_flow_control_sent`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-flow-control-sent)

### `wsrep_flow_control_status`

This variable shows whether a node has flow control enabled for normal traffic.
It does not indicate the status of flow control during SST.

### `wsrep_gcache_pool_size`

This variable shows the size of the page pool and dynamic memory allocated for
GCache (in bytes).

### `wsrep_gcomm_uuid`

This status variable exposes UUIDs in `gvwstate.dat`, which are Galera
view IDs (thus unrelated to cluster state UUIDs). This UUID is unique for each
node. You will need to know this value when using manual eviction feature.

!!! admonition "See also"

    [`Galera status variable: wsrep_gcomm_uuid`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-gcomm-uuid)

### `wsrep_incoming_addresses`

Shows the comma-separated list of incoming node addresses in the cluster.

!!! admonition "See also"

    [`Galera status variable: wsrep_incoming_addresses`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-incoming-addresses)

### `wsrep_ist_receive_status`

This variable displays the progress of IST for joiner node.
If IST is not running, the value is blank.
If IST is running, the value is the percentage of transfer completed.

### `wsrep_ist_receive_seqno_end`

The sequence number of the last transaction in IST.

### `wsrep_ist_receive_seqno_current`

The sequence number of the current transaction in IST.

### `wsrep_ist_receive_seqno_start`

The sequence number of the first transaction in IST.

### `wsrep_last_applied`

Sequence number of the last applied transaction.

### `wsrep_last_committed`

Sequence number of the last committed transaction.

### `wsrep_local_bf_aborts`

Number of local transactions that were aborted by replica transactions while
being executed.

!!! admonition "See also"

    [`Galera status variable: wsrep_local_bf_aborts`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-bf-aborts)

### `wsrep_local_cached_downto`

The lowest sequence number in GCache. This information can be helpful with
determining IST and SST. If the value is `0`, then it means there are no
writesets in GCache (usual for a single node).

!!! admonition "See also"

    [`Galera status variable: wsrep_local_cached_downto`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-cached-downto)

### `wsrep_local_cert_failures`

Number of writesets that failed the certification test.

!!! admonition "See also"

    [`Galera status variable: wsrep_local_cert_failures`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-cert-failures)

### `wsrep_local_commits`

Number of writesets commited on the node.

!!! admonition "See also"

    [`Galera status variable: wsrep_local_commits`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-commits)

### `wsrep_local_index`

Node’s index in the cluster.

!!! admonition "See also"

    [`Galera status variable: wsrep_local_index`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-index)

### `wsrep_local_recv_queue`

Current length of the receive queue (that is, the number of writesets waiting
to be applied).

!!! admonition "See also"

    [`Galera status variable: wsrep_local_recv_queue`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-recv-queue)

### `wsrep_local_recv_queue_avg`

Average length of the receive queue since the last status query. When this
number is bigger than `0` this means node can’t apply writesets as fast as
they are received. This could be a sign that the node is overloaded and it may
cause replication throttling.

!!! admonition "See also"

    [`Galera status variable: wsrep_local_recv_queue_avg`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-recv-queue-avg)

### `wsrep_local_replays`

Number of transaction replays due to *asymmetric lock granularity*.

!!! admonition "See also"

    [`Galera status variable: wsrep_local_replays`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-replays)

### `wsrep_local_send_queue`

Current length of the send queue (that is, the number of writesets waiting to
be sent).

!!! admonition "See also"

    [`Galera status variable: wsrep_local_send_queue`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-send-queue)

### `wsrep_local_send_queue_avg`

Average length of the send queue since the last status query. When cluster
experiences network throughput issues or replication throttling, this value
will be significantly bigger than `0`.

!!! admonition "See also"

    [`Galera status variable: wsrep_local_send_queue_avg`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-send-queue-avg)

### `wsrep_local_state`

Internal Galera cluster FSM state number.

!!! admonition "See also"

    [`Galera status variable: wsrep_local_state`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-state)

### `wsrep_local_state_comment`

Internal number and the corresponding human-readable comment of the node’s
state. Possible values are:

| Num | Comment         | Description                                       |
| --- | --------------- | ------------------------------------------------- |
| 1   | Joining         | Node is joining the cluster                       |
| 2   | Donor/Desynced  | Node is the donor to the node joining the cluster |
| 3   | Joined          | Node has joined the cluster                       |
| 4   | Synced          | Node is synced with the cluster                   |

!!! admonition "See also"

    [`Galera status variable: wsrep_local_state_comment`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-state-comment)

### `wsrep_local_state_uuid`

The [UUID](glossary.md#uuid) of the state stored on the node.

!!! admonition "See also"

    [`Galera status variable: wsrep_local_state_uuid`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-local-state-uuid)

### `wsrep_monitor_status`

The status of the local monitor (local and replicating actions), apply monitor
(apply actions of write-set), and commit monitor (commit actions of write
sets). In the value of this variable, each monitor (L: Local, A: Apply, C:
Commit) is represented as a *last_entered*, and *last_left* pair:

```{.text .no-copy}
wsrep_monitor_status (L/A/C)	[ ( 7, 5), (2, 2), ( 2, 2) ]
```

**last_entered**

Shows which transaction or write-set has recently entered the queue.

**last_left**

Shows which last transaction or write-set has been executed and left the queue.

According to the Galera protocol, transactions can be applied in parallel but
must be committed in a given order. This rule implies that there can be multiple
transactions in the *apply* state at a given point of time but transactions are
*committed* sequentially.

!!! admonition "See also"

    [`Galera Documentation: Database replication`](https://galeracluster.com/library/documentation/tech-desc-introduction.html)

### `wsrep_protocol_version`

Version of the wsrep protocol used.

!!! admonition "See also"

    [`Galera status variable: wsrep_protocol_version`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-protocol-version)

### `wsrep_provider_name`

Name of the wsrep provider (usually `Galera`).

!!! admonition "See also"

    [`Galera status variable: wsrep_provider_name`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-provider-name)

### `wsrep_provider_vendor`

Name of the wsrep provider vendor (usually `Codership Oy`)

!!! admonition "See also"

    [`Galera status variable: wsrep_provider_vendor`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-provider-vendor)

### `wsrep_provider_version`

Current version of the wsrep provider.

!!! admonition "See also"

    [`Galera status variable: wsrep_provider_version`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-provider-version)

### `wsrep_ready`

This variable shows if node is ready to accept queries. If status is `OFF`,
almost all queries will fail with `ERROR 1047 (08S01) Unknown Command` error
(unless the [`wsrep_on`](wsrep-system-index.md#wsrep_on) variable is set to `0`).

!!! admonition "See also"

    [`Galera status variable: wsrep_ready`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-ready)

### `wsrep_received`

Total number of writesets received from other nodes.

!!! admonition "See also"

    [`Galera status variable: wsrep_received`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-received)

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

Total number of writesets sent to other nodes.

!!! admonition "See also"

    [`Galera status variable: wsrep_replicated`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-replicated)

### `wsrep_replicated_bytes`

Total size of replicated writesets. To compute the actual size of bytes sent
over network to cluster peers, multiply the value of this variable by the number
of cluster peers in the given [`network segment`](wsrep-provider-index.md#gmcastsegment).

!!! admonition "See also"

    [`Galera status variable: wsrep_replicated_bytes`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-replicated-bytes)

### `wsrep_sync_wait`

#### What the variable measures

The `wsrep_sync_wait` status variable controls the level of consistency that client queries must observe in a Galera Cluster. It acts as a *synchronization barrier*. When enabled, queries such as `SELECT` wait until the node has applied the most recent transactions, ensuring that the results reflect up-to-date cluster state.  

This variable offers detailed control with bitmask options for different synchronization modes.

#### Expected results

The value of `wsrep_sync_wait` is an integer that represents a bitmask of synchronization modes:  

* `0` — No synchronization. Queries return immediately, without waiting for updates from the cluster. Fastest but may return stale data.  

* `1` — Synchronization for reads (`SELECT`). Ensures causal consistency by waiting for the latest writesets to apply before executing the query.  

* `2` — Synchronization for updates (`INSERT`, `UPDATE`, `DELETE`). Guarantees that updates see a consistent state.  

* `4` — Synchronization for transactions. Affects `START TRANSACTION` commands.  

* Values can be combined (for example, `wsrep_sync_wait = 3` applies both read and update synchronization).  

The recommended value depends on your application’s need for consistency versus performance. For most cases, `1` (causal reads) is sufficient.  

##### Example of use

You can run this command on any node in the cluster; it requires only the `PROCESS` privilege. Results reflect the status of the node where the query is executed.  

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_sync_wait';
```

??? example "Expected output"

    ```{.text .no-copy}
    +------------------+-------+
    | Variable_name    | Value |
    +------------------+-------+
    | wsrep_sync_wait  | 1     |
    +------------------+-------+
    ```

#### Related status variables

| Variable name | Description | Ideal value/interpretation |
|---------------|-------------|-----------------------------|
| [`wsrep_local_recv_queue_avg`](#wsrep_local_recv_queue_avg) | Shows the average length of the receive queue. | High values with `wsrep_sync_wait` enabled can cause queries to stall. |
| [`wsrep_flow_control_paused`](#wsrep_flow_control_paused) | Tracks how often flow control pauses replication. | High values increase latency for queries under `wsrep_sync_wait`. |
| [`wsrep_apply_oooe`](#wsrep_apply_oooe) | Indicates out-of-order execution efficiency for applying transactions. | Low values may increase the lag observed when `wsrep_sync_wait` is active. |

#### Monitor consistency waits

To get a comprehensive view of how `wsrep_sync_wait` interacts with replication and flow control, run this query:

```{.bash data-prompt="$"}
$ SHOW GLOBAL STATUS LIKE 'wsrep_sync_wait|wsrep_local_recv_queue_avg|wsrep_flow_control_paused|wsrep_apply_oooe';
```

This query helps you determine whether query stalls under wsrep_sync_wait are caused by:

* High replication apply lag, indicated by a large `wsrep_local_recv_queue_avg`

* Frequent flow control pauses, shown in `wsrep_flow_control_paused`

* Low parallelization efficiency, reflected in `wsrep_apply_oooe`

When analyzing the output, look for patterns such as:

* High `wsrep_local_recv_queue_avg` together with frequent flow control pauses suggests replication bottlenecks are causing queries to stall.

* Low `wsrep_apply_oooe` alongside non-zero `wsrep_sync_wait` may indicate that the node cannot apply transactions quickly enough, creating unnecessary delays for queries.



