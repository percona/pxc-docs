# *Percona XtraDB Cluster* 8.0.18-9.3


* **Date**

    April 29, 2020



* **Installation**

    [Installing Percona XtraDB Cluster](https://www.percona.com/doc/percona-xtradb-cluster/8.0/install/index.html)


Percona XtraDB Cluster 8.0.18-9.3 includes all of the features and bug fixes available in Percona Server for MySQL. See the corresponding [release notes for Percona Server for MySQL 8.0.18-9](https://www.percona.com/doc/percona-server/LATEST/release-notes/Percona-Server-8.0.18-9.html) for more details on these changes.

## Improvements


* [PXC-2495](https://jira.percona.com/browse/PXC-2495): Modified documentation for wsrep_sst_donor to include results when IP address is used


* [PXC-3002](https://jira.percona.com/browse/PXC-3002): Enhanced service_startup_timeout options to allow it to be disabled


* [PXC-2331](https://jira.percona.com/browse/PXC-2331): Modified the SST process to run mysql_upgrade


* [PXC-2991](https://jira.percona.com/browse/PXC-2991): Enhanced Strict Mode Processing to handle Group Replication Plugin


* [PXC-2985](https://jira.percona.com/browse/PXC-2985): Enabled Service for Automated Startup on Reboot with valid grastate.dat


* [PXC-2980](https://jira.percona.com/browse/PXC-2980): Modified Documentation to include AutoStart Up Process after Installation


* [PXC-2722](https://jira.percona.com/browse/PXC-2722): Enabled Support for Percona XtraBackup (PXB) 8.0.8 in Percona XtraDB Cluster (PXC) 8.0


* [PXC-2602](https://jira.percona.com/browse/PXC-2602): Added Ability to Configure xbstream options with wsrep_sst_xtrabackup


* [PXC-2455](https://jira.percona.com/browse/PXC-2455): Implemented the use of Percona XtraBackup (PXB) 8.0.5 in Percona XtraDB Cluster (PXC) 8.0


* [PXC-2259](https://jira.percona.com/browse/PXC-2259): Updated wsrep-files-index.htrml to include new files created by Percona XtraDB Cluster (PXC)


* [PXC-2197](https://jira.percona.com/browse/PXC-2197): Modified SST Documentation to Include Package Dependencies for Percona XtraBackup (PXB)


* [PXC-2194](https://jira.percona.com/browse/PXC-2194): Improvements to the PXC upgrade guide


* [PXC-2191](https://jira.percona.com/browse/PXC-2191): Revised Documentation on innodb_deadlock to Clarify Cluster Level Deadlock Processing


* [PXC-3017](https://jira.percona.com/browse/PXC-3017): Remove these SST encryption methods. encrypt=1, encrypt=2, and encrypt=3


* [PXC-2189](https://jira.percona.com/browse/PXC-2189): Modified Reference Architecture for Percona XtraDB Cluster (PXC) to include ProxySQL

## Bugs Fixed


* [PXC-2537](https://jira.percona.com/browse/PXC-2537): Modified mysqladmin password command to prevent node crash


* [PXC-2958](https://jira.percona.com/browse/PXC-2958): Modified User Documentation to include wsrep_certification_rules and cert.optimistic_pa


* [PXC-2045](https://jira.percona.com/browse/PXC-2045): Removed debian.cnf reference from logrotate/logcheck configuration Installed on Xenial/Stretch


* [PXC-2292](https://jira.percona.com/browse/PXC-2292): Modified Processing to determine Type of Key Cert when IST/SST


* [PXC-2974](https://jira.percona.com/browse/PXC-2974): Modified Percona XtraDB Cluster (PXC) Dockerfile to Integrate Galera wsrep recovery Process


* [PXC-3145](https://jira.percona.com/browse/PXC-3145): When the joiner fails during an SST, the mysqld process stays around (doesn’t exit)


* [PXC-3128](https://jira.percona.com/browse/PXC-3128): Removed Prior Commit to Allow High Priority High Transaction Processing


* [PXC-3076](https://jira.percona.com/browse/PXC-3076): Modified Galera build to remove python3 components


* [PXC-2912](https://jira.percona.com/browse/PXC-2912): Modified netcat Configuration to Include -N Flag on Donor


* [PXC-2476](https://jira.percona.com/browse/PXC-2476): Modified process to determine and process IST or SST and with keyring_file processing


* [PXC-2204](https://jira.percona.com/browse/PXC-2204): Modified Shutdown using systemd after Bootstrap to provide additional messaging


* [PXB-2142](https://jira.percona.com/browse/PXB-2142): Transition key was written to backup / stream


* [PXC-2969](https://jira.percona.com/browse/PXC-2969): Modified pxc_maint_transition_period Documentation to Include Criteria for Use

## Known Issues


* [PXC-2978](https://jira.percona.com/browse/PXC-2978): Certificate Information not Displayed when pxc-encrypt-cluster-traffic=ON


* [PXC-3039](https://jira.percona.com/browse/PXC-3039): No useful error messages if an SSL-disabled node tries to join SSL-enabled cluster


* [PXC-3043](https://jira.percona.com/browse/PXC-3043): Update required donor version to PXC 5.7.28


* [PXC-3063](https://jira.percona.com/browse/PXC-3063): Data at Rest Encryption not Encrypting Record Set Cache


* [PXC-3092](https://jira.percona.com/browse/PXC-3092): Abort startup if keyring is specified but cluster traffic encryption is turned off


* [PXC-3093](https://jira.percona.com/browse/PXC-3093): Garbd logs Completed SST Transfer Incorrectly (Timing is not correct)


* [PXC-3159](https://jira.percona.com/browse/PXC-3159): Killing the Donor or Connection lost during SST Process Leaves Joiner Hanging
