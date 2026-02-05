# Restart the cluster nodes

To restart a cluster node, shut down MySQL and restart the service. The node leaves the cluster, reducing the total vote count for quorum. 

The quorum refers to the minimum number of votes required for the cluster to operate effectively and make decisions. Each node in the cluster typically represents one vote. When a node leaves the cluster, the total number of votes decreases, affecting the cluster's ability to achieve quorum. If the cluster does not maintain quorum, it may become unable to process transactions or make changes, potentially leading to a split-brain scenario where different parts of the cluster operate independently.

Upon rejoining, the node synchronizes using IST (Incremental State Transfer). IST allows the node to catch up with the current state of the cluster by transferring only the changes that occurred while the node was offline. If the necessary changes for IST do not exist in the `gcache` file on any other node within the cluster, the process will perform SST (State Snapshot Transfer) instead. SST involves transferring a complete database snapshot to the node, which can be more time-consuming but ensures that the node receives all data. This approach makes restarting cluster nodes for rolling configuration changes, or software upgrades straightforward from the cluster’s perspective. 

If a node restarts with an invalid configuration change that prevents MySQL from loading, Galera drops the node’s state and forces an SST for that node.

In the event of a MySQL failure, the system does not remove the PID file because the system deletes this file only during a clean shutdown. As a result, the server does not restart if an existing PID file is present. When MySQL encounters a failure, check the log records for details. You must remove the PID file manually. 

Use the `rm` command in a Unix/Linux shell to do this:

```shell
bash rm /path/to/mysql.pid
```

Replace `/path/to/mysql.pid` with the actual path to your MySQL PID file. The default location for the PID file is often `/var/run/mysqld/mysqld.pid` or `/var/lib/mysql/mysql.pid`, but this can vary based on your configuration. Before executing this command, ensure that MySQL is not running, as removing the PID file while the server is active can lead to issues.