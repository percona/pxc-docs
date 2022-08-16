# Percona XtraDB Cluster 5.7.27-31.39

Percona is happy to announce the release of Percona XtraDB Cluster 5.7.27-31.39 on
September 18, 2019.  Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/) or from our
[software repositories](../install/index.md#install).

Percona XtraDB Cluster 5.7.27-31.39 is now the current release, based on the following:


* [Percona Server for MySQL 5.7.27-30](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.27-30.html)


* Galera/Codership WSREP API Release 5.7.27


* Galera Replication library 3.28

## Bugs Fixed


* [#2432](https://jira.percona.com/browse/PXC-2432): PXC was not updating the information_schema user/client statistics properly.


* [#2555](https://jira.percona.com/browse/PXC-2555): SST initialization delay: fixed a bug where the SST
process took too long to detect if a child process was running.


* [#2557](https://jira.percona.com/browse/PXC-2557): Fixed a crash when a node goes NON-PRIMARY and SHOW STATUS is executed.


* [#2592](https://jira.percona.com/browse/PXC-2592): PXC restarting automatically on data inconsistency.


* [#2605](https://jira.percona.com/browse/PXC-2605): PXC could crash when log_slow_verbosity included InnoDB.  Fixed upstream PS-5820.


* [#2639](https://jira.percona.com/browse/PXC-2639): Fixed an issue where a SQL admin command (like OPTIMIZE) could cause a deadlock.
