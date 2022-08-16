# Percona XtraDB Cluster 5.7.28-31.41

Percona is happy to announce the release of Percona XtraDB Cluster 5.7.28-31.41 on
December 16, 2019.  Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/) or from our
[software repositories](../install/index.md#install).

Percona XtraDB Cluster 5.7.28-31.41 is now the current release, based on the following:


* [Percona Server for MySQL 5.7.28-31](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.28-31.html)


* Galera/Codership WSREP API Release 5.7.28


* Galera Replication library 3.28

Percona XtraDB Cluster 5.7.28-31.41 requires [Percona XtraBackup 2.4.17](https://www.percona.com/doc/percona-xtrabackup/2.4/release-notes/2.4/2.4.17.html).

## Bugs Fixed


* [PXC-2729](https://jira.percona.com/browse/PXC-2729): A cluster node could hang when trying to access a table which was being updated by another node.


* [PXC-2704](https://jira.percona.com/browse/PXC-2704): After a row was updated with a variable-length unique key, the entire cluster could crash.

Other bugs fixed: [PXC-2670](https://jira.percona.com/browse/PXC-2670)
