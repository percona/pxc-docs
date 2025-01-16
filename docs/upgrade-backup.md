# Restore 8.0 backup to 8.4 cluster

Use Percona XtraBackup to back up the source server data and restore the data to a target server, and then upgrade the server to a different version of Percona XtraDB Cluster.

## Restore a database with a different server version

Review [Upgrade Percona XtraDB cluster](upgrade-guide.md).

Upgrade the nodes one at a time. The primary node should be the last node to be upgraded. The following steps are required on each node.

1. Install the new database server version on the target server.

2. Start the new database server version on the restored data directory.

3. Perform any other upgrade steps as necessary.

To ensure the upgrade was successful, verify the data by performing the following checks:

| Areas to check          | Details                                                                 |
|-------------------------|-------------------------------------------------------------------------|
| Data Integrity          | Compare the data before and after the upgrade to ensure no data loss or corruption occurred. |
| Application Functionality | Test the application to confirm it interacts correctly with the upgraded database. |
| Performance Metrics     | Monitor performance metrics to ensure the upgrade did not negatively impact database performance. |
| Logs and Errors         | Review logs for any errors or warnings that might indicate issues during the upgrade process. |

## Next nodes

To create the 2nd and subsequent nodes, follow these steps:

1. Install Percona XtraDB Cluster (PXC) 8.4.

2. Join the cluster using State Snapshot Transfer (SST).

Starting from version 8.0.34, this procedure can also be used to downgrade within the same LTS version.


