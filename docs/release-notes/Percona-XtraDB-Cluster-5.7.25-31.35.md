# Percona XtraDB Cluster 5.7.25-31.35

Percona is glad to announce the release of Percona XtraDB Cluster 5.7.25-31.35 on
February 28, 2019.  Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/) or from our
[software repositories](../install/index.md#install).

This release of Percona XtraDB Cluster includes the support of Ubuntu 18.10 (Cosmic Cuttlefish).
Percona XtraDB Cluster 5.7.25-31.35 is now the current release, based on the following:


* [Percona Server for MySQL 5.7.25](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.25-28.html)


* Galera Replication library 3.25


* Galera/Codership WSREP API Release 5.7.24

## Bugs Fixed


* [#2346](https://jira.percona.com/browse/PXC-2346): `mysqld` could crash when executing `mysqldump
--single-transaction` while the binary log is disabled. This problem was also
reported in [#1711](https://jira.percona.com/browse/PXC-1711), [#2371](https://jira.percona.com/browse/PXC-2371), [#2419](https://jira.percona.com/browse/PXC-2419).


* [#2388](https://jira.percona.com/browse/PXC-2388): In some cases, `DROP FUNCTION` with an explicit name was not
replicated.

Other bugs fixed: [#1711](https://jira.percona.com/browse/PXC-1711), [#2371](https://jira.percona.com/browse/PXC-2371), [#2419](https://jira.percona.com/browse/PXC-2419)
