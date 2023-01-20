# Bootstrap the first node

After you [configure all PXC nodes](configure.md#configure), initialize the cluster by
bootstrapping the first node.  The initial node must contain all the data that
you want to be replicated to other nodes.

Bootstrapping implies starting the first node without any known cluster
addresses: if the `wsrep_cluster_address` variable is empty, Percona XtraDB Cluster assumes that this is the first node and initializes the cluster.

Instead of changing the configuration, start the first node using the following
command:

```{.bash data-prompt="[root@pxc1 ~]#"}
[root@pxc1 ~]# systemctl start mysql@bootstrap.service
```

When you start the node using the previous command,
it runs in bootstrap mode with `wsrep_cluster_address=gcomm://`.
This tells the node to initialize the cluster
with `wsrep_cluster_conf_id` variable set to `1`.
After you [add other nodes](add-node.md#add-node) to the cluster,
you can then restart this node as normal,
and it will use standard configuration again.

!!! note

    A service started with `mysql@bootstrap` must be stopped using the same command. For example, the `systemctl stop mysql` command does not stop an instance started with the `mysql@bootstrap` command.

To make sure that the cluster has been initialized, run the following:

```{.bash data-prompt="mysql@pxc1>"}
mysql@pxc1> show status like 'wsrep%';
```

The output shows that the cluster size is 1 node,
it is the primary component, the node is in the `Synced` state,
it is fully connected and ready for write-set replication.

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

## Next steps

After initializing the cluster, you can [add other nodes](add-node.md#add-node).
