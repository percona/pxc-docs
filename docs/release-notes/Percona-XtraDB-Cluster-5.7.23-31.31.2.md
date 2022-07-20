# Percona XtraDB Cluster 5.7.23-31.31.2

To resolve a critical regression, Percona announces the release of Percona XtraDB Cluster
5.7.23-31.31.2 on October 2, 2018.  Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/) or from our
[software repositories](../install/index.md#install).

This release resolves a critical regression in the upstream wsrep library and
supersedes `5.7.23-31.31`.

Percona XtraDB Cluster 5.7.23-31.31.2 is now the current release,
based on the following:


* [Percona Server 5.7.23-23](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.23-23.html)


* Galera Replication library 3.24


* Galera/Codership WSREP API Release 5.7.23

All Percona software is open-source and free.

## Fixed Bugs


* [#2254](https://jira.percona.com/browse/PXC-2254): A cluster conflict could cause a crash in Percona XtraDB Cluster 5.7.23 if
autocommit=off.
