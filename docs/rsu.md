# Rolling schema upgrade

RSU, or Rolling Schema Upgrade, is a method for altering the structure of a database schema without bringing down the entire cluster. RSU updates one database at a time. This method means there's less downtime for your application. However, it also means there could be periods where different databases have different structures, which can sometimes cause problems.

Thoroughly test RSU procedures in a non-production environment before deploying in production. Closely monitor the cluster during the upgrade to identify and promptly address any issues.

Use RSU in the following scenarios:

* You must perform the schema upgrade with as little disruption to the database service as possible. This is where RSU (Rolling Schema Upgrade) excels. By updating one node at a time, RSU reduces the overall downtime compared to traditional methods that require taking the entire cluster offline.

* If the schema change is considered compatible with different schema versions, it can be applied to a database with an older schema without causing errors or data loss. This means that even though some databases in the cluster might have an older structure, the new schema change can still be applied to them successfully.

  For example, adding a new column to a table is often compatible with different schema versions, as older databases can ignore the new column without affecting existing data. However, removing a column or changing a column's data type might not be compatible, as it could impact data integrity on databases with the older schema.

* There might be short periods where the data on different servers doesn't perfectly match. This is because one server is being updated while others are still using the old database structure version. If your application can handle these minor differences for a short time, then RSU might be a good option for you.


## How RSU works

1. Node Isolation: Isolate a single node from the cluster.

2. Schema Change: Apply the schema change on the isolated node.

3. Rejoining the Cluster: Rejoin the node to the cluster after completing the schema change.

4. Replication: Replicate the schema change to the other nodes in the cluster.

## Advantages

| Advantage       | Description                                                                                      |
|-----------------|--------------------------------------------------------------------------------------------------|
| Reduced Downtime | As only one node is isolated at a time, the overall impact on the cluster is minimized.           |
| Flexibility      | RSU can handle a wider range of schema changes compared to TOI.                                   |

## Disadvantages

| Disadvantage                 | Description                                                                                                                     |
|------------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| Potential Data Inconsistencies | Since nodes have different schema versions during the upgrade, there's a risk of data inconsistencies if applications aren't carefully managed. |
| Increased Complexity           | RSU requires more careful planning and execution compared to TOI.                                                             |
| Longer Upgrade Time            | The entire upgrade process can take longer due to the sequential nature of the updates on each node.                          |





