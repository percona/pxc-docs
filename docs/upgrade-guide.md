# Upgrade Percona XtraDB Cluster

!!! warning "Required: Back up before upgrading"
    Create backups before attempting any upgrade.

This guide describes how to upgrade Percona XtraDB Cluster (PXC) to version 9.7 without downtime. A rolling upgrade updates one node at a time while the cluster remains operational. Rolling upgrades to 9.7 require version 8.4 or later. Upgrade to the latest 8.4 release before upgrading to 9.7.

!!! warning

    A node with a higher protocol version cannot join a cluster running an older Galera Communication System (GCS) protocol version. The cluster enforces this rule to prevent data corruption and incompatibilities.

    Run the following command on a cluster member and on the joining node, then compare the outputs:

    ```sql
    SHOW STATUS LIKE 'wsrep_protocol_version';
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        +------------------------+-------+
        | Variable_name          | Value |
        +------------------------+-------+
        | wsrep_protocol_version | 11    |
        +------------------------+-------+
        ```

Upgrading to Percona Server 9.7 follows the same process as minor 8.4 upgrades. Review the [Percona Server for MySQL 9.7 Upgrade Guide :octicons-link-external-16:](https://docs.percona.com/percona-server/{{vers}}/upgrade.html) for additional details.

--8<--- "get-help-snip.md"

## Changes in Percona XtraDB Cluster 9.7

### Clone plugin enhancements

The Clone plugin supports cloning between consecutive Long-Term Support (LTS) versions. This enhancement affects State Snapshot Transfer (SST) operations that use the clone-based SST method.

### Authentication enhancements

PXC 9.7 supports the PBKDF2 storage format with `caching_sha2_password`. This format uses PBKDF2 with SHA512 and enables migration from existing formats without client-side modifications. The default authentication plugin remains `caching_sha2_password`. ProxySQL 2.6.2 or later supports the `caching_sha2_password` authentication method.

### Replication variable

The `replica_allow_higher_version_source` variable controls replication from a higher-version source to a lower-version replica.

### Keyring component

Installations that use the keyring plugin require configuration updates. Replace the keyring plugin settings in `my.cnf` with keyring component settings. Update the manifest and component configuration file. For installation steps, see [Keyring component installation :octicons-link-external-16:](https://dev.mysql.com/doc/refman/9.7/en/keyring-component-installation.html).

During a rolling upgrade, the donor node running 8.4 can use the keyring plugin. The 9.7 node uses the keyring component. Both methods use the keyring for encryption and remain compatible.

### ProxySQL compatibility

ProxySQL versions before 2.6.2 do not support the [--syncusers](proxysql-v2.md#proxysql-admin-utilities) option with `caching_sha2_password` users. Use the `mysql_native_password` authentication plugin for these versions. Load this authentication plugin manually.

## Default security and compatibility settings

[PXC Strict Mode](strict-mode.md#percona-xtradb-cluster-strict-mode) is enabled by default. This mode denies unsupported operations and may halt the server. For more information, see [PXC Strict Mode](strict-mode.md).

The `pxc-encrypt-cluster-traffic` variable is enabled by default. Configure each node accordingly. Do not join a cluster with unencrypted cluster traffic. For more information, see [Traffic encryption](encrypt-traffic.md#encrypt-pxc-traffic).

## Avoid mixing PXC 8.4 nodes with PXC 9.7 nodes

Percona Server for MySQL versions 8.4 and 9.7 both use Galera 4. No Galera protocol layer issues prevent any node from being a writer in a mixed-version cluster. However, 9.7 includes changes that may not work on 8.4. Replicating these changes from 9.7 to 8.4 can cause node inconsistencies. The cluster may evict the node through the inconsistency voting protocol.

MySQL 9.7 includes changes that may affect mixed-version clusters:

* Atomic DDL improvements: Changes to Generated Invisible Primary Keys (GIPK) behavior and Primary Key Equivalent handling may produce different results on 8.4 nodes

* Optimizer changes: The Hypergraph Optimizer can be enabled per session and may behave differently when replicated to 8.4 nodes

Data Definition Language (DDL) statements replicate as Total Order Isolation (TOI) operations. These statements execute on all nodes. If a statement succeeds on 9.7 but fails on 8.4, the cluster detects the inconsistency and evicts the 8.4 node.

In a mixed-version cluster, use the lower-version node as the writer. Verify that DDL statements behave the same way on all nodes.

## Major upgrade scenarios

Verify your version before upgrading PXC to version 9.7. Direct upgrades from versions older than 8.4 are not supported.

The following upgrade paths are available:

* Version 8.4 clusters can upgrade directly to 9.7

* Clusters older than 8.4 must first upgrade to the latest 8.4 release

The upgrade steps vary based on cluster configuration and workload.

### In-place rolling upgrade

An in-place rolling upgrade updates an N-node PXC cluster with minimal downtime. This process updates nodes one at a time. During the upgrade, direct application writes only to lower-version nodes. Any node can handle reads.

Before upgrading, verify that your application can function with a reduced cluster size. A cluster with an even number of nodes can experience split-brain scenarios. For more information, see [Cluster failover](failover.md).

The upgrade process detects the 8.4 data directory and initiates the upgrade during node startup. The data directory transforms to be compatible with PXC 9.7. The node joins the cluster and enters a synced state. The result is an N-node cluster with N-1 nodes on 8.4 and one node on 9.7.

To upgrade a node from PXC 8.4 to 9.7, complete the following steps:

1. Stop one of the nodes running PXC 8.4.

2. Uninstall the PXC 8.4 packages. Do not delete the data directory.

3. Install the PXC 9.7 packages.

4. Start the `mysqld` service.

## Add a 9.7 node to an 8.4 cluster

You can upgrade a cluster by starting a fresh 9.7 node and joining the existing 8.4 cluster.

This scenario uses an active three-node 8.4 cluster. Complete the following steps:

1. Join the 9.7 node to the cluster. The node receives an SST and joins the cluster. The cluster has four nodes: three running 8.4 and one running 9.7.

2. Stop one of the 8.4 nodes. Repeat the procedure to replace each 8.4 node with 9.7.

3. After all nodes run 9.7, the upgrade is complete.

!!! warning

    * You cannot join an 8.4 node to a PXC 9.7 cluster.

    * You cannot join a 9.7 node to clusters older than 8.4.

If you run a version older than 8.4, upgrade all nodes to the latest 8.4 release first, then upgrade to 9.7.

## Upgrade an async replication replica node

If a PXC node is an async replica of another server, complete the following steps:

1. Stop async replication.

2. Upgrade the PXC node using any described method.

3. Start async replication.

4. Verify that async replication works.

## Minor upgrade

A minor upgrade moves PXC 9.7 to a higher version within the 9.7 series (for example, from 9.7.0 to 9.7.1). These upgrades include bug fixes and minor improvements without major functionality changes.

To upgrade the cluster, complete the following steps for each node:

1. Verify that all nodes are synchronized.

2. Disable fast shutdown to ensure that all data files are prepared before the upgrade:

    ```shell
    mysql -u root -p -e "SET GLOBAL innodb_fast_shutdown=0;"
    ```

3. Stop the `mysql` service:

    ```shell
    sudo service mysql stop
    ```

4. Back up `grastate.dat` to enable restoration if the file is corrupted or zeroed out due to network issues.

5. Upgrade the PXC packages. For more information, see [Install Percona XtraDB Cluster](install-index.md).

6. With the server stopped, run the following command to fetch the UUID and sequence number. Store these values to restore the `grastate.dat` file if corruption or network issues occur:

    ```shell
    mysqld --wsrep_recover --user=mysql --log-error=/tmp/wsrep-recover.log
    ```

7. Start the PXC node with the upgraded packages.

    Starting the `mysql` service runs the node with your previous configuration. For more information, see [Add nodes to a cluster](add-node.md#add-nodes-to-cluster).

    ```shell
    sudo service mysql start
    ```

    On Red Hat Enterprise Linux, the `/etc/my.cnf` configuration file is renamed to `my.cnf.rpmsave`. Rename this file back before joining the upgraded node to the cluster.

    The node upgrades the data directory through one of the following methods:

    * During the node startup process

    * Through a state transfer (Incremental State Transfer or SST) from another node

    The cluster handles the upgrade process. Start the node with the upgraded packages, and PXC manages the data directory upgrade.

8. Repeat this procedure for the next node in the cluster until you upgrade all nodes.

## Downgrade

MySQL-compatible database servers version 8.0.34 and later allow downgrades within the same LTS version scope. Downgrading a node is not recommended because of the following risks:

* Data corruption if the older version reads data written in a format it does not recognize

* Node fails to start or join the cluster

* Node eviction due to inconsistencies with other cluster members

* Replication failures between nodes running different versions

* Schema or metadata incompatibilities that cause errors

If you have used features specific to the newer version, you cannot downgrade. The data format becomes incompatible with the older version. To recover, restore a backup taken before using version-specific features. This restore results in data loss for changes made after that backup.

To downgrade a cluster, use the backup and restore procedure described in [Restore 8.4 backup to 9.7 cluster](upgrade-backup.md). This procedure works in reverse for downgrades within the same LTS version. Verify that no version-specific modifications exist before attempting a downgrade.

Downgrades carry inherent risks. Approach downgrades with caution and thorough planning.
