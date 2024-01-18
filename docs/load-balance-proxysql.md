# Load balance with ProxySQL

[ProxySQL](http://www.proxysql.com/) is a high-performance SQL proxy. ProxySQL runs as a daemon watched by
a monitoring process. The process monitors the daemon and restarts it in case
of a crash to minimize downtime.

The daemon accepts incoming traffic from MySQL clients and forwards it to
backend MySQL servers.

The proxy is designed to run continuously without needing to be restarted.  Most
configuration can be done at runtime using queries similar to SQL statements in
the ProxySQL admin interface.  These include runtime parameters, server
grouping, and traffic-related settings.

!!! admonition "See also"

    [ProxySQL Documentation](https://proxysql.com/documentation/)

[ProxySQL v2](proxysql-v2.md#pxc-proxysql-v2) natively supports Percona XtraDB Cluster. With this version,
`proxysql-admin` tool does not require any custom scripts to keep track of Percona XtraDB Cluster status.

!!! important

    In version {{vers}}, Percona XtraDB Cluster does not support ProxySQL v1.  

## Manual configuration

This section describes how to configure ProxySQL with three Percona XtraDB Cluster nodes.

| Node| Host Name| IP address|
| ---- | -------- | --------- |
| Node 1| pxc1| 192.168.70.71|
| Node 2| pxc2| 192.168.70.72|
| Node 3| pxc3 | 192.168.70.73|
| Node 4| proxysql| 192.168.70.74|

ProxySQL can be configured either using the `/etc/proxysql.cnf` file or
through the admin interface. The admin interface is recommended because this interface can dynamically change the configuration without restarting the proxy.

To connect to the ProxySQL admin interface, you need a `mysql` client.
You can either connect to the admin interface from Percona XtraDB Cluster nodes
that already have the `mysql` client installed (Node 1, Node 2, Node 3)
or install the client on Node 4 and connect locally.
For this tutorial, install Percona XtraDB Cluster on Node 4:

**Changes in the installation procedure**

In Percona XtraDB Cluster {{vers}}, ProxySQL is not installed automatically as a dependency of the ``percona-xtradb-cluster-client-8.0`` package. You should install the ``proxysql`` package separately.

!!! note 

    ProxySQL has multiple versions in the version 2 series.

* On Debian or Ubuntu for ProxySQL 2.x:

```{.bash data-prompt="root@proxysql:~#"}
root@proxysql:~# apt install percona-xtradb-cluster-client
root@proxysql:~# apt install proxysql2
```

* On Red Hat Enterprise Linux or CentOS for ProxySQL 2.x:

```{.bash data-prompt="$"}
$ sudo yum install Percona-XtraDB-Cluster-client-80
$ sudo yum install proxysql2
```

To connect to the admin interface, use the credentials, host name and port specified in the [global variables](https://github.com/sysown/proxysql/blob/master/doc/global_variables.md).

!!! warning

    Do not use default credentials in production!

The following example shows how to connect to the ProxySQL admin interface
with default credentials:

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

    mysql@proxysql>
    ```

To see the ProxySQL databases and tables use the following commands:

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> SHOW DATABASES;
```

The following output shows the ProxySQL databases:

??? example "Expected output"

    ```{.text .no-copy}
    +-----+---------+-------------------------------+
    | seq | name    | file                          |
    +-----+---------+-------------------------------+
    | 0   | main    |                               |
    | 2   | disk    | /var/lib/proxysql/proxysql.db |
    | 3   | stats   |                               |
    | 4   | monitor |                               |
    +-----+---------+-------------------------------+
    4 rows in set (0.00 sec)
    ```

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> SHOW TABLES;
```

The following output shows the ProxySQL tables:

??? example "Expected output"

    ```{.text .no-copy}
    +--------------------------------------+
    | tables                               |
    +--------------------------------------+
    | global_variables                     |
    | mysql_collations                     |
    | mysql_query_rules                    |
    | mysql_replication_hostgroups         |
    | mysql_servers                        |
    | mysql_users                          |
    | runtime_global_variables             |
    | runtime_mysql_query_rules            |
    | runtime_mysql_replication_hostgroups |
    | runtime_mysql_servers                |
    | runtime_scheduler                    |
    | scheduler                            |
    +--------------------------------------+
    12 rows in set (0.00 sec)
    ```

For more information about admin databases and tables,
see [Admin Tables](https://github.com/sysown/proxysql/blob/master/doc/admin_tables.md)

!!! note 
    
    The ProxySQL configuration can reside in the following areas:


    * MEMORY (your current working place)


    * RUNTIME (the production settings)


    * DISK (durable configuration, saved inside an SQLITE database)

    When you change a parameter, you change it in MEMORY area.
    This ability is by design and lets you test the changes
    before pushing the change to production (RUNTIME), or save the change to disk.

### Add cluster nodes to ProxySQL

To configure the backend Percona XtraDB Cluster nodes in ProxySQL,
insert corresponding records into the `mysql_servers` table.

!!! note 
    
    ProxySQL uses the concept of *hostgroups* to group cluster nodes.
    This enables you to balance the load in a cluster by
    routing different types of traffic to different groups.
    There are many ways you can configure hostgroups
    (for example, source and replicas, read and write load, etc.)
    and a every node can be a member of multiple hostgroups.

This example adds three Percona XtraDB Cluster nodes to the default hostgroup (`0`),
which receives both write and read traffic:

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (0,'192.168.70.71',3306);
mysql@proxysql> INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (0,'192.168.70.72',3306);
mysql@proxysql> INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (0,'192.168.70.73',3306);
```

To see the nodes:

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> SELECT * FROM mysql_servers;
```

The following output shows the list of nodes:

??? example "Expected output"

    ```{.text .no-copy}
    +--------------+---------------+------+--------+--------+-------------+-----------------+---------------------+---------+----------------+---------+
    | hostgroup_id | hostname      | port | status | weight | compression | max_connections | max_replication_lag | use_ssl | max_latency_ms | comment |
    +--------------+---------------+------+--------+--------+-------------+-----------------+---------------------+---------+----------------+---------+
    | 0            | 192.168.70.71 | 3306 | ONLINE | 1      | 0           | 1000            | 0                   | 0       | 0              |         |
    | 0            | 192.168.70.72 | 3306 | ONLINE | 1      | 0           | 1000            | 0                   | 0       | 0              |         |
    | 0            | 192.168.70.73 | 3306 | ONLINE | 1      | 0           | 1000            | 0                   | 0       | 0              |         |
    +--------------+---------------+------+--------+--------+-------------+-----------------+---------------------+---------+----------------+---------+
    3 rows in set (0.00 sec)
    ```

### Create ProxySQL monitoring user

To enable monitoring of Percona XtraDB Cluster nodes in ProxySQL,
create a user with `USAGE` privilege on any node in the cluster
and configure the user in ProxySQL.

The following example shows how to add a monitoring user on Node 2:

```{.bash data-prompt="mysql@pxc2>"}
mysql@pxc2> CREATE USER 'proxysql'@'%' IDENTIFIED WITH mysql_native_password by '$3Kr$t';
mysql@pxc2> GRANT USAGE ON *.* TO 'proxysql'@'%';
```

The following example shows how to configure this user on the ProxySQL node:

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> UPDATE global_variables SET variable_value='proxysql'
              WHERE variable_name='mysql-monitor_username';
mysql@proxysql> UPDATE global_variables SET variable_value='ProxySQLPa55'
              WHERE variable_name='mysql-monitor_password';
```

To load this configuration at runtime, issue a `LOAD` command.
To save these changes to disk
(ensuring that they persist after ProxySQL shuts down),
issue a `SAVE` command.

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> LOAD MYSQL VARIABLES TO RUNTIME;
mysql@proxysql> SAVE MYSQL VARIABLES TO DISK;
```

To ensure that monitoring is enabled, check the monitoring logs:

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> SELECT * FROM monitor.mysql_server_connect_log ORDER BY time_start_us DESC LIMIT 6;
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

The previous examples show that ProxySQL is able to connect
and ping the nodes you have added.

To enable monitoring of these nodes, load them at runtime:

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> LOAD MYSQL SERVERS TO RUNTIME;
```

### Create ProxySQL client user

ProxySQL must have users that can access backend nodes
to manage connections.

To add a user, insert credentials into `mysql_users` table:

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> INSERT INTO mysql_users (username,password) VALUES ('sbuser','sbpass');
```

??? example "Expected output"

    ```{.text .no-copy}
    Query OK, 1 row affected (0.00 sec)
    ```

!!! note

    ProxySQL currently doesn’t encrypt passwords.

Load the user into runtime space and save these changes to disk
(ensuring that they persist after ProxySQL shuts down):

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> LOAD MYSQL USERS TO RUNTIME;
mysql@proxysql> SAVE MYSQL USERS TO DISK;
```

To confirm that the user has been set up correctly, you can try to log in as root:

```{.bash data-prompt="root@proxysql:~#"}
root@proxysql:~# mysql -u sbuser -psbpass -h 127.0.0.1 -P 6033
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

To provide read/write access to the cluster for ProxySQL,
add this user on one of the Percona XtraDB Cluster nodes:

```{.bash data-prompt="mysql@pxc3>"}
mysql@pxc3> CREATE USER 'sbuser'@'192.168.70.74' IDENTIFIED BY 'sbpass';
```

??? example "Expected output"

    ```{.text .no-copy}
    Query OK, 0 rows affected (0.01 sec)
    ```

```{.bash data-prompt="mysql@pxc3>"}
mysql@pxc3> GRANT ALL ON *.* TO 'sbuser'@'192.168.70.74';
```

??? example "Expected output"

    ```{.text .no-copy}
    Query OK, 0 rows affected (0.00 sec)
    ```

### Test cluster with sysbench

You can install `sysbench` from Percona software repositories:

* For Debian or Ubuntu:

```{.bash data-prompt="root@proxysql:~#"}
root@proxysql:~# apt install sysbench
```

* For Red Hat Enterprise Linux or CentOS

```{.bash data-prompt="root@proxysql:~#"}
root@proxysql:~# yum install sysbench
```

!!! note

    `sysbench` requires ProxySQL client user credentials that you created in Creating ProxySQL Client User.

1. Create the database that will be used for testing on one of the Percona XtraDB Cluster nodes:

    ```{.bash data-prompt="mysql@pxc1>"}
    mysql@pxc1> CREATE DATABASE sbtest;
    ```

2. Populate the table with data for the benchmark on the ProxySQL node:

    ```{.bash data-prompt="root@proxysql:~#"}
    root@proxysql:~# sysbench --report-interval=5 --num-threads=4 \
    --num-requests=0 --max-time=20 \
    --test=/usr/share/doc/sysbench/tests/db/oltp.lua \
    --mysql-user='sbuser' --mysql-password='sbpass' \
    --oltp-table-size=10000 --mysql-host=127.0.0.1 --mysql-port=6033 \
    prepare
    ```

3. Run the benchmark on the ProxySQL node:

    ```{.bash data-prompt="root@proxysql:~#"}
    root@proxysql:~# sysbench --report-interval=5 --num-threads=4 \
    --num-requests=0 --max-time=20 \
    --test=/usr/share/doc/sysbench/tests/db/oltp.lua \
    --mysql-user='sbuser' --mysql-password='sbpass' \
    --oltp-table-size=10000 --mysql-host=127.0.0.1 --mysql-port=6033 \
    run
    ```

ProxySQL stores collected data in the `stats` schema:

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> SHOW TABLES FROM stats;
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------------------------+
    | tables                         |
    +--------------------------------+
    | stats_mysql_query_rules        |
    | stats_mysql_commands_counters  |
    | stats_mysql_processlist        |
    | stats_mysql_connection_pool    |
    | stats_mysql_query_digest       |
    | stats_mysql_query_digest_reset |
    | stats_mysql_global             |
    +--------------------------------+
    ```

For example, to see the number of commands that run on the cluster:

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> SELECT * FROM stats_mysql_commands_counters;
```

??? example "Expected output"

    ```{.text .no-copy}
    +---------------------------+---------------+-----------+-----------+-----------+---------+---------+----------+----------+-----------+-----------+--------+--------+---------+----------+
    | Command                   | Total_Time_us | Total_cnt | cnt_100us | cnt_500us | cnt_1ms | cnt_5ms | cnt_10ms | cnt_50ms | cnt_100ms | cnt_500ms | cnt_1s | cnt_5s | cnt_10s | cnt_INFs |
    +---------------------------+---------------+-----------+-----------+-----------+---------+---------+----------+----------+-----------+-----------+--------+--------+---------+----------+
    | ALTER_TABLE               | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
    | ANALYZE_TABLE             | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
    | BEGIN                     | 2212625       | 3686      | 55        | 2162      | 899     | 569     | 1        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
    | CHANGE_REPLICATION_SOURCE | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
    | COMMIT                    | 21522591      | 3628      | 0         | 0         | 0       | 1765    | 1590     | 272      | 1         | 0         | 0      | 0      | 0       | 0        |
    | CREATE_DATABASE           | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
    | CREATE_INDEX              | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
    ...
    | DELETE                    | 2904130       | 3670      | 35        | 1546      | 1346    | 723     | 19       | 1        | 0         | 0         | 0      | 0      | 0       | 0        |
    | DESCRIBE                  | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
    ...
    | INSERT                    | 19531649      | 3660      | 39        | 1588      | 1292    | 723     | 12       | 2        | 0         | 1         | 0      | 1      | 2       | 0        |
    ...
    | SELECT                    | 35049794      | 51605     | 501       | 26180     | 16606   | 8241    | 70       | 3        | 4         | 0         | 0      | 0      | 0       | 0        |
    | SELECT_FOR_UPDATE         | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
    ...
    | UPDATE                    | 6402302       | 7367      | 75        | 2503      | 3020    | 1743    | 23       | 3        | 0         | 0         | 0      | 0      | 0       | 0        |
    | USE                       | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
    | SHOW                      | 19691         | 2         | 0         | 0         | 0       | 0       | 1        | 1        | 0         | 0         | 0      | 0      | 0       | 0        |
    | UNKNOWN                   | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
    +---------------------------+---------------+-----------+-----------+-----------+---------+---------+----------+----------+-----------+-----------+--------+--------+---------+----------+
    45 rows in set (0.00 sec)
    ```

### Automatic failover

ProxySQL will automatically detect if a node is not available
or not synced with the cluster.

You can check the status of all available nodes by running:

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> SELECT hostgroup_id,hostname,port,status FROM runtime_mysql_servers;
```

The following output shows the status of all available nodes:

??? example "Expected output"

    ```{.text .no-copy}
    +--------------+---------------+------+--------+
    | hostgroup_id | hostname      | port | status |
    +--------------+---------------+------+--------+
    | 0            | 192.168.70.71 | 3306 | ONLINE |
    | 0            | 192.168.70.72 | 3306 | ONLINE |
    | 0            | 192.168.70.73 | 3306 | ONLINE |
    +--------------+---------------+------+--------+
    3 rows in set (0.00 sec)
    ```

To test problem detection and fail-over mechanism, shut down Node 3:

```{.bash data-prompt="root@pxc3:~#"}
root@pxc3:~# service mysql stop
```

ProxySQL will detect that the node is down and update its status to
`OFFLINE_SOFT`:

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> SELECT hostgroup_id,hostname,port,status FROM runtime_mysql_servers;
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------+---------------+------+--------------+
    | hostgroup_id | hostname      | port | status       |
    +--------------+---------------+------+--------------+
    | 0            | 192.168.70.71 | 3306 | ONLINE       |
    | 0            | 192.168.70.72 | 3306 | ONLINE       |
    | 0            | 192.168.70.73 | 3306 | OFFLINE_SOFT |
    +--------------+---------------+------+--------------+
    3 rows in set (0.00 sec)
    ```

Now start Node 3 again:

```{.bash data-prompt="root@pxc3:~#"}
root@pxc3:~# service mysql start
```

The script will detect the change and mark the node as
`ONLINE`:

```{.bash data-prompt="mysql@proxysql>"}
mysql@proxysql> SELECT hostgroup_id,hostname,port,status FROM runtime_mysql_servers;
```

??? example "Expected output"

    ```{.text .no-copy}
    +--------------+---------------+------+--------+
    | hostgroup_id | hostname      | port | status |
    +--------------+---------------+------+--------+
    | 0            | 192.168.70.71 | 3306 | ONLINE |
    | 0            | 192.168.70.72 | 3306 | ONLINE |
    | 0            | 192.168.70.73 | 3306 | ONLINE |
    +--------------+---------------+------+--------+
    3 rows in set (0.00 sec)
    ```

## Assisted maintenance mode

Usually, to take a node down for maintenance, you need to identify that node,
update its status in ProxySQL to `OFFLINE_SOFT`,
wait for ProxySQL to divert traffic from this node,
and then initiate the shutdown or perform maintenance tasks.
Percona XtraDB Cluster includes a special maintenance mode for nodes
that enables you to take a node down without adjusting ProxySQL manually.

Initiating `pxc_maint_mode=MAINTENANCE` does not disconnect existing connections. You must terminate these connections by either running your application code or forcing a re-connection. With a re-connection, the new connections are re-routed around the PXC node in `MAINTENANCE` mode.

Assisted maintenance mode is controlled via the `pxc_maint_mode` variable,
which is monitored by ProxySQL and can be set to one of the following values:

* `DISABLED`: This value is the default state
that tells ProxySQL to route traffic to the node as usual.

* `SHUTDOWN`: This state is set automatically when you initiate node shutdown.

    You may need to shut down a node when upgrading the OS, adding resources,
    changing hardware parts, relocating the server, etc.

    When you initiate node shutdown, Percona XtraDB Cluster does not send the signal immediately.
    Intead, it changes the state to `pxc_maint_mode=SHUTDOWN`
    and waits for a predefined period (10 seconds by default).
    When ProxySQL detects that the mode is set to `SHUTDOWN`,
    it changes the status of this node to `OFFLINE_SOFT`. This status stops creating new node connections.
    After the transition period, long-running active transactions are aborted.

* `MAINTENANCE`: You can change to this state
if you need to perform maintenance on a node without shutting it down.

    You may need to isolate the node for a specific time
    so that it does not receive traffic from ProxySQL
    while you resize the buffer pool, truncate the undo log,
    defragment, or check disks, etc.

    To do this, manually set `pxc_maint_mode=MAINTENANCE`.
    Control is not returned to the user for a predefined period
    (10 seconds by default). You can increase the transition period
    using the `pxc_maint_transition_period` variable
    to accommodate long-running transactions.
    If the period is long enough for all transactions to finish,
    there should be little disruption in the cluster workload. If you increase 
    the transition period, the packaging script may determine the wait as a server stall.

    When ProxySQL detects that the mode is set to `MAINTENANCE`,
    it stops routing traffic to the node. During the transition period,
    any existing connections continue, but ProxySQL avoids opening new connections and starting transactions.
    Still, the user can open connections to monitor status.
    
    Once control is returned, you can perform maintenance activity.

    !!! note

        Data changes continue to be replicated across the cluster.

    After you finish maintenance, set the mode back to `DISABLED`.
    When ProxySQL detects this, it starts routing traffic to the node again.

**Related sections**

[Setting up a testing environment with ProxySQL](virtual-sandbox.md#testing-env-proxysql-setting-up)
