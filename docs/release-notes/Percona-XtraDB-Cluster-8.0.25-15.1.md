# Percona XtraDB Cluster 8.0.25-15.1


* **Date**

    November 22, 2021



* **Installation**

    [Installing Percona XtraDB Cluster](https://www.percona.com/doc/percona-xtradb-cluster/8.0/install/index.html).


Percona XtraDB Cluster 8.0.25-15.1 includes all of the features and bug fixes available in Percona Server for MySQL. See the corresponding [release notes for Percona Server for MySQL 8.0.25-15](https://www.percona.com/doc/percona-server/LATEST/release-notes/Percona-Server-8.0.25-15.html) for more details on these changes.

Percona XtraDB Cluster (PXC) supports critical business applications in your public, private, or hybrid cloud environment. Our free, open source, enterprise-grade solution includes the high availability and security features your business requires to meet your customer expectations and business goals.

## Release Highlights

A Non-Blocking Operation method for online schema changes in Percona XtraDB Cluster. This mode is similar to the Total Order Isolation (TOI) mode, whereas a data definition language (DDL) statement (for example, `ALTER`) is executed on all nodes in sync. The difference is that in the NBO mode, the DDL statement acquires a metadata lock that locks the table or schema at a late stage of the operation, which is a more efficient locking strategy.

Note that the NBO mode is a **Tech Preview** feature. We do not recommend that you use this mode in a production environment. For more information, see [Non-Blocking Operations (NBO) method for Online Scheme Upgrades (OSU)](/nbo.md#nbo).

The notable changes and bug fixes introduced by Oracle MySQL include the following:


* The `sql_slave_skip_counter` variable only counts the events in the uncompressed transaction payloads.


* A possible deadlock occurred when system variables, read by different clients, were being updated and the binary log file was rotated.


* Sometimes the aggregate function results could return values from a previous statement when using a prepared `SELECT` statement with a `WHERE` clause that is always false.

For more information, see the [MySQL 8.0.24 Release Notes](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-24.html) and the [MySQL 8.0.25 Release Notes](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-25.html).

## New Features


* [PXC-3265](https://jira.percona.com/browse/PXC-3265) Implements the Non-Blocking Operations (NBO) mode for an Online schema upgrade.

## Bugs Fixed


* [PXC-3275](https://jira.percona.com/browse/PXC-3275): Fix the documented APT package list to match the packages listed in the Repo. (Thanks to user Hubertus Krogmann for reporting this issue)


* [PXC-3387](https://jira.percona.com/browse/PXC-3387): Performing an intermediate commit does not call wsrep commit hooks.


* [PXC-3449](https://jira.percona.com/browse/PXC-3449): Fix for missing dependencies which were carried out in replication writesets caused Galera to fail.


* [PXC-3589](https://jira.percona.com/browse/PXC-3589): Documentation: Updates in [Percona XtraDB Cluster Limitations](../limitation.md#limitations) that the `LOCK=NONE` clause is no longer allowed in an INPLACE ALTER TABLE statement. (Thanks to user Brendan Byrd for reporting this issue)


* [PXC-3611](https://jira.percona.com/browse/PXC-3611): Fix that deletes any keyring.backup file if it exists for SST operation.


* [PXC-3608](https://jira.percona.com/browse/PXC-3608): Fix a concurrency issue that caused a server exit when attempting to read a foreign key.


* [PXC-3637](https://jira.percona.com/browse/PXC-3637): Changes the service start sequence to allow more time for mounting local or remote directories with large amounts of data. (Thanks to user Eric Gonyea for reporting this issue)


* [PXC-3679](https://jira.percona.com/browse/PXC-3679): Fix for SST failures after the update of socat to ‘1.7.4.0’.


* [PXC-3706](https://jira.percona.com/browse/PXC-3706): Fix adds a wait to `wsrep_after_commit` until the first thread in a group commit queue is available.


* [PXC-3729](https://jira.percona.com/browse/PXC-3729): Fix for conflicts when multiple applier threads execute certified transactions and are in High-Priority transaction mode.


* [PXC-3731](https://jira.percona.com/browse/PXC-3731): Fix for incorrect writes to the binary log when `sql_log_bin=0`.


* [PXC-3733](https://jira.percona.com/browse/PXC-3733): Fix to clean the WSREP transaction state if a transaction is requested to be re-prepared.
