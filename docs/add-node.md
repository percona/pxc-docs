# Add nodes to cluster

Percona XtraDB Cluster uses the [`wsrep_cluster_address`](wsrep-system-index.md#wsrep_cluster_address) variable to determine how a node starts. An empty value initializes the first node and starts the cluster. See [Bootstrap the first node](bootstrap.md). A value with the address of at least one running node adds the new node to an existing cluster. Percona XtraDB Cluster provisions properly configured new nodes automatically.

!!! note

    Before you add a node to the cluster, review the following:

    * When a node joins the cluster, State Snapshot Transfer replaces its existing data and configuration with data from the DONOR node.

    * When multiple nodes join at the same time, network traffic increases. Add nodes one at a time.

    * SSL certificates must exist on the first node and on every node you plan to add. Use identical key and certificate files on all nodes. See [Generate and copy SSL certificates](bootstrap.md#generate-and-copy-ssl-certificates).

Percona XtraDB Cluster uses [Percona XtraBackup](https://www.percona.com/software/mysql-database/percona-xtrabackup) for [State Snapshot Transfer](glossary.md#sst). The [`wsrep_sst_method`](wsrep-system-index.md#wsrep_sst_method) variable is always set to `xtrabackup-v2`.

## Start the first node

To start the first node, see [Bootstrap the first node](bootstrap.md). The first node initializes the cluster. Nodes added after the first node join an existing cluster.

## Start the second node

Start the second node, `[root@pxc2 ~]#`, using the following command:

```sql
systemctl start mysql
```

The node starts and receives SST automatically.

To check the status of the second node, `mysql@pxc2>`, run the following:

```sql
SHOW STATUS LIKE 'wsrep%';
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

The `SHOW STATUS` output confirms that the second node joined the cluster:

* Cluster size: two nodes

* Cluster status: primary component

* Connection status: fully connected and ready for write-set replication

If the second node's state is `Synced`, the node received full [SST](glossary.md#sst) and synchronized with the cluster. This state matches the previous example. You can add the next node.

!!! note

    If the node's state is `Joiner`, SST has not finished. Do not add new nodes until all other nodes reach `Synced` state.

## Start the third node

To add the third node, `[root@pxc3 ~]#`, start the node:

```sql
systemctl start mysql
```

To check the status of the third node, `mysql@pxc3>`, run the following:

```sql
SHOW STATUS LIKE 'wsrep%';
```

The `SHOW STATUS` output confirms that the third node joined the cluster:

* Cluster size: three nodes

* Cluster status: primary component

* Connection status: fully connected and ready for write-set replication

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

After you add all nodes to the cluster, [verify replication](verify-replication.md#verify-replication). Run queries and change data on different nodes. Confirm that the cluster synchronizes the changes across all nodes.
