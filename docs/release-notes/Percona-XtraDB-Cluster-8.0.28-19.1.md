# Percona XtraDB Cluster 8.0.28-19.1 (2022-07-19)

Percona XtraDB Cluster (PXC) supports critical business applications in your public, private, or hybrid cloud environment. Our free, open source, enterprise-grade solution includes the high availability and security features your business requires to meet your customer expectations and business goals.

## Release Highlights

Improvements and bug fixes introduced by Oracle for *MySQL* 8.0.28 and included in Percona Server for MySQL are the following:


* The `ASCII` shortcut for `CHARACTER SET latin1` and `UNICODE` shortcut for `CHARACTER SET ucs2` are deprecated and raise a warning to use `CHARACTER SET` instead. The shortcuts will be removed in a future version.


* A stored function and a loadable function with the same name can share the same namespace. Add the schema name when invoking a stored function in the shared namespace. The server generates a warning when function names collide.


* InnoDB supports `ALTER TABLE ... RENAME COLUMN` operations when using `ALGORITHM=INSTANT`.


* The limit for `innodb_open_files` now includes temporary tablespace files. The temporary tablespace files were not counted in the `innodb_open_files` in previous versions.

Find the full list of bug fixes and changes in the [MySQL 8.0.28 Release Notes](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-28.html).

## Bugs Fixed


* [PXC-3923](https://jira.percona.com/browse/PXC-3923): When the `read_only` or `super_read_only` option was set, the `ANALYZE TABLE` command removed the node from the cluster.


* [PXC-3388](https://jira.percona.com/browse/PXC-3388): Percona XtraDB Cluster stuck in a DESYNCED state after joiner was killed.


* [PXC-3609](https://jira.percona.com/browse/PXC-3609): The binary log status variables were updated when the binary log was disabled. Now the status variables are not registered when the binary log is disabled. (Thanks to Stofa Kenida for reporting this issue.)


* [PXC-3848](https://jira.percona.com/browse/PXC-3848): The cluster node exited when the `CURRENT_USER()` function was used. (Thanks to Steffen Böhme for reporting this issue.)


* [PXC-3872](https://jira.percona.com/browse/PXC-3872): A user without system_user privilege was able to drop system users. (Thanks to user jackc for reporting this issue.)


* [PXC-3918](https://jira.percona.com/browse/PXC-3918): Galera Arbitrator (garbd) could not connect if the Percona XtraDB Cluster server used encrypted connections. The issue persisted even when the proper certificates were specified.


* [PXC-3924](https://jira.percona.com/browse/PXC-3924): Using `TRUNCATE TABLE X` and `INSERT INTO X` options when the foreign keys were disabled and violated caused the `HA_ERR_FOUND_DUPP_KEY` error on a slave node. (Thanks to Daniel Bartoníček for reporting this issue.)


* [PXC-3062](https://jira.percona.com/browse/PXC-3062): The `wsrep_incoming_addresses` status variable did not contain the garbd IP address.

## Useful Links


* The [Percona XtraDB Cluster installation instructions](https://docs.percona.com/percona-xtradb-cluster/8.0/install-index.html)


* The [Percona XtraDB Cluster downloads](https://www.percona.com/downloads/Percona-XtraDB-Cluster-LATEST/#)


* The [Percona XtraDB Cluster GitHub location](https://github.com/percona/percona-xtradb-cluster)


* To contribute to the documentation, review the [Documentation Contribution Guide](https://github.com/percona/pxc-docs/blob/8.0/contributing.md)
