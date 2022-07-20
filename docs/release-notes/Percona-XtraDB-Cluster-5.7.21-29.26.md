# Percona XtraDB Cluster 5.7.21-29.26

Percona is glad to announce the release of
Percona XtraDB Cluster 5.7.21-29.26 on March 02, 2018.
Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/)
or from our [software repositories](../install/index.md#install).

Percona XtraDB Cluster 5.7.20-29.24 is now the current release,
based on the following:


* [Percona Server for MySQL 5.7.21](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.21-20.html)


* Galera Replication library 3.23


* Galera/Codership WSREP API Release 5.7.21

Starting from now, Percona XtraDB Cluster issue
tracking system was moved from launchpad to
[JIRA](https://jira.percona.com/projects/PXC).
All Percona software is open-source and free.

## Fixed Bugs


* [PXC-2039](https://jira.percona.com/browse/PXC-2039): Node consistency was compromised for
`INSERT INTO ... ON DUPLICATE KEY UPDATE` workload
because the regression introduced in Percona XtraDB Cluster `5.7.17-29.20` made it possible to abort local transactions without further
re-evaluation in case of a lock conflict.

* [PXC-2054](https://jira.percona.com/browse/PXC-2054) Redo optimized DDL operations (like sorted index
build) were not blocked in case of a running backup process, leading
to the SST fail. To fix this, `--lock-ddl` option blocks now
all DDL during the **xtrabackup** backup stage.

* General code improvement was made in the GTID event handling,
when events are captured as a part of the slave replication and
appended to the galera replicated write-set. This fixed
[PXC-2041](https://jira.percona.com/browse/PXC-2041) (starting async slave on a single node
Percona XtraDB Cluster led to a crash) and [PXC-2058](https://jira.percona.com/browse/PXC-2058) (binlog-based
master-slave replication broke the cluster) caused by the
incorrect handling in the GTID append logic.

* An issue caused by noncoincidence between the order of recovered
transaction and the global seqno assigned to the transaction was
fixed ensuring that the updated recovery wsrep coordinates are
persisted.

* [PXC-904](https://jira.percona.com/browse/PXC-904): Replication filters were not working with
account management statements like `CREATE USER` in case of
galera replication; as a result such commands were blocked by the
replication filters on async slave nodes but not on galera ones.

* [PXC-2043](https://jira.percona.com/browse/PXC-2043): SST script was trying to use `pv` (the pipe
viewer) for `progress` and `rlimit` options
even on nodes with no `pv` installed, resulting in SST fail
instead of just ignoring these options for inappropriate nodes.


* [PXC-911](https://jira.percona.com/browse/PXC-911): When node’s own IP address was defined in the `wsrep_cluster_address` variable, the node was receiving
“no messages seen in” warnings from it’s own IP address in the
info log.

This release also contains fixes for the following CVE issues: CVE-2018-2565,
CVE-2018-2573, CVE-2018-2576, CVE-2018-2583, CVE-2018-2586, CVE-2018-2590,
CVE-2018-2612, CVE-2018-2600, CVE-2018-2622, CVE-2018-2640, CVE-2018-2645,
CVE-2018-2646, CVE-2018-2647, CVE-2018-2665, CVE-2018-2667, CVE-2018-2668,
CVE-2018-2696, CVE-2018-2703, CVE-2017-3737.
