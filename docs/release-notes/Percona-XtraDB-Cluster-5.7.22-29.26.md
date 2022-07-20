# Percona XtraDB Cluster 5.7.22-29.26

Percona is glad to announce the release of
Percona XtraDB Cluster 5.7.22-29.26 on June 29, 2018.
Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/)
or from our [software repositories](../install/index.md#install).

Percona XtraDB Cluster 5.7.22-29.26 is now the current release,
based on the following:


* [Percona Server for MySQL 5.7.22](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.22-22.html)


* Galera Replication library 3.23


* Galera/Codership WSREP API Release 5.7.21

## Deprecated

The following variables are deprecated starting from this release:

* `wsrep-force-binlog-format`

* `wsrep_sst_method` = `mysqldump`

As long as the use of `binlog_format=ROW` is enforced in 5.7, `wsrep_forced_binlog_format` variable is much less significant.
The same is related to `mysqldump`, as `xtrabackup` is now the recommended
SST method.

## New features


* [PXC-907](https://jira.percona.com/browse/PXC-907): New variable `wsrep_RSU_commit_timeout` allows
to configure RSU wait for active commit connection timeout (in microseconds).

* [PXC-2111](https://jira.percona.com/browse/PXC-2111): Percona XtraDB Cluster now supports the `keyring_vault` plugin, which
allows to store the master key in a vault server.

* Percona XtraDB Cluster 5.7.22 depends on Percona XtraBackup 2.4.12 in order to fully support vault
plugin functionality.

## Fixed Bugs


* [PXC-2127](https://jira.percona.com/browse/PXC-2127): Percona XtraDB Cluster shutdown process hung if `thread_handling`
option was set to `pool-of-threads` due to a regression in `5.7.21`.

* [PXC-2128](https://jira.percona.com/browse/PXC-2128): Duplicated auto-increment values were set for the
concurrent sessions on cluster reconfiguration due to the erroneous
readjustment.

* [PXC-2059](https://jira.percona.com/browse/PXC-2059): Error message about the necessity of the `SUPER`
privilege appearing in case of the `CREATE TRIGGER` statements fail due to
enabled WSREP was made more clear.

* [PXC-2061](https://jira.percona.com/browse/PXC-2061): Wrong values could be read, depending on timing, when
read causality was enforced with `wsrep_sync_wait=1`, because of waiting on
the commit monitor to be flushed instead of waiting on the apply monitor.

* [PXC-2073](https://jira.percona.com/browse/PXC-2073): `CREATE TABLE AS SELECT` statement was not replicated
in case if result set was empty.

* [PXC-2087](https://jira.percona.com/browse/PXC-2087): Cluster was entering the deadlock state if table had an
unique key and `INSERT ... ON DUPLICATE KEY UPDATE` statement was executed.

* [PXC-2091](https://jira.percona.com/browse/PXC-2091): Check for the maximum number of rows, that can be
replicated as a part of a single transaction because of the Galera limit, was
enforced even when replication was disabled with `wsrep_on=OFF`.

* [PXC-2103](https://jira.percona.com/browse/PXC-2103): Interruption of the local running transaction in a
`COMMIT` state by a replicated background transaction while waiting for the
binlog backup protection caused the commit fail and, eventually, an assert in
Galera.

* [PXC-2130](https://jira.percona.com/browse/PXC-2130): Percona XtraDB Cluster failed to build with Python 3.

* [PXC-2142](https://jira.percona.com/browse/PXC-2142): Replacing Percona Server with Percona XtraDB Cluster on CentOS 7 with the
`yum swap` command produced a broken symlink in place of the
`/etc/my.cnf` configuration file.

* [PXC-2154](https://jira.percona.com/browse/PXC-2154): rsync SST is now aborted with error message if used on
node with `keyring_vault` plugin configured, because it doesn’t support
`keyring_vault`. Also Percona doesn’t recommend using rsync-based SST for
data-at-rest encryption with keyring.

* [PXB-1544](https://jira.percona.com/browse/PXB-1544): `xtrabackup --copy-back` didn’t read which encryption
plugin to use from `plugin-load` setting of the `my.cnf` configuration
file.

* [PXB-1540](https://jira.percona.com/browse/PXB-1540): Meeting a zero sized keyring file, *Percona XtraBackup*
was removing and immediately recreating it, and this could affect external
software noticing the file had undergo some manipulations.

Other bugs fixed: [PXC-2072](https://jira.percona.com/browse/PXC-2072) “flush table <table> for export should be
blocked with mode=ENFORCING”.
