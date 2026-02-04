# High availability

Percona XtraDB Cluster will continue to function in a basic setup with three nodes if you take any of the nodes down. At any point, you can shut down any node to perform maintenance or make configuration changes. Even in unplanned situations such as a node crash or a network failure that makes a node unavailable, Percona XtraDB Cluster continues to operate, and you can run queries on the remaining active nodes.

If data changed while a node was down, the node has two options when rejoining the cluster:


| Option                     | Description |
|----------------------------|-------------|
| State Snapshot Transfer (SST) | SST copies all data from one node to another. The cluster typically uses it when a new node joins and needs to receive the full dataset from an existing node. Percona XtraDB Cluster performs SST using xtrabackup. During this process, xtrabackup does not lock the database for the entire sync; it uses the READ LOCK command only when syncing .frm files, just like in a regular backup. |
| Incremental State Transfer (IST) | IST transfers only incremental changes from one node to another. It avoids the disruption of SST by letting a briefly offline node fetch only the changes made during its downtime without setting the cluster to read-only. Each node implements IST using a caching mechanism. It maintains a ring buffer of a configurable size that stores the last N changes. If the node finds the needed changes within the buffer, it transfers them through IST. If the changes exceed the buffer size, the node falls back to using SST. |

You can monitor the current state of a node using the following command:

```sql
SHOW STATUS LIKE 'wsrep_local_state_comment';
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------------+--------+
    | Variable_name             | Value  |
    +---------------------------+--------+
    | wsrep_local_state_comment | Synced |
    +---------------------------+--------+
    ```

When a node reaches the `Synced` state, the node has completed the process of synchronizing its data with the rest of the cluster. At this point, the cluster considers the node to be fully integrated and operational. The node can now begin processing client requests, participating in read and write operations, and contributing to the overall workload distribution within the cluster.
