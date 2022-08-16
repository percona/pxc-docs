# *Percona XtraDB Cluster* 5.7.11-4beta-25.14.2

Percona is glad to announce the release of
Percona XtraDB Cluster 5.7.11-4beta-25.14.2 on June 9, 2016.
Binaries are available from
[downloads area](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57)
or from our [software repositories](../install/index.md#install).

!!! note 

    This release is available only from the testing repository. It is not meant for upgrade from Percona XtraDB Cluster 5.6 and earlier versions. Only fresh installation is supported.

Percona XtraDB Cluster 5.7.11-4beta-25.14.2 is based on the following:

* [Percona Server 5.7.11-4](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.11-4.html)

* [Galera Replicator 3.14.2](https://github.com/percona/galera/tree/rel-3.14.2)

  This is the first beta release in the Percona XtraDB Cluster 5.7 series.
  It includes all changes from upstream releases
  and the following changes:

* Percona XtraDB Cluster 5.7 does not include `wsrep_sst_xtrabackup`.
  It has been replace by `wsrep_sst_xtrabackup_v2`.

* The `wsrep_mysql_replication_bundle` variable has been removed.
