# Upgrade Percona XtraDB Cluster

___You must make backups before attempting an upgrade.___

This guide explains how to upgrade a Percona XtraDB Cluster to version 8.4 without causing downtime. This process is called a “rolling upgrade,” which means you can upgrade the cluster one node at a time without shutting down the whole cluster. Keep in mind that rolling upgrades to 8.4 are only supported if your current version is 8.0 or newer. Be sure you are running on the latest 8.0 version before you upgrade to 8.4.

!!! warning

    A node with a newer (higher) protocol version cannot join a cluster running an older (lower) Galera Communication System (GCS) protocol version. The cluster enforces this rule to prevent data corruption or incompatibility issues that may arise if a node introduces a feature the cluster doesn't understand.

    Run the command on a current cluster member and on the node that is about to join, then compare the two outputs.

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

Upgrading to Percona Server 8.4 is similar to upgrading between minor versions of 8.0, like from 8.0.x to 8.0.y. There are a few specific details to keep in mind for 8.4, but the overall process isn’t very different. We also recommend checking out the Percona Server upgrade documentation for more information: [Percona Server for MySQL 8.4 Upgrade Guide :octicons-link-external-16:](https://docs.percona.com/percona-server/{{vers}}/upgrade.html).

--8<--- "get-help-snip.md"

## Important changes in Percona XtraDB Cluster 8.4

### Keyring Plugin vs. Keyring Component

Starting with version 8.4, Percona XtraDB Cluster (PXC) no longer supports the keyring plugin. Instead, it uses the keyring component.

| Requirement         | Details  |
|---------------------|----------|
| In-place upgrade     | During an in-place upgrade, you need to update your `my.cnf` configuration. Replace the keyring plugin settings with the keyring component configuration. This includes updating the manifest and the component configuration file. [Learn more about installing the keyring component here :octicons-link-external-16:](https://dev.mysql.com/doc/refman/8.4/en/keyring-component-installation.html). |
| Rolling upgrade      | When performing a rolling upgrade, the donor node (running an older version) can still use the keyring plugin, while the 8.4 node uses the keyring component. Since both use the keyring for encryption, they remain compatible and work together seamlessly. |
| Other requirements   | All other requirements, such as ensuring the SST (State Snapshot Transfer) channel is SSL-encrypted when using the keyring plugin or component, remain the same as in version 8.0. |

### Default authentication plugin

In Percona XtraDB Cluster 8.4, the default authentication plugin is
`caching_sha2_password`. In ProxySQL 2.6.2 or later, use the `caching_sha2_password` authentication method.

### ProxySQL 2.6.2 or later

If you are using a version before ProxySQL 2.6.2, the option [–syncusers](proxysql-v2.md#proxysql-admin-utilities) would not work if the Percona XtraDB Cluster user is
created using `caching_sha2_password`. Use the `mysql_native_password`
authentication plugin in these cases. You must manually load this authentication plugin.

## Default security and compatibility settings

[PXC Strict Mode](strict-mode.md#percona-xtradb-cluster-strict-mode) is enabled by default, which may result in denying any
unsupported operations and may halt the server.

`pxc-encrypt-cluster-traffic` is enabled by default. You need to configure
each node accordingly and avoid joining a cluster with unencrypted cluster
traffic. For more information, see
[Traffic encryption is enabled by default](encrypt-traffic.md#encrypt-pxc-traffic).

## Do not mix PXC 8.0 nodes with PXC 8.4 nodes

In Percona Server for MySQL versions 8.0 and 8.4, both use Galera 4, so there are no issues at the Galera protocol layer that would prevent any node from being a writer in a mixed-version cluster. However, 8.4 introduces changes that might not work on 8.0. If you run these changes on 8.4 and replicate them to 8.0, it could cause node inconsistencies, and the node might be evicted from the cluster due to the inconsistency voting protocol.

Percona Server for MySQL 8.4 also introduces several DDL (Data Definition Language) enhancements that are not supported in 8.0. If you try to run these DDL statements on 8.0, you’ll get errors. A key difference is:

* Foreign Keys Referencing Non-Unique or Partial Keys:

    * 8.4: Supports foreign keys referencing non-unique or partial keys.

    * 8.0: Doesn’t allow this and will fail the DDL statement.

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

Here, parent.value is indexed but not unique. In 8.4, the foreign key reference is allowed, but in 8.0, this operation fails.

Since DDL is replicated as a TOI (transactional operation), it gets executed on all nodes. If it succeeds on 8.4 but fails on 8.0, the cluster will detect the inconsistency and evict the 8.0 node.

In a mixed-version cluster, it’s better to use the lower version node as the writer. When executing DDL, make sure it behaves the same way on all nodes.

## Major upgrade scenarios

Before upgrading your Percona XtraDB Cluster (PXC) to version 8.4, check your current version. You must run version 8.0 before upgrading to 8.4 - direct upgrades from older versions don't work.
Here's what you need to know:

* If you run version 8.0, you can upgrade directly to 8.4

* If you run a version older than 8.0, first upgrade to the latest 8.0 release

The exact upgrade steps depend on your cluster setup and database activity. Your specific configuration and workload will shape the upgrade process, so you'll need to plan accordingly.

### In-place rolling upgrade

If you want to upgrade an N-node Percona XtraDB Cluster (PXC) cluster with minimal downtime, you can use an in-place rolling upgrade. This process updates the nodes one at a time. While upgrading, make sure your application sends writes only to lower-version nodes. You can use any node for reads.

Before upgrading, verify your application can function with a reduced cluster size. When a cluster operates with an even number of nodes, split-brain scenarios become possible.

The upgrade process automatically detects the 8.0 data directory and initiates the upgrade during node boot-up. The data directory transforms to be compatible with PXC 8.4. Afterward, the node joins the cluster and enters a synced state. The result is a N-node cluster with N-1 8.0 and 1 8.4.

Here’s how to upgrade a node from Percona XtraDB Cluster (PXC) 8.0 to 8.4:

1. Shut down an 8.0 node: Pick one of the nodes running PXC 8.0 and stop it.

2. Remove PXC 8.0 packages: Uninstall the PXC 8.0 packages, but make sure you don’t delete the data directory.

3. Install PXC 8.4 packages: Replace the old packages with PXC 8.4 packages.

4. Restart the mysqld service: Start the node again.

## Add 8.4 node to 8.0 cluster

You can upgrade a cluster by booting a fresh 8.4 node and joining it to the existing 8.0 cluster as an additional node instead of performing an in-place rolling upgrade.

In this scenario, you have an active 3-node 8.0 cluster.

1. Join the new 8.4 node to the cluster. It will get a dump of the cluster through SST and stay part of the cluster. You have a four-node cluster: three nodes running 8.0 and one node running 8.4.

2. Shut down one of the 8.0 nodes and repeat the procedure to replace each 8.0 node with 8.4.

3. Once all nodes are running 8.4, the upgrade is complete.

!!! warning

    * You cannot join an 8.0 node to a PXC 8.4 cluster.

    * You cannot join an 8.4 node to clusters older than 8.0.

Therefore, if you are running a version older than 8.0, first upgrade all nodes to the latest 8.0 (using any procedure described), then upgrade to 8.4.

## Upgrade an async replication replica node

If a given PXC node is an async replica of some other server, follow the procedure below:

1. Stop async replication

2. Upgrade PXC node using any described method

3. Start async replication

4. Ensure async replication works

## Minor upgrade

A minor upgrade for Percona XtraDB Cluster 8.4 means upgrading to a newer version within the 8.4 series. For example, moving from version 8.4.0 to 8.4.1. These upgrades include bug fixes and small improvements but don't change major functionality.

To upgrade the cluster, follow these steps for each node:

1. Make sure that all nodes are synchronized.

2. Stop the mysql service:

    ```shell
    sudo service mysql stop
    ```

3. Upgrade Percona XtraDB Cluster packages. For more information, see [Install Percona XtraDB Cluster](install-index.md).

4. Back up `grastate.dat`, so that you can restore it if it is corrupted or zeroed out due to network issues.

5. Start the Percona XtraDB Cluster node with the new packages.

    In most cases, starting the `mysql` service should run the node with your
    previous configuration. For more information, see [Adding Nodes to Cluster](add-node.md#add-nodes-to-cluster).

    ```shell
    sudo service mysql start
    ```

    On Red Hat Enterprise Linux, the `/etc/my.cnf` configuration file is renamed to `my.cnf.rpmsave`. Make sure to rename this file back to the original name before joining the upgraded node back to the cluster.

    The node automatically upgrades its data directory.

    This upgrade happens in one of two ways:

    * During the node startup process

    * Through a state transfer (either IST or SST) from another node

    The cluster handles the upgrade process automatically - you need to start the node with the new packages installed, and PXC manages the data directory upgrade process.

6. Repeat this procedure for the next node in the cluster until you upgrade all nodes.

## Downgrade

Starting from version 8.0.34, MySQL-compatible database servers allow downgrades within the same Long-Term Support (LTS) version scope, provided the server has not applied any new server functionality to the data. While downgrading a node is not recommended, administrators can perform the downgrade using the established procedure.
The key constraint is maintaining data compatibility. New features introduced in a specific point release cannot be retroactively applied to an earlier version. Administrators must carefully verify that no version-specific modifications have been made before attempting a downgrade.

While possible, downgrades carry inherent risks and should be approached with caution and thorough planning.
