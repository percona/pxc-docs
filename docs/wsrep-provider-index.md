# Index of wsrep_provider options

The following variables can be set and checked in the [`wsrep_provider_options`](wsrep-system-index.md#wsrep_provider_options) variable. The value of the variable can be
changed in the *MySQL* configuration file, `my.cnf`, or by setting the variable value in the *MySQL* client.

To change the value in `my.cnf`, the following syntax should be used:

```shell
wsrep_provider_options="variable1=value1;[variable2=value2]"
```

For example to set the size of the Galera buffer storage to 512 MB, specify the following in `my.cnf`:

```shell
wsrep_provider_options="gcache.size=512M"
```

Dynamic variables can be changed from the *MySQL* client using the `SET GLOBAL` command. For example, to change the value of the [`pc.ignore_sb`](wsrep-provider-index.md#pcignore_sb), use the following command:

```{.bash data-prompt="mysql>"}
mysql> SET GLOBAL wsrep_provider_options="pc.ignore_sb=true";
```

## Index

### `base_dir`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | value of `datadir` |

This variable specifies the data directory.

### `base_host`
| Option         |  Description       |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | value of [`wsrep_node_address`](wsrep-provider-index.md#wsrep_node_address)|

This variable sets the value of the node’s base IP. This is an IP address on
which Galera listens for connections from other nodes. Setting this value
incorrectly would stop the node from communicating with other nodes.

### `base_port`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 4567               |

This variable sets the port on which Galera listens for connections from other
nodes. Setting this value incorrectly would stop the node from communicating
with other nodes.

### `cert.log_conflicts`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | no                 |

This variable is used to specify if the details of the certification failures
should be logged.

### `cert.optimistic_pa`

**Enabled**

    Allows the full range of parallelization as determined by the certification
    algorithm.

**Disabled**

    Limits the parallel applying window so that it does not exceed the parallel
    applying window seen on the source. In this case, the action starts applying
    no sooner than all actions on the source are committed.

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |
| Default Value: | No                 |

!!! admonition "See also"

    Galera Cluster Documentation:
    * [Parameter: cert.optimistic_pa](https://galeracluster.com/library/documentation/galera-parameters.html#cert-optimistic-pa)
    * [Setting parallel slave threads](https://galeracluster.com/library/kb/parallel-slave-threads.html)

### `debug`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |
| Default Value: | no                 |

When this variable is set to `yes`, it will enable debugging.

### `evs.auto_evict`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | 0 |

Number of entries allowed on delayed list until auto eviction takes place.
Setting value to `0` disables auto eviction protocol on the node, though node
response times will still be monitored. EVS protocol version
([`evs.version`](wsrep-provider-index.md#evsversion)) `1` is required to enable auto eviction.

### `evs.causal_keepalive_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | value of [`evs.keepalive_period`](wsrep-provider-index.md#evskeepalive_period)|

This variable is used for development purposes and shouldn’t be used by regular
users.

### `evs.debug_log_mask`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |
| Default Value: | 0x1 |

This variable is used for EVS (Extended Virtual Synchrony) debugging. It can be
used only when [`wsrep_debug`](wsrep-system-index.md#wsrep_debug) is set to `ON`.

### `evs.delay_margin`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | PT1S |

Time period that a node can delay its response from expected until it is added
to delayed list. The value must be higher than the highest RTT between nodes.

### `evs.delayed_keep_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | PT30S |

Time period that node is required to remain responsive until one entry is
removed from delayed list.

### `evs.evict`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |

Manual eviction can be triggered by setting the [`evs.evict`](wsrep-provider-index.md#evsevict) to a
certain node value. Setting the [`evs.evict`](wsrep-provider-index.md#evsevict) to an empty string will
clear the evict list on the node where it was set.

### `evs.inactive_check_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT0.5S |

This variable defines how often to check for peer inactivity.

### `evs.inactive_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT15S |

This variable defines the inactivity limit, once this limit is reached the node
will be considered dead.

### `evs.info_log_mask`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0 |

This variable is used for controlling the extra EVS info logging.

### `evs.install_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | PT7.5S |

This variable defines the timeout on waiting for install message
acknowledgments.

### `evs.join_retrans_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT1S |

This variable defines how often to retransmit EVS join messages when forming
cluster membership.

### `evs.keepalive_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT1S |

This variable defines how often to emit keepalive beacons (in the absence of
any other traffic).

### `evs.max_install_timeouts`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 1 |

This variable defines how many membership install rounds to try before giving
up (total rounds will be [`evs.max_install_timeouts`](wsrep-provider-index.md#evsmax_install_timeouts) + 2).

### `evs.send_window`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 10 |

This variable defines the maximum number of data packets in replication at a
time. For WAN setups, the variable can be set to a considerably higher value
than default (for example,512). The value must not be less than [`evs.user_send_window`](wsrep-provider-index.md#evsuser_send_window).

### `evs.stats_report_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT1M   |

This variable defines the control period of EVS statistics reporting.

### `evs.suspect_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT5S   |

This variable defines the inactivity period after which the node is
"suspected" to be dead. If all remaining nodes agree on that, the node will be
dropped out of cluster even before [`evs.inactive_timeout`](wsrep-provider-index.md#evsinactive_timeout) is reached.

### `evs.use_aggregate`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | true   |

When this variable is enabled, smaller packets will be aggregated into one.

### `evs.user_send_window`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | 4   |

This variable defines the maximum number of data packets in replication at a
time. For WAN setups, the variable can be set to a considerably higher value
than default (for example, 512).

### `evs.version`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

This variable defines the EVS protocol version. Auto eviction is enabled when
this variable is set to `1`. Default `0` is set for backwards
compatibility.

### `evs.view_forget_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | P1D   |

This variable defines the timeout after which past views will be dropped from
history.

### `gcache.dir`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | [`datadir`](glossary.md#datadir)   |

This variable can be used to define the location of the `galera.cache`
file.

### `gcache.freeze_purge_at_seqno`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Local, Global             |
| Dynamic:       | Yes                 |
| Default Value: | 0   |

This variable controls the purging of the gcache and enables retaining
more data in it. This variable makes it possible to use [IST (Incremental State Transfer)](glossary.md#ist) when the node rejoins instead of
[SST (State Snapshot Transfer)](glossary.md#sst).

Set this variable on an existing node of the cluster (that will
continue to be part of the cluster and can act as a potential
[donor node](glossary.md#donor-node)). This node continues to retain the write-sets and allows restarting the node to rejoin by using [IST](glossary.md#ist).

!!! admonition "See also"

      Percona Database Performance Blog:

      * [All You Need to Know About GCache (Galera-Cache)](https://www.percona.com/blog/2016/11/16/all-you-need-to-know-about-gcache-galera-cache/)

      * [Want IST Not SST for Node Rejoins? We Have a Solution!](https://www.percona.com/blog/2018/02/13/no-sst-node-rejoins/)

The [`gcache.freeze_purge_at_seqno`](wsrep-provider-index.md#gcachefreeze_purge_at_seqno) variable takes three values:

**-1 (default)**

No freezing of gcache, the purge operates as normal.

**A valid seqno in gcache**

The freeze purge of write-sets may not be smaller than the selected seqno.
The best way to select an optimal value is to use the value of the
variable :variable:`wsrep_last_applied` from the node that you plan to shut down.

**now**
The freeze purge of write-sets is no less than the smallest seqno currently
in gcache. Using this value results in freezing the gcache-purge instantly.
Use this value if selecting a valid seqno in gcache is difficult.

### `gcache.keep_pages_count`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Local, Global             |
| Dynamic:       | Yes                 |
| Default Value: | 0   |

This variable is used to limit the number of overflow pages
rather than the total memory occupied by all overflow pages.
Whenever `gcache.keep_pages_count` is set to a non-zero value,
excess overflow pages will be deleted
(starting from the oldest to the newest).

Whenever either the `gcache.keep_pages_count`
or the [`gcache.keep_pages_size`](wsrep-provider-index.md#gcachekeep_pages_size) variable
is updated at runtime to a non-zero value,
cleanup is called on excess overflow pages to delete them.

### `gcache.keep_pages_size`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Local, Global            |
| Dynamic:       | No                 |
| Default Value: | 0   |

This variable is used to limit the total size of overflow pages
rather than the count of all overflow pages.
Whenever `gcache.keep_pages_size` is set to a non-zero value,
excess overflow pages will be deleted
(starting from the oldest to the newest)
until the total size is below the specified value.

Whenever either the [`gcache.keep_pages_count`](wsrep-provider-index.md#gcachekeep_pages_count) or the `gcache.keep_pages_size` variable
is updated at runtime to a non-zero value,
cleanup is called on excess overflow pages to delete them.

### `gcache.mem_size`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

This variable has been deprecated in `5.6.22-25.8` and shouldn’t be used as it
could cause a node to crash.

This variable was used to define how much RAM is available for the system.

### `gcache.name`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | /var/lib/mysql/galera.cache   |

This variable can be used to specify the name of the Galera cache file.

### `gcache.page_size`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | No                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 128M   |

Size of the page files in page storage. The limit on overall page storage is the
size of the disk. Pages are prefixed by gcache.page.

!!! admonition "See also"

      * [Galera Documentation: gcache.page_size](https://galeracluster.com/library/documentation/galera-parameters.html#gcache-page-size)

      * [Percona Database Performance Blog: All You Need to Know About GCache](https://www.percona.com/blog/2016/11/16/all-you-need-to-know-about-gcache-galera-cache/)

### `gcache.recover`

| Option              | Description |
|---------------------|-------------|
| Command line:       | No          |
| Configuration file: | Yes         |
| Scope:              | Global      |
| Dynamic:            | No          |
| Default value:      | No          |

Attempts to recover a node’s gcache file to a usable state on startup. If the node can successfully recover the gcache file, the node can provide IST to the remaining nodes. This ability can reduce the time needed to bring up the cluster.

An example of enabling the variable in the configuration file:

```text
wsrep_provider_options="gcache.recover=yes"
```


### `gcache.size`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 128M   |

Size of the transaction cache for Galera replication. This defines the size of
the `galera.cache` file which is used as source for [IST](glossary.md#ist). The bigger the
value of this variable, the better are chances that the re-joining node will
get IST instead of [SST](glossary.md#sst).

### `gcomm.thread_prio`

Using this option, you can raise the priority of the gcomm thread to a higher
level than it normally uses.

The format for this variable is: &#60;policy&#62;:&#60;priority&#62;. The priority value is an integer.

other

    Default time-sharing scheduling in Linux. The threads can run
    until blocked by an I/O request or preempted by higher priorities or
    superior scheduling designations.

fifo

    First-in First-out (FIFO) scheduling. These threads always immediately
    preempt any currently running other, batch or idle threads. They can run
    until they are either blocked by an I/O request or preempted by a FIFO thread
    of a higher priority.

rr

    Round-robin scheduling. These threads always preempt any currently running
    other, batch or idle threads. The scheduler allows these threads to run for a
    fixed period of a time. If the thread is still running when this time period is
    exceeded, they are stopped and moved to the end of the list, allowing another
    round-robin thread of the same priority to run in their place. They can
    otherwise continue to run until they are blocked by an I/O request or are
    preempted by threads of a higher priority.

!!! admonition "See also"

    For information, see the [Galera Cluster documentation](https://galeracluster.com/library/documentation/galera-parameters.html#gcomm-thread-prio)

### `gcs.fc_auto_evict_threshold`

| Option | Description |
|---|---|
| Command Line: | Yes |
| Config file : | Yes |
| Scope: | Global |
| Dynamic: | No |
| Default value: | 0.75 |

Implemented in Percona XtraDB Cluster 8.0.33-25.

Defines the threshold that must be reached or crossed before a node is evicted from the cluster. This variable is a ratio of the [`gcs.fc_auto_evict_window`](#gcs.fc_auto_evict_window) variable. The default value is `.075`, but the value can be set to any value between 0.0 and 1.0. 

### `gcs.fc_auto_evict_window`

| Option | Description |
|---|---|
| Command Line: | Yes |
| Config file : | Yes |
| Scope: | Global |
| Dynamic: | No |
| Default value: | 0 |

Implemented in Percona XtraDB Cluster 8.0.33-25.

The variable defines the time window width within which flow controls are observed. The time span of the window is [now - gcs.fc_audot_evict_window, now]. The window is constantly moving ahead as the time passes. And now, within this window if the flow control summary time >= (gcs.fc_audot-evict_window * gcs.fc_audot_evict_threshold), the node self-leaves the cluster.

The default value is 0, which means that the feature is disabled.

The maximum value is `DBL_MAX`.

### `gcs.fc_debug`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

This variable specifies after how many writesets the debug statistics about SST
flow control will be posted.

### `gcs.fc_factor`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | 1   |

This variable is used for replication flow control. Replication is resumed when
the replica queue drops below [`gcs.fc_factor`](wsrep-provider-index.md#gcsfc_factor) * [`gcs.fc_limit`](wsrep-provider-index.md#gcsfc_limit).

### `gcs.fc_limit`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | 100   |

This variable is used for replication flow control. Replication is paused when
the replica queue exceeds this limit. In the default operation mode, flow control
limit is dynamically recalculated based on the amount of nodes in the
cluster, but this recalculation can be turned off with use of the [`gcs.fc_master_slave`](wsrep-provider-index.md#gcsfc_master_slave) variable to make manual setting of the [`gcs.fc_limit`](wsrep-provider-index.md#gcsfc_limit) having an effect  (e.g., for configurations when writing is done to a single node in Percona XtraDB Cluster).

### `gcs.fc_master_slave`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | NO                 |
| Default Value: | NO   |

This variable is used to specify if there is only one source node in the
cluster. It affects whether flow control limit is recalculated dynamically
(when `NO`) or not (when `YES`).

### `gcs.max_packet_size`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 64500   |

This variable is used to specify the writeset size after which they will be fragmented.

### `gcs.max_throttle`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0.25   |

This variable specifies how much the replication can be throttled during the
state transfer in order to avoid running out of memory. Value can be set to
`0.0` if stopping replication is acceptable in order to finish state
transfer.

### `gcs.recv_q_hard_limit`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 9223372036854775807   |

This variable specifies the maximum allowed size of the receive queue. This
should normally be `(RAM + swap) / 2`. If this limit is exceeded, Galera will
abort the server.

### `gcs.recv_q_soft_limit`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0.25   |

This variable specifies the fraction of the [`gcs.recv_q_hard_limit`](wsrep-provider-index.md#gcsrecv_q_hard_limit) after which replication rate will be throttled.

### `gcs.sync_donor`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | No   |

This variable controls if the rest of the cluster should be in sync with the
donor node. When this variable is set to `YES`, the whole cluster will be
blocked if the donor node is blocked with SST.

### `gmcast.listen_addr`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | tcp://0.0.0.0:4567   |

This variable defines the address on which the node listens to connections from
other nodes in the cluster.

### `gmcast.mcast_addr`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | None   |

This variable should be set up if UDP multicast should be used for replication.

### `gmcast.mcast_ttl`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 1   |

This variable can be used to define TTL for multicast packets.

### `gmcast.peer_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT3S   |

This variable specifies the connection timeout to initiate message relaying.

### `gmcast.segment`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

This variable specifies the group segment this member should be a part of. Same
segment members are treated as equally physically close.

### `gmcast.time_wait`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT5S   |

This variable specifies the time to wait until allowing peer declared outside
of stable view to reconnect.

### `gmcast.version`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

This variable shows which gmcast protocol version is being used.

### `ist.recv_addr`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | value of [`wsrep_node_address`](wsrep-provider-index.md#wsrep_node_address)   |

This variable specifies the address on which the node listens for Incremental
State Transfer ([IST](glossary.md#ist)).

### `pc.announce_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT3S   |

Cluster joining announcements are sent every 1/2 second for this period of time
or less if other nodes are discovered.

### `pc.checksum`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | true   |

This variable controls whether replicated messages should be checksummed or
not.

### `pc.ignore_quorum`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | false   |

When this variable is set to `TRUE`, the node will completely ignore quorum
calculations. This should be used with extreme caution even in source-replica
setups, because replicas won’t automatically reconnect to source in this case.

### `pc.ignore_sb`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | false   |

When this variable is set to `TRUE`, the node will process updates even in
the case of a split brain. This should be used with extreme caution in
multi-source setup, but should simplify things in source-replica cluster
(especially if only 2 nodes are used).

### `pc.linger`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT20S   |

This variable specifies the period for which the PC protocol waits for EVS
termination.

### `pc.npvo`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | false   |

When this variable is set to `TRUE`, more recent primary components override
older ones in case of conflicting primaries.

### `pc.recovery`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | true   |

When this variable is set to `true`, the node stores the Primary Component
state to disk. The Primary Component can then recover automatically when all
nodes that were part of the last saved state re-establish communication with
each other. This feature allows automatic recovery from full cluster crashes,
such as in the case of a data center power outage. A subsequent graceful full
cluster restart will require explicit bootstrapping for a new Primary
Component.

### `pc.version`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

This status variable is used to check which PC protocol version is used.

### `pc.wait_prim`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | true   |

When set to `TRUE`, the node waits for a primary component for the period of
time specified in [`pc.wait_prim_timeout`](wsrep-provider-index.md#pcwait_prim_timeout). This is useful to bring up a non-primary component and make it primary with [`pc.bootstrap`](wsrep-provider-index.md#pcbootstrap).

### `pc.wait_prim_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT30S   |

This variable is used to specify the period of time to wait for a primary
component.

### `pc.weight`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | 1   |

This variable specifies the node weight that’s going to be used for Weighted
Quorum calculations.

### `protonet.backend`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | asio   |

This variable is used to define which transport backend should be used.
Currently only `ASIO` is supported.

### `protonet.version`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

This status variable is used to check which transport backend protocol version
is used.

### `repl.causal_read_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | PT30S   |

This variable specifies the causal read timeout.

### `repl.commit_order`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 3   |

This variable is used to specify out-of-order committing (which is used to
improve parallel applying performance). The following values are available:

* `0` - BYPASS: all commit order monitoring is turned off (useful for measuring performance penalty)

* `1` - OOOC: allow out-of-order committing for all transactions

* `2` - LOCAL_OOOC: allow out-of-order committing only for local transactions

* `3` - NO_OOOC: no out-of-order committing is allowed (strict total order committing)

### `repl.key_format`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | FLAT8   |

This variable is used to specify the replication key format. The following
values are available:
 
* `FLAT8` - short key with higher probability of key match false positives

* `FLAT16` - longer key with lower probability of false positives

* `FLAT8A` - same as `FLAT8` but with annotations for debug purposes

* `FLAT16A` - same as `FLAT16` but with annotations for debug purposes

### `repl.max_ws_size`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 2147483647   |

This variable is used to specify the maximum size of a write-set in bytes. This
is limited to 2 gygabytes.

### `repl.proto_max`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 7   |

This variable is used to specify the highest communication protocol version to
accept in the cluster. Used only for debugging.

### `socket.checksum`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 2   |

This variable is used to choose the checksum algorithm for network packets. The ``CRC32-C`` option is optimized and may be hardware accelerated on Intel CPUs. The following values are available:

* `0` - disable checksum

* `1` - plain `CRC32` (used in Galera 2.x)

* `2` - hardware accelerated `CRC32-C`

The following is an example of the variable use:

```text
wsrep_provider_options="socket.checksum=2"
```

### `socket.ssl`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | No                 |

This variable is used to specify if SSL encryption should be used.

### `socket.ssl_ca`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |

This variable is used to specify the path
to the Certificate Authority (CA) certificate file.

### `socket.ssl_cert`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |

This variable is used to specify the path
to the server’s certificate file (in PEM format).

### `socket.ssl_key`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |

This variable is used to specify the path
to the server’s private key file (in PEM format).

### `socket.ssl_compression`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | Yes                |

This variable is used to specify if the SSL compression is to be used.

### `socket.ssl_cipher`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | AES128-SHA         |

This variable is used to specify what cypher will be used for encryption.
