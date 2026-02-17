# Frequently asked questions

## How do I report bugs?

All bugs can be reported on [JIRA :octicons-link-external-16:](https://jira.percona.com/projects/PXC/issues).
Please submit `error.log` files from **all** the nodes.

## How do I solve locking issues like auto-increment?

For auto-increment, Percona XtraDB Cluster changes `auto_increment_offset` for each new node.
In a single-node workload, locking is handled in the same way as *InnoDB*.

In case of write load on several nodes, Percona XtraDB Cluster uses [optimistic locking :octicons-link-external-16:](https://en.wikipedia.org/wiki/Optimistic_concurrency_control) and the application may receive lock error in response to `COMMIT` query.

## What if a node crashes and InnoDB recovery rolls back some transactions?

When a node crashes, after restarting, the node will copy the whole dataset from another node whenever there were changes to data since the crash.

## How can I check the Galera node health?

To check the health of a Galera node, use the following query:

```sql
SELECT 1 FROM dual;
```

The following results of the previous query are possible:

* One row with value 1: the node is healthy.

* Unknown error: the node is online but Galera is not connected or synced with the cluster.

* Connection error: the node is not online.

You can also check a node’s health with the `clustercheck` script.
First set up the `clustercheck` user:

```sql
CREATE USER 'clustercheck'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

??? example "Expected output"

    ```{.text .no-copy}
    Query OK, 0 rows affected (0.00 sec)
    ```

```sql
GRANT PROCESS ON *.* TO 'clustercheck'@'localhost';
```

??? example "Expected output"

    ```{.text .no-copy}
    Query OK, 0 rows affected (0.00 sec)
    ```

You can then check a node’s health by running the `clustercheck` script:

```shell
/usr/bin/clustercheck clustercheck password 0
```

If the node is running, you should get the following status:

```{.text .no-copy}
HTTP/1.1 200 OK
Content-Type: text/plain
Connection: close
Content-Length: 40
Percona XtraDB Cluster Node is synced.
```

In case the node is not synced or is offline, status will look like:

```{.text .no-copy}
HTTP/1.1 503 Service Unavailable
Content-Type: text/plain
Connection: close
Content-Length: 44
Percona XtraDB Cluster Node is not synced.
```

!!! note

    The `clustercheck` script has the following syntax:

    `<user> <pass> <available_when_donor=0|1> <log_file> <available_when_readonly=0|1> <defaults_extra_file>`

    Recommended: `server_args = user pass 1 /var/log/log-file 0 /etc/my.cnf.local`

    Compatibility: `server_args = user pass 1 /var/log/log-file 1 /etc/my.cnf.local`

## How does Percona XtraDB Cluster handle big transactions?

Percona XtraDB Cluster populates write set in memory before replication, and this sets the limit for the size of transactions that make sense. There are wsrep variables for maximum row count and maximum size of write set to make sure that the server does not run out of memory.

## Different table structures on the nodes

For example, if there are four nodes, with four tables: `sessions_a`, `sessions_b`, `sessions_c`, and `sessions_d`, and you want each table in a separate node, this is not possible for InnoDB tables. However, that approach works for MEMORY tables.

## What if a node fails or there is a network issue between nodes?

The quorum mechanism in Percona XtraDB Cluster will decide which nodes can accept traffic and will shut down the nodes that do not belong to the quorum. Later when the failure is fixed, the nodes will need to copy data from the working cluster.

The algorithm for quorum is Dynamic Linear Voting, or DLV. The quorum is preserved only when the sum weight of the nodes in a new component strictly exceeds half that of the preceding Primary Component, minus the nodes which left gracefully.

The mechanism is described in detail in [MariaDB Galera documentation :octicons-link-external-16:](https://mariadb.com/docs/galera-cluster/).

## How would the quorum mechanism handle split brain?

Split brain occurs when parts of the cluster are disconnected and each part believes the other is no longer running; this can lead to data inconsistency. The quorum mechanism cannot resolve split brain. If there is no way to decide on the primary component, Percona XtraDB Cluster has no way to resolve a [split brain](glossary.md#split-brain) situation. The minimal recommendation is to have three nodes. However, you can allow a node to handle traffic with the following option:

```{.text .no-copy}
wsrep_provider_options="pc.ignore_sb = yes"
```

## A node stops accepting commands when the other node fails in a 2-node setup

This is expected behavior to prevent [split brain](glossary.md#split-brain). For more information, see the previous section or [MariaDB Galera documentation :octicons-link-external-16:](https://mariadb.com/docs/galera-cluster/).

## Setting up a cluster without state transfer

You can set up a cluster without state transfer in two ways:

1. By default, Galera reads starting position from a text file `<datadir>/grastate.dat`. Make this file identical on all nodes, and there will be no state transfer after starting a node.

2. Use the [`wsrep_start_position`](wsrep-system-index.md#wsrep_start_position) variable to start the nodes with the same `UUID:seqno` value.

## What TCP ports are used by Percona XtraDB Cluster?

You may need to open up to four ports if you are using a firewall. Add the options below to your server configuration file (for example, `my.cnf` or `my.ini`) in the `[mysqld]` section.

1. **Regular MySQL port.** The default is 3306. You can change it with the `port` option.

2. **Port for group communication.** The default is 4567. To use a different port, add the following to the `[mysqld]` section of your configuration file:

    ```{.text .no-copy}
    wsrep_provider_options ="gmcast.listen_addr=tcp://0.0.0.0:4010; "
    ```

3. **Port for State Snapshot Transfer.** The default is 4444. To use a different address or port, add the following to the `[mysqld]` section:

    ```{.text .no-copy}
    wsrep_sst_receive_address=10.11.12.205:5555
    ```

4. **Port for Incremental State Transfer.** The default is the group communication port plus 1, or 4568. To use a different address or port, add the following to the `[mysqld]` section:

    ```{.text .no-copy}
    wsrep_provider_options = "ist.recv_addr=10.11.12.206:7777; "
    ```

## Is there “async” mode or only “sync” commits are supported?

Percona XtraDB Cluster does not support “async” mode, all commits are synchronous on all nodes.

To be precise, the commits are “virtually” synchronous, which means that the transaction should pass *certification* on nodes, not physical commit. Certification means a guarantee that the transaction does not have conflicts with other transactions on the corresponding node.

## Does Percona XtraDB Cluster work with regular MySQL replication?

Yes. On the node you are going to use as source, enable binary logging and replica updates: set `log_bin` and `log_replica_updates` (or the deprecated `log_slave_updates`) in your configuration.

## Why the MySQL init script does not start?

On systemd-based systems, use `systemctl start mysql` or `systemctl start mysqld` instead of the init script. If the service does not start, SELinux can block it. Prefer configuring SELinux to allow MySQL (for example, setting the correct file context or booleans) rather than disabling it. Only as a last resort for troubleshooting, you can temporarily set SELinux to permissive:

```shell
setenforce 0
```

## What does “nc: invalid option – ‘d’” in the sst.err log file mean?

This error can occur on Debian and Ubuntu when SST uses netcat (`transferfmt=nc`) and the system provides a netcat that does not support the `-d` option. In Percona XtraDB Cluster 8.4 the default transfer format is `socat`, not netcat, so you typically see this only if you set `transferfmt=nc`. To fix it, install the `netcat-openbsd` package and ensure it is used for `nc`, or switch to the default `socat` transfer format. See bug [PXC-941 :octicons-link-external-16:](https://jira.percona.com/browse/PXC-941).
