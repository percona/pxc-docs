# *Percona XtraDB Cluster* 8.0.23-14.1


* **Date**

    June 9, 2021



* **Installation**

    [Installing Percona XtraDB Cluster](https://www.percona.com/doc/percona-xtradb-cluster/8.0/install/index.html).


Percona XtraDB Cluster 8.0.23-14.1 includes all of the features and bug fixes available in Percona Server for MySQL. See the corresponding [release notes for Percona Server for MySQL 8.0.23-14](https://www.percona.com/doc/percona-server/LATEST/release-notes/Percona-Server-8.0.23-14.html) for more details on these changes.

## Improvements


* [PXC-3092](https://jira.percona.com/browse/PXC-3092): Log a warning at startup if a keyring is specified, but the cluster traffic encryption is turned off

## Bugs Fixed


* [PXC-3464](https://jira.percona.com/browse/PXC-3464): Data is not propagated with SET SESSION sql_log_bin = 0


* [PXC-3146](https://jira.percona.com/browse/PXC-3146): Galera/SST is not looking for the default data directory location for SSL certs


* [PXC-3226](https://jira.percona.com/browse/PXC-3226): Results from CHECK TABLE from PXC server can cause the client libraries to crash


* [PXC-3381](https://jira.percona.com/browse/PXC-3381): Modify GTID functions to use a different char set


* [PXC-3437](https://jira.percona.com/browse/PXC-3437): Node fails to join in the endless loop


* [PXC-3446](https://jira.percona.com/browse/PXC-3446): Memory leak during server shutdown


* [PXC-3538](https://jira.percona.com/browse/PXC-3538): Garbd crashes after successful backup


* [PXC-3580](https://jira.percona.com/browse/PXC-3580): Aggressive network outages on one node makes the whole cluster unusable


* [PXC-3596](https://jira.percona.com/browse/PXC-3596): Node stuck in aborting SST


* [PXC-3645](https://jira.percona.com/browse/PXC-3645): Deadlock during ongoing transaction and RSU
