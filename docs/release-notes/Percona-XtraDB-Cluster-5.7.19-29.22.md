# Percona XtraDB Cluster 5.7.19-29.22

Percona is glad to announce the release of
Percona XtraDB Cluster 5.7.19-29.22 on September 22, 2017.
Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/)
or from our [software repositories](../install/index.md#install).

Percona XtraDB Cluster 5.7.19-29.22 is now the current release,
based on the following:


* [Percona Server 5.7.19-17](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.19-17.html)


* Galera Replication library 3.22


* wsrep API version 29

All Percona software is open-source and free.

## Upgrade Instructions

After you upgrade each node to Percona XtraDB Cluster 5.7.19-29.22,
run the following command on one of the nodes:

```shell
$ mysql -uroot -p < /usr/share/mysql/pxc_cluster_view.sql
```

Then restart all nodes, one at a time:

```shell
$ sudo service mysql restart
```

## New Features

* Introduced the `pxc_cluster_view` table
to get a unified view of the cluster.
  This table is exposed through the performance schema.

  ```text
  mysql> select * from pxc_cluster_view;
  -----------------------------------------------------------------------------
  HOST_NAME  UUID                                  STATUS  LOCAL_INDEX  SEGMENT
  -----------------------------------------------------------------------------
  n1         b25bfd59-93ad-11e7-99c7-7b26c63037a2  DONOR   0            0
  n2         be7eae92-93ad-11e7-88d8-92f8234d6ce2  JOINER  1            0
  -----------------------------------------------------------------------------
  2 rows in set (0.01 sec)
  ```

* [PXC-803](https://jira.percona.com/browse/PXC-803): Added support for new features in Percona XtraBackup 2.4.7:


    * `wsrep_debug` enables debug logging


    * [`encrypt_threads`](../manual/xtrabackup_sst.md#cmdoption-arg-encrypt_threads) specifies the number of threads that XtraBackup should use for encrypting data (when `encrypt=1`). This value is passed using the `--encrypt-threads` option in XtraBackup.

    * [`backup_threads`](../manual/xtrabackup_sst.md#cmdoption-arg-backup_threads) specifies the number of threads that XtraBackup should use to create backups.
    See the `--parallel` option in XtraBackup.

## Improvements


* [PXC-835](https://jira.percona.com/browse/PXC-835): Limited `wsrep_node_name` to 64 bytes.


* [PXC-846](https://jira.percona.com/browse/PXC-846): Improved logging to report reason of IST failure.


* [PXC-851](https://jira.percona.com/browse/PXC-851): Added version compatibility check during SST with XtraBackup:


    * If donor is 5.6 and joiner is 5.7:
      A warning is printed to perform `mysql_upgrade`.


    * If donor is 5.7 and joiner is 5.6:
      An error is printed and SST is rejected.

## Fixed Bugs


* [PXC-825](https://jira.percona.com/browse/PXC-825): Fixed script for SST with XtraBackup
(`wsrep_sst_xtrabackup-v2`) to include the `--defaults-group-suffix`
when logging to syslog.
  For more information, see [#1559498](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1559498).


* [PXC-826](https://jira.percona.com/browse/PXC-826): Fixed multi-source replication to PXC node slave.
  For more information, see [#1676464](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1676464).


* [PXC-827](https://jira.percona.com/browse/PXC-827): Fixed handling of different binlog names
between donor and joiner nodes when GTID is enabled.
  For more information, see [#1690398](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1690398).


* [PXC-830](https://jira.percona.com/browse/PXC-830): Rejected the `RESET MASTER` operation
when wsrep provider is enabled and `gtid_mode` is set to `ON`.
  For more information, see [#1249284](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1249284).


* [PXC-833](https://jira.percona.com/browse/PXC-833): Fixed connection failure handling during SST
by making the donor retry connection to joiner every second
for a maximum of 30 retries.
  For more information, see [#1696273](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1696273).


* [PXC-839](https://jira.percona.com/browse/PXC-839): Fixed GTID inconsistency when setting `gtid_next`.


* [PXC-840](https://jira.percona.com/browse/PXC-840): Fixed typo in alias for `systemd` configuration.


* [PXC-841](https://jira.percona.com/browse/PXC-841): Added check to avoid replication of DDL
if `sql_log_bin` is disabled.
  For more information, see [#1706820](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1706820).


* [PXC-842](https://jira.percona.com/browse/PXC-842): Fixed deadlocks during Load Data Infile (LDI)
with `log-bin` disabled
by ensuring that a new transaction (of 10 000 rows)
starts only after the previous one is committed by both wsrep and InnoDB.
  For more information, see [#1706514](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1706514).


* [PXC-843](https://jira.percona.com/browse/PXC-843): Fixed situation where the joiner hangs
after SST has failed
by dropping all transactions in the receive queue.
  For more information, see [#1707633](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1707633).


* [PXC-853](https://jira.percona.com/browse/PXC-853): Fixed cluster recovery by enabling `wsrep_ready`
whenever nodes become PRIMARY.


* [PXC-862](https://jira.percona.com/browse/PXC-862): Fixed script for SST with XtraBackup
(`wsrep_sst_xtrabackup-v2`) to use the `ssl-dhparams` value
from the configuration file.

  !!! note

      As part of fix for [PXC-827](https://jira.percona.com/browse/PXC-827),
      version communication was added to the SST protocol.
      As a result, newer version of PXC (as of 5.7.19 and later)
      cannot act as donor when joining an older version PXC node (prior to 5.7.19).
      It will work fine vice versa:
      old node can act as donor when joining nodes with new version.
