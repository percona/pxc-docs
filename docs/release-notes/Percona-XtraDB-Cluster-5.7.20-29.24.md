# Percona XtraDB Cluster 5.7.20-29.24

Percona is glad to announce the release of
Percona XtraDB Cluster 5.7.20-29.24 on January 26, 2018.
Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/)
or from our [software repositories](../install/index.md#install).

!!! note

    Due to new package dependency, Ubuntu/Debian users should use `apt-get dist-upgrade`, `apt upgrade`, or `apt-get install percona-xtradb-cluster-57` to upgrade.

Percona XtraDB Cluster 5.7.20-29.24 is now the current release,
based on the following:

* [Percona Server 5.7.20-18](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.20-18.html)

* Galera Replication library 3.22

* Galera/Codership WSREP API Release 5.7.20

All Percona software is open-source and free.

## NEW FEATURES:

* Ubuntu 17.10 Artful Aardvark is now supported.

* [PXC-737](https://jira.percona.com/browse/PXC-737): freezing gcache purge was implemented to facilitate node
joining through IST, avoiding time consuming SST process.

* [PXC-822](https://jira.percona.com/browse/PXC-822): a usability improvement was made to timeout error
messages, the name of the configuration variable which caused the
timeout was added to the message.

* [PXC-866](https://jira.percona.com/browse/PXC-866): a new variable `wsrep_last_applied`, in addition to `wsrep_last_committed` one, was introduced to clearly separate last committed and last applied transaction numbers.

* [PXC-868](https://jira.percona.com/browse/PXC-868): on the Joiner, during SST, `tmpdir` variable under `[sst]` section can be used to specify temporary SST files storage different from the default `datadir/.sst` one.

## Fixed Bugs

* [PXC-889](https://jira.percona.com/browse/PXC-889): fixed an issue where a node with an invalid value for `wsrep_provider` was allowed to start up and operate in standalone
mode, which could lead to data inconsistency. The node will now abort in
this case. Bug fixed [#1728774](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1728774)

* [PXC-806](https://jira.percona.com/browse/PXC-806): fixed an abort caused by an early read of the
`query_id`, ensuring valid  ids are assigned to subsequent transactions.

* [PXC-850](https://jira.percona.com/browse/PXC-850): ensured that a node, because of data inconsistency,
isolates itself before leaving the cluster, thus allowing pending nodes
to re-evaluate the quorum. Bug fixed [#1704404](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1704404)


* [PXC-867](https://jira.percona.com/browse/PXC-867): `wsrep_sst_rsync` script was overwriting `wsrep_debug` configuration setting making it not to be taken
into account.


* [PXC-873](https://jira.percona.com/browse/PXC-873): fixed formatting issue in the error message appearing
when SST is not possible due to a timeout. Bug fixed [#1720094](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1720094)


* [PXC-874](https://jira.percona.com/browse/PXC-874): PXC acting as async slave reported unhandled transaction
errors, namely “Rolling back unfinished transaction”.


* [PXC-875](https://jira.percona.com/browse/PXC-875): fixed an issue where toggling `wsrep_provider` off and on failed to reset some internal variables and resulted in PXC
logging an “Unsupported protocol downgrade” warning. Bug fixed [#1379204](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1379204)


* [PXC-877](https://jira.percona.com/browse/PXC-877): fixed PXC hang caused by an internal deadlock.


* [PXC-878](https://jira.percona.com/browse/PXC-878): thread failed to mark exit from the InnoDB server
concurrency and therefore never got un-register in InnoDB concurrency system.


* [PXC-879](https://jira.percona.com/browse/PXC-879): fixed a bug where a `LOAD DATA` command used
with GTIDs was executed on one node, but the other nodes would receive less
rows than the first one. Bug fixed [#1741818](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1741818)


* [PXC-880](https://jira.percona.com/browse/PXC-880): insert to table without primary key was possible with
insertable view if `pxc_strict_mode` variable was set to ENFORCING.
Bug fixed [#1722493](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1722493)


* [PXC-883](https://jira.percona.com/browse/PXC-883): fixed `ROLLBACK TO SAVEPOINT` incorrect operation on
slaves by avoiding useless wsrep plugin register for a savepoint rollback. Bug fixed [#1700593](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1700593)


* [PXC-885](https://jira.percona.com/browse/PXC-885): fixed IST hang when `keyring_file_data` is set.
Bug fixed [#1728688](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1728688)


* [PXC-887](https://jira.percona.com/browse/PXC-887): gcache page files were unnecessarily created due to
an error in projecting gcache free size when configured to recover on
restart.


* [PXC-895](https://jira.percona.com/browse/PXC-895): fixed transaction loss after recovery by avoiding
interruption of the binlog recovery based on wsrep saved position.
Bug fixed :bug:1734113


* [PXC-897](https://jira.percona.com/browse/PXC-897): fixed empty `gtid_executed` variable after
recovering the position of a node with `--wsrep_recover`.


* [PXC-906](https://jira.percona.com/browse/PXC-906): fixed certification failure in the case of a node
restarting at the same time when frequent `TRUNCATE TABLE` commands and
DML writes occur simultaneously on other nodes. Bug fixed [#1737731](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1737731)


* [PXC-909](https://jira.percona.com/browse/PXC-909): qpress package was turned into a dependency from
suggested/recommended one on Debian 9.


* [PXC-903](https://jira.percona.com/browse/PXC-903) and [PXC-910](https://jira.percona.com/browse/PXC-910): init.d/systemctl scripts on Debian
9 were updated to avoid starting wsrep-recover if there was no crash, and to
fix an infinite loop at mysqladmin ping fail because of nonexistent ping
user.


* [PXC-915](https://jira.percona.com/browse/PXC-915): suppressing DDL/TOI replication in case of `sql_log_bin` zero value didn’t work when DDL statement was modifying an existing table, resulting in an error.
