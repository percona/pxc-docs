# Percona XtraDB Cluster 5.7.37-31.57 (2022-05-18)

Percona XtraDB Cluster (PXC) supports critical business applications in your public, private, or hybrid cloud environment. Our free, open source, enterprise-grade solution includes the high availability and security features your business requires to meet your customer expectations and business goals.

## Release Highlights

The following lists a number of the notable updates and fixes for MySQL 5.7.37, provided by Oracle, and included in Percona Server for MySQL:


* The performance on debug builds has been improved by optimizing the buf_validate() function in the *InnoDB* sources.


* Fix for when a query using an index that differs from the primary key of the partitioned table results in excessive CPU load.


* Enabling `PAD_CHAR_TO_FULL_LENGTH` SQL mode on a replica server added trailing spaces to a replication channel’s name in the replication metadata repository tables. Attempts to identify the channel using the padded name caused errors. The SQL mode is disabled when reading from those tables.

Find the complete list of bug fixes and changes in [MySQL 5.7.37 Release Notes](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-37.html).

## Bugs Fixed


* [PXC-3388](https://jira.percona.com/browse/PXC-3388): When the joiner failed and the donor did not abort the SST. The cluster remained in donor/desynced state.


* [PXC-3609](https://jira.percona.com/browse/PXC-3609): The binary log status was updated when the binary log was disabled.


* [PXC-3796](https://jira.percona.com/browse/PXC-3796): The Garbd IP was not visible in the `wsrep_incoming_addresses` status variable.


* [PXC-3848](https://jira.percona.com/browse/PXC-3848): Issuing an `ALTER USER CURRENT_USER()` command crashed the connected cluster node.


* [PXC-3914](https://jira.percona.com/browse/PXC-3914): Upgraded the version of socat used in the Docker image.

## Useful Links

The [Percona XtraDB Cluster installation instructions](https://www.percona.com/doc/percona-xtradb-cluster/5.7/install/index.html)

The [Percona XtraBackup downloads](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/LATEST/)

The [Percona XtraBackup GitHub location](https://github.com/percona/percona-xtradb-cluster)

To contribute to the documentation, review the [Documentation Contribution Guide](https://github.com/percona/pxc-docs/blob/8.0/contributing.md)
