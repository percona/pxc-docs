# Load balancing with HAProxy

The free and open source software, HAProxy, provides a high-availability load balancer and reverse proxy for TCP and HTTP-based applications. HAProxy can distribute requests across multiple servers, ensuring optimal performance and security.

Here are the benefits of using HAProxy:

 - HAProxy supports layer 4 (TCP) and layer 7 (HTTP) load balancing, which means it can handle different network traffic and protocols. HAProxy requires patched backends to tunnel IP traffic in layer 4 load-balancing tunnel mode. This mode also disables some layer 7 advanced features.

- HAProxy has rich features, such as URL rewriting, SSL/TLS termination, gzip compression, caching, observability, health checks, retries, circuit breakers, WebSocket, HTTP/2 and HTTP/3 support, and more.

- HAProxy has a reputation for being fast and efficient in terms of processor and memory usage. The software is written in C and has an event-driven and multithreaded architecture.

- HAProxy has a user-friendly status page that shows detailed information about the load balancer and the backends. The software also integrates well with third-party monitoring tools and services.

- HAProxy supports session retention and cookie guidance, which can help with sticky sessions and affinity.


## Create a user

Access the server as a user with administrative privileges, either `root` or use sudo.

Create a Dedicated HAProxy user account for HAProxy to interact with your MySQL instance. This account enhances security.

Make the following changes to the example `CREATE USER` command to replace the placeholders:

* Replace haproxy_user with your preferred username.

* Substitute `haproxy_server_ip` with the actual IP address of your HAProxy server.

* Choose a robust password for the 'strong_password'.

Execute the following command:

```{.bash data-prompt="mysql>"}
mysql> CREATE USER 'haproxy_user'@'haproxy_server_ip' IDENTIFIED BY 'strong_password';
```

Grant the minimal set of privileges necessary for HAProxy to perform its health checks and monitoring.

Execute the following:

```{.bash data-prompt="mysql>"}
GRANT SELECT ON `mysql`.* TO 'haproxy_user'@'haproxy_server_ip';
FLUSH PRIVILEGES;
```

### Important Considerations

If your MySQL servers are part of a replication cluster, create the user and grant privileges on each node to ensure consistency.

For enhanced security, consider restricting the `haproxy_user` to specific databases or tables to monitor rather than granting permissions to the entire `mysql` database schema.


## Install

Add the HAProxy Enterprise repository to your system by following the instructions for [your operating system](https://www.haproxy.com/documentation/hapee/latest/getting-started/installation/).

Install HAProxy on the [node](glossary.md#node) you intend to use
for load balancing. You can install it using the package manager.

=== "On a Debian-derived distribution"

    ```{.bash data-prompt="$"}
    $ sudo apt update
    $ sudo apt install haproxy
    ```

=== "On a Red Hat-derived distribution"

    ```{.bash data-prompt="$"}
    $ sudo yum update
    $ sudo yum install haproxy
    ```

To start HAProxy, use the `haproxy` command. You may pass any
number of configuration parameters on the command line. To use a
configuration file, add the `-f` option.

```{.bash data-prompt="$"}
$ # Passing one configuration file
$ sudo haproxy -f haproxy-1.cfg

$ # Passing multiple configuration files
$ sudo haproxy -f haproxy-1.cfg haproxy-2.cfg

$ # Passing a directory
$ sudo haproxy -f conf-dir
```

You can pass the name of an existing configuration file or a
directory. HAProxy includes all files with the *.cfg* extension in the supplied directory. Another way to pass multiple files is to use `-f`
multiple times.

For more information, see [`HAProxy Management Guide`](https://docs.haproxy.org/2.5/management.html)
 
For information, see [HAProxy configuration file](haproxy-config.md)


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

## Uninstall

To uninstall haproxy version 2 from a Linux system, follow the [latest instructions](https://www.haproxy.com/documentation/haproxy-enterprise/getting-started/uninstallation/).
