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

State Snapshot Transfer (SST) plays a critical role in Percona XtraDB Cluster by synchronizing data between nodes when a new node joins the cluster or requires a full state transfer. SST ensures data consistency and cluster integrity during these operations.

The xtrabackup SST method is the recommended option for most scenarios. This method is robust, reliable, and non-blocking. The donor node remains operational throughout the transfer. By creating consistent backups and sending them efficiently to the joiner node, xtrabackup reduces downtime and conserves resources while preserving data integrity.

The Clone SST method offers an alternative for users who want a modern and efficient solution. This method uses MySQL’s native cloning to transfer data at the file level. While xtrabackup remains the preferred choice, Clone SST works well when speed and simplicity are top priorities.

Choose the SST method that best matches your environment, performance needs, and available resources. Both methods ensure consistent data across nodes and support reliable cluster synchronization.

### Xtrabackup SST Method  

Percona XtraDB Cluster supports the xtrabackup SST method and recommends this method for most use cases.

Unlike earlier methods, xtrabackup SST uses backup locks and keeps the Galera provider running without interruption. This design ensures seamless data synchronization during State Snapshot Transfers.

Set the `wsrep_sst_method` variable to configure the SST method.

State Snapshot Transfer (SST) performs a full data copy from a donor node to a joining node. When a new node joins the cluster, that node receives data from an existing cluster node to complete synchronization.

Percona XtraDB Cluster uses xtrabackup to handle this transfer process.

Xtrabackup SST uses backup locks, so the Galera provider continues running without any pause. Set the `wsrep_sst_method` variable to enable this method.

!!! note "Note"

    If the `gcs.sync_donor` variable is set to `Yes` (the default is `No`), the whole cluster will be blocked if SST blocks the donor.

The `xtrabackup` SST method uses the [Reduced backup lock time :octicons-link-external-16:](https://docs.percona.com/percona-xtrabackup/{{vers}}/reduction-in-locks.html) feature. This enhancement minimizes blocking on the **Donor node** during SST process while the backup is being prepared. The Percona XtraBackup reduced lock feature is enabled by default. To modify this behavior, set the desired `lock_ddl` value in the [xtrabackup] section of the `my.cnf` configuration file. For more information about the `--lock-ddl` option and its available values, refer to the [xtrabackup command-line options :octicons-link-external-16:](https://docs.percona.com/percona-xtrabackup/{{vers}}/xtrabackup-option-reference.html#lock-ddl) documentation.

## Limitation

Percona XtraDB Cluster requires a local socket for configuration. Define a socket path or allow the server to create one automatically. Do not leave the `socket` variable empty in the `my.cnf` file (for example, `socket=`). An empty value prevents the server from creating a socket.

State Snapshot Transfer (SST) uses the local socket to:
  
* Create backups with Percona XtraBackup 
    
* Check the status of the keyring component

## Choose the SST Donor

If no nodes are available to safely perform Incremental State Transfer (IST), the cluster defaults to State Snapshot Transfer (SST).

If IST-capable nodes are available, the cluster prefers a local donor over a remote one.

The cluster selects a remote donor when no local nodes can perform IST.

When both local and remote nodes can perform IST, the cluster selects the node with the highest `seqno` as the donor.

## Use Percona Xtrabackup

The default SST method is `xtrabackup-v2`, which uses *Percona XtraBackup*.
The least blocking method leverages [backup locks :octicons-link-external-16:](https://docs.percona.com/percona-server/{{vers}}/backup-locks.html).
XtraBackup is run locally on the donor node.

The [datadir](glossary.md#datadir) must be specified in the server configuration file `my.cnf`, otherwise the transfer process will fail.

Detailed information on this method is provided in [Percona XtraBackup SST Configuration](xtrabackup-sst.md#percona-xtrabackup-sst-configuration) documentation.

## SST for tables with tablespaces that are not in the data directory

For example:

```shell
CREATE TABLE t1 (c1 INT PRIMARY KEY) DATA DIRECTORY = '/alternative/directory';
```

### SST using Percona XtraBackup

XtraBackup restores the table to the same location on the joiner node. If the target directory does not exist, XtraBackup creates the directory. If the target file already exists, XtraBackup returns an error because XtraBackup cannot clear tablespaces outside the data directory.

## Other reading

* [State Snapshot Transfer Methods for MySQL :octicons-link-external-16:](https://mariadb.com/docs/galera-cluster/high-availability/state-snapshot-transfers-ssts-in-galera-cluster/introduction-to-state-snapshot-transfers-ssts/)

* [Xtrabackup SST configuration](xtrabackup-sst.md#percona-xtrabackup-sst-configuration)
