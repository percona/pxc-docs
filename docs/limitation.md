# Percona XtraDB Cluster limitations

The following limitations apply to Percona XtraDB Cluster:

* Replication works only with InnoDB storage engine.

    Any writes to tables of other types are not replicated.

* Unsupported queries:

    - `LOCK TABLES` and `UNLOCK TABLES` are not supported in multi-source configurations.

    - Lock functions, such as `GET_LOCK()` and `RELEASE_LOCK()`, are not supported.

* Query log cannot be directed to table.

    If you enable query logging, you must forward the log to a file:

    ```text
    log_output = FILE
    ```

    Use `general_log` and `general_log_file` to choose query logging
    and the log file name.

* Maximum allowed transaction size is defined by the [`wsrep_max_ws_rows`](wsrep-system-index.md#wsrep_max_ws_rows) and [`wsrep_max_ws_size`](wsrep-system-index.md#wsrep_max_ws_size) variables.

    `LOAD DATA INFILE` processing commits every 10 000 rows.
    Percona XtraDB Cluster splits large `LOAD DATA` transactions into a series of smaller transactions.

* Transaction issuing `COMMIT` may still be aborted at that stage.

    Cluster-level optimistic concurrency control allows two transactions to write to the same rows on separate Percona XtraDB Cluster nodes.
    Only one of these transactions commits successfully.
    The other transaction is aborted.
    Percona XtraDB Cluster returns the following deadlock error code for cluster-level aborts:

    ??? example "Error message"

        ```{.text .no-copy}
        (Error: 1213 SQLSTATE: 40001  (ER_LOCK_DEADLOCK)).
        ```

* XA transactions are not supported

    Due to possible rollback on commit.

* Write throughput of the whole cluster is limited by the weakest node.

    A slow node slows the entire cluster.
    Provision matching hardware on every node when you require stable high performance.

* The minimum recommended cluster size is three nodes.

    The third node can act as an arbitrator.

* `enforce_storage_engine=InnoDB` is not compatible with `wsrep_replicate_myisam=OFF`

    [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) is set to `OFF` by default.

* Avoid `ALTER TABLE ... IMPORT/EXPORT` workloads when running Percona XtraDB Cluster in cluster mode.

    These statements can produce node inconsistency when not run in sync on every node.

* All tables must have a primary key.

    A primary key keeps row order identical on every node.
    The `DELETE` statement is not supported on tables without a primary key.

    When [`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode) is `ENFORCING` or `MASTER`, Percona XtraDB Cluster sets the global [`sql_require_primary_key`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_sql_require_primary_key) variable to `ON` and rejects attempts to set it `OFF`.
    `CREATE TABLE` and `ALTER TABLE` statements that would leave a table without a primary key fail.
    For details, see [Primary key requirement](strict-mode.md#primary-key-requirement).

    !!! admonition "See also"

        [MariaDB Galera Documentation: Tables without Primary Keys](https://mariadb.com/docs/server/architecture/server-constraints/primary-key-constraints)

* When configuring Percona XtraDB Cluster, your server must create a local socket.

    Configure the socket by providing a path, or skip the explicit configuration to use the default.
    Do not leave the `socket` variable in `my.cnf` empty, as in `socket=`.
    An empty value prevents the server from creating a socket.

    State Snapshot Transfer (SST) requires the socket for the following tasks:

    - Taking a backup with Percona XtraBackup

    - Detecting keyring component status

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

Starting with version 8.0.21, an `INPLACE` [`ALTER TABLE`](https://dev.mysql.com/doc/refman/8.0/en/alter-table.html) query takes an internal shared lock on the table during query execution.
The `LOCK=NONE` clause is no longer allowed for any `INPLACE ALTER TABLE` query.
The lock change addresses a deadlock that could cause a cluster node to hang in the following scenario:

* An INPLACE `ALTER TABLE` query in one session or being applied as Total Order Isolation (TOI)

* A DML on the same table from another session

Do not use one or more dot characters (.) when defining the values for the following variables:

* [log_bin](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#option_mysqld_log-bin)

* [log_bin_index](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#option_mysqld_log-bin-index)

MySQL and Percona XtraBackup handle the character in different ways and this difference causes unpredictable behavior.
