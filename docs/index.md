# Percona XtraDB Cluster 8.0 Documentation

[Percona XtraDB Cluster](https://www.percona.com/software/mysql-database/percona-xtradb-cluster) is a database clustering solution for MySQL. It ensures high availability, prevents downtime and data loss, and provides linear scalability for a growing environment.

### Features of Percona XtraDB Cluster

| Feature| Details|
| ------ | ------ | 
| Synchronous replication\*\*  | Data is written to all nodes simultaneously, or not written at all in case of a failure even on a single node  |
| Multi-source replication| Any node can trigger a data update. |
| True parallel replication| Multiple threads on replica performing replication on row level |
| Automatic node provisioning | You simply add a node and it automatically syncs.|
| Data consistency| No more unsynchronized nodes. |
| PXC Strict Mode| Avoids the use of tech preview features and unsupported features|
| Configuration script for ProxySQL| Percona XtraDB Cluster includes the `proxysql-admin` tool that automatically configures Percona XtraDB Cluster nodes using ProxySQL. |
| Automatic configuration of SSL encryption| Percona XtraDB Cluster includes the `pxc-encrypt-cluster-traffic` variable that enables automatic configuration of SSL encryption |
| Optimized Performance| Percona XtraDB Cluster performance is optimized to scale with a growing production workload|

Percona XtraDB Cluster 8.0 is fully compatible with MySQL Server Community Edition 8.0 and Percona Server for MySQL 8.0. The cluster has the following compatibilities:

* Data - Use the data created by any MySQL variant.

* Application - No changes or minimal application changes are required for an application to use the cluster.

!!! admonition "See also"

    Overview of changes in the most recent PXC release

     * [Important changes in Percona XtraDB Cluster 8.0](howtos/upgrade_guide.md#upgrade-guide-changed)

     * [MySQL Community Edition](https://www.mysql.com/products/community/)
     
     * [Percona Server for MySQL](https://www.percona.com/doc/percona-server/LATEST/index.html)
    
     * [How We Made Percona XtraDB Cluster Scale](https://www.percona.com/blog/2017/04/19/how-we-made-percona-xtradb-cluster-scale)
   

## Introduction


* [About Percona XtraDB Cluster](intro.md)


* [Percona XtraDB Cluster Limitations](limitation.md)


## Getting Started


* [Quick Start Guide for Percona XtraDB Cluster](overview.md)


* [Installing Percona XtraDB Cluster](install/index.md)


* [Configuring Nodes for Write-Set Replication](configure.md)


* [Bootstrapping the First Node](bootstrap.md)


* [Adding Nodes to Cluster](add-node.md)


* [Verifying Replication](verify.md)


## Features


* [High Availability](features/highavailability.md)


* [PXC Strict Mode](features/pxc-strict-mode.md)


* [Online Schema Upgrade](features/online-schema-upgrade.md)


* [Non-Blocking Operations (NBO) method for Online Scheme Upgrades (OSU)](features/nbo.md)


## PXC Security


* [Security Basics](security/index.md)


* [Securing the Network](security/secure-network.md)


* [Encrypting PXC Traffic](security/encrypt-traffic.md)


* [Enabling AppArmor](security/apparmor.md)


* [Enabling SELinux](security/selinux.md)


## User's Manual


* [State Snapshot Transfer](manual/state_snapshot_transfer.md)


* [Percona XtraBackup SST Configuration](manual/xtrabackup_sst.md)


* [Restarting the cluster nodes](manual/restarting_nodes.md)


* [Cluster Failover](manual/failover.md)


* [Monitoring the cluster](manual/monitoring.md)


* [Certification in Percona XtraDB Cluster](manual/certification.md)


* [Percona XtraDB Cluster threading model](manual/threading_model.md)


* [Understanding GCache and Record-Set Cache](manual/gcache_record-set_cache_difference.md)


* [Perfomance Schema Instrumentation](manual/performance_schema_instrumentation.md)


* [Data at Rest Encryption](management/data_at_rest_encryption.md)


## How-tos


* [Upgrading Percona XtraDB Cluster](howtos/upgrade_guide.md)


* [Crash Recovery](howtos/crash-recovery.md)


* [Configuring Percona XtraDB Cluster on CentOS](howtos/centos_howto.md)


* [Configuring Percona XtraDB Cluster on Ubuntu](howtos/ubuntu_howto.md)


* [Setting up Galera Arbitrator](howtos/garbd_howto.md)


* [How to set up a three-node cluster on a single box](howtos/singlebox.md)


* [How to set up a three-node cluster in EC2 environment](howtos/3nodesec2.md)


* [Load balancing with HAProxy](howtos/haproxy.md)


* [Load balancing with ProxySQL](howtos/proxysql.md)


* [The proxysql-admin Tool with ProxySQL v2](howtos/proxysql-v2.md)


* [Setting up a testing environment with ProxySQL](howtos/virt_sandbox.md)


## Release notes


* [*Percona XtraDB Cluster* 8.0 Release notes](release-notes/release-notes_index.md)


## Reference


* [Index of wsrep status variables](wsrep-status-index.md)


* [Index of wsrep system variables](wsrep-system-index.md)


* [Index of wsrep_provider options](wsrep-provider-index.md)


* [Index of files created by PXC](wsrep-files-index.md)


* [Frequently Asked Questions](faq.md)


* [Glossary](glossary.md)

