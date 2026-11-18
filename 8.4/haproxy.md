> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-xtradb-cluster/8.4/llms.txt).

# Load balancing with HAProxy

This guide describes how to configure HAProxy to work with Percona XtraDB Cluster.

Start by installing HAProxy on a [node](https://docs.percona.com/percona-xtradb-cluster/8.4/glossary.html#node) intended for load balancing. Operating systems that support Percona XtraDB Cluster provide the `haproxy` package, and package managers install the package.

**Debian or Ubuntu**

Install HAProxy from the default APT repositories:

```
sudo apt update
sudo apt install haproxy
```

**Red Hat Enterprise Linux**

Install HAProxy from the default DNF repositories:

```
sudo dnf update
sudo dnf install haproxy
```

On environments that use YUM, use equivalent `yum` commands.

**Supported versions of HAProxy**

Use the HAProxy version provided by your supported operating system repositories.

To start HAProxy, use the `haproxy` command. You may pass any number of configuration parameters on the command line. To use a configuration file, use the `-f` option.

```
# Passing one configuration file
sudo haproxy -f <HAPROXY_CFG_PATH>
# Passing multiple configuration files
sudo haproxy -f <HAPROXY_CFG_PATH> <HAPROXY_EXTRA_CFG_PATH>
# Passing a directory
sudo haproxy -f <HAPROXY_CFG_DIRECTORY>
```

You can pass the name of an existing configuration file or a directory. HAProxy includes all files with the *.cfg* extension in the the supplied directory. Another way to pass multiple files is to use `-f` multiple times.

Replace placeholder values such as `<NODE1_IP>`, `<NODE2_IP>`, `<NODE3_IP>`, `<HAPROXY_CFG_PATH>`, and `<HEALTHCHECK_PORT>` with values from your environment.

Options set in the configuration file

| HAProxy option (with links to HAProxy documentation)                                   | Description                                                                                                                   |
| -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| global                                                                                 | A section in the configuration file for process-wide parameters                                                               |
| defaults                                                                               | A section in the configuration file for default parameters for all other following sections                                   |
| listen                                                                                 | A section in the configuration file that defines a complete proxy with its frontend and backend parts combined in one section |
| [balance](https://docs.haproxy.org/3.0/configuration.html#4-balance)                   | Load balancing algorithm to be used in a backend                                                                              |
| [timeout client](https://docs.haproxy.org/3.0/configuration.html#4-timeout%20client)   | Set the maximum inactivity time on the client side                                                                            |
| [timeout connect](https://docs.haproxy.org/3.0/configuration.html#4-timeout%20connect) | Set the maximum time to wait for a connection attempt to a server to succeed.                                                 |
| [daemon](https://docs.haproxy.org/3.0/configuration.html#daemon)                       | Makes the process fork into background (recommended mode of operation)                                                        |
| [gid](https://docs.haproxy.org/3.0/configuration.html#3.1-gid)                         | Changes the process’ group ID to <number>                                                                                     |
| [log](https://docs.haproxy.org/3.0/configuration.html#3.1-log)                         | Adds a global syslog server                                                                                                   |
| [maxconn](https://docs.haproxy.org/3.0/configuration.html#3.2-maxconn)                 | Sets the maximum per-process number of concurrent connections to <number>                                                     |
| [mode](https://docs.haproxy.org/3.0/configuration.html#4-mode)                         | Set the running mode or protocol of the instance                                                                              |
| option dontlognull                                                                     | Disable logging of null connections                                                                                           |
| [option tcplog](https://docs.haproxy.org/3.0/configuration.html#4.2-option%20tcplog)   | Enable advanced logging of TCP connections with session state and timers                                                      |
| [redispatch](https://docs.haproxy.org/3.0/configuration.html#4.2-redispatch)           | Enable or disable session redistribution in case of connection failure                                                        |
| [retries](https://docs.haproxy.org/3.0/configuration.html#4.2-retries)                 | Set the number of retries to perform on a server after a connection failure                                                   |
| [server](https://docs.haproxy.org/3.0/configuration.html#4.2-retries)                  | Declare a server in a backend                                                                                                 |
| [timeout server](https://docs.haproxy.org/3.0/configuration.html#4-timeout%20server)   | Set the maximum inactivity time on the server side                                                                            |
| [uid](https://docs.haproxy.org/3.0/configuration.html#3.1-uid)                         | Changes the process’ user ID to <number>                                                                                      |

With the following legacy configuration, HAProxy balances the load between three nodes. The health check only verifies that `mysqld` listens on port 3306, but the health check does not account for node state. As a result, HAProxy can send queries to a node with `mysqld` running even when the node state is `JOINING` or `DISCONNECTED`. To check the current status of a node we need a more complex check. The following approach uses `clustercheck` with HAProxy `httpchk` health checks.

For Percona XtraDB Cluster 8.4 deployments, configure `clustercheck` on each node through a `systemd` socket/service workflow that listens on port `<HEALTHCHECK_PORT>`.

The legacy `xinetd`-based workflow with `/etc/xinetd.d/mysqlchk` remains available for environments that still use `xinetd`.

For the legacy `xinetd` workflow, add the following service entry on each node:

```
mysqlchk        9200/tcp                # mysqlchk
```

## Configure HAProxy with `clustercheck` health checks (recommended)

Example of the HAProxy configuration file

```
global
        log 127.0.0.1   local0
        log 127.0.0.1   local1 notice
        maxconn 4096
        uid 99
        gid 99
        #daemon
        debug
        #quiet
defaults
        log     global
        mode    http
        option  tcplog
        option  dontlognull
        retries 3
        redispatch
        maxconn 2000
        timeout connect 5000
        timeout client  50000
        timeout server  50000
listen mysql-cluster 0.0.0.0:<PXC_SERVICE_PORT>
    mode tcp
    balance roundrobin
    option  httpchk
    server db01 <NODE1_IP>:<PXC_SERVICE_PORT> check port <HEALTHCHECK_PORT> inter 12000 rise 3 fall 3
    server db02 <NODE2_IP>:<PXC_SERVICE_PORT> check port <HEALTHCHECK_PORT> inter 12000 rise 3 fall 3
    server db03 <NODE3_IP>:<PXC_SERVICE_PORT> check port <HEALTHCHECK_PORT> inter 12000 rise 3 fall 3
```

For the `option httpchk` configuration, HAProxy checks node health through the `clustercheck` endpoint on port `<HEALTHCHECK_PORT>`. The `clustercheck` health check path does not require HAProxy to authenticate to MySQL with a user authentication plugin.

## Verify HAProxy configuration

1. Validate the HAProxy configuration syntax:

```
haproxy -c -f <HAPROXY_CFG_PATH>
```

Expected output

```
Configuration file is valid
```

1. Restart HAProxy and confirm service state:

```
sudo systemctl restart haproxy
sudo systemctl is-active haproxy
```

Expected output

```
active
```

1. Verify that HAProxy listens on port `<PXC_SERVICE_PORT>`:

```
ss -ltn | rg ':<PXC_SERVICE_PORT>'
```

Expected output

```
LISTEN 0      128      0.0.0.0:<PXC_SERVICE_PORT>      0.0.0.0:*
```

1. Verify the node health endpoint on each cluster node:

```
curl -sS -i http://<NODE1_IP>:<HEALTHCHECK_PORT>/
```

Expected output

```
HTTP/1.1 200 OK
Content-Type: text/plain
```

1. Run an end-to-end query through HAProxy:

```
mysql -h <HAPROXY_IP> -P <PXC_SERVICE_PORT> -u <APP_USER> -p -e "SELECT @@hostname;"
```

Expected output

```
+------------+
| @@hostname |
+------------+
| <NODE_NAME>|
+------------+
```

## Legacy HAProxy health check example (TCP-only)

The following example uses a basic TCP/MySQL check. The following example does not verify node state and can route traffic to nodes that are not ready for application traffic.

Legacy HAProxy configuration example

```
global
        log 127.0.0.1   local0
        log 127.0.0.1   local1 notice
        maxconn 4096
        uid 99
        gid 99
        daemon
        #debug
        #quiet
defaults
        log     global
        mode    http
        option  tcplog
        option  dontlognull
        retries 3
        redispatch
        maxconn 2000
        timeout connect 5000
        timeout client  50000
        timeout server  50000
listen mysql-cluster 0.0.0.0:<PXC_SERVICE_PORT>
    mode tcp
    balance roundrobin
    option mysql-check user root
    server db01 <NODE1_IP>:<PXC_SERVICE_PORT> check
    server db02 <NODE2_IP>:<PXC_SERVICE_PORT> check
    server db03 <NODE3_IP>:<PXC_SERVICE_PORT> check
```

See also

- [Configure a cluster on Red Hat-based distributions](https://docs.percona.com/percona-xtradb-cluster/8.4/configure-cluster-rhel.html)
- [Configure a cluster on Debian or Ubuntu](https://docs.percona.com/percona-xtradb-cluster/8.4/configure-cluster-ubuntu.html)
- [High availability](https://docs.percona.com/percona-xtradb-cluster/8.4/high-availability.html)
- [Load balancing with ProxySQL](https://docs.percona.com/percona-xtradb-cluster/8.4/load-balance-proxysql.html)
- [Manual monitoring](https://docs.percona.com/percona-xtradb-cluster/8.4/monitoring.html#manual-monitoring)
- [Alerting](https://docs.percona.com/percona-xtradb-cluster/8.4/monitoring.html#alerting)
- [Failover](https://docs.percona.com/percona-xtradb-cluster/8.4/failover.html)
- [Start the second node](https://docs.percona.com/percona-xtradb-cluster/8.4/add-node.html#start-the-second-node)
- [Starting the Third Node](https://docs.percona.com/percona-xtradb-cluster/8.4/add-node.html#starting-the-third-node)
- [Verify replication](https://docs.percona.com/percona-xtradb-cluster/8.4/verify-replication.html)
- [Bootstrapping the First Node](https://docs.percona.com/percona-xtradb-cluster/8.4/bootstrap.html#bootstrap-the-first-node)
- [HAProxy Documentation](https://www.haproxy.org/#docs)
- [MySQL Documentation: CREATE USER statement](https://dev.mysql.com/doc/refman/8.4/en/create-user.html)
