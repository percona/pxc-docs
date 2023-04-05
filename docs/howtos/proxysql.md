# Load balancing with ProxySQL

[ProxySQL](https://www.proxysql.com/) is a high-performance SQL proxy.  ProxySQL runs as a daemon watched by
a monitoring process.  The process monitors the daemon and restarts it in case
of a crash to minimize downtime.

The daemon accepts incoming traffic from *MySQL* clients and forwards it to
backend *MySQL* servers.

The proxy is designed to run continuously without needing to be restarted.  Most
configuration can be done at runtime using queries similar to SQL statements.
These include runtime parameters, server grouping, and traffic-related settings.

!!! note

      For more information about ProxySQL, see [ProxySQL documentation](https://proxysql.com/documentation/).

ProxySQL is available from the Percona software repositories in two
versions. ProxySQL v1 does not natively support Percona XtraDB Cluster and requires
custom bash scripts to keep track of the status of Percona XtraDB Cluster nodes using the
ProxySQL scheduler.

ProxySQL v2 natively supports Percona XtraDB Cluster. With this version,
`proxysql-admin` tool does not require custom scripts to keep track of
Percona XtraDB Cluster status.

* [Using ProxySQL v1 with `proxysql-admin`](proxysql-v1.md)

* [Installing ProxySQL v1](proxysql-v1.md#installing-proxysql-v1)

* [Automatic Configuration](proxysql-v1.md#automatic-configuration)

    * [Preparing Configuration File](proxysql-v1.md#preparing-configuration-file)

    * [Enabling ProxySQL](proxysql-v1.md#enabling-proxysql)

    * [Disabling ProxySQL](proxysql-v1.md#disabling-proxysql)

    * [Additional Options](proxysql-v1.md#additional-options)

    * [ProxySQL Status script](proxysql-v1.md#proxysql-status-script)

* [The **proxysql-admin** Tool with ProxySQL 2.0.x](proxysql-v2.md)


## Manual Configuration

This tutorial describes how to configure ProxySQL with three Percona XtraDB Cluster nodes.

| Node    | Host Name | IP address    |
| ------- | --------- | ------------- |
| Node 1  | pxc1      | 192.168.70.61 |
| Node 2  | pxc2      | 192.168.70.62 |
| Node 3  | pxc3      | 192.168.70.63 |
| Node 4  | proxysql  | 192.168.70.64 |

ProxySQL can be configured either using the `/etc/proxysql.cnf` file
or through the admin interface.
Using the admin interface is preferable,
because it allows you to change the configuration dynamically
(without having to restart the proxy).

To connect to the ProxySQL admin interface, you need a `mysql` client.
You can either connect to the admin interface from Percona XtraDB Cluster nodes
that already have the `mysql` client installed (Node 1, Node 2, Node 3)
or install the client on Node 4 and connect locally.
For this tutorial, install Percona XtraDB Cluster on Node 4:

* On Debian or Ubuntu:

```shell
root@proxysql:~# apt install percona-xtradb-cluster-client-5.7
```

* On Red Hat Enterprise Linux or CentOS:

```shell
[root@proxysql ~]# yum install Percona-XtraDB-Cluster-client-57
```

To connect to the admin interface,
use the credentials, host name and port specified in the [global variables](https://github.com/sysown/proxysql/blob/master/doc/global_variables.md).

!!! warning

      Do not use default credentials in production!

The following example shows how to connect to the ProxySQL admin interface
with default credentials:

```shell
root@proxysql:~# mysql -u admin -padmin -h 127.0.0.1 -P 6032
```

The example of the output is the following:

```text
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.1.30 (ProxySQL Admin Module)

Copyright (c) 2009-2016 Percona LLC and/or its affiliates
Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql@proxysql>
```

To see the ProxySQL databases and tables use the following commands:

```sql
mysql@proxysql> SHOW DATABASES;
```

The example of the output is the following:

```text
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

```sql
mysql@proxysql> SHOW TABLES;
```

The example of the output is the following:

```text
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

      ProxySQL has 3 areas where the configuration can reside:
      
      * MEMORY (your current working place)
      
      * RUNTIME (the production settings)
      
      * DISK (durable configuration, saved inside an SQLITE database)
      
      When you change a parameter, you change it in MEMORY area.
      That is done by design to allow you to test the changes
      before pushing to production (RUNTIME), or save them to disk.

### Adding cluster nodes to ProxySQL

To configure the backend Percona XtraDB Cluster nodes in ProxySQL,
insert corresponding records into the `mysql_servers` table.

!!! note

      ProxySQL uses the concept of *hostgroups* to group cluster nodes.
      This enables you to balance the load in a cluster by
      routing different types of traffic to different groups.
      There are many ways you can configure hostgroups
      (for example source and replicas, read and write load, etc.)
      and a every node can be a member of multiple hostgroups.

This example adds three Percona XtraDB Cluster nodes to the default hostgroup (`0`),
which receives both write and read traffic:

```sql
mysql@proxysql> INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (0,'192.168.70.61',3306);
mysql@proxysql> INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (0,'192.168.70.62',3306);
mysql@proxysql> INSERT INTO mysql_servers(hostgroup_id, hostname, port) VALUES (0,'192.168.70.63',3306);
```

To see the nodes:

```sql
mysql@proxysql> SELECT * FROM mysql_servers;
```

The example of the output is the following:

```text
+--------------+---------------+------+--------+--------+-------------+-----------------+---------------------+---------+----------------+---------+
| hostgroup_id | hostname      | port | status | weight | compression | max_connections | max_replication_lag | use_ssl | max_latency_ms | comment |
+--------------+---------------+------+--------+--------+-------------+-----------------+---------------------+---------+----------------+---------+
| 0            | 192.168.70.61 | 3306 | ONLINE | 1      | 0           | 1000            | 0                   | 0       | 0              |         |
| 0            | 192.168.70.62 | 3306 | ONLINE | 1      | 0           | 1000            | 0                   | 0       | 0              |         |
| 0            | 192.168.70.63 | 3306 | ONLINE | 1      | 0           | 1000            | 0                   | 0       | 0              |         |
+--------------+---------------+------+--------+--------+-------------+-----------------+---------------------+---------+----------------+---------+
3 rows in set (0.00 sec)
```

### Creating ProxySQL Monitoring User

To enable monitoring of Percona XtraDB Cluster nodes in ProxySQL,
create a user with `USAGE` privilege on any node in the cluster
and configure the user in ProxySQL.

The following example shows how to add a monitoring user on Node 2:

```sql
mysql@pxc2> CREATE USER 'proxysql'@'%' IDENTIFIED BY 'ProxySQLPa55';
mysql@pxc2> GRANT USAGE ON *.* TO 'proxysql'@'%';
```

The following example shows how to configure this user on the ProxySQL node:

```sql
mysql@proxysql> UPDATE global_variables SET variable_value='proxysql'
              WHERE variable_name='mysql-monitor_username';
mysql@proxysql> UPDATE global_variables SET variable_value='ProxySQLPa55'
              WHERE variable_name='mysql-monitor_password';
```

To load this configuration at runtime, issue a `LOAD` command.
To save these changes to disk
(ensuring that they persist after ProxySQL shuts down),
issue a `SAVE` command.

```sql
mysql@proxysql> LOAD MYSQL VARIABLES TO RUNTIME;
mysql@proxysql> SAVE MYSQL VARIABLES TO DISK;
```

To ensure that monitoring is enabled,
check the monitoring logs:

```sql
mysql@proxysql> SELECT * FROM monitor.mysql_server_connect_log ORDER BY time_start_us DESC LIMIT 6;
```

The example of the output is the following:

```text
+---------------+------+------------------+----------------------+---------------+
| hostname      | port | time_start_us    | connect_success_time | connect_error |
+---------------+------+------------------+----------------------+---------------+
| 192.168.70.61 | 3306 | 1469635762434625 | 1695                 | NULL          |
| 192.168.70.62 | 3306 | 1469635762434625 | 1779                 | NULL          |
| 192.168.70.63 | 3306 | 1469635762434625 | 1627                 | NULL          |
| 192.168.70.61 | 3306 | 1469635642434517 | 1557                 | NULL          |
| 192.168.70.62 | 3306 | 1469635642434517 | 2737                 | NULL          |
| 192.168.70.63 | 3306 | 1469635642434517 | 1447                 | NULL          |
+---------------+------+------------------+----------------------+---------------+
6 rows in set (0.00 sec)
```

```sql
mysql> SELECT * FROM monitor.mysql_server_ping_log ORDER BY time_start_us DESC LIMIT 6;
```

The example of the output is the following:

```text
+---------------+------+------------------+-------------------+------------+
| hostname      | port | time_start_us    | ping_success_time | ping_error |
+---------------+------+------------------+-------------------+------------+
| 192.168.70.61 | 3306 | 1469635762416190 | 948               | NULL       |
| 192.168.70.62 | 3306 | 1469635762416190 | 803               | NULL       |
| 192.168.70.63 | 3306 | 1469635762416190 | 711               | NULL       |
| 192.168.70.61 | 3306 | 1469635702416062 | 783               | NULL       |
| 192.168.70.62 | 3306 | 1469635702416062 | 631               | NULL       |
| 192.168.70.63 | 3306 | 1469635702416062 | 542               | NULL       |
+---------------+------+------------------+-------------------+------------+
6 rows in set (0.00 sec)
```

The previous examples show that ProxySQL is able to connect
and ping the nodes you added.

To enable monitoring of these nodes, load them at runtime:

```sql
mysql@proxysql> LOAD MYSQL SERVERS TO RUNTIME;
```

### Creating ProxySQL Client User

ProxySQL must have users that can access backend nodes
to manage connections.

To add a user, insert credentials into `mysql_users` table:

```sql
mysql@proxysql> INSERT INTO mysql_users (username,password) VALUES ('sbuser','sbpass');
```

The example of the output is the following:

```text
Query OK, 1 row affected (0.00 sec)
```

!!! note

    ProxySQL currently doesn’t encrypt passwords.

Load the user into runtime space and save these changes to disk
(ensuring that they persist after ProxySQL shuts down):

```sql
mysql@proxysql> LOAD MYSQL USERS TO RUNTIME;
mysql@proxysql> SAVE MYSQL USERS TO DISK;
```

To confirm that the user has been set up correctly, you can try to log in:

```sql
root@proxysql:~# mysql -u sbuser -psbpass -h 127.0.0.1 -P 6033
```

The example of the output is the following:

```text
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1491
Server version: 5.1.30 (ProxySQL)

Copyright (c) 2009-2016 Percona LLC and/or its affiliates
Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

To provide read/write access to the cluster for ProxySQL,
add this user on one of the Percona XtraDB Cluster nodes:

```sql
mysql@pxc3> CREATE USER 'sbuser'@'192.168.70.64' IDENTIFIED BY 'sbpass';
```

```text
Query OK, 0 rows affected (0.01 sec)

```sql
mysql@pxc3> GRANT ALL ON *.* TO 'sbuser'@'192.168.70.64';
```

```text
Query OK, 0 rows affected (0.00 sec)
```

### Adding Galera Support in ProxySQL v1

ProxySQL v2 supports monitoring the status Percona XtraDB Cluster nodes. ProxySQL v1 cannot
detect a node which is not in `Synced` state.  To monitor the status of Percona XtraDB Cluster
nodes in ProxySQL v1, use the `proxysql_galera_checker` script.  The
script is located here: `/usr/bin/proxysql_galera_checker`.

To use this script, load it into ProxySQL v1
[Scheduler](https://github.com/sysown/proxysql/blob/master/doc/scheduler.md).

The following example shows how you can load the script
for default ProxySQL v1 configuration:

```text
INSERT INTO scheduler (active,interval_ms,filename,arg1,comment)
VALUES (1,10000,'/usr/bin/proxysql_galera_checker','--config-file=/etc/proxysql-admin.cnf
--write-hg=10 --read-hg=11 --writer-count=1 --mode=singlewrite
--priority=192.168.100.20:3306,192.168.100.40:3306,192.168.100.10:3306,192.168.100.30:3306
--log=/var/lib/proxysql/cluster_one_proxysql_galera_check.log','cluster_one');
```

This scheduler script accepts the following options in the `arg1` argument:

| Option             |  Name                 |  Required | Description                                                                               |
| ------------------ | --------------------- | --------- | ----------------------------------------------------------------------------------------- |
| ``--config-file``  | Configuration File    | Yes       | Specify ``proxysql-admin`` configuration file.                                            |
| ``--write-hg``     | ``HOSTGROUP WRITERS`` | No        | Specify ProxySQL write hostgroup.                                                         |
| ``--read-hg``      | ``HOSTGROUP READERS`` | No        | Specify ProxySQL read hostgroup.                                                          |
| ``--writer-count`` | ``NUMBER WRITERS``    | No        | Specify write nodes count. ``0`` for ``loadbal`` mode and ``1`` for ``singlewrite`` mode. |
| ``--mode``         | ``MODE``              | No        | Specify ProxySQL read/write configuration mode.                                           |
| ``--priority``     | ``WRITER PRIORITY``   | No        | Specify write nodes priority.                                                             |
| ``--log``          | ``LOG FILE``          | No        | Specify ``proxysql_galera_checker`` log file.                                             |

!!! note

    Specify cluster name in comment column.

To load the scheduler changes into the runtime space:

```sql
mysql@proxysql> LOAD SCHEDULER TO RUNTIME;
```

To make sure that the script has been loaded,
check the `runtime_scheduler` table:

```sql
mysql@proxysql> SELECT * FROM scheduler\G;
```
The example of the output is the following:

```text
*************************** 1. row ***************************
         id: 1
     active: 1
interval_ms: 10000
   filename: /bin/proxysql_galera_checker
       arg1: --config-file=/etc/proxysql-admin.cnf --write-hg=10 --read-hg=11
             --writer-count=1 --mode=singlewrite
             --priority=192.168.100.20:3306,192.168.100.40:3306,192.168.100.10:3306,192.168.100.30:3306
             --log=/var/lib/proxysql/cluster_one_proxysql_galera_check.log
       arg2: NULL
       arg3: NULL
       arg4: NULL
       arg5: NULL
    comment: cluster_one
1 row in set (0.00 sec)
```

To check the status of available nodes, run the following command:

```sql
mysql@proxysql> SELECT hostgroup_id,hostname,port,status FROM mysql_servers;
```

The example of the output is the following:

```text
+--------------+---------------+------+--------+
| hostgroup_id | hostname      | port | status |
+--------------+---------------+------+--------+
| 0            | 192.168.70.61 | 3306 | ONLINE |
| 0            | 192.168.70.62 | 3306 | ONLINE |
| 0            | 192.168.70.63 | 3306 | ONLINE |
+--------------+---------------+------+--------+
3 rows in set (0.00 sec)
```

!!! note

    Each node can have the following status:

      * `ONLINE`: backend node is fully operational.

      * `SHUNNED`: backend node is temporarily taken out of use,
      because either too many connection errors hapenned in a short time,
      or replication lag exceeded the allowed threshold.

      * `OFFLINE_SOFT`: new incoming connections aren’t accepted,
      while existing connections are kept until they become inactive.
      In other words, connections are kept in use
      until the current transaction is completed.
      This allows to gracefully detach a backend node.

      * `OFFLINE_HARD`: existing connections are dropped,
      and new incoming connections aren’t accepted.
      This is equivalent to deleting the node from a hostgroup,
      or temporarily taking it out of the hostgroup for maintenance.

### Testing Cluster with sysbench

You can install `sysbench` from Percona software repositories:

* For Debian or Ubuntu:

```shell
root@proxysql:~# apt install sysbench
```

* For Red Hat Enterprise Linux or CentOS

```shell
[root@proxysql ~]# yum install sysbench
```

!!! note

    `sysbench` requires ProxySQL client user credentials
    that you creted in [Creating ProxySQL Client User](#creating-proxysql-client-user).

* Create the database that will be used for testing on one of the Percona XtraDB Cluster nodes:

```sql
mysql@pxc1> CREATE DATABASE sbtest;
```

* Populate the table with data for the benchmark on the ProxySQL node:

```text
root@proxysql:~# sysbench --report-interval=5 --num-threads=4 \
--num-requests=0 --max-time=20 \
--test=/usr/share/doc/sysbench/tests/db/oltp.lua \
--mysql-user='sbuser' --mysql-password='sbpass' \
--oltp-table-size=10000 --mysql-host=127.0.0.1 --mysql-port=6033 \
prepare
```

* Run the benchmark on the ProxySQL node:

```text
root@proxysql:~# sysbench --report-interval=5 --num-threads=4 \
  --num-requests=0 --max-time=20 \
  --test=/usr/share/doc/sysbench/tests/db/oltp.lua \
  --mysql-user='sbuser' --mysql-password='sbpass' \
  --oltp-table-size=10000 --mysql-host=127.0.0.1 --mysql-port=6033 \
  run
```

ProxySQL stores collected data in the `stats` schema:

```sql
mysql@proxysql> SHOW TABLES FROM stats;
```

The example of the output is the following:

```text
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

```sql
mysql@proxysql> SELECT * FROM stats_mysql_commands_counters;
```

The example of the output is the following:

```text
+-------------------+---------------+-----------+-----------+-----------+---------+---------+----------+----------+-----------+-----------+--------+--------+---------+----------+
| Command           | Total_Time_us | Total_cnt | cnt_100us | cnt_500us | cnt_1ms | cnt_5ms | cnt_10ms | cnt_50ms | cnt_100ms | cnt_500ms | cnt_1s | cnt_5s | cnt_10s | cnt_INFs |
+-------------------+---------------+-----------+-----------+-----------+---------+---------+----------+----------+-----------+-----------+--------+--------+---------+----------+
| ALTER_TABLE       | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
| ANALYZE_TABLE     | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
| BEGIN             | 2212625       | 3686      | 55        | 2162      | 899     | 569     | 1        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
| CHANGE_MASTER     | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
| COMMIT            | 21522591      | 3628      | 0         | 0         | 0       | 1765    | 1590     | 272      | 1         | 0         | 0      | 0      | 0       | 0        |
| CREATE_DATABASE   | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
| CREATE_INDEX      | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
...
| DELETE            | 2904130       | 3670      | 35        | 1546      | 1346    | 723     | 19       | 1        | 0         | 0         | 0      | 0      | 0       | 0        |
| DESCRIBE          | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
...
| INSERT            | 19531649      | 3660      | 39        | 1588      | 1292    | 723     | 12       | 2        | 0         | 1         | 0      | 1      | 2       | 0        |
...
| SELECT            | 35049794      | 51605     | 501       | 26180     | 16606   | 8241    | 70       | 3        | 4         | 0         | 0      | 0      | 0       | 0        |
| SELECT_FOR_UPDATE | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
...
| UPDATE            | 6402302       | 7367      | 75        | 2503      | 3020    | 1743    | 23       | 3        | 0         | 0         | 0      | 0      | 0       | 0        |
| USE               | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
| SHOW              | 19691         | 2         | 0         | 0         | 0       | 0       | 1        | 1        | 0         | 0         | 0      | 0      | 0       | 0        |
| UNKNOWN           | 0             | 0         | 0         | 0         | 0       | 0       | 0        | 0        | 0         | 0         | 0      | 0      | 0       | 0        |
+-------------------+---------------+-----------+-----------+-----------+---------+---------+----------+----------+-----------+-----------+--------+--------+---------+----------+
45 rows in set (0.00 sec)
```

### Automatic Fail-over

ProxySQL will automatically detect if a node is not available
or not synced with the cluster.

You can check the status of all available nodes by running:

```sql
mysql@proxysql> SELECT hostgroup_id,hostname,port,status FROM mysql_servers;
```

The example of the output is the following:

```text
+--------------+---------------+------+--------+
| hostgroup_id | hostname      | port | status |
+--------------+---------------+------+--------+
| 0            | 192.168.70.61 | 3306 | ONLINE |
| 0            | 192.168.70.62 | 3306 | ONLINE |
| 0            | 192.168.70.63 | 3306 | ONLINE |
+--------------+---------------+------+--------+
3 rows in set (0.00 sec)
```

To test problem detection and fail-over mechanism, shut down Node 3:

```shell
root@pxc3:~# service mysql stop
```

ProxySQL will detect that the node is down and update its status to
`OFFLINE_SOFT`:

```sql
mysql@proxysql> SELECT hostgroup_id,hostname,port,status FROM mysql_servers;
```

The example of the output is the following:

```text
+--------------+---------------+------+--------------+
| hostgroup_id | hostname      | port | status       |
+--------------+---------------+------+--------------+
| 0            | 192.168.70.61 | 3306 | ONLINE       |
| 0            | 192.168.70.62 | 3306 | ONLINE       |
| 0            | 192.168.70.63 | 3306 | OFFLINE_SOFT |
+--------------+---------------+------+--------------+
3 rows in set (0.00 sec)
```

Now start Node 3 again:

```shell
root@pxc3:~# service mysql start
```

The script will detect the change and mark the node as `ONLINE`:

```sql
mysql@proxysql> SELECT hostgroup_id,hostname,port,status FROM mysql_servers;
```

The example of the output is the following:

```text
+--------------+---------------+------+--------+
| hostgroup_id | hostname      | port | status |
+--------------+---------------+------+--------+
| 0            | 192.168.70.61 | 3306 | ONLINE |
| 0            | 192.168.70.62 | 3306 | ONLINE |
| 0            | 192.168.70.63 | 3306 | ONLINE |
+--------------+---------------+------+--------+
3 rows in set (0.00 sec)
```

## Assisted Maintenance Mode

Usually, to take a node down for maintenance, you need to identify that node,
update its status in ProxySQL to `OFFLINE_SOFT`,
wait for ProxySQL to divert traffic from this node,
and then initiate the shutdown or perform maintenance tasks.
Percona XtraDB Cluster includes a special *maintenance mode* for nodes
that enables you to take a node down without adjusting ProxySQL manually.
This mode is controlled using the [`pxc_maint_mode`](../wsrep-system-index.md#pxc_maint_mode) variable,
which is monitored by ProxySQL and can be set to one of the following values:

* `DISABLED`: This is the default state that tells ProxySQL to route traffic to the node as usual.

* `SHUTDOWN`: This state is set automatically when you initiate node shutdown.

You may need to shut down a node when upgrading the OS, adding resources,
changing hardware parts, relocating the server, etc.

When you initiate node shutdown, Percona XtraDB Cluster does not send the signal immediately.
Instead, it changes the state to `pxc_maint_mode=SHUTDOWN`
and waits for a predefined period,
which is determined by the value of the [`pxc_maint_transition_period`](../wsrep-system-index.md#pxc_maint_transition_period).
After detecting that the maintenance mode is set to `SHUTDOWN`,
ProxySQL changes the status of this node to `OFFLINE_SOFT`,
which stops creating new connections for the node.
After the transition period ends,
any long-running transactions that are still active are aborted.

* `MAINTENANCE`: You can change to this state if you need to perform maintenace on a node without shutting it down.

You may need to isolate the node for some time,
so that it does not receive traffic from ProxySQL
while you resize the buffer pool, truncate the undo log,
defragment, or check disks, etc.

To do this, manually set `pxc_maint_mode=MAINTENANCE`.
Control is not returned to the user for the predefined period set by `pxc-maint_transaction_period`,
10 seconds by default.
ProxySQL marks the node as OFFLINE, and avoids opening new connections for any DML transactions. ProxySQL does not terminate existing connections.

Once control is returned, you can perform maintenance activity. 

!!! note

    Any data changes will still be replicated across the cluster.
 
    After you finish maintenance, set the mode back to `DISABLED`.
    When ProxySQL detects this, it starts routing traffic to the node again.
