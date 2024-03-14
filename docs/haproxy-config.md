# HAProxy configuration file

## Example of HAProxy v1 configuration file

??? example "Example of the HAProxy v1 configuration file"

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

## Differences between version 1 configuration file and version 2 configuration file

### Version Declaration:

v1: The configuration file typically omits an explicit version declaration.

v2: You must explicitly declare the version using the version keyword followed by the specific version number (e.g., version = 2.0).

### Global Parameters:

v1 and v2: Both versions utilize a global section to define global parameters, but certain parameters might have different names or functionalities across versions. Refer to the official documentation for specific changes.

### Configuration Blocks:

v1 and v2: Both versions use similar indentation-based structure to define configuration blocks like frontend and backend. However, v2 introduces new blocks and keywords not present in v1 (e.g., process, http-errors).

### Directives:

v1 and v2: While many directives remain consistent, some might have renamed keywords, altered syntax, or entirely new functionalities in v2. Consult the official documentation for a comprehensive comparison of directives and their usage between versions.

### Comments:

v1 and v2: Both versions support comments using the # symbol. However, v2 introduces multi-line comments using /* ... */ syntax, which v1 does not support.

## Version 2 configuration file

This simplified example is for load balancing. HAProxy offers numerous features for advanced configurations and fine-tuning.

This example demonstrates a basic HAProxy v2 configuration file for load balancing HTTP traffic across two backend servers.

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
    option tcpka
```

### Frontend Section

The following settings are defined in this section:

* Create a frontend named "webserver" that listens on port 80 for incoming HTTP requests.

* Enable the [`httpclose`](https://docs.haproxy.org/2.5/configuration.html#4.2-option%20httpclose) option to terminate idle client connections efficiently.

* Specify the default backend for this frontend.


```{.text .no-copy}
frontend webserver
    bind 192.168.1.10:8080     # Listen on port 80 for incoming traffic
    default_backend servers
```

### Backend Section

The following settings are defined in this section:

* Specify a backend named "servers" to handle requests forwarded by the frontend.

* Choose the "roundrobin" [`balance`](https://docs.haproxy.org/2.5/configuration.html#4.2-balance) method to distribute traffic evenly among backend servers.

* Enable [`httpchk`](https://docs.haproxy.org/2.5/configuration.html#4.2-option%20httpchk) to perform health checks using HTTP requests.

* Specify two servers named "server1" and "server2" with their respective IP addresses, ports, weights, and maximum connections.

!!! note:

    Replace the server details (IP addresses, ports) with your actual backend server information.
    Adjust weights and connection limits according to your specific needs and server capabilities.

```{.text .no-copy}
backend servers
    balance roundrobin    # Distribute traffic in a round-robin fashion
    option tcpka          # TCP keepalive packets
    server server1 192.168.1.10:8080 weight 1 maxconn 200
    server server2 192.168.1.20:8080 weight 2 maxconn 100
    server server3 192.168.1.30:8080 weight 3 maxconn 100
```

 [`More information about how to configure HAProxy`](https://docs.haproxy.org/2.5/configuration.html)
