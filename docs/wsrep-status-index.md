# Index of wsrep status variables

This variable shows parallelization efficiency, how often writests have been
applied out of order.

This variable shows how often a writeset with a higher sequence number was
applied before one with a lower sequence number.

Average distance between highest and lowest concurrently applied sequence
numbers.

Shows the number of writesets processed while the variable


```
:variable:`wsrep_causal_reads`
```

 was set to `ON`.

This variable, shows the number of cells in the certification index
hash-table.

Average distance between highest and lowest sequence number that can be
possibly applied in parallel.

Number of entries in the certification index.

Average number of write-sets received while a transaction replicates.

Number of cluster membership changes that have taken place.

Current number of nodes in the cluster.

This variable contains [UUID](glossary.md#term-UUID) state of the cluster. When this value is
the same as the one in 

```
:variable:`wsrep_local_state_uuid`
```

, node is synced with
the cluster.

Status of the cluster component. Possible values are:

> 
> * `Primary`


> * `Non-Primary`


> * `Disconnected`

This variable shows how often a transaction was committed out of order.

This variable currently has no meaning.

Average distance between highest and lowest concurrently committed sequence
number.

This variable shows if the node is connected to the cluster. If the value is
`OFF`, the node has not yet connected to any of the cluster components. This
may be due to misconfiguration.

Comma separated list of nodes that are considered delayed. The node format is
`<uuid>:<address>:<count>`, where `<count>` is the number of entries on
delayed list for that node.

List of UUIDs of the evicted nodes.

This status variable provides information regarding group communication
replication latency. This latency is measured in seconds from when a message is
sent out to when a message is received.

The format of the output is `<min>/<avg>/<max>/<std_dev>/<sample_size>`.

Internal EVS protocol state.

This variable shows the lower and upper limits for Galera flow control.
The upper limit is the maximum allowed number of requests in the queue.
If the queue reaches the upper limit, new requests are denied.
As existing requests get processed, the queue decreases,
and once it reaches the lower limit, new requests will be allowed again.

Shows the upper limit for flow control to trigger.

Shows the lower limit for flow control to stop.

Time since the last status query that was paused due to flow control.

Total time spent in a paused state measured in nanoseconds.

The number of `FC_PAUSE` events received since the last status query. Unlike most status variables, this counter does not reset each time you run the query. This counter is reset when the server restarts.

The number of `FC_PAUSE` events sent since the last status query. Unlike most status variables, this counter does not reset each time you run the query. This counter is reset when the server restarts.

This variable shows whether a node has flow control enabled for normal traffic.
It does not indicate the status of flow control during SST.

This variable shows the size of the page pool and dynamic memory allocated for
GCache (in bytes).

This status variable exposes UUIDs in `gvwstate.dat`, which are Galera
view IDs (thus unrelated to cluster state UUIDs). This UUID is unique for each
node. You will need to know this value when using manual eviction feature.

Shows the comma-separated list of incoming node addresses in the cluster.

Displays the progress of IST for joiner node.
If IST is not running, the value is blank.
If IST is running, the value is the percentage of transfer completed.

The sequence number of the last transaction in IST.

The sequence number of the current transaction in IST.

The sequence number of the first transaction in IST.

Sequence number of the last applied transaction.

Sequence number of the last committed transaction.

Number of local transactions that were aborted by replica transactions while
being executed.

The lowest sequence number in GCache. This information can be helpful with
determining IST and SST. If the value is `0`, then it means there are no
writesets in GCache (usual for a single node).

Number of writesets that failed the certification test.

Number of writesets commited on the node.

Node’s index in the cluster.

Current length of the receive queue (that is, the number of writesets waiting
to be applied).

Average length of the receive queue since the last status query. When this
number is bigger than `0` this means node can’t apply writesets as fast as
they are received. This could be a sign that the node is overloaded and it may
cause replication throttling.

Number of transaction replays due to *asymmetric lock granularity*.

Current length of the send queue (that is, the number of writesets waiting to
be sent).

Average length of the send queue since the last status query. When cluster
experiences network throughput issues or replication throttling, this value
will be significantly bigger than `0`.

Internal number and the corresponding human-readable comment of the node’s
state. Possible values are:

| Num

 | Comment

 | Description

 |
| ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 1

                                                                            | Joining

                                                                                                                                                                                                                                                                                                                                                                            | Node is joining the cluster

                                                                                                                                                            |
| 2

                                                                            | Donor/Desynced

                                                                                                                                                                                                                                                                                                                                                                     | Node is the donor to the node joining the cluster

                                                                                                                                      |
| 3

                                                                            | Joined

                                                                                                                                                                                                                                                                                                                                                                             | Node has joined the cluster

                                                                                                                                                            |
| 4

                                                                            | Synced

                                                                                                                                                                                                                                                                                                                                                                             | Node is synced with the cluster

                                                                                                                                                        |
The [UUID](glossary.md#term-UUID) of the state stored on the node.

Version of the wsrep protocol used.

Name of the wsrep provider (usually `Galera`).

Name of the wsrep provider vendor (usually `Codership Oy`)

Current version of the wsrep provider.

This variable shows if node is ready to accept queries. If status is `OFF`,
almost all queries will fail with `ERROR 1047 (08S01) Unknown Command` error
(unless the 

```
:variable:`wsrep_on`
```

 variable is set to `0`).

Total number of writesets received from other nodes.

Total size (in bytes) of writesets received from other nodes.

Total size (in bytes) of data replicated.

Total number of keys replicated.

Total size (in bytes) of keys replicated.

Total size of other bits replicated.

Total number of writesets sent to other nodes.

Total size (in bytes) of writesets sent to other nodes.
