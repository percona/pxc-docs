# Load balancing with HAProxy

This manual describes how to configure HAProxy to work with Percona XtraDB Cluster.

Start by installing HAProxy on a [node](glossary.md#node) that you intend to use
for load balancing. The operating systems that support Percona XtraDB Cluster provide
the haproxy package and you can install it using the package manager.

Debian or Ubuntu

    ```{.bash data-prompt="$"}
    $ sudo apt update
    $ sudo apt install haproxy
    ```

Red Hat or CentOS:

    ```{.bash data-prompt="$"}
    $ sudo yum update
    $ sudo yum install haproxy
    ```

**Supported versions of HAProxy**

   The lowest supported version of HAProxy is 1.4.20. 

To start HAProxy, use the `haproxy` command. You may pass any
number of configuration parameters on the command line. To use a
configuration file, use the `-f` option.

```{.bash data-prompt="$"}
$ # Passing one configuration file
$ sudo haproxy -f haproxy-1.cfg

$ # Passing multiple configuration files
$ sudo haproxy -f haproxy-1.cfg haproxy-2.cfg

$ # Passing a directory
$ sudo haproxy -f conf-dir
```

You can pass the name of an existing configuration file or a
directory. HAProxy includes all files with the *.cfg* extension in the the
supplied directory. Another way to pass multiple files is to use `-f`
multiple times.

!!! admonition "See also"

    HAProxy Documentation:
     * [`Managing HAProxy (including available options)`](http://cbonte.github.io/haproxy-dconv/2.0/management.html)
     * [`More information about how to configure HAProxy`](http://cbonte.github.io/haproxy-dconv/2.0/configuration.html#2)

??? example "Example of the HAProxy configuration file"

        ```{.text .no-copy}
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
                contimeout      5000
                clitimeout      50000
                srvtimeout      50000

        listen mysql-cluster 0.0.0.0:3306
            mode tcp
            balance roundrobin
            option mysql-check user root

            server db01 10.4.29.100:3306 check
            server db02 10.4.29.99:3306 check
            server db03 10.4.29.98:3306 check

Options set in the configuration file

|HAProxy option (with links to HAProxy documentation)|Description|
| -------------------------------------------------- |-----------|
|global|A section in the configuration file for process-wide parameters|
|defaults|A section in the configuration file for default parameters for all other following sections|
|listen|A section in the configuration file that defines a complete proxy with its frontend and backend parts combined in one section|
|[balance](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4-balance)|Load balancing algorithm to be used in a backend|
|[clitimeout](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4-clitimeout)|Set the maximum inactivity time on the client side|
|[contimeout](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4-contimeout)|Set the maximum time to wait for a connection attempt to a server to succeed.|
|[daemon](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#daemon)| Makes the process fork into background (recommended mode of operation)|
|[gid](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#3.1-gid)|Changes the process' group ID to &#60;number&#62;|
|[log](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#3.1-log)|Adds a global syslog server|
|[maxconn](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#3.2-maxconn)|Sets the maximum per-process number of concurrent connections to &#60;number&#62;|
|[mode](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4-mode)|Set the running mode or protocol of the instance|
|option dontlognull|Disable logging of null connections|
|[option tcplog](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4.2-option%20tcplog)|Enable advanced logging of TCP connections with session state and timers|
|[redispatch](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4.2-redispatch)|Enable or disable session redistribution in case of connection failure|
|[retries](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4.2-retries)|Set the number of retries to perform on a server after a connection failure|
|[server](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4.2-retries)|Declare a server in a backend|
|[srvtimeout](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#4.2-srvtimeout)|Set the maximum inactivity time on the server side|
|[uid](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#3.1-uid)|Changes the process' user ID to &#60;number&#62;|

With this configuration, HAProxy will balance the load between three nodes.
In this case, it only checks if `mysqld` listens on port 3306,
but it doesn’t take into an account the state of the node.
So it could be sending queries to the node that has `mysqld` running
even if it’s in `JOINING` or `DISCONNECTED` state.

To check the current status of a node we need a more complex check.
This idea was taken from [codership-team google groups](https://groups.google.com/group/codership-team/browse_thread/thread/44ee59c8b9c458aa/98b47d41125cfae6).

To implement this setup, you will need two scripts:

* **clustercheck** (located in `/usr/local/bin`) and a config for `xinetd`

* **mysqlchk** (located in `/etc/xinetd.d`) on each node

Both scripts are available in binaries and source distributions of Percona XtraDB Cluster.

Change the `/etc/services` file
by adding the following line on each node:

```text
mysqlchk        9200/tcp                # mysqlchk
```

??? example "Example of the HAProxy configuration file"

        ```{.text .no-copy}
        # this config needs haproxy-1.4.20

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
                contimeout      5000
                clitimeout      50000
                srvtimeout      50000

        listen mysql-cluster 0.0.0.0:3306
            mode tcp
            balance roundrobin
            option  httpchk

            server db01 10.4.29.100:3306 check port 9200 inter 12000 rise 3 fall 3
            server db02 10.4.29.99:3306 check port 9200 inter 12000 rise 3 fall 3
            server db03 10.4.29.98:3306 check port 9200 inter 12000 rise 3 fall 3
        ```

!!! important

    In Percona XtraDB Cluster 8.0, the default authentication plugin is
    ``caching_sha2_password``. HAProxy does not support this authentication
    plugin. Create a mysql user using the ``mysql_native_password``
    authentication plugin.

    ```{.bash data-prompt="mysql>"}
    mysql> CREATE USER 'haproxy_user'@'%' IDENTIFIED WITH mysql_native_password by '$3Kr$t';
    ```

    !!! admonition "See also"

        [MySQL Documentation: CREATE USER statement](https://dev.mysql.com/doc/refman/8.0/en/create-user.html)