# *Percona XtraDB Cluster* 8.0.19-10


* **Date**

    June 18, 2020



* **Installation**

    [Installing Percona XtraDB Cluster](https://www.percona.com/doc/percona-xtradb-cluster/8.0/install/index.html)


Percona XtraDB Cluster 8.0.19-10 includes all of the features and bug fixes available in Percona Server for MySQL. See the corresponding [release notes for Percona Server for MySQL 8.0.19-10](https://www.percona.com/doc/percona-server/LATEST/release-notes/Percona-Server-8.0.19-10.html) for more details on these changes.

## Improvements


* [PXC-2189](https://jira.percona.com/browse/PXC-2189): Modify Reference Architecture for Percona XtraDB Cluster (PXC) to include ProxySQL


* [PXC-3182](https://jira.percona.com/browse/PXC-3182): Modify processing to not allow writes on 8.0 nodes while 5.7 nodes are still on the cluster


* [PXC-3187](https://jira.percona.com/browse/PXC-3187): Add dependency package installation note in PXC binary tarball installation doc.


* [PXC-3138](https://jira.percona.com/browse/PXC-3138): Document mixed cluster write (PXC8 while PXC5.7 nodes are still part of the cluster) should not be completed.


* [PXC-3066](https://jira.percona.com/browse/PXC-3066): Document that pxc-encrypt-cluster-traffic=OFF is not just about traffic encryption


* [PXC-2993](https://jira.percona.com/browse/PXC-2993): Document the dangers of running with strict mode disabled and Group Replication at the same time


* [PXC-2980](https://jira.percona.com/browse/PXC-2980): Modify Documentation to include AutoStart Up Process after Installation


* [PXC-2604](https://jira.percona.com/browse/PXC-2604): Modify garbd processing to support Operator

## Bugs Fixed


* [PXC-3298](https://jira.percona.com/browse/PXC-3298): Correct galera_var_reject_queries test to remove display value width


* [PXC-3320](https://jira.percona.com/browse/PXC-3320): Correction on PXC installation doc


* [PXC-3270](https://jira.percona.com/browse/PXC-3270): Modify wsrep_ignore_apply_errors variable default to restore 5.x behavior


* [PXC-3179](https://jira.percona.com/browse/PXC-3179): Correct replication of CREATE USER … RANDOM PASSWORD


* [PXC-3080](https://jira.percona.com/browse/PXC-3080): Modify to process the ROTATE_LOG_EVENT synchronously to perform proper cleanup


* [PXC-2935](https://jira.percona.com/browse/PXC-2935): Remove incorrect assertion when –thread_handling=pool-of-threads is used


* [PXC-2500](https://jira.percona.com/browse/PXC-2500): Modify ALTER USER processing when executing thread is Galera applier thread to correct assertion


* [PXC-3234](https://jira.percona.com/browse/PXC-3234): Correct documentation link in spec file


* [PXC-3204](https://jira.percona.com/browse/PXC-3204): Modify to set wsrep_protocol_version correctly when wsrep_auto_increment_control is disabled


* [PXC-3189](https://jira.percona.com/browse/PXC-3189): Correct SST processing for super_read_only


* [PXC-3184](https://jira.percona.com/browse/PXC-3184): Modify startup to correct crash when socat not found and SST Fails


* [PXC-3169](https://jira.percona.com/browse/PXC-3169): Modify wsrep_reject_queries to enhance error messaging


* [PXC-3165](https://jira.percona.com/browse/PXC-3165): Allow COM_FIELD_LIST to be executed when WSREP is not ready


* [PXC-3145](https://jira.percona.com/browse/PXC-3145): Modify to end mysqld process when the joiner fails during an SST


* [PXC-3043](https://jira.percona.com/browse/PXC-3043): Update required donor version to PXC 5.7.28 (previously was Known Issue)


* [PXC-3036](https://jira.percona.com/browse/PXC-3036): Document correct method for starting, stopping, bootstrapping


* [PXC-3287](https://jira.percona.com/browse/PXC-3287): Correct link displayed on help client command


* [PXC-3031](https://jira.percona.com/browse/PXC-3031): Modify processing for garbd to prevent issues when multiple requests are started at approximately the same time and request an SST transfers to prevent SST from hanging

## Known Issues


* [PXC-3039](https://jira.percona.com/browse/PXC-3039): No useful error messages if an SSL-disabled node tries to join SSL-enabled cluster


* [PXC-3092](https://jira.percona.com/browse/PXC-3092): Abort startup if keyring is specified but cluster traffic encryption is turned off


* [PXC-3093](https://jira.percona.com/browse/PXC-3093): Garbd logs Completed SST Transfer Incorrectly (Timing is not correct)


* [PXC-3159](https://jira.percona.com/browse/PXC-3159): Killing the Donor or Connection lost during SST Process Leaves Joiner Hanging
