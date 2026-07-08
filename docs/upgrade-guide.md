# Upgrade Percona XtraDB Cluster

!!! warning "Required: Back up before upgrading"
    Create backups before attempting any upgrade.

This guide describes how to upgrade Percona XtraDB Cluster to version 8.4 without downtime. A rolling upgrade updates one node at a time while the cluster remains operational. Rolling upgrades to 8.4 require version 8.0 or later. Upgrade to the latest 8.0 release before upgrading to 8.4.

!!! warning

    A node with a newer protocol version cannot join a cluster running an older Galera Communication System (GCS) protocol version. The cluster enforces this rule to prevent data corruption and incompatibility issues.

    Run the following command on a cluster member and on the joining node, then compare the outputs.

    ```sql
    SHOW GLOBAL STATUS LIKE 'wsrep_protocol_version';
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        +------------------------+-------+
        | Variable_name          | Value |
        +------------------------+-------+
        | wsrep_protocol_version | 11    |
        +------------------------+-------+
        ```

Upgrading to Percona Server 8.4 follows the same process as minor 8.0 upgrades. Review the [Percona Server for MySQL 8.4 Upgrade Guide :octicons-link-external-16:](https://docs.percona.com/percona-server/{{vers}}/upgrade.html) for additional details.

--8<--- "get-help-snip.md"

## Important changes in Percona XtraDB Cluster 8.4

### Keyring Plugin vs. Keyring Component

Percona XtraDB Cluster (PXC) 8.4 replaces the keyring plugin with the keyring component.

| Requirement         | Details  |
|---------------------|----------|
| In-place upgrade     | During an in-place upgrade, update your `my.cnf` configuration. Replace the keyring plugin settings with the keyring component configuration. This change includes updating the manifest and the component configuration file. [Learn more about installing the keyring component :octicons-link-external-16:](https://dev.mysql.com/doc/refman/8.4/en/keyring-component-installation.html). |
| Rolling upgrade      | During a rolling upgrade, the donor node (running an older version) can use the keyring plugin. The 8.4 node uses the keyring component. Both use the keyring for encryption and remain compatible. |
| Other requirements   | All other requirements remain the same as in version 8.0. For example, ensure the State Snapshot Transfer (SST) channel is SSL-encrypted when using the keyring plugin or component. |

### Default authentication plugin

In Percona XtraDB Cluster 8.4, the default authentication plugin is `caching_sha2_password`. In ProxySQL 2.6.2 or later, use the `caching_sha2_password` authentication method.

### ProxySQL 2.6.2 or later

If you use a version before ProxySQL 2.6.2, the option [--syncusers](proxysql-v2.md#proxysql-admin-utilities) does not work if the Percona XtraDB Cluster user is created using `caching_sha2_password`. Use the `mysql_native_password` authentication plugin in these cases. You must manually load this authentication plugin.

## Default security and compatibility settings

[PXC Strict Mode](strict-mode.md#percona-xtradb-cluster-strict-mode) is enabled by default. This mode denies unsupported operations and may halt the server.

`pxc-encrypt-cluster-traffic` is enabled by default. Configure each node accordingly and avoid joining a cluster with unencrypted cluster traffic. For more information, see [Traffic encryption is enabled by default](encrypt-traffic.md#encrypt-pxc-traffic).

## Do not mix PXC 8.0 nodes with PXC 8.4 nodes

Percona Server for MySQL versions 8.0 and 8.4 both use Galera 4. No Galera protocol layer issues prevent any node from being a writer in a mixed-version cluster. However, 8.4 introduces changes that may not work on 8.0. Running these changes on 8.4 and replicating them to 8.0 can cause node inconsistencies. The cluster may evict the node due to the inconsistency voting protocol.

Percona Server for MySQL 8.4 introduces several Data Definition Language (DDL) enhancements not supported in 8.0. Running these DDL statements on 8.0 produces errors. A key difference is foreign key handling:

* Foreign keys referencing non-unique or partial keys:

    * 8.4: Supports foreign keys referencing non-unique or partial keys

    * 8.0: Does not allow this configuration and fails the DDL statement

The following example shows a foreign key referencing a non-unique index:

```sql
CREATE TABLE parent (
    id INT,
    value INT,
    INDEX (value)
);
CREATE TABLE child (
    id INT,
    parent_value INT,
    FOREIGN KEY (parent_value) REFERENCES parent(value)
);
```

In this example, `parent.value` is indexed but not unique. Version 8.4 allows the foreign key reference. Version 8.0 fails this operation.

DDL statements replicate as Total Order Isolation (TOI) operations and execute on all nodes. If a statement succeeds on 8.4 but fails on 8.0, the cluster detects the inconsistency and evicts the 8.0 node.

In a mixed-version cluster, use the lower version node as the writer. Verify that DDL statements behave the same way on all nodes.

## Major upgrade scenarios

Verify your version before upgrading PXC to version 8.4. Direct upgrades from versions older than 8.0 are not supported.

* Version 8.0 clusters can upgrade directly to 8.4

* Clusters older than 8.0 must first upgrade to the latest 8.0 release

The upgrade steps vary based on cluster configuration and workload.

### In-place rolling upgrade

An in-place rolling upgrade updates an N-node PXC cluster with minimal downtime. This process updates nodes one at a time. During the upgrade, direct application writes only to lower-version nodes. Any node can handle reads.

Before upgrading, verify your application can function with a reduced cluster size. A cluster with an even number of nodes can experience split-brain scenarios.

The upgrade process automatically detects the 8.0 data directory and initiates the upgrade during node startup. The data directory transforms to be compatible with PXC 8.4. The node then joins the cluster and enters a synced state. The result is an N-node cluster with N-1 nodes on 8.0 and one node on 8.4.

To upgrade a node from PXC 8.0 to 8.4:

1. Stop one of the nodes running PXC 8.0.

2. Uninstall the PXC 8.0 packages. Do not delete the data directory.

3. Install the PXC 8.4 packages.

4. Start the `mysqld` service.

## Add 8.4 node to 8.0 cluster

You can upgrade a cluster by starting a fresh 8.4 node and joining the existing 8.0 cluster as an additional node.

This scenario assumes an active three-node 8.0 cluster:

1. Join the 8.4 node to the cluster. The node receives a State Snapshot Transfer (SST) and joins the cluster. The cluster has four nodes: three running 8.0 and one running 8.4.

2. Stop one of the 8.0 nodes. Repeat the procedure to replace each 8.0 node with 8.4.

3. After all nodes run 8.4, the upgrade is complete.

!!! warning

    * You cannot join an 8.0 node to a PXC 8.4 cluster.

    * You cannot join an 8.4 node to clusters older than 8.0.

If you run a version older than 8.0, first upgrade all nodes to the latest 8.0 release, then upgrade to 8.4.

## Upgrade an async replication replica node

If a PXC node is an async replica of another server, follow this procedure:

1. Stop async replication.

2. Upgrade the PXC node using any described method.

3. Start async replication.

4. Verify async replication works.

## Minor upgrade

A minor upgrade moves Percona XtraDB Cluster 8.4 to a newer version within the 8.4 series (for example, from 8.4.0 to 8.4.1). These upgrades include bug fixes and minor improvements without major functionality changes.

To upgrade the cluster, follow these steps for each node:

1. Verify all nodes are synchronized.

2. Disable fast shutdown to ensure all data files are fully prepared before the upgrade:

    ```{.bash data-prompt="$"}
    $ mysql -u root -p -e "SET GLOBAL innodb_fast_shutdown=0;"
    ```

3. Stop the `mysql` service:

    ```{.bash data-prompt="$"}
    $ sudo service mysql stop
    ```

4. Upgrade Percona XtraDB Cluster packages. For more information, see [Install Percona XtraDB Cluster](install-index.md).

5. Back up the `grastate.dat` file. You can restore this file if corruption or network issues zero out the values.

6. With the server stopped, run the following command to fetch the UUID and sequence number. Store these values to restore the `grastate.dat` file if needed:

    ```{.bash data-prompt="$"}
    $ mysqld --wsrep_recover --user=mysql --log-error=/tmp/wsrep-recover.log
    ```

7. Start the Percona XtraDB Cluster node with the new packages.

    Starting the `mysql` service runs the node with your previous configuration. For more information, see [Adding Nodes to Cluster](add-node.md#add-nodes-to-cluster).

    ```{.bash data-prompt="$"}
    $ sudo service mysql start
    ```

    On Red Hat Enterprise Linux, the `/etc/my.cnf` configuration file is renamed to `my.cnf.rpmsave`. Rename this file back to the original name before joining the upgraded node to the cluster.

    The node automatically upgrades the data directory. This upgrade happens in one of two ways:

    * During the node startup process

    * Through a state transfer (Incremental State Transfer (IST) or SST) from another node

    The cluster handles the upgrade process automatically. Start the node with the new packages installed, and PXC manages the data directory upgrade.

8. Repeat this procedure for the next node in the cluster until all nodes are upgraded.

## Downgrade

MySQL-compatible database servers (version 8.0.34 and later) allow downgrades within the same Long-Term Support (LTS) version scope. The server must not have applied any new functionality to the data. Downgrading a node is not recommended, but administrators can perform the downgrade using the established procedure.

The key constraint is data compatibility. New features introduced in a specific point release cannot be retroactively applied to an earlier version. Verify that no version-specific modifications have been made before attempting a downgrade.

Downgrades carry inherent risks. Approach downgrades with caution and thorough planning.
