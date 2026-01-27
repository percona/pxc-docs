# State snapshot transfer

??? example "Key takeaways"

    Here are the top three takeaways from the "State Snapshot Transfer" documentation:
    
    * **Purpose of SST**:  
       State Snapshot Transfer (SST) is a critical process in Percona XtraDB Cluster that synchronizes a new or recovering node (joiner) with the cluster by transferring a consistent data snapshot from an existing node (donor).
    
    * **Xtrabackup as the Recommended Method**:  
       The default and most recommended SST method is `xtrabackup-v2`, which uses Percona XtraBackup. It is a non-blocking method that ensures the donor node remains operational during the transfer, leveraging backup locks for efficiency and consistency.
    
    * **Configuration and Variables**:  
       SST methods are configured using the `wsrep_sst_method` variable. Proper configuration of SST-related variables, such as `gcs.sync_donor`, is essential to avoid cluster-wide blocking and ensure smooth synchronization.
    
### Overview  

State Snapshot Transfer (SST) is essential in Percona XtraDB Cluster for synchronizing data between nodes when a new node joins the cluster or requires a full state transfer. The SST process ensures data consistency and cluster integrity during these operations.  

The [**`xtrabackup` SST method**](xtrabackup-sst.md) is the recommended choice for most scenarios. It is a robust, reliable, and non-blocking method that allows the donor node to remain operational during the process. By creating consistent backups and efficiently transferring them to the joiner node, `xtrabackup` minimizes downtime and resource usage while maintaining data integrity.  

The [**Clone SST method**](clone-sst.md) is another option for users seeking a modern and efficient approach. This method leverages MySQL’s native cloning capabilities to transfer data at the file level. While `xtrabackup` remains the primary choice, the Clone SST method can be particularly useful for scenarios where speed and simplicity are prioritized.  

Choosing the appropriate SST method depends on your environment, requirements for performance, and resource considerations. Both options ensure consistency between nodes and support seamless cluster synchronization.

### Xtrabackup SST Method  

Percona XtraDB Cluster supports the **xtrabackup** SST method, which is the recommended option for most use cases.  

Xtrabackup SST uses [backup locks](https://docs.percona.com/percona-server/8.0/backup-locks.html), ensuring that the Galera provider remains operational without being paused, unlike earlier approaches. This method ensures a seamless data synchronization process during State Snapshot Transfers.  

The SST method is configured using the [`wsrep_sst_method`](wsrep-system-index.md#wsrep_sst_method) variable.  

!!! note  

    If the [`gcs.sync_donor`](wsrep-provider-index.md#gcssync_donor) variable is set to `Yes` (default is `No`), the entire cluster will be blocked if the donor node is blocked by SST.  


## Limitation

When configuring Percona XtraDB Cluster, your server must create a local socket. You can set up a socket by providing a path, or you can skip creating one explicitly. However, do not leave the <socket> variable in my.cnf empty, like this: `socket=`. If you do, the server won’t create a socket. State Snapshot Transfer (SST) requires the local socket for the following tasks:
  
* Taking backup using Percona XtraBackup
    
* Detecting keyring component status

## Choose the SST Donor

If there are no nodes available
that can safely perform incremental state transfer (IST),
the cluster defaults to SST.

If there are nodes available that can perform IST,
the cluster prefers a local node over remote nodes to serve as the donor.

If there are no local nodes available that can perform IST,
the cluster chooses a remote node to serve as the donor.

If there are several local and remote nodes that can perform IST,
the cluster chooses the node with the highest `seqno` to serve as the donor.

## Use Percona Xtrabackup

The default SST method is `xtrabackup-v2` which uses *Percona XtraBackup*.
This is the least blocking method that leverages [backup locks](https://docs.percona.com/percona-xtrabackup/8.0/xtrabackup-option-reference.html#backup-locks).
XtraBackup is run locally on the donor node.

The [datadir](glossary.md#datadir) needs to be specified in the server configuration file `my.cnf`, otherwise the transfer process will fail.

Detailed information on this method is provided in [Percona XtraBackup SST Configuration](xtrabackup-sst.md#percona-xtrabackup-sst-configuration) documentation.

## SST for tables with tablespaces that are not in the data directory

For example:

```sql
CREATE TABLE t1 (c1 INT PRIMARY KEY) DATA DIRECTORY = '/alternative/directory';
```

### SST using Percona XtraBackup

XtraBackup will restore the table to the same location on the joiner node.  If
the target directory does not exist, it will be created. If the target file
already exists, an error will be returned, because XtraBackup cannot clear
tablespaces not in the data directory.

For more information, see:  

* [Clone SST](clone-sst.md)  

* [Xtrabackup SST configuration](xtrabackup-sst.md#percona-xtrabackup-sst-configuration)  

* [State Snapshot Transfer Methods for MySQL](https://mariadb.com/docs/galera-cluster/high-availability/state-snapshot-transfers-ssts-in-galera-cluster/introduction-to-state-snapshot-transfers-ssts/)  
