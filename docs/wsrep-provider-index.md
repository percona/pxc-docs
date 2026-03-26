
# Index of wsrep_provider options

You can set and check the following options in the [`wsrep_provider_options`](wsrep-system-index.md#wsrep_provider_options) system variable. You can change option values in the *MySQL* configuration file, `my.cnf`, or by setting options in the *MySQL* client.

To change the value in `my.cnf`, use the following syntax:

```shell
wsrep_provider_options="option1=value1;[option2=value2]"
```

For example, to set the size of the Galera buffer storage (gcache) to 512 MB, specify the following in `my.cnf`:

```shell
wsrep_provider_options="gcache.size=512M"
```

You can change dynamic options from the *MySQL* client using the `SET GLOBAL` command (for example, `SET GLOBAL wsrep_provider_options="option=value";`). For each option below, only the config file syntax is shown; options with Dynamic: Yes in the table can also be set at runtime in the same way.

## Index

### `base_dir`
| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | value of `datadir` |

This option specifies the data directory.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="base_dir=/var/lib/mysql"
```

### `base_host`
| Option         |  Description       |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | value of `wsrep_node_address`|

This option sets the value of the node’s base IP. This is an IP address on which Galera listens for connections from other nodes. Setting this value incorrectly would stop the node from communicating with other nodes.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="base_host=/var/lib/mysql"
```

### `base_port`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 4567               |

This option sets the port on which Galera listens for connections from other nodes. Setting this value incorrectly would stop the node from communicating with other nodes.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="base_port=4567"
```

### `cert.log_conflicts`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | no                 |

This option specifies whether the cluster logs the details of certification failures.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="cert.log_conflicts=yes"
```

### `cert.optimistic_pa`

This option controls whether parallel applying of replicated transactions is optimistic (allowed to run ahead of the source) or conservative (limited to the source’s apply window). The default is `No` (Disabled).

Enabled (`Yes`): Allows the full range of parallelization as determined by the certification algorithm.

Disabled (`No`, default): Limits the parallel applying window so that the window does not exceed the parallel applying window seen on the source. In this case, the action starts applying no sooner than all actions on the source are committed.

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |
| Default Value: | No                 |

### `debug`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |
| Default Value: | no                 |

When you set this option to `yes`, debugging is enabled.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="debug=yes"
```

### `evs.auto_evict`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | 0 |

This option specifies the number of entries allowed on the delayed list until auto eviction runs. Set the value to `0` to disable auto eviction on the node (the cluster still monitors node response times). Auto eviction requires EVS (Extended Virtual Synchrony) protocol version ([`evs.version`](wsrep-provider-index.md#evsversion)) `1`.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.auto_evict=5"
```

### `evs.causal_keepalive_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | value of [`evs.keepalive_period`](wsrep-provider-index.md#evskeepalive_period)|

Use this option only for development. Do not use the option in production.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.causal_keepalive_period=/var/lib/mysql"
```

### `evs.debug_log_mask`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |
| Default Value: | 0x1 |

Use this option for EVS (Extended Virtual Synchrony) debugging. You can use the option only when [`wsrep_debug`](wsrep-system-index.md#wsrep_debug) is set to `ON`.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.debug_log_mask=0x1"
```

### `evs.delay_margin`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | PT1S |

This option specifies the time period that a node can delay its response from expected before the node is added to the delayed list. The value must be higher than the highest RTT (round-trip time) between nodes.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.delay_margin=PT30S"
```

### `evs.delayed_keep_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | PT30S |

This option specifies the time period that a node must remain responsive before one entry is removed from the delayed list.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.delayed_keep_period=PT30S"
```

### `evs.evict`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |

Manual eviction can be triggered by setting the [`evs.evict`](wsrep-provider-index.md#evsevict) to a certain node value. Setting the [`evs.evict`](wsrep-provider-index.md#evsevict) to an empty string will clear the evict list on the node where the option was set.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.evict=value"
```

### `evs.inactive_check_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT0.5S |

This option defines how often to check for peer inactivity.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.inactive_check_period=PT30S"
```

### `evs.inactive_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT15S |

This option defines the inactivity limit. Once the cluster reaches this limit, the cluster considers the node dead.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.inactive_timeout=PT30S"
```

### `evs.info_log_mask`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0 |

This option controls extra EVS (Extended Virtual Synchrony) info logging.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.info_log_mask=0"
```

### `evs.install_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | PT7.5S |

This option defines the timeout on waiting for install message acknowledgments.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.install_timeout=PT30S"
```

### `evs.join_retrans_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT1S |

This option defines how often to retransmit EVS (Extended Virtual Synchrony) join messages when forming cluster membership.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.join_retrans_period=PT30S"
```

### `evs.keepalive_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT1S |

This option defines how often to emit keepalive beacons (in the absence of any other traffic).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.keepalive_period=PT30S"
```

### `evs.max_install_timeouts`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 1 |

This option defines how many membership install rounds to try before giving up (total rounds will be [`evs.max_install_timeouts`](wsrep-provider-index.md#evsmax_install_timeouts) + 2).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.max_install_timeouts=1"
```

### `evs.send_window`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 10 |

This option defines the maximum number of data packets in replication at a time. For WAN setups, you can set this option to a considerably higher value than the default (for example, 512). The value must not be less than [`evs.user_send_window`](wsrep-provider-index.md#evsuser_send_window).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.send_window=10"
```

### `evs.stats_report_period`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT1M   |

This option defines the control period of EVS (Extended Virtual Synchrony) statistics reporting.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.stats_report_period=PT1M"
```

### `evs.suspect_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                |
| Default Value: | PT5S   |

This option defines the inactivity period after which the node is "suspected" to be dead. If all remaining nodes agree, the cluster drops the node even before [`evs.inactive_timeout`](wsrep-provider-index.md#evsinactive_timeout) is reached.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.suspect_timeout=PT30S"
```

### `evs.use_aggregate`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | true   |

When you enable this option, the cluster aggregates smaller packets into one.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.use_aggregate=no"
```

### `evs.user_send_window`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | 4   |

This option defines the maximum number of data packets in replication at a time. For WAN setups, you can set this option to a considerably higher value than the default (for example, 512).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.user_send_window=4"
```

### `evs.version`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

This option defines the EVS (Extended Virtual Synchrony) protocol version. Setting this option to `1` enables auto eviction. The default `0` keeps backwards compatibility.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.version=0"
```

### `evs.view_forget_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | P1D   |

This option defines the timeout after which the cluster drops past views from history.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="evs.view_forget_timeout=P1D"
```

### `gcache.dir`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | [`datadir`](glossary.md#datadir)   |

Use this option to define the location of the `galera.cache` file.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcache.dir=/var/lib/mysql"
```

### `gcache.freeze_purge_at_seqno`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Local, Global             |
| Dynamic:       | Yes                 |
| Default Value: | -1  |

This option controls the purging of the gcache (Galera cache) and enables retaining more data in the gcache. This option makes possible the use of [IST (Incremental State Transfer)](glossary.md#ist) when the node rejoins instead of [SST (State Snapshot Transfer)](glossary.md#sst).

Set this option on an existing node of the cluster (that will continue to be part of the cluster and can act as a potential [donor node](glossary.md#donor-node)). This node continues to retain the write-sets (replicated transaction data) and allows restarting the node to rejoin by using [IST](glossary.md#ist).

!!! admonition "See also"

      Percona Database Performance Blog:

      * [All You Need to Know About GCache (Galera-Cache) :octicons-link-external-16:](https://www.percona.com/blog/2016/11/16/all-you-need-to-know-about-gcache-galera-cache/)

      * [Want IST Not SST for Node Rejoins? We Have a Solution! :octicons-link-external-16:](https://www.percona.com/blog/2018/02/13/no-sst-node-rejoins/)

The [`gcache.freeze_purge_at_seqno`](wsrep-provider-index.md#gcachefreeze_purge_at_seqno) option accepts one of the following:

-1 (default): No freezing of gcache; purge operates as normal.

A numeric sequence number (seqno): The freeze purge of write-sets may not be smaller than the chosen seqno. To choose a value, use the status variable :variable:`wsrep_last_applied` on the node that you plan to shut down.

The literal string `now`: Set the value to the word `now` (as a string, not a number). The freeze purge of write-sets is then no less than the smallest seqno currently in gcache, which freezes the gcache purge immediately. Use `now` when picking a specific seqno is impractical.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcache.freeze_purge_at_seqno=-1"
```

To freeze purge immediately using the current minimum seqno:

```text
wsrep_provider_options="gcache.freeze_purge_at_seqno=now"
```

### `gcache.keep_pages_count`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Local, Global             |
| Dynamic:       | Yes                 |
| Default Value: | 0   |

This option limits the number of overflow pages rather than the total memory occupied by all overflow pages.
Whenever you set `gcache.keep_pages_count` to a non-zero value, the cluster deletes excess overflow pages
(starting from the oldest to the newest).

Whenever you update either `gcache.keep_pages_count` or [`gcache.keep_pages_size`](wsrep-provider-index.md#gcachekeep_pages_size) at runtime to a non-zero value, the cluster runs cleanup and deletes excess overflow pages.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcache.keep_pages_count=0"
```

### `gcache.keep_pages_size`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Local, Global            |
| Dynamic:       | No                 |
| Default Value: | 0   |

This option limits the total size of overflow pages rather than the count of all overflow pages. Whenever you set `gcache.keep_pages_size` to a non-zero value, the cluster deletes excess overflow pages (starting from the oldest to the newest) until the total size is below the specified value.

Whenever you update either [`gcache.keep_pages_count`](wsrep-provider-index.md#gcachekeep_pages_count) or `gcache.keep_pages_size` at runtime to a non-zero value, the cluster runs cleanup and deletes excess overflow pages.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcache.keep_pages_size=0"
```

### `gcache.name`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | /var/lib/mysql/galera.cache   |

Use this option to specify the name of the Galera cache file.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcache.name=/var/lib/mysql/galera.cache"
```

### `gcache.page_size`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | No                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 128M   |

Size of the page files in page storage. The limit on overall page storage is the size of the disk. Page file names use the gcache.page prefix.

!!! admonition "See also"

    [Percona Database Performance Blog: All You Need to Know About GCache :octicons-link-external-16:](https://www.percona.com/blog/2016/11/16/all-you-need-to-know-about-gcache-galera-cache/)

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcache.page_size=128M"
```

### `gcache.recover`

| Option              | Description |
|---------------------|-------------|
| Command line:       | No          |
| Configuration file: | Yes         |
| Scope:              | Global      |
| Dynamic:            | No          |
| Default value:      | Yes         |

This option attempts to recover a node’s gcache file to a usable state during startup. If the node successfully recovers the gcache file, the node provides Incremental State Transfer (IST) to the remaining nodes. This capability reduces the time required to bring up the cluster.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcache.recover=value"
```

### `gcache.size`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 128M   |

Size of the transaction cache for Galera replication. This option defines the size of the `galera.cache` file, which the cluster uses as the source for [IST](glossary.md#ist). The higher the value of this option, the better the chance that a re-joining node gets IST instead of [SST](glossary.md#sst).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcache.size=512M"
```

### `gcomm.thread_prio`

Using this option, you can raise the priority of the gcomm thread to a higher level than the thread normally uses.

The format for this option is: &#60;policy&#62;:&#60;priority&#62;. The priority value is an integer.

other

    Default time-sharing scheduling in Linux. The threads can run until blocked by an I/O request or preempted by higher priorities or superior scheduling designations.

fifo

    First-in First-out (FIFO) scheduling. These threads always immediately preempt any currently running other, batch or idle threads. They can run until they are either blocked by an I/O request or preempted by a FIFO thread of a higher priority.

rr

    Round-robin scheduling. These threads always preempt any currently running other, batch or idle threads. The scheduler allows these threads to run for a
    fixed period of a time. If the thread is still running when this time period is exceeded, they are stopped and moved to the end of the list, allowing another
    round-robin thread of the same priority to run in their place. They can otherwise continue to run until they are blocked by an I/O request or are
    preempted by threads of a higher priority.

!!! admonition "See also"

    For information, see the [MariaDB Galera Cluster documentation :octicons-link-external-16:](https://mariadb.com/docs/galera-cluster/reference/wsrep-variable-details/wsrep_provider_options#gcomm.thread_prio)

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcomm.thread_prio=value"
```

### `gcs.fc_auto_evict_threshold`

| Option | Description |
|---|---|
| Command Line: | Yes |
| Config file : | Yes |
| Scope: | Global |
| Dynamic: | No |
| Default value: | 0.75 |

This option defines the threshold that the cluster must reach or cross before evicting a node. The option is a ratio of the [`gcs.fc_auto_evict_window`](#gcsfc_auto_evict_window) option. The default is `.075`; you can set the value to any number between 0.0 and 1.0.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcs.fc_auto_evict_threshold=value"
``` 

### `gcs.fc_auto_evict_window`

| Option | Description |
|---|---|
| Command Line: | Yes |
| Config file : | Yes |
| Scope: | Global |
| Dynamic: | No |
| Default value: | 0 |

This option defines the time window width within which the cluster observes flow control. The time span of the window is [now - gcs.fc_audot_evict_window, now]. The window is constantly moving ahead as the time passes. And now, within this window if the flow control summary time >= (gcs.fc_audot-evict_window * gcs.fc_audot_evict_threshold), the node self-leaves the cluster.

The default value is 0, which means that the feature is disabled.

The maximum value is `DBL_MAX`.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcs.fc_auto_evict_window=value"
```

### `gcs.fc_debug`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

This option specifies after how many writesets the cluster posts debug statistics about SST (State Snapshot Transfer) flow control.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcs.fc_debug=0"
```

### `gcs.fc_factor`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | 1   |

This option controls replication flow. Replication resumes when the replica queue drops below [`gcs.fc_factor`](wsrep-provider-index.md#gcsfc_factor) * [`gcs.fc_limit`](wsrep-provider-index.md#gcsfc_limit).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcs.fc_factor=1"
```

### `gcs.fc_limit`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | 100   |

This option controls replication flow. Replication pauses when the replica queue exceeds this limit. In the default operation mode, the cluster recalculates the flow control limit dynamically based on the number of nodes in the cluster. You can turn off this recalculation by using the [`gcs.fc_master_slave`](wsrep-provider-index.md#gcsfc_master_slave) option so that manual setting of [`gcs.fc_limit`](wsrep-provider-index.md#gcsfc_limit) takes effect (e.g., when only one node in Percona XtraDB Cluster accepts writes).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcs.fc_limit=100"
```

### `gcs.fc_master_slave`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | NO                 |
| Default Value: | NO   |

This option specifies if there is only one source node in the cluster. It affects whether flow control limit is recalculated dynamically
(when `NO`) or not (when `YES`).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcs.fc_master_slave=NO"
```

### `gcs.max_packet_size`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 64500   |

This option specifies the writeset (replicated transaction data) size after which the cluster fragments them.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcs.max_packet_size=64500"
```

### `gcs.max_throttle`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0.25   |

This option specifies how much replication can be throttled during state transfer to avoid running out of memory. You can set the value to `0.0` if stopping replication is acceptable to finish state transfer.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcs.max_throttle=0.25"
```

### `gcs.recv_q_hard_limit`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 9223372036854775807   |

This option specifies the maximum allowed size of the receive queue. Set this to `(RAM + swap) / 2` in most cases. If the limit is exceeded, Galera aborts the server.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcs.recv_q_hard_limit=9223372036854775807"
```

### `gcs.recv_q_soft_limit`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0.25   |

This option specifies the fraction of the [`gcs.recv_q_hard_limit`](wsrep-provider-index.md#gcsrecv_q_hard_limit) after which the cluster throttles the replication rate.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcs.recv_q_soft_limit=0.25"
```

### `gcs.sync_donor`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | No   |

This option controls whether the rest of the cluster stays in sync with the donor node. When you set this option to `YES`, the whole cluster blocks if the donor node is blocked with SST (State Snapshot Transfer).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gcs.sync_donor=yes"
```

### `gmcast.listen_addr`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | tcp://0.0.0.0:4567   |

This option defines the address on which the node listens to connections from other nodes in the cluster.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gmcast.listen_addr=tcp://0.0.0.0:4567"
```

### `gmcast.mcast_addr`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | None   |

Set this option if you use UDP multicast for replication.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gmcast.mcast_addr=None"
```

### `gmcast.mcast_ttl`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 1   |

Use this option to define TTL (time to live) for multicast packets.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gmcast.mcast_ttl=1"
```

### `gmcast.peer_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT3S   |

This option specifies the connection timeout to initiate message relaying.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gmcast.peer_timeout=PT30S"
```

### `gmcast.segment`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

This option specifies the group segment this member belongs to. The cluster treats members in the same segment as equally physically close.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gmcast.segment=0"
```

### `gmcast.time_wait`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT5S   |

This option specifies the time to wait until allowing peer declared outside of stable view to reconnect.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gmcast.time_wait=PT30S"
```

### `gmcast.version`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

This option shows which gmcast protocol version is being used.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="gmcast.version=0"
```

### `ist.recv_addr`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | value of `wsrep_node_address`   |

This option specifies the address on which the node listens for Incremental State Transfer ([IST](glossary.md#ist)).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="ist.recv_addr=/var/lib/mysql"
```

### `pc.announce_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT3S   |

The node sends cluster joining announcements every half second (0.5 s) for this period or less if the node discovers other nodes.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.announce_timeout=PT30S"
```

### `pc.bootstrap`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | No                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | FALSE   |

`pc.bootstrap` is a configuration option that instructs a node to become the initial starting point for a new cluster or rejoin an existing one as the primary component (PC). You can only set this option in the configuration file.

```text
wsrep_provider_options="pc.bootstrap=TRUE"
```

You can use `pc.bootstrap` in the following:

| Action | Description |
|---|---|
| Bootstrap a New Cluster | When setting up a new Galera cluster, designate one node as the bootstrap node by setting `pc.bootstrap=YES` on that node. This allows the node to initialize the cluster and become the primary component. Other nodes can then discover and join this primary node. |
| Rejoining a Cluster (Use with Caution) | If a Galera node needs to rejoin the cluster as the primary component (for example, after a disconnect or restart), you can use `pc.bootstrap=TRUE`. This is generally not recommended because bootstrapping can lead to data loss if other cluster nodes made changes while the bootstrapping node was offline. |

By default, pc.bootstrap is disabled (FALSE).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.bootstrap=FALSE"
```

### `pc.checksum`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | true   |

This option controls whether the cluster checksums replicated messages.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.checksum=no"
```

### `pc.ignore_quorum`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | false   |

When you set this option to `TRUE`, the node completely ignores quorum calculations. Use this option with extreme caution even in source-replica setups, because replicas won’t automatically reconnect to source in this case.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.ignore_quorum=false"
```

### `pc.ignore_sb`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | false   |

When you set this option to `TRUE`, the node processes updates even in the case of a split brain. Use this option with extreme caution in multi-source setup, and can simplify things in a source-replica cluster (especially if only two nodes are used).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.ignore_sb=false"
```

### `pc.linger`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT20S   |

This option specifies the period for which the PC (Primary Component) protocol waits for EVS (Extended Virtual Synchrony) termination.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.linger=PT30S"
```

### `pc.npvo`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | false   |

When you set this option to `TRUE`, more recent primary components override older ones in case of conflicting primaries.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.npvo=false"
```

### `pc.recovery`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | true   |

When you set this option to `true`, the node stores the Primary Component state to disk. The Primary Component can then recover automatically when all nodes that were part of the last saved state re-establish communication with each other. This feature allows automatic recovery from full cluster crashes, such as in the case of a data center power outage. A subsequent graceful full cluster restart requires explicit bootstrapping for a new Primary Component.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.recovery=no"
```

### `pc.version`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

Use this option to check which PC (Primary Component) protocol version the cluster uses.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.version=0"
```

### `pc.wait_prim`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | true   |

When set to `TRUE`, the node waits for a primary component for the period of time specified in [`pc.wait_prim_timeout`](wsrep-provider-index.md#pcwait_prim_timeout). Use this to bring up a non-primary component and make that component primary with [`pc.bootstrap`](wsrep-provider-index.md#pcbootstrap).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.wait_prim=no"
```

### `pc.wait_prim_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | PT30S   |

This option specifies the period of time to wait for a primary component.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.wait_prim_timeout=PT30S"
```

### `pc.wait_restored_prim_timeout`

| Option   | Description   |
| ---| ---|
| Command Line: | Yes |
| Config File: | Yes|
| Scope: | Global |
| Dynamic | No |
| Default Value: | PT0S |

This option specifies the wait period for a primary component when the cluster restores the primary component from the `gvwstate.dat` file after an outage.

The default value is `PT0S` (zero seconds). The node waits for an infinite time, which is the current behavior.

You can define a wait time with `PTNS`, replace the `N` value with the number of seconds. For example, to wait for 90 seconds, set the value to `PT90S`.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.wait_restored_prim_timeout=PT30S"
```

### `pc.weight`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | 1   |

This option specifies the node weight that’s going to be used for Weighted Quorum calculations.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="pc.weight=1"
```

### `protonet.backend`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | asio   |

This option defines which transport backend to use. Currently the cluster supports only ASIO (Asynchronous I/O library).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="protonet.backend=asio"
```

### `protonet.version`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 0   |

Use this option to check which transport backend protocol version the cluster uses.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="protonet.version=0"
```

### `repl.causal_read_timeout`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | PT30S   |

This option specifies the causal read timeout.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="repl.causal_read_timeout=PT30S"
```

### `repl.commit_order`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 3   |

This option specifies out-of-order committing (which improves parallel applying performance). You can use the following values:

* `0` - BYPASS: all commit order monitoring is turned off (useful for measuring performance penalty)

* `1` - OOOC: allow out-of-order committing for all transactions

* `2` - LOCAL_OOOC: allow out-of-order committing only for local transactions

* `3` - NO_OOOC: no out-of-order committing is allowed (strict total order committing)

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="repl.commit_order=3"
```

### `repl.key_format`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | Yes                 |
| Default Value: | FLAT8   |

This option specifies the replication key format. You can use the following values:
 
* `FLAT8` - short key with higher probability of key match false positives

* `FLAT16` - longer key with lower probability of false positives

* `FLAT8A` - same as `FLAT8` but with annotations for debug purposes

* `FLAT16A` - same as `FLAT16` but with annotations for debug purposes

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="repl.key_format=FLAT8"
```

### `repl.max_ws_size`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 2147483647   |

This option specifies the maximum size of a write-set (replicated transaction data) in bytes. The maximum is 2 gigabytes.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="repl.max_ws_size=2147483647"
```

### `repl.proto_max`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 7   |

This option specifies the highest communication protocol version to accept in the cluster. Use this option only for debugging.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="repl.proto_max=7"
```

### `socket.checksum`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | 2   |

This option selects the checksum algorithm for network packets. The ``CRC32-C`` option is optimized and may be hardware accelerated on Intel CPUs. You can use the following values:

* `0` - disable checksum

* `1` - plain `CRC32` (used in Galera 2.x)

* `2` - hardware accelerated `CRC32-C`

The following is an example of the option:

```text
wsrep_provider_options="socket.checksum=2"
```

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

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

This option specifies whether to use SSL (Secure Sockets Layer) encryption.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="socket.ssl=yes"
```

### `socket.ssl_ca`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |

This option specifies the path to the Certificate Authority (CA) certificate file.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="socket.ssl_ca=/path/to/ca.pem"
```

### `socket.ssl_cert`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |

This option specifies the path to the server’s certificate file (in PEM, Privacy Enhanced Mail, format).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="socket.ssl_cert=/path/to/server-cert.pem"
```

### `socket.ssl_key`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |

This option specifies the path to the server’s private key file (in PEM, Privacy Enhanced Mail, format).

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="socket.ssl_key=/path/to/server-key.pem"
```

### `socket.ssl_compression`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | Yes                |

This option specifies whether to use SSL (Secure Sockets Layer) compression.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="socket.ssl_compression=no"
```

### `socket.ssl_cipher`

| Option         | Description        |
| -------------- | ------------------ |
| Command Line:  | Yes                |
| Config File:   | Yes                |
| Scope:         | Global             |
| Dynamic:       | No                 |
| Default Value: | AES128-SHA         |

This option specifies which cipher to use for encryption.

Example (config file): Add to the `[mysqld]` section of `my.cnf`:

```text
wsrep_provider_options="socket.ssl_cipher=AES128-SHA"
```