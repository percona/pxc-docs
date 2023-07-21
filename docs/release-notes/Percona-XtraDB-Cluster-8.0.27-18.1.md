# Percona XtraDB Cluster 8.0.27-18.1

Date: April 11, 2022

Percona XtraDB Cluster (PXC) supports critical business applications in your public, private, or hybrid cloud environment. Our free, open source, enterprise-grade solution includes the high availability and security features your business requires to meet your customer expectations and business goals.

## Release Highlights

The following lists a number of the bug fixes for *MySQL* 8.0.27, provided by Oracle, and included in Percona Server for MySQL:


* The `default_authentication_plugin` is deprecated. Support for this plugin may be removed in future versions. Use the `authentication_policy` variable.


* The `binary` operator is deprecated. Support for this operator may be removed in future versions. Use `CAST(... AS BINARY)`.


* Fix for when a parent table initiates a cascading `SET NULL` operation on the child table, the virtual column can be set to NULL instead of the value derived from the parent table.

Find the full list of bug fixes and changes in the [MySQL 8.0.27 Release Notes](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-27.html).

## Bugs Fixed


* [PXC-3831](https://jira.percona.com/browse/PXC-3831): Allowed certified high-priority transactions to proceed without lock conflicts.


* [PXC-3766](https://jira.percona.com/browse/PXC-3766): Stopped every XtraBackup-based SST operation from executing the version-check procedure.


* [PXC-3704](https://jira.percona.com/browse/PXC-3704): Based the maximum writeset size on `repl.max_ws_size` when both `repl.max_ws_size` and `wsrep_max_ws_size` values are passed during startup.

## Useful Links


* The [Percona XtraDB Cluster installation instructions](https://www.percona.com/doc/percona-xtradb-cluster/8.0/install/index.html)


* The [Percona XtraDB Cluster downloads](https://www.percona.com/downloads/Percona-XtraDB-Cluster-LATEST/#)


* The [Percona XtraDB Cluster GitHub location](https://github.com/percona/percona-xtradb-cluster)


* To contribute to the documentation, review the [Documentation Contribution Guide](https://github.com/percona/pxc-docs/blob/8.0/contributing.md)
