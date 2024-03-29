# HAProxy configuration file

## Example of HAProxy v1 configuration file

??? example "HAProxy v1 configuration file"

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
            timeout connect 160000
            timeout client 240000
            timeout server 240000

    listen mysql-cluster 0.0.0.0:3306
        mode tcp
        balance roundrobin
        option mysql-check user root

        server db01 10.4.29.100:3306 check
        server db02 10.4.29.99:3306 check
        server db03 10.4.29.98:3306 check
    ```

Options set in the configuration file

## Differences between version 1 configuration file and version 2 configuration file

### Version Declaration:

v1: The configuration file typically omits an explicit version declaration.

v2: You must explicitly declare the version using the version keyword followed by the specific version number (e.g., version = 2.0).

### Global Parameters:

v1 and v2: Both versions utilize a global section to define global parameters, but certain parameters might have different names or functionalities across versions. Refer to the official documentation for specific changes.

### Configuration Blocks:

v1 and v2: Both versions use a similar indentation-based structure to define configuration blocks like frontend and backend. However, v2 introduces new blocks and keywords not present in v1 (e.g., process, http-errors).

### Directives:

v1 and v2: While many directives remain consistent, some might have renamed keywords, altered syntax, or entirely new functionalities in v2. Consult the official documentation for a comprehensive comparison of directives and their usage between versions.

### Comments:

v1 and v2: Both versions support comments using the # symbol. However, v2 introduces multi-line comments using /* ... */ syntax, which v1 does not support.

## Version 2 configuration file

This simplified example is for load balancing. HAProxy offers numerous features for advanced configurations and fine-tuning.

This example demonstrates a basic HAProxy v2 configuration file for load-balancing HTTP traffic across two backend servers.

### Global Section

The following settings are defined in the Global section:

* The [maximum number of concurrent connections](xhttps://docs.haproxy.org/2.5/configuration.html#4.2-maxconn) allowed by HAProxy.

* The user and group under which HAProxy should run.

* A [UNIX socket for accessing HAProxy statistics](https://docs.haproxy.org/2.5/configuration.html#3.1-stats%20socket).

In the `defaults` block, we set the [operating mode](https://docs.haproxy.org/2.5/configuration.html#4.2-mode) to TCP and define [`option tcpka`](https://docs.haproxy.org/2.5/configuration.html#option%20tcpka)

```{.text .no-copy}
global
    maxconn 4000           # Maximum concurrent connections (adjust as needed)
    user haproxy          # User to run HAProxy process
    group haproxy          # Group to run HAProxy process
    stats socket /var/run/haproxy.sock mode 666 level admin

defaults
    mode tcp             # Set operating mode to TCP
    #option tcpka
```

### Frontend Section

The following settings are defined in this section:

* Create a frontend named "webserver" that listens on port 80 for incoming HTTP requests.

* Enable the [`httpclose`](https://docs.haproxy.org/2.5/configuration.html#4.2-option%20httpclose) option to terminate idle client connections efficiently.

* Specify the default backend for this frontend.


```{.text .no-copy}
frontend gr-prod-rw
    bind 0.0.0.0:3307     
    mode tcp
    option contstats
    option dontlognull
    option clitcpka
    default_backend gr-prod-rw
```

You should add the following options:



| option | Description |
|---|---|
|`contstats` | Provides continuous updates to the statistics of your connections. This option ensures that your traffic counters are updated in real-time, rather than only after a connection closes, giving you a more accurate and immediate view of your traffic patterns. |
|`dontlognull` | Does not log requests that don't transfer any data, like health check pings. |
|`clitcpka` | Configures TCP keepalive settings for client connections. This option allows the operating system to detect and terminate inactive connections, even if HAProxy isn't actively checking them. |


### Backend Section

In this section, you specify the backend servers that will handle requests forwarded by the frontend. List each server with their respective IP addresses, ports, and weights.

You set up a health check with `check inter 10000`. This option means that HAProxy performs a health check on each server every 10,000 milliseconds or 10 seconds. If a server fails a health check, it is temporarily removed from the pool until it passes subsequent checks, ensuring smooth and reliable client service. This proactive monitoring is crucial for maintaining an efficient and uninterrupted backend service.

Set the number of retries to put the service down and up. For example, you set the `rise` parameter to `1`, which means the server only needs to pass one health check before the server is considered healthy again. The `fall` parameter is set to `2`, requiring two consecutive failed health checks before the server is marked as unhealthy. 

The `weight 50 backup` setting is crucial for load balancing; this setting determines that this server only receives traffic if the primary servers are down. The weight of 50 indicates the relative amount of traffic the server will handle compared to other servers in the backup role. This method ensures the server can handle a significant load even in backup mode, but not as much as a primary server.

The following example lists these options. Replace the server details (IP addresses, ports) with your backend server information. Adjust weights and other options according to your specific needs and server capabilities.

```{.text .no-copy}
backend servers
    server server1 10.0.68.39:3307 check inter 10000 rise 1 fall 2 weight 50
    server server1 10.0.68.74:3307 check inter 10000 rise 1 fall 2 weight 50 backup
    server server1 10.0.68.20:3307 check inter 10000 rise 1 fall 2 weight 1 backup
```

 [`More information about how to configure HAProxy`](https://docs.haproxy.org/2.5/configuration.html)
