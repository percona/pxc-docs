# Percona XtraDB Cluster 5.7.36-31.55


* **Date**

    February 16, 2022



* **Installation**

    [Installing Percona XtraDB Cluster](https://www.percona.com/doc/percona-xtradb-cluster/5.7/install/index.html)


Percona XtraDB Cluster (PXC) supports critical business applications in your public, private, or hybrid cloud environment. Our free, open source, enterprise-grade solution includes the high availability and security features your business requires to meet your customer expectations and business goals.

## Release Highlights

The following list contains some of the bug fixes for MySQL 5.7.36, provided by Oracle, and included in Percona XtraDB Cluster for MySQL:


* Fix for the possibility of a deadlock or failure when an undo log truncate operation is initiated after an upgrade from *MySQL* 5.6 to *MySQL* 5.7.


* Fix for when a parent table initiates a cascading `SET NULL` operation on the child table, the virtual column can be set to NULL instead of the value derived from the parent table.


* On a view, the query digest for each SELECT statement is now based on the SELECT statement and not the view definition, which was the case for earlier versions.

Find the complete list of bug fixes and changes in [MySQL 5.7.36 Release Notes](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/news-5-7-36.html).

## Bugs Fixed


* [PXC-3838](https://jira.percona.com/browse/PXC-3838): Documentation - remove extra space in the Bootstrapping the first node document.


* [PXC-3739](https://jira.percona.com/browse/PXC-3739): Fix for FLUSH TABLES … FOR EXPORT staying locked after a session ends.


* [PXC-3766](https://jira.percona.com/browse/PXC-3766): Fix the behavior when SST always runs a version-check procedure that causes unallowed external network communication.
