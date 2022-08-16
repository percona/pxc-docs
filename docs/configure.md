# Configuring Nodes for Write-Set Replication

After installing Percona XtraDB Cluster on a node,
configure it with information about the cluster.

!!! note

    Make sure that the Percona XtraDB Cluster server is not running.
    
    ```shell
    $ sudo service mysql stop
    ```

Configuration examples assume there are three Percona XtraDB Cluster nodes:

| Node | Host | IP |
| ---- | ---- | -- |
| Node 1 | pxc1 | 192.168.70.61 |
| Node 2 | pxc2 | 192.168.70.62 |
| Node 3 | pxc3 | 192.168.70.63 |

If you are running Debian or Ubuntu,
add the following configuration variables to */etc/percona-xtradb-cluster.conf.d/wsrep.cnf* on the first node:

```text
[mysqld]
wsrep_provider=/usr/lib/libgalera_smm.so
wsrep_cluster_name=pxc-cluster
wsrep_cluster_address=gcomm://192.168.70.61,192.168.70.62,192.168.70.63
wsrep_node_name=pxc1
wsrep_node_address=192.168.70.61
wsrep_sst_method=xtrabackup-v2
wsrep_sst_auth=sstuser:passw0rd
pxc_strict_mode=ENFORCING
binlog_format=ROW
default_storage_engine=InnoDB
innodb_autoinc_lock_mode=2
```

If you are running Red Hat or CentOS,
add the following configuration variables to */etc/percona-xtradb-cluster.conf.d/wsrep.cnf*
on the first node:

```text
[mysqld]
 wsrep_provider=/usr/lib64/galera3/libgalera_smm.so
 wsrep_cluster_name=pxc-cluster
 wsrep_cluster_address=gcomm://192.168.70.61,192.168.70.62,192.168.70.63
 wsrep_node_name=pxc1
 wsrep_node_address=192.168.70.61
 wsrep_sst_method=xtrabackup-v2
 wsrep_sst_auth=sstuser:passw0rd
 pxc_strict_mode=ENFORCING
 binlog_format=ROW
 default_storage_engine=InnoDB
 innodb_autoinc_lock_mode=2
```

Use the same configuration for the second and third nodes,
except the `wsrep_node_name` and `wsrep_node_address` variables:

* For the second node:

  ```text
  wsrep_node_name=pxc2
  wsrep_node_address=192.168.70.62
  ```

* For the third node:

  ```text
  wsrep_node_name=pxc3
  wsrep_node_address=192.168.70.63
  ```

## Configuration Reference

[`wsrep_provider`](wsrep-system-index.md#wsrep_provider)

Specify the path to the Galera library.

!!! note

    The location depends on the distribution:

     * Debian or Ubuntu: `/usr/lib/libgalera_smm.so`

     * Red Hat or CentOS: `/usr/lib64/galera3/libgalera_smm.so`

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

    No addresses are required for the initial node in the cluster. However, it is recommended to specify them and [properly bootstrap the first node](bootstrap.md#bootstrap). This will ensure that the node is able to rejoin the cluster if it goes down in the future.

[`wsrep_node_name`](wsrep-system-index.md#wsrep_node_name)

Specify the logical name for each individual node.
If this variable is not specified, the host name will be used.

[`wsrep_node_address`](wsrep-system-index.md#wsrep_node_address)

Specify the IP address of this particular node.

[`wsrep_sst_method`](wsrep-system-index.md#wsrep_sst_method)

By default, Percona XtraDB Cluster uses [Percona XtraBackup](https://www.percona.com/software/mysql-database/percona-xtrabackup) for *State Snapshot Transfer* ([SST](glossary.md#sst)).
Setting `wsrep_sst_method=xtrabackup-v2` is highly recommended.
This method requires a user for SST to be set up on the initial node.
Provide SST user credentials with the `wsrep_sst_auth` variable.

[`wsrep_sst_auth`](wsrep-system-index.md#wsrep_sst_auth)

Specify authentication credentials for [SST](glossary.md#sst) as `<sstuser>:<sst_pass>`.
You must create this user when [Bootstrapping the First Node](bootstrap.md#bootstrap) and provide necessary privileges for it:

```text
mysql> CREATE USER 'sstuser'@'localhost' IDENTIFIED BY 'passw0rd';
mysql> GRANT RELOAD, LOCK TABLES, PROCESS, REPLICATION CLIENT ON *.* TO
  'sstuser'@'localhost';
mysql> FLUSH PRIVILEGES;
```

For more information, see [Privileges for Percona XtraBackup](https://www.percona.com/doc/percona-xtrabackup/2.4/using_xtrabackup/privileges.html).

[`pxc_strict_mode`](wsrep-system-index.md#pxc_strict_mode)

[PXC Strict Mode](features/pxc-strict-mode.md#pxc_strict_mode) is enabled by default and set to `ENFORCING`, which blocks the use of experimental and unsupported features in Percona XtraDB Cluster.

[`binlog_format`](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_binlog_format)

Galera supports only row-level replication, so set `binlog_format=ROW`.

[`default_storage_engine`](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_default_storage_engine)

Galera fully supports only the InnoDB storage engine.
It will not work correctly with MyISAM
or any other non-transactional storage engines.
Set this variable to `default_storage_engine=InnoDB`.

[`innodb_autoinc_lock_mode`](https://dev.mysql.com/doc/refman/5.7/en/innodb-parameters.html#sysvar_innodb_autoinc_lock_mode)

Galera supports only interleaved (`2`) lock mode for InnoDB.
Setting the traditional (`0`) or consecutive (`1`) lock mode
can cause replication to fail due to unresolved deadlocks.
Set this variable to `innodb_autoinc_lock_mode=2`.

## Next Steps

After you configure all your nodes,
initialize Percona XtraDB Cluster by bootstrapping the first node
according to the procedure described in [Bootstrapping the First Node](bootstrap.md#bootstrap).
