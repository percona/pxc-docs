# Running Percona XtraDB Cluster in a Docker Container

## {{post}} Docker containers

You must request access to the {{post}} repository from [Percona Support] to be able to download the {{post}} Docker image. This request provides you with a client ID and an access token, which are required to download the binary tarball that contains the Docker image.

After providing the access token, download the tar file. The following is an example of the download command. Provide a password before the download begins.

```{.bash data-prompt="$"}
$ sudo docker load -i percona-xtradb-cluster-5.7.44-31.65.2.docker.tar
```

You can view the currently available Docker images on your local system. The following command lists these images. 

```{.bash data-prompt="$"}
$ sudo docker images
```

The result of the command contains the following information:

* Repository name

* Tag - if the image is a specific version or latest

* Image ID - a unique identifier of the image

* Size - the size of the image on the disk

??? example "Expected output"

    ```{.text .no-copy}
    Repository                      TAG            IMAGE ID       CREATED       SIZE
    percona/percona-xtradb-cluster  5.7           (image number) (when created) (size of file)
    percona/percona-xtradb-cluster  5.7.44-31.65.2 
    ```

After downloading, you can run the Docker image to [set up a cluster](#set-up-a-cluster).

## Docker images for versions that are not {{post}}

Docker images of Percona XtraDB Cluster are hosted publicly on Docker Hub at
[https://hub.docker.com/r/percona/percona-xtradb-cluster/](https://hub.docker.com/r/percona/percona-xtradb-cluster/).

For more information about using Docker, see the [Docker Docs](https://docs.docker.com/).

Be sure to install the latest official version of Docker for the latest features. The versions provided via `apt` and `yum` may lag behind the latest Docker features.

By default, Docker pulls the image from Docker Hub if it is unavailable locally.

## Set up a cluster

The following procedure describes how to set up a simple 3-node cluster
for evaluation and testing purposes, with all nodes running Percona XtraDB Cluster 5.7 in separate containers on one host:

1. Create a Docker network:

    ```{.bash data-prompt="$"}
    docker network create pxc-network
    ```

2. Bootstrap the cluster (create the first node):

    ```{.bash data-prompt="$"}
    docker run -d \
      -e MYSQL_ROOT_PASSWORD=root \
      -e CLUSTER_NAME=cluster1 \
      --name=node1 \
      --net=pxc-network \
      percona/percona-xtradb-cluster:5.7
    ```

3. Join the second node:

    ```{.bash data-prompt="$"}
    docker run -d \
      -e MYSQL_ROOT_PASSWORD=root \
      -e CLUSTER_NAME=cluster1 \
      -e CLUSTER_JOIN=node1 \
      --name=node2 \
      --net=pxc-network \
      percona/percona-xtradb-cluster:5.7
    ```

4. Join the third node:

    ```{.bash data-prompt="$"}
    docker run -d \
      -e MYSQL_ROOT_PASSWORD=root \
      -e CLUSTER_NAME=cluster1 \
      -e CLUSTER_JOIN=node1 \
      --name=node3 \
      --net=pxc-network \
      percona/percona-xtradb-cluster:5.7
    ```

To ensure that the cluster is running:

1. Access the MySQL client. For example, on the first node:

    ```{.bash data-prompt="$"}
    $ sudo docker exec -it node1 /usr/bin/mysql -uroot -proot
    ```

    ??? example "Expected output"

        ```{.text .no-copy}

        mysql: [Warning] Using a password on the command line interface can be insecure.
        Welcome to the MySQL monitor. Commands end with ; or \g.
        Your MySQL connection id is 12
        Server version: 5.7.19-17-57-log Percona XtraDB Cluster (GPL), Release rel17, Revision c10027a, WSREP version 29.22, wsrep_29.22

        Copyright (c) 2009-2017 Percona LLC and/or its affiliates
        Copyright (c) 2000, 2017, Oracle and/or its affiliates. All rights reserved.

        Oracle is a registered trademark of Oracle Corporation and/or its
        affiliates. Other names may be trademarks of their respective
        owners.

        Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

        mysql@node1>
        ```

2. View the wsrep status variables by running the following command:

    ```{sql}
    mysql@node1> show status like 'wsrep%';
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

[Percona Support]: https://www.percona.com/services/support