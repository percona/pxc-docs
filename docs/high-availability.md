# High availability

In a basic setup with three nodes, if you take any of the nodes down, Percona
 XtraDB Cluster continues to function.
At any point, you can shut down any node to perform maintenance
or configuration changes.

Even in unplanned situations (like a node crashing or if it becomes unavailable
 over the network), you can run queries on working nodes. If a node is down and
 the data has changed, there are two methods that the node may use when it
 joins the cluster again:

| Method | What happens | Description  |
|---|---|--- |
| SST  | The joiner node receives a full copy of the database state from the donor node. | You initiate a Solid State Transfer (SST) when adding a new node to a Galera cluster or when a node has fallen too far out of sync|
| IST  | Only incremental changes are copied from one node to another. | This operation can be used when a node is down for a short period.|                                                                         
                              
## SST

The primary benefit of SST is that it ensures data consistency across the
 cluster by providing a complete snapshot of the database at a point in time.
However, SST can be resource-intensive and time-consuming if the operation transfers significant data. The donor node is locked during this transfer, impacting cluster performance.

You initiate a state snapshot transfer (SST) when a node joins a cluster
 without the complete data set. This process involves transferring a full data
 copy from one node to another, ensuring that the joining node has an exact
 replica of the cluster's current state. Technically, SST is performed by
 halting the donor node's database operations momentarily to create a consistent
 snapshot of its data. The snapshot is then transferred over the network to the
 joining node, which applies it to its database system.

Even without locking your cluster in a read-only state, SST may be intrusive
 and disrupt the regular operation of your services.
IST avoids disruption. A node fetches only the changes that happened while that
node was unavailable. IST uses a caching mechanism on nodes.

## IST 

Incremental State Transfer (IST) is a method that allows a node to request only
 the missing transactions from another node in the cluster. This process is
 beneficial because it reduces the amount of data that must be transferred,
 leading to faster recovery times for nodes that are out of sync. Additionally,
 IST minimizes the network bandwidth required for state transfer, which is
 particularly advantageous in environments with limited resources.

However, there are drawbacks to consider. Reliance on another node's state means that an SST operation is necessary if no node in the cluster has the required information. 

When a node joins the cluster with a state slightly behind the current cluster state, IST does not require the joining node to copy the entire database state. Technically, IST transfers only the missing write-sets that the joining node needs to catch up with the cluster. The donor node, the node with the most recent state, sends the write-sets to the joining node through a dedicated channel. The joining node then applies these write-sets to its database state incrementally until it synchronizes with the cluster's current state. The donor node can experience a performance impact during an IST operation, typically less severe than during SST.

## Monitor the node state

The `wsrep_state_comment` variable returns the current state of a Galera node in the cluster, providing information about the node's role and status. The value can vary depending on the specific state of the Galera node, such as the following: 

* "Synced" 

* "Donor/Desynced" 

* "Donor/Joining"

* "Joined"

You can monitor the current state of a node using the following command:

```{.bash data-prompt="mysql>"}
mysql> SHOW STATUS LIKE 'wsrep_local_state_comment';
```

If the node is in `Synced (6)` state, that node is part of the cluster
and can handle the traffic.
