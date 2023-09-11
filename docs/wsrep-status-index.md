# Index of wsrep status variables

### `wsrep_apply_oooe`

This variable shows parallelization efficiency, how often writests have been
applied out of order.

!!! admonition "See also"

    [`Galera status variable: wsrep_apply_oooe`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-apply-oooe)

### `wsrep_apply_oool`

This variable shows how often a writeset with a higher sequence number was
applied before one with a lower sequence number.

!!! admonition "See also"

    [`Galera status variable: wsrep_apply_oool`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-apply-oool)

### `wsrep_apply_window`

Average distance between highest and lowest concurrently applied sequence
numbers.

!!! admonition "See also"

    [`Galera status variable: wsrep_apply_window`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-apply-window)

### `wsrep_causal_reads`

Shows the number of writesets processed while the variable [`wsrep_causal_reads`](wsrep-system-index.md#wsrep_causal_reads) was set to `ON`.

!!! admonition "See also"

    [`MySQL wsrep options: wsrep_causal_reads`](https://galeracluster.com/library/documentation/mysql-wsrep-options.html#wsrep-causal-reads)

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
of cluster peers in the given [`network segment`](wsrep-provider-index.md#gmcast.segment).

!!! admonition "See also"

    [`Galera status variable: wsrep_replicated_bytes`](https://galeracluster.com/library/documentation/galera-status-variables.html#wsrep-replicated-bytes)