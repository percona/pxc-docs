# Configure nodes for write-set replication

After installing Percona XtraDB Cluster on each node, you need to configure the cluster.
In this section, we will demonstrate how to configure a three node cluster:

| Node | Host | IP |
| ---- | ---- | -- |
| Node 1 | pxc1 | 192.168.70.61 |
| Node 2 | pxc2 | 192.168.70.62 |
| Node 3 | pxc3 | 192.168.70.63 |

1. Stop the Percona XtraDB Cluster server. After the installation completes the server is not started. You need this step if you have started the server manually.

    ```shell
    sudo service mysql stop
    ```

2. Generate the SSL certificates on the first node. Ensure that all nodes use 
   identical key and certificate files to maintain a consistent security setup. 
   Store the certificates in `/etc/`, not in the data directory. After generating 
   the certificates, copy them to all other nodes.

    ```shell
    sudo openssl req -newkey rsa:2048 -nodes -keyout /etc/server-key.pem \
    -x509 -days 365 -out /etc/server-cert.pem
    sudo cp /etc/server-key.pem pxc2:/etc/
    sudo cp /etc/server-cert.pem pxc2:/etc/
    sudo cp /etc/server-key.pem pxc3:/etc/
    sudo cp /etc/server-cert.pem pxc3:/etc/
    ```
    
3. Edit the configuration file of the first node to provide the cluster settings.

    *If you use Debian or Ubuntu*, edit `/etc/mysql/mysql.conf.d/mysqld.cnf`:

    ```ini
    wsrep_provider=/usr/lib/galera4/libgalera_smm.so
    wsrep_cluster_name=pxc-cluster
    wsrep_cluster_address=gcomm://192.168.70.61,192.168.70.62,192.168.70.63
    ```

    *If you use Red Hat Enterprise Linux*, edit `/etc/my.cnf`. Note that on these systems you set
    the `wsrep_provider option` to a different value:

    ```ini
    wsrep_provider=/usr/lib64/galera4/libgalera_smm.so
    wsrep_cluster_name=pxc-cluster
    wsrep_cluster_address=gcomm://192.168.70.61,192.168.70.62,192.168.70.63
    ```

4. Configure *node 1*.

    ```ini
    wsrep_node_name=pxc1
    wsrep_node_address=192.168.70.61
    pxc_strict_mode=ENFORCING
    ```

