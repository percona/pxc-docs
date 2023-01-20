# Upgrade Percona XtraDB Cluster

<!-- This guide describes the procedure for upgrading Percona XtraDB Cluster without downtime
(*rolling upgrade*) to the Percona XtraDB Cluster 8.0. -->
<!-- A "rolling upgrade" means there is no need to take down the complete
cluster during the upgrade. -->
The following documents contain details about relevant changes in the 8.0 series
of MySQL and Percona Server for MySQL. Make sure you deal with any incompatible features
and variables mentioned in these documents when upgrading to Percona XtraDB Cluster 8.0.

* [Changed in Percona Server for MySQL 8.0](https://www.percona.com/doc/percona-server/8.0/changed_in_version.html)

* [Upgrading MySQL](http://dev.mysql.com/doc/refman/8.0/en/upgrading.html)

* [Upgrading from MySQL 5.7 to 8.0](https://dev.mysql.com/doc/refman/8.0/en/upgrading-from-previous-series.html)

## Important changes in Percona XtraDB Cluster 8.0

* [Traffic encryption is enabled by default](#traffic-encryption-is-enabled-by-default)
* [Not recommended to mix PXC 5.7 nodes with PXC 8.0 nodes](#not-recommended-to-mix-pxc-57-nodes-with-pxc-80-nodes)
* [PXC Strict Mode is enabled by default](#pxc-strict-mode-is-enabled-by-default)
* [The configuration file layout has changed in PXC 8.0](#the-configuration-file-layout-has-changed-in-pxc-80)
* [caching_sha2_password is the default authentication plugin](#cachingsha2password-is-the-default-authentication-plugin)
* [mysql_upgrade is part of SST](#mysqlupgrade-is-part-of-sstglossarymdsst)

### Traffic encryption is enabled by default

The `pxc_encrypt_cluster_traffic` variable, which enables traffic
encryption, is set to `ON` by default in Percona XtraDB Cluster 8.0.

Unless you configure a node accordingly (each node in your cluster must use the
same SSL certificates) or try to join a cluster running PXC 5.7 which
unencrypted cluster traffic, the node will not be able to join resulting in an
error.

??? example "The error message"

    ```{.text .no-copy}
    ... [ERROR] ... [Galera] handshake with remote endpoint ...
    This error is often caused by SSL issues. ...
    ```

!!! admonition "See also"

    sections [Encrypting PXC Traffic](../security/encrypt-traffic.md#encrypt-traffic), [Configuring Nodes for Write-Set Replication](../configure.md#configure)

<!-- Rolling upgrades to |version| from versions older than 5.7 are not supported
--------------------------------------------------------------------------------

Therefore, if you are running Percona XtraDB Cluster version 5.6, shut down all
nodes, then remove and re-create the cluster from scratch. Alternatively,
you can perform a `rolling upgrade from PXC 5.6 to 5.7 <https://www.percona.com/doc/percona-xtradb-cluster/5.7/howtos/upgrade_guide.html>`_.
and then follow the current procedure to upgrade from 5.7 to 8.0. -->
### Not recommended to mix PXC 5.7 nodes with PXC 8.0 nodes

Shut down the cluster and upgrade each node to PXC 8.0. It is
important that you make backups before attempting an upgrade.

<!-- If downtime is acceptable
Shut down the cluster and upgrade all nodes to PXC
8.0. It is important that you make backups before attempting an upgrade. -->
<!-- If downtime is not possible
The rolling upgrade is supported but ensure the
traffic is controlled during the upgrade and writes are directed only to 5.7
nodes until all nodes are upgraded to 8.0. -->
### [PXC strict mode] is enabled by default

Percona XtraDB Cluster in 8.0 runs with [PXC Strict Mode](../features/pxc-strict-mode.md#pxc-strict-mode) enabled by default. This will deny any unsupported operations and may halt the server if [a strict mode validation fails](../features/pxc-strict-mode.md#validations). It is recommended to first start the node with
the `pxc_strict_mode` variable set to `PERMISSIVE` in the MySQL
configuration file (on Debian and Ubuntu `/etc/mysql/mysql.conf.d/mysqld.cnf`; on CentOS and Red Hat /etc/my.cnf).

After you check the log for any tech preview features or unsupported features
and you have fixed any of the encountered incompatibilities, set the variable
back to `ENFORCING` at run time:

```{.bash data-prompt="mysql>"}
mysql> SET pxc_strict_mode=ENFORCING;
```

Also, switching back to `ENFORCING` may be done by restarting the node with
the updated configuration file (on Debian and Ubuntu `/etc/mysql/mysql.conf.d/mysqld.cnf`; on CentOS and Red Hat /etc/my.cnf).

### The configuration file layout has changed in PXC 8.0

All configuration settings are stored in the default MySQL configuration file:

* Path on Debian and Ubuntu: `/etc/mysql/mysql.conf.d/mysqld.cnf`

* Path on Red Hat and CentOS: /etc/my.cnf

Before you start the upgrade, move your custom settings from
`/etc/mysql/percona-xtradb-cluster.conf.d/wsrep.cnf` (on Debian and
Ubuntu) or from `/etc/percona-xtradb-cluster.conf.d/wsrep.cnf` (on Red Hat
and CentOS) to the new location accordingly.

### caching_sha2_password is the default authentication plugin

In Percona XtraDB Cluster 8.0, the default authentication plugin is
`caching_sha2_password`. The ProxySQL option
[–syncusers](proxysql-v2.md#pxc-proxysql-v2-admin-tool-syncusers) will not work if the Percona XtraDB Cluster user is
created using `caching_sha2_password`. Use the `mysql_native_password`
authentication plugin in these cases.

Be sure you are running on the latest 5.7 version before you upgrade to 8.0.

### mysql_upgrade is part of [SST](../glossary.md#sst)

**mysql_upgrade** is now run automatically as part of [SST](../glossary.md#sst). You do not have
to run it manually when upgrading your system from an older version.

<!-- Rolling Upgrade of a 3-Node PXC from 5.7 to 8.0
================================================================================

The major upgrade involves upgrading the PXC from the previous major version
5.7 to the new major version 8.0.

To upgrade the cluster, follow these steps for each node:

#. Make sure that all nodes are synchronized.

#. Stop the ``mysql`` service: -->
<!-- .. code-block: bash

$ sudo service mysql stop -->
<!-- #. Remove existing Percona XtraDB Cluster and Percona XtraBackup packages, then install Percona XtraDB Cluster version 8.0 packages.  For more information, see :ref:`install`.

For example, if you have Percona software repositories configured,
you might use the following commands: -->
<!-- * On CentOS or RHEL:

.. code-block: bash

   $ sudo yum remove percona-xtrabackup* Percona-XtraDB-Cluster*
   $ sudo yum install percona-xtradb-cluster -->
<!-- * On Debian or Ubuntu:

.. code-block: bash

   $ sudo apt-get remove percona-xtrabackup* percona-xtradb-cluster*
   $ sudo apt-get install percona-xtradb-cluster -->
<!-- #. Back up :file:`grastate.dat`, so that you can restore it
if it is corrupted or zeroed out due to network issue. -->
<!-- #. Now, start the cluster node with 8.0 packages installed, PXC will upgrade
the data directory as needed - either as part of the startup process or a
state transfer (IST/SST).

In most cases, starting the ``mysql`` service should run the node with your
previous configuration. For more information, see :ref:`add-node`.

.. code-block: bash

   $ sudo service mysql start

.. note:

   On CentOS, the `/etc/my.cnf` configuration file is renamed to
   `my.cnf.rpmsave`.  Make sure to rename it back before
   joining the upgraded node back to the cluster.

|strict-mode| is enabled by default, which may result in denying any unsupported operations and may halt the server. For more information, see    :ref:`upgrade-guide-changed-strict-mode`.

|opt.encrypt-cluster-traffic| is enabled by default. You need to configure
each node accordingly and avoid joining a cluster with unencrypted cluster
traffic: all nodes in your cluster must have traffic encryption enabled. For
more information, see :ref:`upgrade-guide-changed-traffic-encryption`. -->
<!-- #. Repeat this procedure for the next node in the cluster until you upgrade all nodes. -->
## Major upgrade scenarios

Upgrading PXC from 5.7 to 8.0 may have slightly different strategies depending
on the configuration and workload on your PXC cluster.

Note that the new default value of `pxc-encrypt-cluster-traffic` (set to *ON*
versus *OFF* in PXC 5.7) requires additional care. You cannot join a 5.7 node
to a PXC 8.0 cluster unless the node has traffic encryption enabled as the
cluster may not have some nodes with traffic encryption enabled and some nodes
with traffic encryption disabled. For more information, see
[Traffic encryption is enabled by default](#traffic-encryption-is-enabled-by-default).

### Scenario: No active parallel workload or with read-only workload

If there is no active parallel workload or the cluster has read-only workload
while upgrading the nodes, complete the following procedure for each node in the
cluster:

1. Shutdown one of the node 5.7 cluster nodes.

2. Remove 5.7 PXC packages without removing the data-directory.

3. Install PXC 8.0 packages.

4. Restart the mysqld service.

!!! important

    Before upgrading, make sure your application can work with a reduced cluster
    size.  Also, during the upgrade, the cluster will operate with an even number
    of nodes - the cluster may run into the split-brain problem.

This upgrade flow auto-detects the presence of the 5.7 data directory and trigger
the upgrade as part of the node bootup process. The data directory is upgraded
to be compatible with PXC 8.0. Then the node joins the cluster and
enters synced state. The 3-node cluster is restored with 2 nodes running PXC
5.7 and 1 node running PXC 8.0.

!!! note

    Since [SST](../glossary.md#sst) is not involved, [SST](../glossary.md#sst) based auto-upgrade flow is not started.

PXC 8.0 uses Galera 4 while PXC 5.7 uses Galera-3. The cluster will
continue to use the protocol version 3 used in Galera 3 effectively limiting
some of the functionality. With all nodes upgraded to version 8.0,
protocol version 4 is applied.

!!! admonition "Tip"

    The protocol version is stored in the ``protocol_version`` column of the ``wsrep_cluster`` table.

    ```{.bash data-prompt="mysql>"}
    mysql> USE mysql;
    ```

    ```{.bash data-prompt="mysql>"}
    mysql> SELECT protocol_version from wsrep_cluster;
    ```

    The example of the output is the following:

    ```{.text .no-copy}
    +------------------+
    | protocol_version |
    +------------------+
    |                4 |
    +------------------+
    1 row in set (0.00 sec)
    ```

As soon as the last 5.7 node shuts down, the configuration of the remaining
two nodes is updated to use protocol version 4. A new upgraded node will then join
using protocol version 4 and the whole cluster will maintain
protocol version 4 enabling the support for additional Galera 4 facilities.

It may take longer to join the last upgraded node since it will invite
[IST](../glossary.md#ist) to obtain the configuration changes.

!!! note

    Starting from Galera 4, the configuration changes are cached to `gcache`
    and the configuration changes are donated as part of [IST](../glossary.md#ist) or
    [SST](../glossary.md#sst) to help build the certification queue on the JOINING node. As
    other nodes (say n2 and n3), already using protocol version 4, donate the
    configuration changes when the JOINER node is booted.

The situation was different for the previous and penultimate nodes since the
donation of the configuration changes is not supported by protocol
version 3 that they used.

With [IST](../glossary.md#ist) involved on joining the last node, the smart IST flow is triggered
to take care of the upgrade even before MySQL starts to look at the
data directory.

!!! important

    It is not recommended to restart the last node without upgrading it.

<!-- Scenario: Upgrading with active parallel read-write workload
-------------------------------------------------------------------------------- -->
<!-- The upgrade prosess is similar to that described in
:ref:`upgrading-rolling-no-active-read-only-parallel-workload`: since
the cluster state keeps changing while each node is taken down and
upgraded, :term:`IST` or :term:`SST` is triggered when rejoining the
node that you upgrading. In |version|, PXC clears the configuration
of each replica node (via `RESET SLAVE ALL <https://dev.mysql.com/doc/refman/8.0/en/reset-slave.html>`_). -->
<!-- important:

It is important that the joining node have enough replica threads to catch up
IST write-sets and cluster write-sets. -->
<!-- Scenario: Adding 8.0 node to a 5.7 cluster
--------------------------------------------------------------------------------

Instead of upgrading a node in a 5.7 cluster to PXC 8.0, you are booting a
fresh 8.0 node and try joining it as an additional node.

Suppose the 5.7 cluster is already active and has processed write-sets. The new
8.0 node joins the cluster and gets a dump of the cluster through :term:`SST`
and remains part of the cluster.

Since the underline protocol version negotiated in both cases is based on Galera 3,
the 8.0 node will fail to service galera 4 features. As soon as all 5.7
nodes leave the cluster, the 8.0 nodes re-negotiates using protocol version 4
and gets a proper local index and other properties assigned.

.. warning:

   Joining a 5.7 node to a PXC |version| cluster is not supported. -->
<!-- Scenario: Upgrading a node which is an async replication replica
--------------------------------------------------------------------------------

In this case, you need to configure the source-replica replication. Make one of the
PXC nodes as an async replica. Upgrade the PXC async replica node and check the
replication status. -->
### Scenario: Upgrade from PXC 5.6 to PXC 8.0

First, upgrade PXC from 5.6 to the latest version of PXC 5.7. Then proceed
with the upgrade using the procedure described in
[Scenario: No active parallel workload or with read-only workload](#scenario-no-active-parallel-workload-or-with-read-only-workload).

## Minor upgrade

To upgrade the cluster, follow these steps for each node:

1. Make sure that all nodes are synchronized.

2. Stop the `mysql` service:

    ```{.bash data-prompt="$"}
    $ sudo service mysql stop
    ```

3. Upgrade Percona XtraDB Cluster and Percona XtraBackup packages.
For more information, see [Installing Percona XtraDB Cluster](../install/index.md#install).

4. Back up `grastate.dat`, so that you can restore it
if it is corrupted or zeroed out due to network issue.

5. Now, start the cluster node with 8.0 packages installed, PXC will upgrade
the data directory as needed - either as part of the startup process or a
state transfer (IST/SST).

    In most cases, starting the `mysql` service should run the node with your
    previous configuration. For more information, see [Adding Nodes to Cluster](../add-node.md#add-node).

    ```{.bash data-prompt="$"}
    $ sudo service mysql start
    ```

    !!! note

        On CentOS, the /etc/my.cnf configuration file is renamed to `my.cnf.rpmsave`. Make sure to rename it back before joining the upgraded node back to the cluster.


    [PXC Strict Mode](../features/pxc-strict-mode.md#pxc-strict-mode) is enabled by default, which may result in denying any
    unsupported operations and may halt the server. For more information, see
    [pxc-strict-mode is enabled by default](#pxc-strict-mode-is-enabled-by-default).

    `pxc-encrypt-cluster-traffic` is enabled by default. You need to configure
    each node accordingly and avoid joining a cluster with unencrypted cluster
    traffic. For more information, see
    [Traffic encryption is enabled by default](#traffic-encryption-is-enabled-by-default).

6. Repeat this procedure for the next node in the cluster
until you upgrade all nodes.

