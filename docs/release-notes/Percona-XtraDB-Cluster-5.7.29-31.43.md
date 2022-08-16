# *Percona XtraDB Cluster* 5.7.29-31.43


* **Date**

    May 8, 2020



* **Installation**

    [Installing Percona XtraDB Cluster](https://www.percona.com/doc/percona-xtradb-cluster/5.7/install/index.html)


## Improvements


* [PXC-3002](https://jira.percona.com/browse/PXC-3002): The PXC mysql-systemd parameter service_startup_timeout terminates any startup process after a configurable time. Added a “Disable” option to the service_startup_timeout for workloads which require more time.


* [PXC-2259](https://jira.percona.com/browse/PXC-2259): Updated the “Index of files created by PXC” document with additional file names and descriptions.


* [PXC-2197](https://jira.percona.com/browse/PXC-2197): Modified the SST Documentation to Include Package Dependencies for Percona XtraBackup (PXB).


* [PXC-2602](https://jira.percona.com/browse/PXC-2602): Added the ability to configure xbstream options with wsrep_sst_xtrabackup.

## Bugs Fixed


* [PXC-2954](https://jira.percona.com/browse/PXC-2954): DDL to add FK on one node fails but completes on other nodes causing inconsistency


* [PXC-2705](https://jira.percona.com/browse/PXC-2705): Executing parallel LOAD DATA queries against the same table and same source data and the number of rows is greater then 10k, the cluster consistency was broken.


* [PXC-2683](https://jira.percona.com/browse/PXC-2683): An issue may occur if a user set innodb_locks_unsafe_for_binlog=1. The variable is deprecated. The processing for the variable was removed.


* [PXC-3202](https://jira.percona.com/browse/PXC-3202): In CentOS 8, the POSTUN scriptlet in an rpm package would expand into an “empty string” and caused an error.


* [PXC-3190](https://jira.percona.com/browse/PXC-3190): Added to the execution of Cleanup functions to remove CREATE USER from SHOW PROCESSLIST.


* [PXC-3076](https://jira.percona.com/browse/PXC-3076): Modified the Galera SConstruct file to remove python3 components.


* [PXC-2969](https://jira.percona.com/browse/PXC-2969): Modified the “Load balancing with Proxy-SQL” documentation to include the Criteria for Use.


* [PXC-2904](https://jira.percona.com/browse/PXC-2904): Ensured the “Percona-XtraDB-Cluster-57” yum package installed the required xtrabackup version.


* [PXC-2958](https://jira.percona.com/browse/PXC-2958): Modified the User Documentation to include wsrep_certification_rules and the cert.optimistic_pa option.


* [PXC-2912](https://jira.percona.com/browse/PXC-2912): Modified the netcat Configuration to Include the -N option, which is required by more recent versions of netcat. The option allows the shutdown of the network socket after the input EOF.


* [PXC-2974](https://jira.percona.com/browse/PXC-2974): Modified Percona XtraDB Cluster (PXC) Dockerfile to Integrate the Galera WSREP recovery process.


* [PS-6979](https://jira.percona.com/browse/PS-6979): Modify the processing to call clean up functions to remove CREATE USER statement from the processlist after the statement has completed (Upstream [#99200](http://bugs.mysql.com/bug.php?id=99200))


* [PXC-2684](https://jira.percona.com/browse/PXC-2684): Modified error handling to prevent deadlock when stored procedure was aborted.
