# Total Order Isolation



TOI (Total Order Isolation) is a method used in  Percona XtraDB Cluster (PXC) to handle DDL (Data Definition Language) statements. DDL statements are commands that define or alter the structure of database objects, like tables or indexes. TOI ensures that these changes happen consistently across all nodes in the cluster.

TOI offers strong data consistency guarantees for online schema changes in PXC. However, it comes with the trade-off of potentially impacting performance and introducing complexities. Weigh the advantages and disadvantages based on your specific needs and the complexity of your schema changes.

## Advantages

| Advantage              | Description                                                                                                                                                                      |
|------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Strong Data Consistency| TOI makes sure that all nodes in the PXC cluster apply schema changes in the exact same order. This means all nodes have identical schemas at any given time, preventing inconsistencies and potential data corruption. |
| Simplicity             | You can set the configuration option, `wsrep_OSU_method=TOI`.                                                                                                                     |
| Predictability         | Since all nodes apply changes in the same order, you can predict the outcome, minimizing surprises during schema updates.                                                         |

## Disadvantages

| Disadvantage            | Description                                                                                                                                                                                                                  |
|-------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Blocking Commits        | During a TOI schema change, all other transactions on the cluster are blocked until the schema change is completed on all nodes. This can lead to temporary performance slowdowns or pauses in application activity depending on the complexity of the schema change and workload. |
| Increased Load          | Replicating the complete schema change statement to all nodes can put additional load on the network and cluster resources.                                                                                                  |
| Potential Deadlocks     | In rare cases, complex schema changes involving multiple tables or operations might lead to deadlocks. YOu may need to manually resolve these issues.                                                                        |
| Error Handling          | If the schema change fails on one node, TOI doesn't perform a rollback on other nodes. This can leave the cluster in an inconsistent state, requiring additional steps to recover.                                              |
| Complexity of Changes   | The impact of TOI increases with the complexity of the schema changes. Simple changes like adding columns might have minimal performance impact, while complex changes involving multiple tables or data manipulation could cause significant slowdowns. |
| Testing                 | Thoroughly testing schema changes in a non-production environment before applying them in a production PXC cluster is crucial.                                                                                               |


## How TOI handles DDL statements

| Action | Description |
|--------|-------------|
| Statement reception | When a DDL statement is issued, the node that receives it broadcasts this statement to all other nodes in the cluster. |
| Write suspension | All nodes in the cluster temporarily suspend write operations. This means that no new data modifications can occur during this period. |
| Transaction queue | Any ongoing transactions are allowed to complete, but new transactions are queued up and wait for the DDL operation to finish. |
| Synchronization point | The cluster establishes a synchronization point where all nodes agree to apply the DDL statement. |
| Schema change application | Once the synchronization point is reached, all nodes simultaneously apply the DDL statement to their local database schemas. |
| Verification | Each node verifies that the schema change has been applied successfully. |
| Write resumption | After successful verification, the cluster resumes normal operations. Queued transactions are then processed, and new write operations are allowed. |
| Completion notification | The system notifies the client that issued the DDL statement that the operation has been completed successfully. |

