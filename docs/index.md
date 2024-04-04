# Percona XtraDB Cluster 5.7 Documentation

Percona XtraDB Cluster is a database cluster solution for MySQL.
It ensures high availability, prevents downtime and data loss,
and provides linear scalability for a growing environment.

!!! note ""

    This documentation is for the latest release: Percona XtraDB Cluster {{release}} ([Release Notes](release-notes/{{release}}.md)).

Features of Percona XtraDB Cluster include:

**Synchronous replication &#60;Database replication&#62;**

Data is written to all nodes simultaneously or not written at all if it fails even on a single node.

**Multi-source replication**

Any node can trigger a data update.

**True parallel replication &#60;Database replication&#62;**

Multiple threads on slave performing replication on row level.

**Automatic node provisioning&#42;&#42;**

Add a node and that node automatically syncs.

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

Percona XtraDB Cluster is fully compatible with [MySQL Server Community Edition](https://www.percona.com/doc/percona-xtradb-cluster/8.0/index.html), [Percona Server](https://www.percona.com/software/mysql-database/percona-server).

## {{post}}

Leverage [{{post}}](https://www.percona.com/navigating-mysql-5-7-end-of-life) to ensure continued security and performance for your MySQL applications. While this service provides critical bug fixes and security patches, consider migrating to the next major version to unlock a wider range of benefits:

* Enhanced performance and scalability: Gain access to performance improvements and handle growing data volumes.

* Improved security features: Benefit from the latest security advancements to effectively safeguard your data.

* Access to the latest features and functionality: Use the newest features and functionalities to improve your database management experience.

### Releases

* [Percona XtraDB Cluster 5.7.44-31.65.2](release-notes/5.7.44-31.65.2.md)

### Installation links

* [Install {{post}} releases](install/install-eol.md)

* [Download {{post}} releases from binary tarballs](install/tarball.md)

## For Monitoring and Management

Percona Monitoring and Management (PMM )monitors and provides actionable performance data for MySQL variants, including Percona Server for MySQL, Percona XtraDB Cluster, Oracle MySQL Community Edition, Oracle MySQL Enterprise Edition, and MariaDB. PMM captures metrics and data for the InnoDB, XtraDB, and MyRocks storage engines and has specialized dashboards for specific engine details.

[Install PMM and connect your server instances to it.](https://docs.percona.com/percona-monitoring-and-management/get-started/index.html)
