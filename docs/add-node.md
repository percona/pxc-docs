# Adding Nodes to Cluster

New `nodes <Node>` that are [properly configured](configure.md#configure)
are provisioned automatically.
When you start a node with the address of at least one other running `node`
in the [`wsrep_cluster_address`](wsrep-system-index.md#wsrep_cluster_address) variable,
it automatically joins the cluster and synchronizes with it.

!!! note

    Any existing data and configuration will be overwritten
    to match the data and configuration of the DONOR node.
    Do not join several nodes at the same time
    to avoid overhead due to large amounts of traffic when a new node joins. 

By default, Percona XtraDB Cluster uses [Percona XtraBackup](https://www.percona.com/software/mysql-database/percona-xtrabackup) for *State Snapshot Transfer* (SST).
This requires the following:

* Set the `wsrep_sst_method` variable to `xtrabackup-v2` and provide SST user credentials with the `wsrep_sst_auth` variable.

  For more information, see [Configuring Nodes for Write-Set Replication](configure.md#configure).

* Create a user for SST on the initial node.

  For more information, see [Bootstrapping the First Node](bootstrap.md#bootstrap).

## Starting the Second Node

Start the second node by using either of the following commands:

```shell
[root@pxc2 ~]# /etc/init.d/mysql start
```

or

```shell
[root@pxc2 ~]# systemctl start mysql
```

After the server starts, it should receive SST automatically.

To check the status of the second node, run the following:

```sql
mysql@pxc2> show status like 'wsrep%';
```

The output shows that the new node
has been successfully added to the cluster.
Cluster size is now 2 nodes, it is the primary component,
and it is fully connected and ready to receive write-set replication.

```text
+----------------------------+--------------------------------------+
| Variable_name              | Value                                |
+----------------------------+--------------------------------------+
| wsrep_local_state_uuid     | c2883338-834d-11e2-0800-03c9c68e41ec |
| ...                        | ...                                  |
| wsrep_local_state          | 4                                    |
| wsrep_local_state_comment  | Synced                               |
| ...                        | ...                                  |
| wsrep_cluster_size         | 2                                    |
| wsrep_cluster_status       | Primary                              |
| wsrep_connected            | ON                                   |
| ...                        | ...                                  |
| wsrep_ready                | ON                                   |
+----------------------------+--------------------------------------+
40 rows in set (0.01 sec)
```

If the state of the second node is `Synced` as in the previous example,
then the node received full SST, is synchronized with the cluster,
and you can proceed to add the next node.

!!! note

    If the state of the node is `Joiner`, it means that SST hasn’t finished. Do not add new nodes until all others are in `Synced` state. 


## Starting the Third Node

To add the third node, start the node using either command:

```shell
[root@pxc3 ~]# /etc/init.d/mysql start
```

or

```shell
[root@pxc3 ~]# systemctl start mysql
```

To check the status of the third node, run the following:

```sql
mysql@pxc3> show status like 'wsrep%';
```

The output shows that the new node
has been successfully added to the cluster.
Cluster size is now 3 nodes, it is the primary component,
and it is fully connected and ready to receive write-set replication.

```text
+----------------------------+--------------------------------------+
| Variable_name              | Value                                |
+----------------------------+--------------------------------------+
| wsrep_local_state_uuid     | c2883338-834d-11e2-0800-03c9c68e41ec |
| ...                        | ...                                  |
| wsrep_local_state          | 4                                    |
| wsrep_local_state_comment  | Synced                               |
| ...                        | ...                                  |
| wsrep_cluster_size         | 3                                    |
| wsrep_cluster_status       | Primary                              |
| wsrep_connected            | ON                                   |
| ...                        | ...                                  |
| wsrep_ready                | ON                                   |
+----------------------------+--------------------------------------+
40 rows in set (0.01 sec)
```
 
## Next Steps

When you add all nodes to the cluster,
you can [verify replication](verify.md#verify)
by running queries and manipulating data on nodes
to see if these changes are synchronized accross the cluster.
