# Set up a testing environment with ProxySQL

This section describes how to set up Percona XtraDB Cluster in a virtualized testing environment
based on ProxySQL. To test the cluster, we will use the sysbench benchmark
tool.

It is assumed that each PXC node is installed on Amazon EC2 micro instances
running CentOS 7. However, the information in this section should apply if you
used another virtualization technology (for example, VirtualBox) with any Linux
distribution.

Each of the tree Percona XtraDB Cluster nodes is installed on a separate virtual machine. One
more virtual machine has ProxySQL, which redirects requests to the nodes.

!!! admonition "Tip"

    Running ProxySQL on an application server, instead of having it as a dedicated entity, removes the unnecessary extra network roundtrip, because the load balancing layer in Percona XtraDB Cluster scales well with application servers.

1.  Install Percona XtraDB Cluster on three cluster nodes, as described in [Configuring Percona XtraDB Cluster on CentOS](configure-cluster-rhel.md#centos-howto).

2.  On the client node, install [ProxySQL](load-balance-proxysql.md#load-balancing-with-proxysql) and `sysbench`:

    ```{.bash data-prompt="$"}
    $ yum -y install proxysql2 sysbench
    ```

3.  When all cluster nodes are started, configure ProxySQL using the admin
    interface.

        !!! admonition "Tip"

            To connect to the ProxySQL admin interface, you need a ``mysql`` client.
            You can either connect to the admin interface from Percona XtraDB Cluster nodes
            that already have the ``mysql`` client installed (Node 1, Node 2, Node 3)
            or install the client on Node 4 and connect locally.

        To connect to the admin interface, use the credentials, host name and port
        specified in the [global variables](https://github.com/sysown/proxysql/blob/master/doc/global_variables.md).

        !!! warning

            Do not use default credentials in production!

        The following example shows how to connect to the ProxySQL admin interface
        with default credentials (assuming that ProxySQL IP is 192.168.70.74):

        ```{.bash data-prompt="root@proxysql:~#"}
        root@proxysql:~# mysql -u admin -padmin -h 127.0.0.1 -P 6032
        ```

        ??? example "Expected output"

            ```{.text .no-copy}
            Welcome to the MySQL monitor.  Commands end with ; or \g.
            Your MySQL connection id is 2
            Server version: 5.5.30 (ProxySQL Admin Module)

            Copyright (c) 2009-2020 Percona LLC and/or its affiliates
            Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

            Oracle is a registered trademark of Oracle Corporation and/or its
            affiliates. Other names may be trademarks of their respective
            owners.

            Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

            mysql>
            ```

        To see the ProxySQL databases and tables use the `SHOW DATABASES` and
        `SHOW TABLES` commands:

        ```{.bash data-prompt="mysql>"}
        mysql> SHOW DATABASES;
        ```

        The following output shows the list of the ProxySQL databases:

        ??? example "Expected output"

            ```{.text .no-copy}
            +-----+---------------+-------------------------------------+
            | seq | name          | file                                |
            +-----+---------------+-------------------------------------+
            | 0   | main          |                                     |
            | 2   | disk          | /var/lib/proxysql/proxysql.db       |
            | 3   | stats         |                                     |
            | 4   | monitor       |                                     |
            | 5   | stats_monitor | /var/lib/proxysql/proxysql_stats.db |
            +-----+---------------+-------------------------------------+
            5 rows in set (0.00 sec)
            ```

        ```{.bash data-prompt="mysql>"}
        mysql> SHOW TABLES;
        ```

        The following output shows the list of tables:

        ??? example "Expected output"

            ```{.text .no-copy}
            +----------------------------------------------------+
            | tables                                             |
            +----------------------------------------------------+
            | global_variables                                   |
            | mysql_aws_aurora_hostgroups                        |
            | mysql_collations                                   |
            | mysql_firewall_whitelist_rules                     |
            | mysql_firewall_whitelist_sqli_fingerprints         |
            | mysql_firewall_whitelist_users                     |
            | mysql_galera_hostgroups                            |
            | mysql_group_replication_hostgroups                 |
            | mysql_query_rules                                  |
            | mysql_query_rules_fast_routing                     |
            | mysql_replication_hostgroups                       |
            | mysql_servers                                      |
            | mysql_users                                        |
            | proxysql_servers                                   |
            | restapi_routes                                     |
            | runtime_checksums_values                           |
            | runtime_global_variables                           |
            | runtime_mysql_aws_aurora_hostgroups                |
            | runtime_mysql_firewall_whitelist_rules             |
            | runtime_mysql_firewall_whitelist_sqli_fingerprints |
            | runtime_mysql_firewall_whitelist_users             |
            | runtime_mysql_galera_hostgroups                    |
            | runtime_mysql_group_replication_hostgroups         |
            | runtime_mysql_query_rules                          |
            | runtime_mysql_query_rules_fast_routing             |
            | runtime_mysql_replication_hostgroups               |
            | runtime_mysql_servers                              |
            | runtime_mysql_users                                |
            | runtime_proxysql_servers                           |
            | runtime_restapi_routes                             |
            | runtime_scheduler                                  |
            | scheduler                                          |
            +----------------------------------------------------+
            32 rows in set (0.00 sec)
            ```

        For more information about admin databases and tables, see [Admin Tables](https://github.com/sysown/proxysql/blob/master/doc/admin_tables.md)

        !!! note

            ProxySQL has 3 areas where the configuration can reside:

            * MEMORY (your current working place)

            * RUNTIME (the production settings)

            * DISK (durable configuration, saved inside an SQLITE database)

            When you change a parameter, you change it in MEMORY area.
            That is done by design to allow you to test the changes
            before pushing to production (RUNTIME), or saving them to disk.

### Adding cluster nodes to ProxySQL

To configure the backend Percona XtraDB Cluster nodes in ProxySQL, insert corresponding
records into the mysql_servers table.

```sql
INSERT INTO mysql_servers (hostname,hostgroup_id,port,weight) VALUES ('192.168.70.71',10,3306,1000);
INSERT INTO mysql_servers (hostname,hostgroup_id,port,weight) VALUES ('192.168.70.72',10,3306,1000);
INSERT INTO mysql_servers (hostname,hostgroup_id,port,weight) VALUES ('192.168.70.73',10,3306,1000);
```

ProxySQL v2.0 supports PXC natlively. It uses the concept of _hostgroups_
(see the value of hostgroup_id in the mysql_servers table) to group cluster
nodes to balance the load in a cluster by routing different types of traffic to
different groups.

This information is stored in the [runtime_]mysql_galera_hostgroups table.

**Columns of the `[runtime_]mysql_galera_hostgroups` table**

| Column name             | Description                                                                                                                                                                                         |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| writer_hostgroup:       | The ID of the hostgroup that refers to the WRITER node                                                                                                                                              |
| backup_writer_hostgroup | The ID of the hostgroup that contains candidate WRITER servers                                                                                                                                      |
| reader_hostgroup        | The ID of the hostgroup that contains candidate READER servers                                                                                                                                      |
| offline_hostgroup       | The ID of the hostgroup that will eventually contain the WRITER node that will be put OFFLINE                                                                                                       |
| active                  | `1` (Yes) to inidicate that this configuration should be used; `0` (No) - otherwise                                                                                                                 |
| max_writers             | The maximum number of WRITER nodes that must operate simultaneously. For most cases, a reasonable value is `1`. The value in this column may not exceed the total number of nodes.                  |
| writer_is_also_reader   | `1` (Yes) to keep the given node in both `reader_hostgroup` and `writer_hostgroup`. `0` (No) to remove the given node from `reader_hostgroup` if it already belongs to `writer_hostgroup`.          |
| max_transactions_behind | As soon as the value of :variable:`wsrep_local_recv_queue` exceeds the number stored in this column the given node is set to `OFFLINE`. Set the value carefully based on the behaviour of the node. |
| comment                 | Helpful extra information about the given node                                                                                                                                                      |

Make sure that the variable mysql-server_version refers to the correct
version. For Percona XtraDB Cluster {{vers}}, set it to {{vers}} accordingly:

```{.bash data-prompt="mysql>"}
mysql> UPDATE GLOBAL_VARIABLES
SET variable_value='8.0'
WHERE variable_name='mysql-server_version';

mysql> LOAD MYSQL SERVERS TO RUNTIME;
mysql> SAVE MYSQL SERVERS TO DISK;
```

!!! admonition "See also"

    Percona Blogpost: ProxySQL Native Support for Percona XtraDB Cluster (PXC) https://www.percona.com/blog/2019/02/20/proxysql-native-support-for-percona-xtradb-cluster-pxc/

Given the nodes from the mysql_servers table, you may set up the hostgroups as
follows:

```{.bash data-prompt="mysql>"}
mysql> INSERT INTO mysql_galera_hostgroups (
writer_hostgroup, backup_writer_hostgroup, reader_hostgroup,
offline_hostgroup, active, max_writers, writer_is_also_reader,
max_transactions_behind)
VALUES (10, 12, 11, 13, 1, 1, 2, 100);
```

This command configures ProxySQL as follows:

WRITER hostgroup

    hostgroup `10`

READER hostgroup

    hostgroup `11`

BACKUP WRITER hostgroup

    hostgroup `12`

OFFLINE hostgroup

    hostgroup `13`

Set up ProxySQL query rules for read/write split using the mysql_query_rules
table:

```{.bash data-prompt="mysql>"}
mysql> INSERT INTO mysql_query_rules (
username,destination_hostgroup,active,match_digest,apply)
VALUES ('appuser',10,1,'^SELECT.*FOR UPDATE',1);

mysql> INSERT INTO mysql_query_rules (
username,destination_hostgroup,active,match_digest,apply)
VALUES ('appuser',11,1,'^SELECT ',1);

mysql> LOAD MYSQL QUERY RULES TO RUNTIME;
mysql> SAVE MYSQL QUERY RULES TO DISK;

mysql> select hostgroup_id,hostname,port,status,weight from runtime_mysql_servers;
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------+----------------+------+--------+--------+
    | hostgroup_id | hostname       | port | status | weight |
    +--------------+----------------+------+--------+--------+
    | 10           | 192.168.70.73 | 3306  | ONLINE | 1000   |
    | 11           | 192.168.70.72 | 3306  | ONLINE | 1000   |
    | 11           | 192.168.70.71 | 3306  | ONLINE | 1000   |
    | 12           | 192.168.70.72 | 3306  | ONLINE | 1000   |
    | 12           | 192.168.70.71 | 3306  | ONLINE | 1000   |
    +--------------+----------------+------+--------+--------+
    5 rows in set (0.00 sec)
    ```

!!! admonition "See also"

    ProxySQL Blog: MySQL read/write split with ProxySQL
      https://proxysql.com/blog/configure-read-write-split/
    ProxySQL Documentation: `mysql_query_rules` table
      https://github.com/sysown/proxysql/wiki/Main-(runtime)#mysql_query_rules

### ProxySQL failover behavior

Notice that all servers were inserted into the mysql_servers table with the
READER hostgroup set to 10 (see the value of the hostgroup_id column):

```{.bash data-prompt="mysql>"}
mysql> SELECT * FROM mysql_servers;
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------+---------------+------+--------+     +---------+
    | hostgroup_id | hostname      | port | weight | ... | comment |
    +--------------+---------------+------+--------+     +---------+
    | 10           | 192.168.70.71 | 3306 | 1000   |     |         |
    | 10           | 192.168.70.72 | 3306 | 1000   |     |         |
    | 10           | 192.168.70.73 | 3306 | 1000   |     |         |
    +--------------+---------------+------+--------+     +---------+
    3 rows in set (0.00 sec)
    ```

This configuration implies that ProxySQL elects the writer automatically. If
the elected writer goes offline, ProxySQL assigns another (failover). You
might tweak this mechanism by assigning a higher weight to a selected
node. ProxySQL directs all write requests to this node. However, it also
becomes the mostly utilized node for reading requests. In case of a failback (a
node is put back online), the node with the highest weight is automatically
elected for write requests.

<!-- seealso: :ref:`proxysql.automatic-failover` -->

### Creating a ProxySQL monitoring user

To enable monitoring of Percona XtraDB Cluster nodes in ProxySQL, create a user with `USAGE`
privilege on any node in the cluster and configure the user in ProxySQL.

The following example shows how to add a monitoring user on Node 2 if you are using the deprecated `mysql_native_password` authentication method:

```{.bash data-prompt="mysql>"}
mysql> CREATE USER 'proxysql'@'%' IDENTIFIED WITH mysql_native_password BY 'ProxySQLPa55';
```

The following example adds a monitoring user on Node 2 if you are using the `caching_sha2_password` authentication method:

```{.bash data-prompt="mysql>"}
mysql> CREATE USER 'proxysql'@'%' \
        IDENTIFIED WITH caching_sha2_password \
        BY 'ProxySQLPa55';
```

For either authentication method, run the following command to give the user account named 'proxysql' permission to connect to any database and perform basic actions like checking if the database is read-only. This privilege is often used for tools that need to monitor or interact with a MySQL server.

````{.bash data-prompt="mysql>"}
mysql> GRANT USAGE ON *.* TO 'proxysql'@'%';
```

The following example shows how to configure this user on the ProxySQL node:

```{.bash data-prompt="mysql>"}
mysql> UPDATE global_variables SET variable_value='proxysql'
WHERE variable_name='mysql-monitor_username';

mysql> UPDATE global_variables SET variable_value='ProxySQLPa55'
WHERE variable_name='mysql-monitor_password';
````

### Saving and loading the configuration

To load this configuration at runtime, issue the `LOAD` command. To save these
changes to disk (ensuring that they persist after ProxySQL shuts down), issue
the `SAVE` command.

```{.bash data-prompt="mysql>"}
mysql> LOAD MYSQL VARIABLES TO RUNTIME;
mysql> SAVE MYSQL VARIABLES TO DISK;
```

To ensure that monitoring is enabled, check the monitoring logs:

```{.bash data-prompt="mysql>"}
mysql> SELECT * FROM monitor.mysql_server_connect_log ORDER BY time_start_us DESC LIMIT 6;
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------+------+------------------+----------------------+---------------+
    | hostname      | port | time_start_us    | connect_success_time | connect_error |
    +---------------+------+------------------+----------------------+---------------+
    | 192.168.70.71 | 3306 | 1469635762434625 | 1695                 | NULL          |
    | 192.168.70.72 | 3306 | 1469635762434625 | 1779                 | NULL          |
    | 192.168.70.73 | 3306 | 1469635762434625 | 1627                 | NULL          |
    | 192.168.70.71 | 3306 | 1469635642434517 | 1557                 | NULL          |
    | 192.168.70.72 | 3306 | 1469635642434517 | 2737                 | NULL          |
    | 192.168.70.73 | 3306 | 1469635642434517 | 1447                 | NULL          |
    +---------------+------+------------------+----------------------+---------------+
    6 rows in set (0.00 sec)
    ```

```{.bash data-prompt="mysql>"}
mysql> SELECT * FROM monitor.mysql_server_ping_log ORDER BY time_start_us DESC LIMIT 6;
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------+------+------------------+-------------------+------------+
    | hostname      | port | time_start_us    | ping_success_time | ping_error |
    +---------------+------+------------------+-------------------+------------+
    | 192.168.70.71 | 3306 | 1469635762416190 | 948               | NULL       |
    | 192.168.70.72 | 3306 | 1469635762416190 | 803               | NULL       |
    | 192.168.70.73 | 3306 | 1469635762416190 | 711               | NULL       |
    | 192.168.70.71 | 3306 | 1469635702416062 | 783               | NULL       |
    | 192.168.70.72 | 3306 | 1469635702416062 | 631               | NULL       |
    | 192.168.70.73 | 3306 | 1469635702416062 | 542               | NULL       |
    +---------------+------+------------------+-------------------+------------+
    6 rows in set (0.00 sec)
    ```

The previous examples show that ProxySQL is able to connect and ping the nodes
you added.

To enable monitoring of these nodes, load them at runtime:

```{.bash data-prompt="mysql>"}
mysql> LOAD MYSQL SERVERS TO RUNTIME;
```

### Creating ProxySQL Client User

ProxySQL must have users that can access backend nodes to manage connections.

To add a user, insert credentials into `mysql_users` table:

```{.bash data-prompt="mysql>"}
mysql> INSERT INTO mysql_users (username,password) VALUES ('appuser','$3kRetp@$sW0rd');
```

The example of the output is the following:

??? example "Expected output"

    ```{.text .no-copy}
    Query OK, 1 row affected (0.00 sec)
    ```

!!! note

    ProxySQL currently doesn’t encrypt passwords.

    !!! admonition "See also"

        [More information about password encryption in ProxySQL](https://github.com/sysown/proxysql/wiki/MySQL-{{vers}})

Load the user into runtime space and save these changes to disk (ensuring that
they persist after ProxySQL shuts down):

```{.bash data-prompt="mysql>"}
mysql> LOAD MYSQL USERS TO RUNTIME;
mysql> SAVE MYSQL USERS TO DISK;
```

To confirm that the user has been set up correctly, you can try to log in:

```{.bash data-prompt="root@proxysql:~#"}
root@proxysql:~# mysql -u appuser -p$3kRetp@$sW0rd -h 127.0.0.1 -P 6033
```

??? example "Expected output"

    ```{.text .no-copy}
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 1491
    Server version: 5.5.30 (ProxySQL)

    Copyright (c) 2009-2020 Percona LLC and/or its affiliates
    Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

    Oracle is a registered trademark of Oracle Corporation and/or its
    affiliates. Other names may be trademarks of their respective
    owners.

    Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
    ```

The following example adds an `appuser` user account, if you are using the deprecated `mysql_native_password` authentication method:

```{.bash data-prompt="mysql>"}
mysql> CREATE USER 'appuser'@'192.168.70.74'
IDENTIFIED WITH mysql_native_password by '$3kRetp@$sW0rd';
```

The following example adds an `appuser` user account if you are using the `caching_sha2_password` authentication method:

```{.bash data-prompt="mysql>"}
mysql> CREATE USER 'appuser'@'192.168.70.74' \
        IDENTIFIED WITH caching_sha2_password \
        BY '$3kRetp@$sW0rd';
```

The following example command grants the `appuser` account all privileges on all databases and tables.

```{.bash data-prompt="mysql>"}
mysql> GRANT ALL ON *.* TO 'appuser'@'192.168.70.74';
```

## Testing the cluster with the sysbench benchmark tool

After you set up Percona XtraDB Cluster in your testing environment, you can test it using
the `sysbench` benchmarking tool.

1.  Create a database (sysbenchdb in this example; you can use a
    different name):

        ```{.bash data-prompt="mysql>"}
        mysql> CREATE DATABASE sysbenchdb;
        ```

        The following output confirms that a new database has been created:

        ??? example "Expected output"

            ```{.text .no-copy}
            Query OK, 1 row affected (0.01 sec)
            ```

2.  Populate the table with data for the benchmark. Note that you
    should pass the database you have created as the value of the
    `--mysql-db` parameter, and the name of the user who has full
    access to this database as the value of the `--mysql-user`
    parameter:

    ```{.bash data-prompt="$"}
    $ sysbench /usr/share/sysbench/oltp_insert.lua --mysql-db=sysbenchdb \
    --mysql-host=127.0.0.1 --mysql-port=6033 --mysql-user=appuser \
    --mysql-password=$3kRetp@$sW0rd --db-driver=mysql --threads=10 --tables=10 \
    --table-size=1000 prepare
    ```

3.  Run the benchmark on port 6033:

    ```{.bash data-prompt="$"}
    $ sysbench /usr/share/sysbench/oltp_read_write.lua --mysql-db=sysbenchdb \
    --mysql-host=127.0.0.1 --mysql-port=6033 --mysql-user=appuser \
    --mysql-password=$3kRetp@$sW0rd --db-driver=mysql --threads=10 --tables=10 \
    --skip-trx=true --table-size=1000 --time=100 --report-interval=10 run
    ```

---

**Related sections and additional reading**

- [Load balancing with ProxySQL](load-balance-proxysql.md#load-balancing-with-proxysql)
- [Configuring Percona XtraDB Cluster on CentOS](configure-cluster-rhel.md#centos-howto)
- [Percona Blog post: ProxySQL Native Support for Percona XtraDB Cluster (PXC)](https://www.percona.com/blog/2019/02/20/proxysql-native-support-for-percona-xtradb-cluster-pxc/)
- [GitHub repository for the sysbench benchmarking tool](https://github.com/akopytov/sysbench/)
