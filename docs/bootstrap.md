# Bootstrap the first node

After you [configure all PXC nodes](configure-nodes.md#configure-nodes-for-write-set-replication), initialize the cluster by
bootstrapping the first node. This node must contain all the data you want to replicate to other nodes.

Start the first node without any known cluster
addresses: if the `wsrep_cluster_address` variable is empty, Percona XtraDB Cluster treats the node as the first node and initializes the cluster.

Instead of changing the configuration, start the first node, `[root@pxc1 ~]#`, using the following command:

```shell
systemctl start mysql@bootstrap.service
```

When you start the node using this command,
the node runs in bootstrap mode with `wsrep_cluster_address=gcomm://`.
Bootstrap mode initializes the cluster and sets the `wsrep_cluster_conf_id` variable to `1`.

After you [add other nodes](add-node.md) to the cluster,
you can then restart the first node as normal. Then the node uses the standard configuration again.

!!! note

    Stop a node started with `mysql@bootstrap` by running `systemctl stop mysql@bootstrap`. The `systemctl stop mysql` command does not stop an instance started with `mysql@bootstrap`.

To check the status of the first node, `mysql@pxc1>`, run the following:

```sql
SHOW STATUS LIKE 'wsrep%';
```

??? example "Expected output"

    ```{.text .no-copy}
    +----------------------------+--------------------------------------+
    | Variable_name              | Value                                |
    +----------------------------+--------------------------------------+
    | wsrep_local_state_uuid     | c2883338-834d-11e2-0800-03c9c68e41ec |
    | ...                        | ...                                  |
    | wsrep_local_state          | 4                                    |
    | wsrep_local_state_comment  | Synced                               |
    | ...                        | ...                                  |
    | wsrep_cluster_size         | 1                                    |
    | wsrep_cluster_status       | Primary                              |
    | wsrep_connected            | ON                                   |
    | ...                        | ...                                  |
    | wsrep_ready                | ON                                   |
    +----------------------------+--------------------------------------+
    40 rows in set (0.01 sec)
    ```

The output confirms that the first node initialized the cluster:

* Cluster size: one node

* Cluster status: primary component

* Node state: Synced

* Connection status: fully connected and ready for write-set replication

## Generate and copy SSL certificates
 
Generate the SSL certificates on this node before you add other nodes to the cluster. Use identical key and certificate files on every node. Store the certificates in `/etc/`, outside the data directory.
 
1. Generate the SSL certificates, `[root@pxc1 ~]#`:

    ```shell
      openssl req -newkey rsa:2048 -nodes -keyout /etc/server-key.pem \
      -x509 -days 365 -out /etc/server-cert.pem
    ```
 
2. Copy the certificates to the nodes you plan to add. For example:

    ```shell
      scp /etc/server-key.pem pxc2:/etc/
      scp /etc/server-cert.pem pxc2:/etc/
      scp /etc/server-key.pem pxc3:/etc/
      scp /etc/server-cert.pem pxc3:/etc/
    ```

## Next steps

After initializing the cluster, you can [add other nodes](add-node.md).
