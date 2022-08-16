# Percona XtraDB Cluster 5.7.17-29.20

Percona is glad to announce the release of
*Percona XtraDB Cluster* 5.7.17-29.20 on April 19, 2017.
Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/)
or from our [software repositories](../install/index.md#install).

Percona XtraDB Cluster 5.7.17-29.20 is now the current release,
based on the following:


* [Percona Server 5.7.17-13](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.17-13.html)


* Galera Replication library 3.20


* wsrep API version 29

All Percona software is open-source and free.

## Performance Improvements

This release was focused on performance
and scaling capability with increasing workload threads.
Tests show up to 10 times increase in performance.

## Fixed Bugs


* Improved parallelism for better scaling with multiple threads.


* Updated semantics for gcache page cleanup
to trigger when either `gcache.keep_pages_size`
or `gcache.keep_pages_count` exceeds the limit,
instead of both at the same time.


* Improved SST and IST log messages
for better readability and unification.


* Excluded the `garbd` node from flow control calculations.


* Added extra checks to verify that SSL files
(certificate, certificate authority, and key)
are compatible before opening connection.


* Added validations for `DISCARD TABLESPACE`
and `IMPORT TABLESPACE` in [PXC Strict Mode](../features/pxc-strict-mode.md#pxc-strict-mode) to prevent data inconsistency.


* Added support for passing the XtraBackup buffer pool size
with the `use-memory` option under `[xtrabackup]`
and the `innodb_buffer_pool_size` option under `[mysqld]`
when the `--use-memory` option is not passed
with the `inno-apply-opts` option under `[sst]`.

* Added the `wsrep_flow_control_status` variable
to indicate if node is in flow control (paused).

* Fixed gcache page cleanup not triggering
when limits are exceeded.

* [PXC-766](https://jira.percona.com/browse/PXC-766): Added the  `wsrep_ist_receive_status` variable
to show progress during an IST.

* Allowed `CREATE TABLE ... AS SELECT` (CTAS) statements
with temporary tables (`CREATE TEMPORARY TABLE ... AS SELECT`)
in [PXC Strict Mode](../features/pxc-strict-mode.md#pxc-strict-mode).
  For more information, see [#1666899](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1666899).

* [PXC-782](https://jira.percona.com/browse/PXC-782): Updated `xtrabackup-v2` script to use the [`tmpdir`](../manual/xtrabackup_sst.md#cmdoption-arg-tmpdir) option (if it is set under `[sst]`, `[xtrabackup]` or `[mysqld]`,
in that order).

* [PXC-783](https://jira.percona.com/browse/PXC-783): Improved the wsrep stage framework.


* [PXC-784](https://jira.percona.com/browse/PXC-784): Fixed the `pc.recovery` procedure to abort
if the `gvwstate.dat` file is empty or invalid,
and fall back to normal joining process.
  For more information, see [#1669333](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1669333).


* [PXC-794](https://jira.percona.com/browse/PXC-794): Updated the [`sockopt`](../manual/xtrabackup_sst.md#cmdoption-arg-sockopt) option
to include a comma at the beginning if it is not set by the user.


* [PXC-795](https://jira.percona.com/browse/PXC-795): Set `--parallel=4` as default option for `wsrep_sst_xtrabackup-v2` to run four threads with XtraBackup.


* [PXC-797](https://jira.percona.com/browse/PXC-797): Blocked `wsrep_desync` toggling while node is paused
to avoid halting the cluster when running `FLUSH TABLES WITH READ LOCK`. 
  For more information, see [#1370532](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1370532).


* [PXC-805](https://jira.percona.com/browse/PXC-805): Inherited upstream fix
to avoid using deprecated variables,
such as `INFORMATION_SCHEMA.SESSION_VARIABLE`.
  For more information, see [#1676401](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1676401).


* [PXC-811](https://jira.percona.com/browse/PXC-811): Changed default values for the following variables:


    * `fc_limit` from `16` to `100`


    * `send_window` from `4` to `10`


    * `user_send_window` from `2` to `4`


* Moved wsrep settings into a separate configuration file
(`/etc/my.cnf.d/wsrep.cnf`).


* Fixed `mysqladmin shutdown` to correctly stop the server
on systems using `systemd`.


* Fixed several minor packaging and dependency issues.
