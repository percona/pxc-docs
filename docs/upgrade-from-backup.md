<!--------- Ask whether we need to change this doc --------->

# Restore a 5.7 backup to an 8.0 cluster

Use Percona XtraBackup to back up the source server data and restore the data to a target server, and then upgrade the server to a different version of Percona XtraDB Cluster.

[Downgrading is not supported](https://docs.percona.com/percona-server/8.0/downgrade.html).

## Restore a database with a different server version

Review [Upgrade Percona XtraDB cluster](upgrade-guide.md).

Upgrade the nodes one at a time. The primary node should be the last node to be upgraded. The following steps are required on each node.

1. Back up the data on the source server.

2. Install the same database version as the source server on the target server.

3. Restore with a `copy-back` operation on the target server.

4. Start the database server on the target server.

5. Do a slow shutdown of the database server with the `SET GLOBAL innodb_fast_shutdown=0` statement. This shutdown type flushes InnoDB operations before completing and may take longer.

6. Install the new database server version on the target server.

7. Start the new database server version on the restored data directory.

8. Perform any other upgrade steps as necessary.

To ensure the upgrade was successful, check the data.

