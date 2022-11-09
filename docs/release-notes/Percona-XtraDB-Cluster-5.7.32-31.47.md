# Percona XtraDB Cluster 5.7.32-31.47


* **Date**

    January 12, 2021



* **Installation**

    [Installing Percona XtraDB Cluster](https://www.percona.com/doc/percona-xtradb-cluster/5.7/install/index.html)


## Bugs Fixed


* [PXC-3468](https://jira.percona.com/browse/PXC-3468): Resolve package conflict when installing PXC 5.7 on RHEL/CentOS8


* [PXC-3459](https://jira.percona.com/browse/PXC-3459): Modify to pass correct data from row_ins_foreign_check_on_constraint() to wsrep_append_foreign_key()


* [PXC-3418](https://jira.percona.com/browse/PXC-3418): Prevent DDL-DML deadlock by making in-place ALTER take shared MDL for the whole duration.


* [PXC-2264](https://jira.percona.com/browse/PXC-2264): Update Data at Rest Encryption documentation on upgrade and compatibility issues to explain incompatibility when the donor is <= 5.7.21 and joiner is >= 5.7.22


* [PXC-3501](https://jira.percona.com/browse/PXC-3501): Modify wsrep_row_upd_check_foreign_constraints() to include foreign key dependencies in the writesets for DELETE query (Thanks to user Steven Gales for reporting this issue)


* [PXC-3442](https://jira.percona.com/browse/PXC-3442): Fix crash when log_slave_updates=ON and consistency check statement is executed


* [PXC-3424](https://jira.percona.com/browse/PXC-3424): Fix error handling when the donor is not able to serve SST