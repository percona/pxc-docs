# How to set up a three-node cluster on a single box

This tutorial describes how to set up a 3-node cluster
on a single physical box.

For the purposes of this tutorial, assume the following:

* The local IP address is `192.168.2.21`.

* Percona XtraDB Cluster is extracted from binary tarball into
`/usr/local/Percona-XtraDB-Cluster-8.0.x86_64`

To set up the cluster:

1. Create three MySQL configuration files for the corresponding nodes:

    * `/etc/my.4000.cnf`

    ```{.text .no-copy}
    [mysqld]
    port = 4000
    socket=/tmp/mysql.4000.sock
    datadir=/data/bench/d1
    basedir=/usr/local/Percona-XtraDB-Cluster-8.0.x86_64
    user=mysql
    log_error=error.log
    binlog_format=ROW
    wsrep_cluster_address='gcomm://192.168.2.21:5030,192.168.2.21:6030'
    wsrep_provider=/usr/local/Percona-XtraDB-Cluster-8.0.x86_64/lib/libgalera_smm.so
    wsrep_sst_receive_address=192.168.2.21:4020
    wsrep_node_incoming_address=192.168.2.21
    wsrep_cluster_name=trimethylxanthine
    wsrep_provider_options = "gmcast.listen_addr=tcp://192.168.2.21:4030;"
    wsrep_sst_method=xtrabackup-v2
    wsrep_node_name=node4000
    innodb_autoinc_lock_mode=2
    ```

    * `/etc/my.5000.cnf`

    ```{.text .no-copy}
    [mysqld]
    port = 5000
    socket=/tmp/mysql.5000.sock
    datadir=/data/bench/d2
    basedir=/usr/local/Percona-XtraDB-Cluster-8.0.x86_64
    user=mysql
    log_error=error.log
    binlog_format=ROW
    wsrep_cluster_address='gcomm://192.168.2.21:4030,192.168.2.21:6030'
    wsrep_provider=/usr/local/Percona-XtraDB-Cluster-8.0.x86_64/lib/libgalera_smm.so
    wsrep_sst_receive_address=192.168.2.21:5020
    wsrep_node_incoming_address=192.168.2.21

    wsrep_cluster_name=trimethylxanthine
    wsrep_provider_options = "gmcast.listen_addr=tcp://192.168.2.21:5030;"
    wsrep_sst_method=xtrabackup-v2
    wsrep_node_name=node5000
    innodb_autoinc_lock_mode=2
    ```

    * `/etc/my.6000.cnf`

    ```{.text .no-copy}
    [mysqld]
    port = 6000
    socket=/tmp/mysql.6000.sock
    datadir=/data/bench/d3
    basedir=/usr/local/Percona-XtraDB-Cluster-8.0.x86_64
    user=mysql
    log_error=error.log
    binlog_format=ROW
    wsrep_cluster_address='gcomm://192.168.2.21:4030,192.168.2.21:5030'
    wsrep_provider=/usr/local/Percona-XtraDB-Cluster-8.0.x86_64/lib/libgalera_smm.so
    wsrep_sst_receive_address=192.168.2.21:6020
    wsrep_node_incoming_address=192.168.2.21
    wsrep_cluster_name=trimethylxanthine
    wsrep_provider_options = "gmcast.listen_addr=tcp://192.168.2.21:6030;"
    wsrep_sst_method=xtrabackup-v2
    wsrep_node_name=node6000
    innodb_autoinc_lock_mode=2
    ```

2. Create three data directories for the nodes:

    * `/data/bench/d1`

    * `/data/bench/d2`

    * `/data/bench/d3`

3. Start the first node using the following command (from the Percona XtraDB Cluster install directory):

    ```{.bash data-prompt="$"}
    $ bin/mysqld_safe --defaults-file=/etc/my.4000.cnf --wsrep-new-cluster
    ```

    If the node starts correctly, you should see the following output:

    ??? example "Expected output"

        ```{.text .no-copy}
        111215 19:01:49 [Note] WSREP: Shifting JOINED -> SYNCED (TO: 0)
        111215 19:01:49 [Note] WSREP: New cluster view: global state: 4c286ccc-2792-11e1-0800-94bd91e32efa:0, view# 1: Primary, number of nodes: 1, my index: 0, protocol version 1
        ```

    To check the ports, run the following command:

    ```{.bash data-prompt="$"}
    $ netstat -anp | grep mysqld
    tcp        0      0 192.168.2.21:4030           0.0.0.0:*                   LISTEN      21895/mysqld
    tcp        0      0 0.0.0.0:4000                0.0.0.0:*                   LISTEN      21895/mysqld
    ```

4. Start the second and third nodes:

    ```{.bash data-prompt="$"}
    $ bin/mysqld_safe --defaults-file=/etc/my.5000.cnf
    $ bin/mysqld_safe --defaults-file=/etc/my.6000.cnf
    ```

    If the nodes start and join the cluster successful,
    you should see the following output:

    ```{.text .no-copy}
    111215 19:22:26 [Note] WSREP: Shifting JOINER -> JOINED (TO: 2)
    111215 19:22:26 [Note] WSREP: Shifting JOINED -> SYNCED (TO: 2)
    111215 19:22:26 [Note] WSREP: Synchronized with group, ready for connections
    ```

    To check the cluster size, run the following command:

    ```{.bash data-prompt="$"}
    $ mysql -h127.0.0.1 -P6000 -e "show global status like 'wsrep_cluster_size';"
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        +--------------------+-------+
        | Variable_name      | Value |
        +--------------------+-------+
        | wsrep_cluster_size | 3     |
        +--------------------+-------+
        ```

    After that you can connect to any node and perform queries,
    which will be automatically synchronized with other nodes.
    For example, to create a database on the second node,
    you can run the following command:

    ```{.bash data-prompt="$"}
    $ mysql -h127.0.0.1 -P5000 -e "CREATE DATABASE hello_peter"
    ```
