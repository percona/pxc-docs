# Percona XtraDB Cluster 8.0.20-11


* **Date**

    October 1, 2020



* **Installation**

    [Installing Percona XtraDB Cluster](https://www.percona.com/doc/percona-xtradb-cluster/8.0/install/index.html)


Percona XtraDB Cluster 8.0.20-11 includes all of the features and bug fixes available in Percona Server for MySQL. See the corresponding [release notes for Percona Server for MySQL 8.0.20-11](https://www.percona.com/doc/percona-server/LATEST/release-notes/Percona-Server-8.0.20-11.html) for more details on these changes.

## Improvements


* [PXC-2603](https://jira.percona.com/browse/PXC-2603): Update Index for PXC status variables - apply consistent definitions

## Bugs Fixed


* [PXC-3159](https://jira.percona.com/browse/PXC-3159): Modify error handling to close the communication channels and abort the joiner node when donor crashes (previously was Known Issue)


* [PXC-3352](https://jira.percona.com/browse/PXC-3352): Modify wsrep_row_upd_check_foreign_constraints() to remove the check for DELETE


* [PXC-3371](https://jira.percona.com/browse/PXC-3371): Fix Directory creation in build-binary.sh


* [PXC-3370](https://jira.percona.com/browse/PXC-3370): Provide binary tarball with shared libs and glibc suffix & minimal tarballs


* [PXC-3360](https://jira.percona.com/browse/PXC-3360): Update sysbench commands in PXC-ProxySQL configuration doc page


* [PXC-3312](https://jira.percona.com/browse/PXC-3312): Prevent cleanup of statement diagnostic area in case of transaction replay.


* [PXC-3167](https://jira.percona.com/browse/PXC-3167): Correct GCache buffer repossession processing


* [PXC-3347](https://jira.percona.com/browse/PXC-3347): Modify PERCONA_SERVER_EXTENSION for bintarball and modify MYSQL_SERVER_SUFFIX

## Known Issues (unfixed problems that you should be aware of)


* [PXC-3039](https://jira.percona.com/browse/PXC-3039): No useful error messages if an SSL-disabled node tries to join SSL-enabled cluster


* [PXC-3092](https://jira.percona.com/browse/PXC-3092): Log warning at startup if keyring is specified but cluster traffic encryption is turned off


* [PXC-3093](https://jira.percona.com/browse/PXC-3093): Garbd logs Completed SST Transfer Incorrectly (Timing is not correct)
