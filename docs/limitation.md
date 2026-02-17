# Percona XtraDB Cluster limitations

The following limitations apply to Percona XtraDB Cluster:

* Replication works only with InnoDB storage engine.

    Any writes to tables of other types are not replicated.

* Unsupported queries:

    `LOCK TABLES` and `UNLOCK TABLES` is not supported in multi-source setups

    Lock functions, such as `GET_LOCK()`, `RELEASE_LOCK()`, and so on

* Query log cannot be directed to table.

    If you enable query logging, you must forward the log to a file:

    ```text
    log_output = FILE
    ```

    Use `general_log` and `general_log_file` to choose query logging
    and the log file name.

* Maximum allowed transaction size is defined by the [`wsrep_max_ws_rows`](wsrep-system-index.md#wsrep_max_ws_rows) and [`wsrep_max_ws_size`](wsrep-system-index.md#wsrep_max_ws_size) variables. 

    `LOAD DATA INFILE` processing will commit every 10 000 rows.  So large
    transactions due to `LOAD DATA` will be split to series of small
    transactions.

* Transaction issuing `COMMIT` may still be aborted at that stage.

    Due to cluster-level optimistic concurrency control,  there can be two
    transactions writing to the same rows and committing in separate Percona XtraDB Cluster nodes,
    and only one of them can successfully commit. The failing one will be
    aborted. For cluster-level aborts, Percona XtraDB Cluster gives back deadlock error code:

    ??? example "Error message"

        ```{.text .no-copy}
        (Error: 1213 SQLSTATE: 40001  (ER_LOCK_DEADLOCK)).
        ```

* XA transactions are not supported

    Due to possible rollback on commit.

* Write throughput of the whole cluster is limited by the weakest node.

    If one node becomes slow, the whole cluster slows down.  If you have
    requirements for stable high performance, then it should be supported by
    corresponding hardware.

* Minimal recommended size of cluster is 3 nodes.

    The 3rd node can be an arbitrator.

* `enforce_storage_engine=InnoDB` is not compatible with `wsrep_replicate_myisam=OFF`

    [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) is set to `OFF` by default.

* Avoid `ALTER TABLE ... IMPORT/EXPORT` workloads when running Percona XtraDB Cluster in cluster mode.

    It can lead to node inconsistency if not executed in sync on all nodes.

* All tables must have a primary key.

    This ensures that the same rows appear in the same order on different
    nodes. The `DELETE` statement is not supported on tables without a primary
    key.

    !!! admonition "See also"

        [MariaDB Galera Documentation: Tables without Primary Keys](https://mariadb.com/docs/server/architecture/server-constraints/primary-key-constraints)
        
* When configuring Percona XtraDB Cluster, your server must create a local socket.

    You can set up a socket by providing a path, or you can skip creating one explicitly. However, do not leave the <socket> variable in my.cnf empty, like this: `socket=`. If you do, the server won’t create a socket.
   
    State Snapshot Transfer (SST) requires the socket for the following tasks:
  
    * Taking backup using Percona XtraBackup
    
    * Detecting keyring component status

* Avoid reusing the names of persistent tables for temporary tables

    Although MySQL does allow having temporary tables named the same as
    persistent tables, this approach is not recommended.

    Galera Cluster blocks the replication of those persistent tables
    the names of which match the names of temporary tables.

    With wsrep_debug set to *1*, the error log may contain the following message:

    ??? example "Error message"

        ```{.text .no-copy}
        ... [Note] WSREP: TO BEGIN: -1, 0 : create table t (i int) engine=innodb
        ... [Note] WSREP: TO isolation skipped for: 1, sql: create table t (i int) engine=innodb.Only temporary tables affected.
        ```

    !!! admonition "See also"

        [MySQL Documentation: Problems with temporary tables](https://dev.mysql.com/doc/refman/8.0/en/temporary-table-problems.html)

* Audit Log Filter state synchronization

    Percona XtraDB Cluster supports the Audit Log Filter component. While the underlying configuration tables (`mysql.audit_log_filter` and `mysql.audit_log_user`) are replicated to all nodes via Galera, the in-memory state of the audit component is local to each node.

    To ensure consistent auditing across the cluster, keep the following limitations in mind:

    * Manual Cache Refresh: Running an `INSERT`, `UPDATE`, or `DELETE` on the audit filter tables on one node replicates the data, but does not trigger a cache refresh on the other nodes.

    * Required Action: You must manually execute `SELECT audit_log_filter_flush();` on every node in the cluster after modifying the filter tables to ensure the nodes are using the same rules.

    * If the `mysql.audit_log_filter` or `mysql.audit_log_user` tables are modified using one of the following user-defined functions (UDFs), then `audit_log_filter_flush()` must also be run on all the other nodes:

        * `audit_log_filter_set_filter`

        * `audit_log_filter_set_user`

        * `audit_log_filter_remove_filter`

        * `audit_log_filter_remove_user`

    * Session Persistence: The audit log filter is applied at the start of a connection. Even after flushing the cache, existing sessions will continue to be governed by the previous filtering rules. Users must disconnect and reconnect for new filters to take effect.

    !!! admonition "See also"

        For more information on the audit log filter, see the [Percona Server 8.0 Audit Log Filter overview](https://docs.percona.com/percona-server/8.0/audit-log-filter-overview.html).

As of version 8.0.21, an INPLACE [ALTER TABLE](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html) query takes an internal shared lock on the table during the execution of the query. The `LOCK=NONE` clause is no longer allowed for all of the INPLACE ALTER TABLE queries due to this change.

This change addresses a deadlock, which could cause a cluster node to hang in the following scenario:

* An INPLACE `ALTER TABLE` query in one session or being applied as Total Order Isolation (TOI)

* A DML on the same table from another session

Do not use one or more dot characters (.) when defining the values for the following variables:

* [log_bin](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#option_mysqld_log-bin)

* [log_bin_index](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#option_mysqld_log-bin-index)

MySQL and Percona XtraBackup handle the character in different ways and this difference causes unpredictable behavior.
