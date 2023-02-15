# Percona XtraDB Cluster 5.7.23-31.31

**This release has been superseded by 5.7.23-31.31.2 after a critical regression was found.** 

[Please update to the latest release](Percona-XtraDB-Cluster-5.7.23-31.31.2.md).

Percona is glad to announce the release of
Percona XtraDB Cluster 5.7.23-31.31 on September 26, 2018.
Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/)
or from our [software repositories](../install/index.md#install).

Percona XtraDB Cluster 5.7.23-31.31 is now the current release,
based on the following:


* [Percona Server for MySQL 5.7.23](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.23-23.html)


* Galera Replication library 3.24


* Galera/Codership WSREP API Release 5.7.23

## Deprecated

The following variables are deprecated starting from this release:

* `wsrep_convert_lock_to_trx`

This variable, which defines whether locking sessions should be converted to
transactions, is deprecated in Percona XtraDB Cluster 5.7.23-31.31 because it is
rarely used in practice.

## Fixed Bugs


* [PXC-1017](https://jira.percona.com/browse/PXC-1017): Memcached access to InnoDB was not replicated by Galera


* [PXC-2164](https://jira.percona.com/browse/PXC-2164): The  script prevented SELinux from being enabled


* [PXC-2155](https://jira.percona.com/browse/PXC-2155): `wsrep_sst_xtrabackup-v2` did not delete all folders on cleanup


* [PXC-2160](https://jira.percona.com/browse/PXC-2160): In some cases, the MySQL version was not detected correctly with the `Xtrabackup-v2` method of .


* [PXC-2199](https://jira.percona.com/browse/PXC-2199): When the `DROP TRIGGER IF EXISTS` statement was run for a not existing trigger, the node GTID was incremented instead of the cluster GTID.


* [PXC-2209](https://jira.percona.com/browse/PXC-2209): The compression dictionary was not replicated in PXC.


* [PXC-2202](https://jira.percona.com/browse/PXC-2202): In some cases, a disconnected cluster node was not shut down.


* [PXC-2165](https://jira.percona.com/browse/PXC-2165):  could fail if either `wsrep_node_address` or `wsrep_sst_receive_address` were not specified.


* [PXC-2213](https://jira.percona.com/browse/PXC-2213): NULL/VOID DDL transactions could commit in a wrong order.