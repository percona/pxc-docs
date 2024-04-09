# Percona XtraDB Cluster Limitations

The following limitations apply to Percona XtraDB Cluster:

* Replication works only with *InnoDB* storage engine.
Any writes to tables of other types, including system (`mysql.\*`) tables,
are not replicated.
However, `DDL` statements are replicated in the statement level,
and changes to `mysql.\*` tables are replicated that way.
So you can safely issue `CREATE USER...`,
but issuing `INSERT INTO mysql.user...` will not be replicated.
You can enable experimental *MyISAM* replication support
using the [`wsrep_replicate_myisam`](wsrep-system-index.md#wsrep_replicate_myisam) variable.

* Unsupported queries:

    * `LOCK TABLES` and `UNLOCK TABLES` is not supported in multi-source setups

    * Lock functions, such as `GET_LOCK()`, `RELEASE_LOCK()`, and so on

    !!! admonition "See also"

         *MySQL* Documentation:

         * [LOCK TABLES AND UNLOCK TABLES statements](https://dev.mysql.com/doc/refman/8.0/en/lock-tables.html)

         * [Locking functions](https://dev.mysql.com/doc/refman/5.7/en/locking-functions.html)

    * Query log cannot be directed to table.

  If you enable query logging, you must forward the log to a file:

  ```text
  log_output = FILE
  ```

  Use `general_log` and `general_log_file` to choose query logging
  and the log file name.

* Maximum allowed transaction size is defined by the [`wsrep_max_ws_rows`](wsrep-system-index.md#wsrep_max_ws_rows) and [`wsrep_max_ws_size`](wsrep-system-index.md#wsrep_max_ws_size) variables. 
  `LOAD DATA INFILE` processing will commit every 10,000 rows.
  So large transactions due to `LOAD DATA`
  will be split to series of small transactions.

* Due to cluster-level optimistic concurrency control, a
  transaction issuing a `COMMIT` may still be aborted at that stage.
  There can be two transactions writing to the same rows
  and committing in separate Percona XtraDB Cluster [nodes](glossary.md#node),
  and only one of the them can successfully commit.
  The failing one will be aborted.
  For cluster-level aborts, Percona XtraDB Cluster gives back deadlock error code:

  ```text
  (Error: 1213 SQLSTATE: 40001  (ER_LOCK_DEADLOCK)).
  ```

* [XA transactions](https://dev.mysql.com/doc/refman/5.7/en/xa.html) are not supported due to possible rollback on commit.

* The write throughput of the whole cluster is limited by the weakest [node](glossary.md#node_1).  If one node becomes slow, the whole cluster slows down. If you have requirements for stable high performance, then it should be supported by corresponding hardware.

* The minimal recommended size of cluster is three nodes.  The third node can be an [arbitrator](https://galeracluster.com/library/documentation/arbitrator.html).

* [InnoDB fake changes feature](https://www.percona.com/doc/percona-server/5.5/management/innodb_fake_changes.html) is not supported. This feature has been removed.

* `enforce_storage_engine=InnoDB` is not compatible with `wsrep_replicate_myisam=OFF` (default).

!!! admonition "See also"

    [Percona Server for MySQL documentation: enforcing storage engine](https://www.percona.com/doc/percona-server/5.7/management/enforce_engine.html)


* When running Percona XtraDB Cluster in cluster mode,
avoid `ALTER TABLE ... IMPORT/EXPORT` workloads. It can lead to [node](glossary.md#node_1) inconsistency if not executed in sync on all nodes.

* All tables must have the primary key. This ensures that the same rows appear
in the same order on different [nodes](glossary.md#node). The `DELETE` statement is not supported on tables without a primary key.

* Percona Server 5.7 data at rest encryption is similar to the [MySQL 5.7 data-at-rest encryption](https://dev.mysql.com/doc/refman/5.7/en/innodb-data-encryption.html). Review the available encryption features for [Percona Server for MySQL 5.7](https://www.percona.com/doc/percona-server/5.7/security/data-at-rest-encryption.html). Percona Server 8.0 provides more encryption features and options which are not available in this version.

!!! admonition "See also"

    [Galera Documentation: Tables without Primary Keys](https://galeracluster.com/documentation-webpages/limitations.html#tables-without-primary-keys)

* Avoid reusing the names of a persistent table for a temporary table. Although MySQL allows a temporary table and a persistent table to have the same name, this approach is not recommended. If a persistent table name matches a temporary table name, Galera Cluster blocks the replication to that table.

  With wsrep_debug set to *1*, the error log may contain the following message:

  ```text
  ... [Note] WSREP: TO BEGIN: -1, 0 : create table t (i int) engine=innodb
  ... [Note] WSREP: TO isolation skipped for: 1, sql: create table t (i int) engine=innodb.Only temporary tables affected.
  ```

* As of version 5.7.32-13.47, an INPLACE [ALTER TABLE](https://dev.mysql.com/doc/refman/5.7/en/alter-table.html)  query takes an internal shared lock on the table during the execution of the query. The `LOCK=NONE` clause is no longer allowed for all of the INPLACE ALTER TABLE queries due to this change.

  This change addresses a deadlock, which could cause a cluster node to hang in the following scenario:

* An INPLACE `ALTER TABLE` query in one session or being applied as Total Order Isolation (TOI)

* A DML on the same table from another session

  Do not use one or more dot characters (.) when defining the values for the following variables:

* [log_bin](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#option_mysqld_log-bin)

* [log_bin_index](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#option_mysqld_log-bin-index)

  MySQL and **XtraBackup** handles the value in different ways and this difference causes unpredictable behavior.
