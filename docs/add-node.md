# Add nodes to cluster

New nodes that are [properly configured](configure-nodes.md#configure) are provisioned
automatically.  When you start a node with the address of at least one other
running node in the [`wsrep_cluster_address`](wsrep-system-index.md#wsrep_cluster_address) variable, this node automatically joins and synchronizes with the cluster.

!!! note

    Any existing data and configuration will be overwritten
    to match the data and configuration of the DONOR node.
    Do not join several nodes at the same time
    to avoid overhead due to large amounts of traffic when a new node joins. 

Percona XtraDB Cluster uses [Percona XtraBackup](https://www.percona.com/software/mysql-database/percona-xtrabackup) for [State Snapshot Transfer](glossary.md#sst) and the `wsrep_sst_method` variable is always set to `xtrabackup-v2`.

## Start the second node

Start the second node using the following command:

```{.bash data-prompt="[root@pxc2 ~]#"}
[root@pxc2 ~]# systemctl start mysql
```

After the server starts, it receives [SST](glossary.md#sst) automatically.

To check the status of the second node, run the following:

```{.bash data-prompt="mysql@pxc2>"}
mysql@pxc2> show status like 'wsrep%';
```

??? example "Expected output"

    ```{.text .no-copy}
    +----------------------------------+--------------------------------------------------+
    | Variable_name                    | Value                                            |
    +----------------------------------+--------------------------------------------------+
    | wsrep_local_state_uuid           | a08247c1-5807-11ea-b285-e3a50c8efb41             |
    | ...                              | ...                                              |
    | wsrep_local_state                | 4                                                |
    | wsrep_local_state_comment        | Synced                                           |
    | ...                              |                                                  |
    | wsrep_cluster_size               | 2                                                |
    | wsrep_cluster_status             | Primary                                          |
    | wsrep_connected                  | ON                                               |
    | ...                              | ...                                              |
    | wsrep_provider_capabilities      | :MULTI_MASTER:CERTIFICATION: ...                 |
    | wsrep_provider_name              | Galera                                           |
    | wsrep_provider_vendor            | Codership Oy <info@codership.com>                |
    | wsrep_provider_version           | 4.3(r752664d)                                    |
    | wsrep_ready                      | ON                                               |
    | ...                              | ...                                              | 
    +----------------------------------+--------------------------------------------------+
    75 rows in set (0.00 sec)
    ```

The output of `SHOW STATUS` shows that the new node has been successfully
added to the cluster.  The cluster size is now 2 nodes, it is the primary
component, and it is fully connected and ready to receive write-set replication.

If the state of the second node is `Synced` as in the previous example, then
the node received full [SST](glossary.md#sst) is synchronized with the cluster, and you can
proceed to add the next node.

!!! note

    If the state of the node is `Joiner`, it means that SST hasn’t finished. Do not add new nodes until all others are in `Synced` state. 

## Starting the Third Node

To add the third node, start it as usual:

```{.bash data-prompt="[root@pxc3 ~]#"}
[root@pxc3 ~]# systemctl start mysql
```

To check the status of the third node, run the following:

```{.bash data-prompt="mysql@pxc3>"}
mysql@pxc3> show status like 'wsrep%';
```

The output shows that the new node has been successfully added to the
cluster. Cluster size is now 3 nodes, it is the primary component, and it is
fully connected and ready to receive write-set replication.

??? example "Expected output"

    ```{.text .no-copy}
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

## Next steps

When you add all nodes to the cluster, you can [verify replication](verify-replication.md#verify) by running queries and manipulating data on nodes to see if these changes are synchronized across the cluster.

