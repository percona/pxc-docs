> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-xtradb-cluster/9.7/llms.txt).

# Percona XtraDB Cluster 9.7 Documentation

This documentation is for the latest release: [Percona XtraDB Cluster 9.7.2-3](https://docs.percona.com/percona-xtradb-cluster/9.7/release-notes/9.7.2-3.md) release notes.

[Percona XtraDB Cluster](https://www.percona.com/software/mysql-database/percona-xtradb-cluster) is a database clustering solution for MySQL. It ensures high availability, prevents downtime and data loss, and provides linear scalability for a growing environment.

### Features of Percona XtraDB Cluster

| Feature                                   | Details                                                                                                                              |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Synchronous replication                   | Data is written to all nodes simultaneously, or not written at all in case of a failure even on a single node                        |
| Multi-source replication                  | Any node can trigger a data update.                                                                                                  |
| True parallel replication                 | Multiple threads on replica performing replication on row level                                                                      |
| Automatic node provisioning               | You simply add a node and it automatically syncs.                                                                                    |
| Data consistency                          | No more unsynchronized nodes.                                                                                                        |
| PXC Strict Mode                           | Avoids the use of tech preview features and unsupported features                                                                     |
| Configuration script for ProxySQL         | Percona XtraDB Cluster includes the `proxysql-admin` tool that automatically configures Percona XtraDB Cluster nodes using ProxySQL. |
| Automatic configuration of SSL encryption | Percona XtraDB Cluster includes the `pxc-encrypt-cluster-traffic` variable that enables automatic configuration of SSL encryption    |
| Optimized Performance                     | Percona XtraDB Cluster performance is optimized to scale with a growing production workload                                          |

Percona XtraDB Cluster 9.7 is fully compatible with MySQL Server Community Edition 9.7 and Percona Server for MySQL 9.7. The cluster has the following compatibilities:

- Data - use the data created by any MySQL variant.
- Application - no changes or minimal application changes are required for an application to use the cluster.

See also

Overview of changes in the most recent PXC release

- [Important changes in Percona XtraDB Cluster 9.7](https://docs.percona.com/percona-xtradb-cluster/9.7/upgrade-guide.md#important-changes-in-percona-xtradb-cluster-97)
- [MySQL Community Edition](https://www.mysql.com/products/community/)
- [Percona Server for MySQL](https://www.percona.com/doc/percona-server/9.7/index.html)
- [How We Made Percona XtraDB Cluster Scale](https://www.percona.com/blog/2017/04/19/how-we-made-percona-xtradb-cluster-scale)

## Quickstart guide

Get started quickly with our Quickstart guide.

[Quickstart guide](https://docs.percona.com/percona-xtradb-cluster/9.7/quickstart-overview.md)

### Installation guides

Find the best installation solution with our step-by-step installation instructions.

[Installation instructions](https://docs.percona.com/percona-xtradb-cluster/9.7/install-index.md)

### High availability

Learn about how Percona XtraDB Cluster delivers high availability.

[High availability](https://docs.percona.com/percona-xtradb-cluster/9.7/high-availability.md)

## State Snapshot Transfer (SST) Method using Clone plugin

Learn about a modern and efficient method that leverages MySQL’s native cloning capabilities to transfer data from a donor node to a Joiner node.

[SST](https://docs.percona.com/percona-xtradb-cluster/9.7/clone-sst.md)
