# Index of [`wsrep_provider`](wsrep-system-index.md#wsrep_provider_options) options

The following variables can be set and checked in the [`wsrep_provider_options`](wsrep-system-index.md#wsrep_provider_options) variable. The value of the variable can be
changed in the *MySQL* configuration file, `my.cnf`, or by setting the variable value in the *MySQL* client.

To change the value in `my.cnf`, the following syntax should be used:

```default
wsrep_provider_options="variable1=value1;[variable2=value2]"
```

For example to set the size of the Galera buffer storage to 512 MB, specify the following in `my.cnf`:

```default
wsrep_provider_options="gcache.size=512M"
```

Dynamic variables can be changed from the *MySQL* client using the `SET GLOBAL` command. For example, to change the value of the [`pc.ignore_sb`](wsrep-provider-index.md#pc.ignore_sb), use the following command:

```mysql
mysql> SET GLOBAL wsrep_provider_options="pc.ignore_sb=true";
```

## Index

### `base_dir`
|  Command Line: | Yes                |
| -------------- | ------------------ |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | value of `datadir` |

This variable specifies the data directory.

### `base_host`
| Command Line:  |  Yes               |
| -------------- | ------------------ |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | value of `wsrep_node_address` |

This variable sets the value of the node’s base IP. This is an IP address on
which Galera listens for connections from other nodes. Setting this value
incorrectly would stop the node from communicating with other nodes.

This variable sets the port on which Galera listens for connections from other
nodes. Setting this value incorrectly would stop the node from communicating
with other nodes.

This variable is used to specify if the details of the certification failures
should be logged.

When this variable is set to `yes`, it will enable debugging.

Number of entries allowed on delayed list until auto eviction takes place.
Setting value to `0` disables auto eviction protocol on the node, though node
response times will still be monitored. EVS protocol version
(

```
:variable:`evs.version`
```

) `1` is required to enable auto eviction.

This variable is used for development purposes and shouldn’t be used by regular
users.

This variable is used for EVS (Extended Virtual Synchrony) debugging. It can be
used only when 

```
:variable:`wsrep_debug`
```

 is set to `ON`.

Time period that a node can delay its response from expected until it is added
to delayed list. The value must be higher than the highest RTT between nodes.

Time period that node is required to remain responsive until one entry is
removed from delayed list.

Manual eviction can be triggered by setting the 

```
:variable:`evs.evict`
```

 to a
certain node value. Setting the 

```
:variable:`evs.evict`
```

 to an empty string will
clear the evict list on the node where it was set.

This variable defines how often to check for peer inactivity.

This variable defines the inactivity limit, once this limit is reached the node
will be considered dead.

This variable is used for controlling the extra EVS info logging.

This variable defines the timeout on waiting for install message
acknowledgments.

This variable defines how often to retransmit EVS join messages when forming
cluster membership.

This variable defines how often to emit keepalive beacons (in the absence of
any other traffic).

This variable defines how many membership install rounds to try before giving
up (total rounds will be 

```
:variable:`evs.max_install_timeouts`
```

 + 2).

This variable defines the maximum number of data packets in replication at a
time. For WAN setups, the variable can be set to a considerably higher value
than default (for example,512). The value must not be less than


```
:variable:`evs.user_send_window`
```

.

This variable defines the control period of EVS statistics reporting.

This variable defines the inactivity period after which the node is
“suspected” to be dead. If all remaining nodes agree on that, the node will be
dropped out of cluster even before 

```
:variable:`evs.inactive_timeout`
```

 is reached.

When this variable is enabled, smaller packets will be aggregated into one.

This variable defines the maximum number of data packets in replication at a
time. For WAN setups, the variable can be set to a considerably higher value
than default (for example, 512).

This variable defines the EVS protocol version. Auto eviction is enabled when
this variable is set to `1`. Default `0` is set for backwards
compatibility.

This variable defines the timeout after which past views will be dropped from
history.

This variable can be used to define the location of the `galera.cache`
file.

This variable controls the purging of the gcache and enables retaining
more data in it. This variable makes it possible to use [IST
(Incremental State Transfer)](glossary.md#term-IST) when the node rejoins instead of
[SST (State Snapshot Transfer)](glossary.md#term-SST).

Set this variable on an existing node of the cluster (that will
continue to be part of the cluster and can act as a potential
[donor node](glossary.md#term-donor-node)). This node continues to retain the write-sets and
allows restarting the node to rejoin by using [IST](glossary.md#term-IST).

The 

```
:variable:`gcache.freeze_purge_at_seqno`
```

 variable takes three values:

-1 (default)

    No freezing of gcache, the purge operates as normal.

A valid seqno in gcache

    The freeze purge of write-sets may not be smaller than the selected seqno.
    The best way to select an optimal value is to use the value of the
    variable 

    ```
    :variable:`wsrep_last_applied`
    ```

     from the node that you plan to shut down.

*now*

    The freeze purge of write-sets is no less than the smallest seqno currently
    in gcache. Using this value results in freezing the gcache-purge instantly.
    Use this value if selecting a valid seqno in gcache is difficult.

This variable is used to limit the number of overflow pages
rather than the total memory occupied by all overflow pages.
Whenever `gcache.keep_pages_count` is set to a non-zero value,
excess overflow pages will be deleted
(starting from the oldest to the newest).

Whenever either the `gcache.keep_pages_count`
or the 

```
:variable:`gcache.keep_pages_size`
```

 variable
is updated at runtime to a non-zero value,
cleanup is called on excess overflow pages to delete them.

This variable is used to limit the total size of overflow pages
rather than the count of all overflow pages.
Whenever `gcache.keep_pages_size` is set to a non-zero value,
excess overflow pages will be deleted
(starting from the oldest to the newest)
until the total size is below the specified value.

Whenever either the 

```
:variable:`gcache.keep_pages_count`
```


or the `gcache.keep_pages_size` variable
is updated at runtime to a non-zero value,
cleanup is called on excess overflow pages to delete them.

This variable was used to define how much RAM is available for the system.

**WARNING**: This variable has been deprecated and shouldn’t be used as it
could cause a node to crash.

This variable can be used to specify the name of the Galera cache file.

Size of the page files in page storage. The limit on overall page storage is the
size of the disk. Pages are prefixed by gcache.page.

Attempts to recover a node’s gcache file to a usable state on startup. If the node can successfully
recover the gcache file, the node can provide IST to the remaining nodes. This ability can
reduce the time needed to bring up the cluster.

An example of setting the value to yes in the configuration file:

```text
wsrep_provider_options="gcache.recover=yes"
```

Size of the transaction cache for Galera replication. This defines the size of
the `galera.cache` file which is used as source for [IST](glossary.md#term-IST). The bigger the
value of this variable, the better are chances that the re-joining node will
get IST instead of [SST](glossary.md#term-SST).

Raises the gcomm thread priority to a higher level. Use this variable when the gcomm thread does not receive enough CPU time due to other competing threads. For example, if the gcomm threads are not frequently run, a node may drop from the cluster because of the timeout.

The format for this variable is: <policy>:<priority>. The policy value supports the following options: `other`, `fifo`, and `rr`. The priority value is an
integer.

**NOTE**: Setting a priority value of 99 is not recommended. This value blocks system threads.

An example of the variable:

```text
wsrep_provider_options="gcomm.thread_prio=fifo:3"
```

The description of the `policy` parameter follows:

| Option

 | Description

 |
| ---------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| other

                                                                        | This policy is the default Linux time-sharing scheduling. Threads run until one of the following events occur:

> 
> * Thread exit


> * I/O request blocks the thread


> * Higher priority thread preempts the thread

                                                                            |
| fifo

                                                                         | The policy uses a First-in First-out (FIFO) scheduling. These threads always immediately preempt any currently running other, batch or idle threads. The threads are run in a FIFO manner until completion, unless a higher priority thread preempts or blocks them. This policy does not use time slicing.

 |
| rr

                                                                           | The threads use round-robin scheduling. This thread always preempts a currently running other, batch or idle thread. The scheduler runs threads with the same priority for a fixed time in a round-robin style. When this time period is exceeded, the scheduler stops the thread and moves it to the end of the list, and runs another round-robin thread with the same priority.

 |
This variable specifies after how many writesets the debug statistics about SST
flow control will be posted.

This variable is used for replication flow control. Replication is resumed when
the replica queue drops below 

```
:variable:`gcs.fc_factor`
```

 \*


```
:variable:`gcs.fc_limit`
```

.

This variable is used for replication flow control. Replication is paused when
the replica queue exceeds this limit. In the default operation mode, flow control
limit is dynamically recalculated based on the amount of nodes in the
cluster, but this recalculation can be turned off with use of the


```
:variable:`gcs.fc_master_slave`
```

 variable to make manual setting of the 

```
:variable:`gcs.fc_limit`
```

 having an effect  (e.g. for configurations
when writing is done to a single node in Percona XtraDB Cluster).

This variable is used to specify if there is only one source node in the
cluster. It affects whether flow control limit is recalculated dynamically
(when `NO`) or not (when `YES`).

This variable is used to specify the writeset size after which they will be
fragmented.

This variable specifies how much the replication can be throttled during the
state transfer in order to avoid running out of memory. Value can be set to
`0.0` if stopping replication is acceptable in order to finish state
transfer.

This variable specifies the maximum allowed size of the receive queue. This
should normally be `(RAM + swap) / 2`. If this limit is exceeded, Galera will
abort the server.

This variable specifies the fraction of the 

```
:variable:`gcs.recv_q_hard_limit`
```


after which replication rate will be throttled.

This variable controls if the rest of the cluster should be in sync with the
donor node. When this variable is set to `YES`, the whole cluster will be
blocked if the donor node is blocked with SST.

This variable defines the address on which the node listens to connections from
other nodes in the cluster.

This variable should be set up if UDP multicast should be used for replication.

This variable can be used to define TTL for multicast packets.

This variable specifies the connection timeout to initiate message relaying.

This variable specifies the group segment this member should be a part of. Same
segment members are treated as equally physically close.

This variable specifies the time to wait until allowing peer declared outside
of stable view to reconnect.

This variable shows which gmcast protocol version is being used.

This variable specifies the address on which the node listens for Incremental
State Transfer ([IST](glossary.md#term-IST)).

Cluster joining announcements are sent every 1/2 second for this period of time
or less if other nodes are discovered.

This variable controls whether replicated messages should be checksummed or
not.

When this variable is set to `TRUE`, the node will completely ignore quorum
calculations. This should be used with extreme caution even in source-replica
setups, because replicas won’t automatically reconnect to source in this case.

When this variable is set to `TRUE`, the node will process updates even in
the case of a split brain. This should be used with extreme caution in
multi-source setup, but should simplify things in source-replica cluster
(especially if only 2 nodes are used).

This variable specifies the period for which the PC protocol waits for EVS
termination.

When this variable is set to `TRUE`, more recent primary components override
older ones in case of conflicting primaries.

When this variable is set to `true`, the node stores the Primary Component
state to disk. The Primary Component can then recover automatically when all
nodes that were part of the last saved state re-establish communication with
each other. This feature allows automatic recovery from full cluster crashes,
such as in the case of a data center power outage. A subsequent graceful full
cluster restart will require explicit bootstrapping for a new Primary
Component.

This status variable is used to check which PC protocol version is used.

When set to `TRUE`, the node waits for a primary component for the period of
time specified in 

```
:variable:`pc.wait_prim_timeout`
```

. This is useful to bring up
a non-primary component and make it primary with 

```
:variable:`pc.bootstrap`
```

.

This variable is used to specify the period of time to wait for a primary
component.

This variable specifies the node weight that’s going to be used for Weighted
Quorum calculations.

This variable is used to define which transport backend should be used.
Currently only `ASIO` is supported.

This status variable is used to check which transport backend protocol version
is used.

This variable specifies the causal read timeout.

This variable is used to specify out-of-order committing (which is used to
improve parallel applying performance). The following values are available:

> 
> * `0` - BYPASS: all commit order monitoring is turned off (useful for
> measuring performance penalty)


> * `1` - OOOC: allow out-of-order committing for all transactions


> * `2` - LOCAL_OOOC: allow out-of-order committing only for local
> transactions


> * `3` - NO_OOOC: no out-of-order committing is allowed (strict total order
> committing)

This variable is used to specify the replication key format. The following
values are available:

> 
> * `FLAT8` - short key with higher probability of key match false positives


> * `FLAT16` - longer key with lower probability of false positives


> * `FLAT8A` - same as `FLAT8` but with annotations for debug purposes


> * `FLAT16A` - same as `FLAT16` but with annotations for debug purposes

This variable is used to specify the maximum size of a write-set in bytes. This
is limited to 2 gygabytes.

This variable is used to specify the highest communication protocol version to
accept in the cluster. Used only for debugging.

This variable is used to choose the checksum algorithm for network packets. The
following values are available:

> 
> * `0` - disable checksum


> * `1` - plain `CRC32` (used in Galera 2.x)


> * `2` - hardware accelerated `CRC32-C`

This variable is used to specify if SSL encryption should be used.

This variable is used to specify the path
to the Certificate Authority (CA) certificate file.

This variable is used to specify the path
to the server’s certificate file (in PEM format).

This variable is used to specify the path
to the server’s private key file (in PEM format).

This variable is used to specify if the SSL compression is to be used.

This variable is used to specify what cypher will be used for encryption.

<!-- Products -->
<!-- Platforms and rel. products -->
