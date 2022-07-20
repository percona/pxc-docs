# *Percona XtraDB Cluster* 5.7.31-31.45


* **Date**

    September 24, 2020



* **Installation**

    [Installing Percona XtraDB Cluster](https://www.percona.com/doc/percona-xtradb-cluster/5.7/install/index.html)


## Improvements


* [PXC-2187](https://jira.percona.com/browse/PXC-2187): Enhance SST documentation to include a warning about the use of command-line parameters

## Bugs Fixed


* [PXC-3352](https://jira.percona.com/browse/PXC-3352): Modify wsrep_row_upd_check_foreign_constraints() to remove the check for DELETE


* [PXC-3243](https://jira.percona.com/browse/PXC-3243): Modify the BF-abort process to propagate and abort and retry the Stored Procedure instead of the statement


* [PXC-3371](https://jira.percona.com/browse/PXC-3371): Fix Directory creation in build-binary.sh


* [PXC-3370](https://jira.percona.com/browse/PXC-3370): Provide binary tarball with shared libs and glibc suffix & minimal tarballs


* [PXC-3281](https://jira.percona.com/browse/PXC-3281): Modify config to add default socket location
