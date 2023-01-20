# Configure Percona XtraDB Cluster on CentOS

This tutorial describes how to install and configure three Percona XtraDB Cluster nodes
on CentOS 7 servers, using the packages from Percona repositories.

* Node 1

    * Host name: `percona1`

    * IP address: `192.168.70.71`

* Node 2

    * Host name: `percona2`

    * IP address: `192.168.70.72`

* Node 3

    * Host name: `percona3`

    * IP address: `192.168.70.73`

## Prerequisites

The procedure described in this tutorial requires the following:

* All three nodes have CentOS 7 installed.

* The firewall on all nodes is configured to allow connecting
to ports 3306, 4444, 4567 and 4568.

* SELinux on all nodes is disabled.

**Different from previous version**

   The variable `wsrep_sst_auth` has been removed. Percona XtraDB Cluster 8.0 automatically creates the system user [`mysql.pxc.internal.session`](../glossary.md#mysqlpxcinternalsession). During [SST](../glossary.md#sst), the user `mysql.pxc.sst.user` and the role [`mysql.pxc.sst.role`](../glossary.md#mysqlpxcsstrole) are created on the donor node.

## Step 1. Installing PXC

Install Percona XtraDB Cluster on all three nodes as described in [Installing Percona XtraDB Cluster on Red Hat Enterprise Linux and CentOS](../install/yum.md#yum).

## Step 2. Configuring the first node

Individual nodes should be configured to be able to bootstrap the cluster.
For more information about bootstrapping the cluster, see [Bootstrapping the First Node](../bootstrap.md#bootstrap).

1. Make sure that the configuration file `/etc/my.cnf`
on the first node (`percona1`) contains the following:

    ```{.text .no-copy}
    [mysqld]

    datadir=/var/lib/mysql
    user=mysql

    # Path to Galera library
    wsrep_provider=/usr/lib64/galera4/libgalera_smm.so

    # Cluster connection URL contains the IPs of node#1, node#2 and node#3
    wsrep_cluster_address=gcomm://192.168.70.71,192.168.70.72,192.168.70.73

    # In order for Galera to work correctly binlog format should be ROW
    binlog_format=ROW

    # Using the MyISAM storage engine is not recommended.
    default_storage_engine=InnoDB

    # This InnoDB autoincrement locking mode is a requirement for Galera
    innodb_autoinc_lock_mode=2

    # Node 1 address
    wsrep_node_address=192.168.70.71

    # SST method
    wsrep_sst_method=xtrabackup-v2

    # Cluster name
    wsrep_cluster_name=my_centos_cluster
    ```

2. Start the first node with the following command:

    ```{.bash data-prompt="[root@percona1 ~] #"}
    [root@percona1 ~] # systemctl start mysql@bootstrap.service
    ```

    The previous command will start the cluster
    with initial [`wsrep_cluster_address`](../wsrep-system-index.md#wsrep_cluster_address) variable set to `gcomm://`. If the node or MySQL are restarted later, there will be no need to change the configuration file.

3. After the first node has been started, cluster status can be checked with the following command:

    ```{.bash data-prompt="mysql>"}
    mysql> show status like 'wsrep%';
    ```
    
    This output shows that the cluster has been successfully bootstrapped.

    ??? example "Expected output"

        ```{.text .no-copy}
        +----------------------------+--------------------------------------+
        | Variable_name              | Value                                |
        +----------------------------+--------------------------------------+
        | wsrep_local_state_uuid     | c2883338-834d-11e2-0800-03c9c68e41ec |
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

    Copy the automatically generated temporary password for the superuser account:

    ```{.bash data-prompt="$"}
    $ sudo grep 'temporary password' /var/log/mysqld.log
    ```

    Use this password to log in as root:

    ```{.bash data-prompt="$"}
    $ mysql -u root -p
    ```

    Change the password for the superuser account and log out. For example:

    ```{.bash data-prompt="mysql>"}
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'r00tP@$$';
    ```
   
    ??? example "Expected output"

        ```{.text .no-copy}
        Query OK, 0 rows affected (0.00 sec)
        ```

## Step 3. Configuring the second node

1. Make sure that the configuration file `/etc/my.cnf` on the second node (`percona2`) contains the following:

    ```{.text .no-copy}
    [mysqld]

    datadir=/var/lib/mysql
    user=mysql

    # Path to Galera library
    wsrep_provider=/usr/lib64/galera4/libgalera_smm.so

    # Cluster connection URL contains IPs of node#1, node#2 and node#3
    wsrep_cluster_address=gcomm://192.168.70.71,192.168.70.72,192.168.70.73

    # In order for Galera to work correctly binlog format should be ROW
    binlog_format=ROW

    # Using the MyISAM storage engine is not recommended
    default_storage_engine=InnoDB

    # This InnoDB auto_increment locking mode is a requirement for Galera
    innodb_autoinc_lock_mode=2

    # Node 2 address
    wsrep_node_address=192.168.70.72

    # Cluster name
    wsrep_cluster_name=my_centos_cluster

    # SST method
    wsrep_sst_method=xtrabackup-v2
    ```

2. Start the second node with the following command:

    ```{.bash data-prompt="[root@percona2 ~] #"}
    [root@percona2 ~]# systemctl start mysql
    ```

3. After the server has been started, it should receive SST automatically.
    Cluster status can be checked on both nodes.
    The following is an example of status from the second node (`percona2`):

    ```{.bash data-prompt="mysql>"}
    mysql> show status like 'wsrep%';
    ```

    The output shows that the new node has been successfully added to the cluster.

    ??? example "Expected output"

        ```{.text .no-copy}
        +----------------------------+--------------------------------------+
        | Variable_name              | Value                                |
        +----------------------------+--------------------------------------+
        | wsrep_local_state_uuid     | c2883338-834d-11e2-0800-03c9c68e41ec |
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

## Step 4. Configuring the third node

1. Make sure that the MySQL configuration file `/etc/my.cnf` on the third node (`percona3`) contains the following:

    ```{.text .no-copy}
    [mysqld]

    datadir=/var/lib/mysql
    user=mysql

    # Path to Galera library
    wsrep_provider=/usr/lib64/galera4/libgalera_smm.so

    # Cluster connection URL contains IPs of node#1, node#2 and node#3
    wsrep_cluster_address=gcomm://192.168.70.71,192.168.70.72,192.168.70.73

    # In order for Galera to work correctly binlog format should be ROW
    binlog_format=ROW

    # Using the MyISAM storage engine is not recommended
    default_storage_engine=InnoDB

    # This InnoDB auto_increment locking mode is a requirement for Galera
    innodb_autoinc_lock_mode=2

    # Node #3 address
    wsrep_node_address=192.168.70.73

    # Cluster name
    wsrep_cluster_name=my_centos_cluster

    # SST method
    wsrep_sst_method=xtrabackup-v2
    ```

2. Start the third node with the following command:

    ```{.bash data-prompt="[root@percona3 ~] #"}
    [root@percona3 ~]# systemctl start mysql
    ```

3. After the server has been started, it should receive SST automatically.
    Cluster status can be checked on all three nodes.
    The following is an example of status from the third node (`percona3`):

    ```{.bash data-prompt="mysql>"}
    mysql> show status like 'wsrep%';
    ```

    The output confirms that the third node has joined the cluster.
    
    ??? example "Expected output"

        ```{.text .no-copy}
        +----------------------------+--------------------------------------+
        | Variable_name              | Value                                |
        +----------------------------+--------------------------------------+
        | wsrep_local_state_uuid     | c2883338-834d-11e2-0800-03c9c68e41ec |
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

## Testing replication

To test replication, lets create a new database on second node,
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

    ```{.bash data-prompt="mysql@percona2"}
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