5. Set up *node 2* and *node 3* in the same way: Stop the server and update the configuration file applicable to your system. All settings are the same except for [`wsrep_node_name`](wsrep-system-index.md#wsrep_node_name) and [`wsrep_node_address`](wsrep-system-index.md#wsrep_node_address).

    For node 2
        
        ```ini
        wsrep_node_name=pxc2
        wsrep_node_address=192.168.70.62
        ```

    For node 3

        ```ini
        wsrep_node_name=pxc3
        wsrep_node_address=192.168.70.63
        ```
        
5. Set up the traffic encryption settings. Each node of the cluster must use the same SSL certificates stored in `/etc/`.

    ```ini
    [mysqld]
    wsrep_provider_options=”socket.ssl_key=server-key.pem;socket.ssl_cert=server-cert.pem;socket.ssl_ca=ca.pem”

    [sst]
    encrypt=4
    ssl-key=server-key.pem
    ssl-ca=ca.pem
    ssl-cert=server-cert.pem
    ```

!!! important

    In Percona XtraDB Cluster {{vers}}, the [Encrypting Replication Traffic](encrypt-traffic.md#encrypt-replication-traffic) is
    enabled by default by the `pxc-encrypt-cluster-traffic` variable.

    The replication traffic encryption cannot be enabled on a running cluster. If
    it was disabled before the cluster was bootstrapped, the cluster must to
    stopped. Then set up the encryption, and bootstrap (see [`Bootstrapping the First Node`](bootstrap.md#bootstrap-the-first-node))
    again.

    !!! admonition "See also"

        More information about the security settings in Percona XtraDB Cluster
         * [`Security basics`](security-index.md#security-basics)
	     * [`Encrypting PXC traffic`](encrypt-traffic.md#encrypt-pxc-traffic)
	     * [`SSL automatic configuration`](encrypt-traffic.md#ssl-automatic-configuration)


## Template of the configuration file

Here is an example of a full configuration file installed on Red Hat Enterprise Linux to
`/etc/my.cnf`.

```text
# Template my.cnf for PXC
# Edit to your requirements.
[client]
socket=/var/lib/mysql/mysql.sock
[mysqld]
server-id=1
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
# Binary log expiration period is 604800 seconds, which equals 7 days
binlog_expire_logs_seconds=604800
######## wsrep ###############
# Path to Galera library
wsrep_provider=/usr/lib64/galera4/libgalera_smm.so
# Cluster connection URL contains IPs of nodes
#If no IP is found, this implies that a new cluster needs to be created,
#in order to do that you need to bootstrap this node
wsrep_cluster_address=gcomm://
# Slave thread to use
wsrep_applier_threads=8
wsrep_log_conflicts
# This changes how InnoDB autoincrement locks are managed and is a requirement for Galera
innodb_autoinc_lock_mode=2
# Node IP address
#wsrep_node_address=192.168.70.63
# Cluster name
wsrep_cluster_name=pxc-cluster
#If wsrep_node_name is not specified,  then system hostname will be used
wsrep_node_name=pxc-cluster-node-1
#pxc_strict_mode allowed values: DISABLED,PERMISSIVE,ENFORCING,MASTER
pxc_strict_mode=ENFORCING
# SST method
wsrep_sst_method=xtrabackup-v2
```

## Next Steps: Bootstrap the first node

After you configure all your nodes, initialize Percona XtraDB Cluster by bootstrapping the first
node according to the procedure described in [Bootstrapping the First Node](bootstrap.md#bootstrap-the-first-node).

## Essential configuration variables

[`wsrep_provider`](wsrep-system-index.md#wsrep_provider)

Specify the path to the Galera library. The location depends on the distribution:

* Debian and Ubuntu: `/usr/lib/galera4/libgalera_smm.so`

* Red Hat Enterprise Linux: `/usr/lib64/galera4/libgalera_smm.so`

[`wsrep_cluster_name`](wsrep-system-index.md#wsrep_cluster_name)

Specify the logical name for your cluster.
It must be the same for all nodes in your cluster.

[`wsrep_cluster_address`](wsrep-system-index.md#wsrep_cluster_address)

Specify the IP addresses of nodes in your cluster.
At least one is required for a node to join the cluster,
but it is recommended to list addresses of all nodes.
This way if the first node in the list is not available,
the joining node can use other addresses.

!!! note

    No addresses are required for the initial node in the cluster.
    However, it is recommended to specify them
    and [properly bootstrap the first node](bootstrap.md#bootstrap-the-first-node).
    This will ensure that the node is able to rejoin the cluster if it goes down in the future.

[`wsrep_node_name`](wsrep-system-index.md#wsrep_node_name)

Specify the logical name for each individual node.
If this variable is not specified, the host name will be used.

[`wsrep_node_address`](wsrep-system-index.md#wsrep_node_address)

Specify the IP address of this particular node.

[`wsrep_sst_method`](wsrep-system-index.md#wsrep_sst_method)

By default, Percona XtraDB Cluster uses Percona [XtraBackup :octicons-link-external-16:](https://www.percona.com/software/mysql-database/percona-xtrabackup) for [State Snapshot Transfer](glossary.md#sst). `xtrabackup-v2` is the only supported option for this variable.
This method requires a user for SST to be set up on the initial node.

[`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode)

[PXC Strict Mode](strict-mode.md#percona-xtradb-cluster-strict-mode) is enabled by default and set to `ENFORCING`, which blocks the use of tech preview features and unsupported features in Percona XtraDB Cluster.

[`default_storage_engine` :octicons-link-external-16:](https://dev.mysql.com/doc/refman/{{vers}}/en/server-system-variables.html#sysvar_default_storage_engine)

Galera fully supports only the InnoDB storage engine.
It will not work correctly with MyISAM
or any other non-transactional storage engines.
Set this variable to `default_storage_engine=InnoDB`.

[`innodb_autoinc_lock_mode` :octicons-link-external-16:](https://dev.mysql.com/doc/refman/{{vers}}/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode)

Galera supports only interleaved (`2`) lock mode for InnoDB.
Setting the traditional (`0`) or consecutive (`1`) lock mode
can cause replication to fail due to unresolved deadlocks.
Set this variable to `innodb_autoinc_lock_mode=2`.

