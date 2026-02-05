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
    and only one of the them can successfully commit. The failing one will be
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

        [MariaDB Galera Documentation: Tables without Primary Keys :octicons-link-external-16:](https://mariadb.com/docs/server/server-usage/tables/mariadb-indexes-guide-1#finding-tables-without-primary-keys)

* Avoid reusing the names of persistent tables for temporary tables

    Even though MySQL allows temporary tables to have the same names as persistent tables, using this approach is discouraged. Galera Cluster prevents replication of persistent tables with the same names as temporary tables.

    With wsrep_debug set to *1*, the error log may contain the following message:

    ??? example "Error message"

        ```{.text .no-copy}
        ... [Note] WSREP: TO BEGIN: -1, 0 : create table t (i int) engine=innodb
        ... [Note] WSREP: TO isolation skipped for: 1, sql: create table t (i int) engine=innodb.Only temporary tables affected.
        ```

    !!! admonition "See also"

        [MySQL Documentation: Problems with temporary tables :octicons-link-external-16:](https://dev.mysql.com/doc/refman/{{vers}}/en/temporary-table-problems.html)


An INPLACE [ALTER TABLE :octicons-link-external-16:](https://dev.mysql.com/doc/refman/{{vers}}/en/alter-table.html) query takes an internal shared lock on the table during the execution of the query. Due to this change, the `LOCK=NONE` clause is no longer allowed for all `INPLACE ALTER TABLE` queries.

This change addresses a deadlock, which could cause a cluster node to hang in the following scenario:

* An INPLACE `ALTER TABLE` query in one session or being applied as Total Order Isolation (TOI)

* A DML on the same table from another session

Do not use one or more dot characters (.) when defining the values for the following variables:

* [log_bin :octicons-link-external-16:](https://dev.mysql.com/doc/refman/{{vers}}/en/replication-options-binary-log.html#option_mysqld_log-bin)

* [log_bin_index :octicons-link-external-16:](https://dev.mysql.com/doc/refman/{{vers}}/en/replication-options-binary-log.html#option_mysqld_log-bin-index)

MySQL and **XtraBackup** handle the value in different ways, and this difference causes unpredictable behavior.
