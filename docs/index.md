# Percona XtraDB Cluster 5.7 Documentation

Percona XtraDB Cluster is a database clustering solution for MySQL.
It ensures high availability, prevents downtime and data loss,
and provides linear scalability for a growing environment.

Features of Percona XtraDB Cluster include:

**Synchronous replication &#60;Database replication&#62;**

Data is written to all nodes simultaneously, or not written at all if it fails even on a single node.

**Multi-source replication**

Any node can trigger a data update.

**True parallel replication &#60;Database replication&#62;**

Multiple threads on slave performing replication on row level.

**Automatic node provisioning&#42;&#42;**

You simply add a node and it automatically syncs.

**Data consistency**

Percona XtraDB Cluster ensures that data is automatically synchronized on all `nodes <Node>` in your cluster.

[PXC Strict Mode](features/pxc-strict-mode.md#pxc-strict-mode)

Avoids the use of experimental and unsupported features.

**Configuration script for ProxySQL**

Percona provides a ProxySQL package with the `proxysql-admin`
tool that automatically configures Percona XtraDB Cluster nodes.

!!! admonition "See also"

    [Load balancing with ProxySQL](howtos/proxysql.md#load-balancing-with-proxysql) 

**Automatic configuration of SSL encryption**

Percona XtraDB Cluster includes the `pxc-encrypt-cluster-traffic` variable that enables automatic configuration of SSL encryption.

**Optimized Performance**

Percona XtraDB Cluster performance is optimized to scale with a growing production workload.

For more information, see the following blog posts:

 * [How We Made Percona XtraDB Cluster Scale](https://www.percona.com/blog/2017/04/19/how-we-made-percona-xtradb-cluster-scale/)

 * [Performance improvements in Percona XtraDB Cluster 5.7.17-29.20](https://www.percona.com/blog/2017/04/19/performance-improvements-percona-xtradb-cluster-5-7-17/)

Percona XtraDB Cluster is fully compatible with [MySQL Server Community Edition](https://www.percona.com/doc/percona-xtradb-cluster/8.0/index.html), [Percona Server](https://www.percona.com/software/mysql-database/percona-server), and [MariaDB](https://www.mariadb.com/). It provides a robust application compatibility: there is no or minimal application changes required.

## Introduction


* [About Percona XtraDB Cluster](intro.md)


* [Percona XtraDB Cluster Limitations](limitation.md)


## Getting Started


* [Overview](overview.md)


* [Installing Percona XtraDB Cluster](install/index.md)


* [Configuring Nodes for Write-Set Replication](configure.md)


* [Bootstrapping the First Node](bootstrap.md)


* [Adding Nodes to Cluster](add-node.md)


* [Verifying Replication](verify.md)


## Features


* [High Availability](features/highavailability.md)


* [Multi-Source Replication](features/multimaster-replication.md)


* [PXC Strict Mode](features/pxc-strict-mode.md)


## PXC Security


* [Security Basics](security/index.md)


* [Securing the Network](security/secure-network.md)


* [Encrypting PXC Traffic](security/encrypt-traffic.md)


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


## Flexibility


* [Binlogging and replication improvements](flexibility/binlogging_replication_improvements.md)


* [InnoDB Full-Text Search improvements](flexibility/innodb_fts_improvements.md)


* [Multiple page asynchronous I/O requests](performance/aio_page_requests.md)


## Diagnostics


* [*InnoDB* Page Fragmentation Counters](diagnostics/innodb_fragmentation_count.md)


* [Using libcoredumper](diagnostics/libcoredumper.md)


* [Stack Trace](diagnostics/stacktrace.md)


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


* [Setting up PXC reference architecture with HAProxy](howtos/virt_sandbox.md)


## Reference


* [*Percona XtraDB Cluster* 5.7 Release notes](release-notes/release-notes_index.md)


* [Index of wsrep status variables](wsrep-status-index.md)


* [Index of wsrep system variables](wsrep-system-index.md)


* [Index of wsrep_provider options](wsrep-provider-index.md)


* [Index of files created by PXC](wsrep-files-index.md)


* [Frequently Asked Questions](faq.md)


* [Glossary](glossary.md)

