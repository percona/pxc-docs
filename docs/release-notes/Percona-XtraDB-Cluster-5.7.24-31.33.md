# Percona XtraDB Cluster 5.7.24-31.33

Percona is glad to announce the release of
Percona XtraDB Cluster 5.7.24-31.33 on January 4, 2019.
Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/)
or from our [software repositories](../install/index.md#install).

Percona XtraDB Cluster 5.7.24-31.33 is now the current release,
based on the following:


* [Percona Server for MySQL 5.7.24](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.24-26.html)


* Galera Replication library 3.25


* Galera/Codership WSREP API Release 5.7.24

## Deprecated

The following variables are deprecated starting from this release:


* `wsrep_preordered` was used to turn on transparent handling of
preordered replication events applied locally first before being replicated
to other nodes in the cluster. It is not needed anymore due to the [carried
out performance fix](https://jira.percona.com/browse/PXC-2128) eliminating
the lag in asynchronous replication channel and cluster replication.


* `innodb_disallow_writes` usage to make *InnoDB* avoid writes during was deprecated in favor of the `innodb_read_only` variable.


* `wsrep_drupal_282555_workaround` avoided the duplicate value
creation caused by buggy auto-increment logic, but the correspondent bug is
already fixed.


* session-level variable `binlog_format=STATEMENT` was enabled
only for `pt-table-checksum`, which would be addressed in following
releases of the *Percona Toolkit*.

## Fixed Bugs


* [PXC-2220](https://jira.percona.com/browse/PXC-2220): Starting two instances of Percona XtraDB Cluster on the same node could
cause writing transactions to a page store instead of a galera.cache ring
buffer, resulting in huge memory consumption because of retaining already
applied write-sets.


* [PXC-2230](https://jira.percona.com/browse/PXC-2230): 

  `gcs.fc_limit=0` not allowed as dynamic
  setting to avoid generating flow control on every message was still possible
  in `my.cnf` due to the inconsistent check.


* [PXC-2238](https://jira.percona.com/browse/PXC-2238): setting `read_only=1` caused race condition.


* [PXC-1131](https://jira.percona.com/browse/PXC-1131): `mysqld-systemd` threw an error at *MySQL* restart in
case of non-existing error-log in Centos/RHEL7.


* [PXC-2269](https://jira.percona.com/browse/PXC-2269): being not dynamic, the `pxc_encrypt_cluster_traffic` variable was erroneously allowed to
be changed by a `SET GLOBAL` statement.


* [PXC-2275](https://jira.percona.com/browse/PXC-2275): checking `wsrep_node_address` value in the
`wsrep_sst_common` command line parser caused parsing the wrong variable.
