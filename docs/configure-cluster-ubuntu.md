# Configure a cluster on Debian or Ubuntu

This tutorial describes how to install and configure three Percona XtraDB Cluster nodes
on Ubuntu 14 LTS servers, using the packages from Percona repositories.

* Node 1

    * Host name: `pxc1`

    * IP address: `192.168.70.61`

* Node 2

    * Host name: `pxc2`

    * IP address: `192.168.70.62`

* Node 3

    * Host name: `pxc3`

    * IP address: `192.168.70.63`

## Prerequisites

The procedure described in this tutorial requires he following:

* All three nodes have Ubuntu 14 LTS installed.

* Firewall on all nodes is configured to allow connecting
to ports 3306, 4444, 4567 and 4568.

* AppArmor profile for MySQL is [disabled](https://www.percona.com/blog/percona-xtradb-cluster-selinux-is-not-always-the-culprit/).

## Step 1. Install PXC

Install Percona XtraDB Cluster on all three nodes as described in [Installing Percona XtraDB Cluster on Debian or Ubuntu](apt.md#apt).

!!! note

    Debian/Ubuntu installation prompts for root password.
    For this tutorial, set it to `Passw0rd`.
    After the packages have been installed,
    `mysqld` will start automatically.
    Stop `mysqld` on all three nodes using `sudo systemctl stop mysql`.

## Step 2. Configure the first node

Individual nodes should be configured to be able to bootstrap the cluster.
For more information about bootstrapping the cluster, see [Bootstrapping the First Node](bootstrap.md#bootstrap).

1. Make sure that the configuration file `/etc/mysql/my.cnf`
for the first node (`pxc1`) contains the following:

    ```{.text .no-copy}
    [mysqld]

    datadir=/var/lib/mysql
    user=mysql

    # Path to Galera library
    wsrep_provider=/usr/lib/libgalera_smm.so

    # Cluster connection URL contains the IPs of node#1, node#2 and node#3
    wsrep_cluster_address=gcomm://192.168.70.61,192.168.70.62,192.168.70.63

    # In order for Galera to work correctly binlog format should be ROW
    binlog_format=ROW

    # Using the MyISAM storage engine is not recommended
    default_storage_engine=InnoDB

    # This InnoDB autoincrement locking mode is a requirement for Galera
    innodb_autoinc_lock_mode=2

    # Node #1 address
    wsrep_node_address=192.168.70.61

    # SST method
    wsrep_sst_method=xtrabackup-v2

    # Cluster name
    wsrep_cluster_name=my_ubuntu_cluster
    ```

1. Start the first node with the following command:

    ```{.bash data-prompt="[root@pxc1 ~]#"}
    [root@pxc1 ~]# systemctl start mysql@bootstrap.service
    ```

    This command will start the first node and bootstrap the cluster.

2. After the first node has been started,
cluster status can be checked with the following command:

    ```{.bash data-prompt="mysql>"}
    mysql> show status like 'wsrep%';
    ``` 

    The following outut shows the cluste status:

    ??? example "Expected output"

        ```{.text .no-copy}
        +----------------------------+--------------------------------------+
        | Variable_name              | Value                                |
        +----------------------------+--------------------------------------+
        | wsrep_local_state_uuid     | b598af3e-ace3-11e2-0800-3e90eb9cd5d3 |
        ...
        | wsrep_local_state          | 4                                    |
        | wsrep_local_state_comment  | Synced                               |
        ...
        | wsrep_cluster_size         | 1                                    |
        | wsrep_cluster_status       | Primary                              |
        | wsrep_connected            | ON                                   |
        ...
        | wsrep_ready                | ON                                   |
        +----------------------------+--------------------------------------+
        75 rows in set (0.00 sec)
        ```

    This output shows that the cluster has been successfully bootstrapped.

To perform [State Snapshot Transfer](state-snapshot-transfer.md#state-snapshot-transfer) using *XtraBackup*,
set up a new user with proper [privileges](https://docs.percona.com/percona-xtrabackup/8.0/privileges.html):

```{.bash data-prompt="mysql@pxc1>"}
mysql@pxc1> CREATE USER 'sstuser'@'localhost' IDENTIFIED BY 's3cretPass';
mysql@pxc1> GRANT PROCESS, RELOAD, LOCK TABLES, REPLICATION CLIENT ON *.* TO 'sstuser'@'localhost';
mysql@pxc1> FLUSH PRIVILEGES;
```

!!! note

    MySQL root account can also be used for performing SST, but it is more secure to use a different (non-root) user for this.

## Step 3. Configure the second node

1. Make sure that the configuration file `/etc/mysql/my.cnf`
on the second node (`pxc2`) contains the following:

    ```{.text .no-copy}
    [mysqld]

    datadir=/var/lib/mysql
    user=mysql

    # Path to Galera library
    wsrep_provider=/usr/lib/libgalera_smm.so

    # Cluster connection URL contains IPs of node#1, node#2 and node#3
    wsrep_cluster_address=gcomm://192.168.70.61,192.168.70.62,192.168.70.63

    # In order for Galera to work correctly binlog format should be ROW
    binlog_format=ROW

    # Using the MyISAM storage engine is not recommended
    default_storage_engine=InnoDB

    # This InnoDB autoincrement locking mode is a requirement for Galera
    innodb_autoinc_lock_mode=2

    # Node #2 address
    wsrep_node_address=192.168.70.62

    # Cluster name
    wsrep_cluster_name=my_ubuntu_cluster

    # SST method
    wsrep_sst_method=xtrabackup-v2
    ```

2. Start the second node with the following command:

    ```{.bash data-prompt="[root@pxc2 ~]#"}
    [root@pxc2 ~]# systemctl start mysql
    ```

3. After the server has been started, it should receive SST automatically.
    Cluster status can now be checked on both nodes.
    The following is an example of status from the second node (`pxc2`):

    ```{.bash data-prompt="mysql>"}
    mysql> show status like 'wsrep%';
    ```

    The following output shows that the new node has been successfully added to the cluster.

    ??? example "Expected output"

        ```{.text .no-copy}
        +----------------------------+--------------------------------------+
        | Variable_name              | Value                                |
        +----------------------------+--------------------------------------+
        | wsrep_local_state_uuid     | b598af3e-ace3-11e2-0800-3e90eb9cd5d3 |
        ...
        | wsrep_local_state          | 4                                    |
        | wsrep_local_state_comment  | Synced                               |
        ...
        | wsrep_cluster_size         | 2                                    |
        | wsrep_cluster_status       | Primary                              |
        | wsrep_connected            | ON                                   |
        ...
        | wsrep_ready                | ON                                   |
        +----------------------------+--------------------------------------+
        40 rows in set (0.01 sec)
        ```

## Step 4. Configure the third node


1. Make sure that the MySQL configuration file `/etc/mysql/my.cnf`
on the third node (`pxc3`) contains the following:

    ```{.text .no-copy}
    [mysqld]

    datadir=/var/lib/mysql
    user=mysql

    # Path to Galera library
    wsrep_provider=/usr/lib/libgalera_smm.so

    # Cluster connection URL contains IPs of node#1, node#2 and node#3
    wsrep_cluster_address=gcomm://192.168.70.61,192.168.70.62,192.168.70.63

    # In order for Galera to work correctly binlog format should be ROW
    binlog_format=ROW

    # Using the MyISAM storage engine is not recommended
    default_storage_engine=InnoDB

    # This InnoDB autoincrement locking mode is a requirement for Galera
    innodb_autoinc_lock_mode=2

    # Node #3 address
    wsrep_node_address=192.168.70.63

    # Cluster name
    wsrep_cluster_name=my_ubuntu_cluster

    # SST method
    wsrep_sst_method=xtrabackup-v2
    ```

2. Start the third node with the following command:

    ```{.bash data-prompt="[root@pxc3 ~]#"}
    [root@pxc3 ~]# systemctl start mysql
    ```

3. After the server has been started,
it should receive SST automatically.
Cluster status can be checked on all nodes.
The following is an example of status from the third node (`pxc3`):

```{.bash data-prompt="mysql>"}
mysql> show status like 'wsrep%';
```

The following output confirms that the third node has joined the cluster.

??? example "Expected output"

    ```{.text .no-copy}
    +----------------------------+--------------------------------------+
    | Variable_name              | Value                                |
    +----------------------------+--------------------------------------+
    | wsrep_local_state_uuid     | b598af3e-ace3-11e2-0800-3e90eb9cd5d3 |
    ...
    | wsrep_local_state          | 4                                    |
    | wsrep_local_state_comment  | Synced                               |
    ...
    | wsrep_cluster_size         | 3                                    |
    | wsrep_cluster_status       | Primary                              |
    | wsrep_connected            | ON                                   |
    ...
    | wsrep_ready                | ON                                   |
    +----------------------------+--------------------------------------+
    40 rows in set (0.01 sec)
    ```

## Test replication

To test replication, lets create a new database on the second node,
create a table for that database on the third node,
and add some records to the table on the first node.

1. Create a new database on the second node:

    ```{.bash data-prompt="mysql@percona2>"}
    mysql@percona2> CREATE DATABASE percona;
    ```

    The following output confirms that a new database has been created:

    ??? example "Expected output"

        ```{.text .no-copy}
        Query OK, 1 row affected (0.01 sec)
        ```

2. Switch to a newly created database:

    ```{.bash data-prompt="mysql@percona3>"}
    mysql@percona3> USE percona;
    ```
    
    The following output confirms that a database has been changed:

    ??? example "Expected output"

        ```{.text .no-copy}
        Database changed
        ```

3. Create a table on the third node:

    ```{.bash data-prompt="mysql@percona3>"}
    mysql@percona3> CREATE TABLE example (node_id INT PRIMARY KEY, node_name VARCHAR(30));
    ```

    The following output confirms that a table has been created:

    ??? example "Expected output"

        ```{.text .no-copy}
        Query OK, 0 rows affected (0.05 sec)
        ```

4. Insert records on the first node:

    ```{.bash data-prompt="mysql@percona1>"}
    mysql@percona1> INSERT INTO percona.example VALUES (1, 'percona1');
    ```

    The following output confirms that the records have been inserted:

    ??? example "Expected output"

        ```{.text .no-copy}
        Query OK, 1 row affected (0.02 sec)
        ```

5. Retrieve all the rows from that table on the second node:

    ```{.bash data-prompt="mysql@percona2>"}
    mysql@percona2> SELECT * FROM percona.example;
    ```
    
    The following output confirms that all the rows have been retrieved:

    ??? example "Expected output"

        ```{.text .no-copy}
        +---------+-----------+
        | node_id | node_name |
        +---------+-----------+
        |       1 | percona1  |
        +---------+-----------+
        1 row in set (0.00 sec)
        ```

    This simple procedure should ensure that all nodes in the cluster
    are synchronized and working as intended.

