# Upgrading Percona XtraDB Cluster

This guide describes the procedure for upgrading Percona XtraDB Cluster without downtime
(*rolling upgrade*) to the latest 5.7 version. A “rolling upgrade” means
there is no need to take down the complete cluster during the upgrade.

Both major upgrades (from 5.6 to 5.7 version) and minor ones (from 5.7.x
to 5.7.y) can be done in this way. Rolling upgrades to 5.7 from versions
older than 5.6 are not supported. Therefore if you are running Percona XtraDB Cluster version
5.5, it is recommended to shut down all nodes, then remove and re-create
the cluster from scratch. Alternatively, you can perform a
[rolling upgrade from PXC 5.5 to 5.6](https://www.percona.com/doc/percona-xtradb-cluster/5.6/upgrading_guide_55_56.html),
and then follow the current procedure to upgrade from 5.6 to 5.7.

The following documents contain details about relevant changes
in the 5.7 series of MySQL and Percona Server.
Make sure you deal with any incompatible features and variables
mentioned in these documents when upgrading to Percona XtraDB Cluster 5.7.

* [Changed in Percona Server 5.7](https://www.percona.com/doc/percona-server/5.7/changed_in_57.html)

* [Upgrading MySQL](https://dev.mysql.com/doc/refman/5.7/en/upgrading.html)

* [Upgrading from MySQL 5.6 to 5.7](https://dev.mysql.com/doc/refman/5.7/en/upgrading-from-previous-series.html)

## Major upgrade

To upgrade the cluster, follow these steps for each node:

1. Make sure that all nodes are synchronized.

2. Stop the `mysql` service:

    ```shell
    $ sudo service mysql stop
    ```

3. Remove existing Percona XtraDB Cluster and Percona XtraBackup packages,
   then install Percona XtraDB Cluster version 5.7 packages.
   For more information, see [Installing Percona XtraDB Cluster](../install/index.md#install).

    For example, if you have Percona software repositories configured,
    you might use the following commands:

    * On CentOS or RHEL:

    ```shell
    $ sudo yum remove percona-xtrabackup* Percona-XtraDB-Cluster*
    $ sudo yum install Percona-XtraDB-Cluster-57
    ```

    * On Debian or Ubuntu:

    ```shell
    $ sudo apt remove percona-xtrabackup* percona-xtradb-cluster*
    $ sudo apt install percona-xtradb-cluster-57
    ```

4. In case of Debian or Ubuntu, the `mysql` service starts automatically after install.
    Stop the service:

    ```shell
    $ sudo service mysql stop
    ```

5. Back up `grastate.dat`, so that you can restore it if it is corrupted or zeroed out due to network issue.

6. Start the node outside the cluster (in standalone mode) by setting the [`wsrep_provider`](../wsrep-system-index.md#wsrep_provider) variable to `none`.
   
    For example:

    ```shell
    sudo mysqld --skip-grant-tables --user=mysql --wsrep-provider='none'
    ```
   
    !!! note

        As of Percona XtraDB Cluster 5.7.6, the `--skip-grant-tables` option
        is not required.

    !!! note

        To prevent any users from accessing this node while performing
        work on it, you may add [–skip-networking](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_skip_networking)
        to the startup options and use a local socket to connect, or
        alternatively you may want to divert any incoming traffic from your
        application to other operational nodes.

7. Open another session and run `mysql_upgrade`.

8. When the upgrade is done, stop the `mysqld` process.
   You can either run `sudo kill` on the `mysqld` process ID,
   or `sudo mysqladmin shutdown` with the MySQL root user credentials.

    !!! note

        On CentOS, the `my.cnf` configuration file
        is renamed to `my.cnf.rpmsave`.
        Make sure to rename it back
        before joining the upgraded node back to the cluster. 

9. Now you can join the upgraded node back to the cluster.

    In most cases, starting the `mysql` service
    should run the node with your previous configuration:

    ```shell
    $ sudo service mysql start
    ```

    For more information, see [Adding Nodes to Cluster](../add-node.md#add-node).

    !!! note

        As of version 5.7,
        Percona XtraDB Cluster runs with [PXC Strict Mode](../features/pxc-strict-mode.md#pxc-strict-mode) enabled by default.
        This will deny any unsupported operations and may halt the server
        upon encountering a failed validation.

        If you are not sure, it is recommended to first start the node
        with the [`pxc_strict_mode`](../wsrep-system-index.md#pxc_strict_mode) variable set to `PERMISSIVE` in the in the *MySQL* configuration file, `my.cnf`.

        After you check the log for any experimental or unsupported features
        and fix any encountered incompatibilities,
        you can set the variable back to `ENFORCING` at run time:

        ```shell
        mysql> SET pxc_strict_mode=ENFORCING;
        ```

        Also switch back to `ENFORCING` may be done by restarting the node
        with updated `my.cnf`. 

10. Repeat this procedure for the next node in the cluster until you upgrade all nodes.

It is important that on rejoining, the node should synchronize using
[IST](../glossary.md#ist). For this, it is best not to leave the cluster node being
upgraded offline for an extended period. More on this below.

When performing any upgrade (major or minor), [SST](../glossary.md#sst) could
be initiated by the joiner node after the upgrade if the server
was offline for some time. After [SST](../glossary.md#sst) completes, the data
directory structure needs to be upgraded (using mysql_upgrade)
once more time to ensure compatibility with the newer version
of binaries.

!!! note

    In case of [SST](../glossary.md#sst) synchronization, the error log
    contains statements like “Check if state gap can be
    serviced using IST … State gap can’t be serviced
    using IST. Switching to SST” instead of “Receiving
    IST: …” lines appropriate to  [IST](../glossary.md#sst)
    synchronization.

## Minor upgrade

To upgrade the cluster, follow these steps for each node:

1. Make sure that all nodes are synchronized.

2. Stop the `mysql` service:
 
    ```shell
    $ sudo service mysql stop
    ```

3. Upgrade Percona XtraDB Cluster and Percona XtraBackup packages.
   For more information, see [Installing Percona XtraDB Cluster](../install/index.md#install).

    For example, if you have Percona software repositories configured,
    you might use the following commands:

    * On CentOS or RHEL:

      ```shell
      $ sudo yum update Percona-XtraDB-Cluster-57
      ```

    * On Debian or Ubuntu:

      ```shell
      $ sudo apt install --only-upgrade percona-xtradb-cluster-57
      ```

4. In case of Debian or Ubuntu,
   the `mysql` service starts automatically after install.
    
    Stop the service:

     ```shell
     $ sudo service mysql stop
     ```

5. Back up `grastate.dat`, so that you can restore it if it is corrupted or zeroed out due to network issue.

6. Start the node outside the cluster (in standalone mode)
   by setting the [`wsrep_provider`](../wsrep-system-index.md#wsrep_provider) variable to `none`.
    
    For example:

    ```shell
    sudo mysqld --skip-grant-tables --user=mysql --wsrep-provider='none'
    ```

    !!! note

        As of Percona XtraDB Cluster 5.7.6, the `--skip-grant-tables` option
        is not required.

    !!! note

        To prevent any users from accessing this node while performing
        work on it, you may add [–skip-networking](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_skip_networking)
        to the startup options and use a local socket to connect, or
        alternatively you may want to divert any incoming traffic from your
        application to other operational nodes.

7. Open another session and run `mysql_upgrade`.

8. When the upgrade is done, stop the `mysqld` process.
   You can either run `sudo kill` on the `mysqld` process ID,
   or `sudo mysqladmin shutdown` with the MySQL root user credentials.

    !!! note

        On CentOS, the `my.cnf` configuration file
        is renamed to `my.cnf.rpmsave`.
        Make sure to rename it back
        before joining the upgraded node back to the cluster.

9. Now you can join the upgraded node back to the cluster.

    In most cases, starting the `mysql` service
    should run the node with your previous configuration:

    ```shell
    $ sudo service mysql start
    ```

    For more information, see [Adding Nodes to Cluster](../add-node.md#add-node).
   
    !!! note

        As of version 5.7, Percona XtraDB Cluster runs with [PXC Strict Mode](../features/pxc-strict-mode.md#pxc-strict-mode) enabled by default.
        This will deny any unsupported operations and may halt the server
        upon encountering a failed validation.

        If you are not sure, it is recommended to first start the node
        with the [`pxc_strict_mode`](../wsrep-system-index.md#pxc_strict_mode) variable set to `PERMISSIVE` in the in the *MySQL* configuration file, `my.cnf`.

        After you check the log for any experimental or unsupported features
        and fix any encountered incompatibilities,
        you can set the variable back to `ENFORCING` at run time:

        ```shell
        mysql> SET pxc_strict_mode=ENFORCING;
        ```

        Also switch back to `ENFORCING` may be done by restarting the node
        with updated `my.cnf`.

10. Repeat this procedure for the next node in the cluster until you upgrade all nodes.

## Dealing with IST/SST synchronization while upgrading

It is important that on rejoining, the node should synchronize using
[IST](../glossary.md#ist). For this, it is best not to leave the cluster node being
upgraded offline for an extended period. More on this below.

When performing any upgrade (major or minor), [SST](../glossary.md#sst) could
be initiated by the joiner node after the upgrade if the server
was offline for some time. After [SST](../glossary.md#sst) completes, the data
directory structure needs to be upgraded (using mysql_upgrade)
once more time to ensure compatibility with the newer version
of binaries.

!!! note

    In case of [SST](../glossary.md#sst) synchronization, the error log
    contains statements like “Check if state gap can be
    serviced using IST … State gap can’t be serviced
    using IST. Switching to SST” instead of “Receiving
    IST: …” lines appropriate to  [IST](../glossary.md#ist)
    synchronization.

The following additional steps should be made to upgrade the data
directory structure after [SST](../glossary.md#sst) (after the normal major or minor upgrade steps):

1. Shutdown the node that rejoined the cluster using [SST](../glossary.md#sst):

    ```shell
    $ sudo service mysql stop
    ```

2. Restart the node in standalone mode by setting the [`wsrep_provider`](../wsrep-system-index.md#wsrep_provider) variable to `none`, for example:

    ```shell
    sudo mysqld --skip-grant-tables --user=mysql --wsrep-provider='none'
    ```

3. Run `mysql-upgrade`

4. Restart the node in cluster mode (e.g., by executing `sudo service mysql start` and make sure the cluster joins back using [IST](../glossary.md#ist).
