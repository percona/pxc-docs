> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-xtradb-cluster/9.7/llms.txt).

# Get started with Percona XtraDB Cluster

This guide describes the procedure for setting up Percona XtraDB Cluster.

Examples provided in this guide assume there are three Percona XtraDB Cluster nodes, as a common choice for trying out and testing:

| Node   | Host | IP            |
| ------ | ---- | ------------- |
| Node 1 | pxc1 | 192.168.70.61 |
| Node 2 | pxc2 | 192.168.70.62 |
| Node 3 | pxc3 | 192.168.70.63 |

Note

Avoid creating a cluster with two or any even number of nodes, because this can lead to *split brain*.

The following procedure provides an overview with links to details for every step:

- [Install Percona XtraDB Cluster](https://docs.percona.com/percona-xtradb-cluster/9.7/install-index.html#install-percona-xtradb-cluster) on all nodes and set up root access for them.

It is recommended to install from official Percona repositories:

- On Red Hat and CentOS, [install using YUM](https://docs.percona.com/percona-xtradb-cluster/9.7/yum.html#install-from-percona-software-repository).
- On Debian and Ubuntu, [install using APT](https://docs.percona.com/percona-xtradb-cluster/9.7/apt.html#install-from-repository).
- [Configure all nodes](https://docs.percona.com/percona-xtradb-cluster/9.7/configure-nodes.html#configure-nodes-for-write-set-replication) with relevant settings required for write-set replication.

This includes path to the Galera library, location of other nodes, etc.

- [Bootstrap the first node](https://docs.percona.com/percona-xtradb-cluster/9.7/bootstrap.html#bootstrap-the-first-node) to initialize the cluster.

This must be the node with your main database, which will be used as the data source for the cluster.

- [Add other nodes](https://docs.percona.com/percona-xtradb-cluster/9.7/add-node.html#add-nodes-to-cluster) to the cluster.

Data on new nodes joining the cluster is overwritten in order to synchronize it with the cluster.

- [Verify replication](https://docs.percona.com/percona-xtradb-cluster/9.7/verify-replication.html#verify-replication).

Although cluster initialization and node provisioning is performed automatically, it is a good idea to ensure that changes on one node actually replicate to other nodes.

- [Install ProxySQL](https://docs.percona.com/percona-xtradb-cluster/9.7/load-balance-proxysql.html#load-balance-with-proxysql).

To complete the deployment of the cluster, a high-availability proxy is required. We recommend installing [ProxySQL](https://www.proxysql.com/) on client nodes for efficient workload management across the cluster without any changes to the applications that generate queries.

## Percona Monitoring and Management

[Percona Monitoring and Management](https://www.percona.com/software/database-tools/percona-monitoring-and-management) is the best choice for managing and monitoring Percona XtraDB Cluster performance. It provides visibility for the cluster and enables efficient troubleshooting.
