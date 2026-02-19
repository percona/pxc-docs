# Run in a Docker container

Docker images of Percona XtraDB Cluster are hosted publicly on Docker Hub at
[https://hub.docker.com/r/percona/percona-xtradb-cluster/ :octicons-link-external-16:](https://hub.docker.com/r/percona/percona-xtradb-cluster/).

For more information about using Docker, see the [Docker Docs :octicons-link-external-16:](https://docs.docker.com/). Make
sure that you are using the latest version of Docker. The ones
provided via `apt` and `yum` may be outdated and cause errors.

We gather [Telemetry data](telemetry.md) in the Percona packages and Docker images.

--8<--- "get-help-snip.md"

By default, Docker pulls the image from Docker Hub if the image is not available locally. The image contains only the most essential Percona XtraDB Cluster binaries. Utilities included in a Percona Server for MySQL or MySQL installation might be missing from this image.

The following procedure describes how to set up a simple 3-node cluster
for evaluation and testing purposes. Do not use these instructions in a
production environment because the MySQL certificates generated in this
procedure are self-signed. For a
production environment, you should generate and store the certificates to be used by Docker and configure proper storage, security, backup, and monitoring systems.

In this procedure, all of the nodes run Percona XtraDB Cluster {{vers}} in separate containers on one host:
{.power-number}

1.  Create a ~/pxc-docker-test/config directory.

2.  Create a custom.cnf file with the following contents, and place the file in the new directory:

    ```{.text .no-copy}
    [mysqld]
    ssl-ca = /cert/ca.pem
    ssl-cert = /cert/server-cert.pem
    ssl-key = /cert/server-key.pem
    [client]
    ssl-ca = /cert/ca.pem
    ssl-cert = /cert/client-cert.pem
    ssl-key = /cert/client-key.pem
    [sst]
    encrypt = 4
    ssl-ca = /cert/ca.pem
    ssl-cert = /cert/server-cert.pem
    ssl-key = /cert/server-key.pem
    ```

3.  Create a ~/pxc-docker-test/cert directory to store self-signed SSL cert:

    ```shell
    mkdir -m 777 -p ~/pxc-docker-test/cert
    ```

4.  Create a create-ssl-certs.sh file with the following contents, and place the file in the cert directory

    ```shell
    #!/bin/bash
    set -e
    OUTPUT_DIR="/cert"
    openssl genrsa 2048 > "${OUTPUT_DIR}/ca-key.pem"
    openssl req -new -x509 -nodes -days 3600 -subj "/C=/ST=/L=/O=/CN=" -key "${OUTPUT_DIR}/ca-key.pem" -out "${OUTPUT_DIR}/ca.pem"
    openssl req -newkey rsa:2048 -days 3600 -subj "/C=/ST=/L=/O=/CN=" \
            -nodes -keyout "${OUTPUT_DIR}/server-key.pem" -out "${OUTPUT_DIR}/server-req.pem"
    openssl rsa -in "${OUTPUT_DIR}/server-key.pem" -out "${OUTPUT_DIR}/server-key.pem"    
    openssl x509 -req -in "${OUTPUT_DIR}/server-req.pem" -days 3600 -subj "/C=/ST=/L=/O=/CN=" \
            -CA "${OUTPUT_DIR}/ca.pem" -CAkey "${OUTPUT_DIR}/ca-key.pem" -set_serial 01 -out "${OUTPUT_DIR}/server-cert.pem"
    openssl req -newkey rsa:2048 -days 3600 -subj "/C=/ST=/L=/O=/CN=" \
            -nodes -keyout "${OUTPUT_DIR}/client-key.pem" -out "${OUTPUT_DIR}/client-req.pem"
    openssl rsa -in "${OUTPUT_DIR}/client-key.pem" -out "${OUTPUT_DIR}/client-key.pem"
    openssl x509 -req -in "${OUTPUT_DIR}/client-req.pem" -days 3600 -subj "/C=/ST=/L=/O=/CN=" \
            -CA "${OUTPUT_DIR}/ca.pem" -CAkey "${OUTPUT_DIR}/ca-key.pem" -set_serial 01 -out "${OUTPUT_DIR}/client-cert.pem"
    openssl verify -CAfile "${OUTPUT_DIR}/ca.pem" "${OUTPUT_DIR}/server-cert.pem" "${OUTPUT_DIR}/client-cert.pem"
    ```

5.  Generate the self-signed certs

    ```shell
    docker run --name pxc-cert --rm  -v ~/pxc-docker-test/cert:/cert percona/percona-xtradb-cluster:8.4 /bin/bash /cert/create-ssl-certs.sh
    ```

6.  Create a Docker network:

    ```shell
    docker network create pxc-network
    ```

7.  Bootstrap the cluster (create the first node):

    ```shell
    docker run -d \
      -e MYSQL_ROOT_PASSWORD=test1234# \
      -e CLUSTER_NAME=pxc-cluster1 \
      --name=pxc-node1 \
      --net=pxc-network \
      -v ~/pxc-docker-test/cert:/cert \
      -v ~/pxc-docker-test/config:/etc/percona-xtradb-cluster.conf.d \
      percona/percona-xtradb-cluster:{{vers}}
    ```

8.  Join the second node:

    ```shell
    docker run -d \
      -e MYSQL_ROOT_PASSWORD=test1234# \
      -e CLUSTER_NAME=pxc-cluster1 \
      -e CLUSTER_JOIN=pxc-node1 \
      --name=pxc-node2 \
      --net=pxc-network \
      -v ~/pxc-docker-test/cert:/cert \
      -v ~/pxc-docker-test/config:/etc/percona-xtradb-cluster.conf.d \
      percona/percona-xtradb-cluster:{{vers}}
    ```

9.  Join the third node:

    ```shell
    docker run -d \
      -e MYSQL_ROOT_PASSWORD=test1234# \
      -e CLUSTER_NAME=pxc-cluster1 \
      -e CLUSTER_JOIN=pxc-node1 \
      --name=pxc-node3 \
      --net=pxc-network \
      -v ~/pxc-docker-test/cert:/cert \
      -v ~/pxc-docker-test/config:/etc/percona-xtradb-cluster.conf.d \
      percona/percona-xtradb-cluster:{{vers}}
    ```

To verify the cluster is available, do the following:

1.  Access the MySQL client. For example, on the first node:

    ```shell
    sudo docker exec -it pxc-node1 /usr/bin/mysql -uroot -ptest1234#
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        mysql: [Warning] Using a password on the command line interface can be insecure.
        Welcome to the MySQL monitor.  Commands end with ; or \g.
        Your MySQL connection id is 12
        ...
        You are enforcing ssl connection via unix socket. Please consider
        switching ssl off as it does not make connection via unix socket
        any more secure
        mysql>
        ```

2.  View the wsrep status variables:

    ```sql
    show status like 'wsrep%';
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        +------------------------------+-------------------------------------------------+
        | Variable_name                | Value                                           |
        +------------------------------+-------------------------------------------------+
        | wsrep_local_state_uuid       | 625318e2-9e1c-11e7-9d07-aee70d98d8ac            |
        ...
        | wsrep_local_state_comment    | Synced                                          |
        ...
        | wsrep_incoming_addresses     | 172.18.0.2:3306,172.18.0.3:3306,172.18.0.4:3306 |
        ...
        | wsrep_cluster_conf_id        | 3                                               |
        | wsrep_cluster_size           | 3                                               |
        | wsrep_cluster_state_uuid     | 625318e2-9e1c-11e7-9d07-aee70d98d8ac            |
        | wsrep_cluster_status         | Primary                                         |
        | wsrep_connected              | ON                                              |
        ...
        | wsrep_ready                  | ON                                              |
        +------------------------------+-------------------------------------------------+
        59 rows in set (0.02 sec)
        ```

[Telemetry data]: telemetry.md
