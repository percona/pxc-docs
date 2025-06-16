# Use status variables to monitor

Percona XtraDB Cluster uses wsrep (Write Set Replication) status variables to track cluster health and performance. These variables begin with the `wsrep_` prefix and provide real-time information about replication, node status, and cluster operations.

You can view all wsrep variables with a single query:

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_%';
```

??? example "Expected output"

    ```{.text .no-copy}

    +-------------------------+-------+
    | Variable_name          | Value |
    +------------------------+-------+
    | wsrep_protocol_version | 11    |
    | wsrep_ready            | ON    |
    | wsrep_connected        | ON    |
    | wsrep_cluster_size     | 3     |
    | wsrep_cluster_status   | Primary |
    +------------------------+-------+
    ```

## Essential monitoring variables

### Cluster membership and health

**wsrep_cluster_size**
Shows the number of nodes currently active in the cluster. A healthy cluster maintains the expected number of nodes.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size';
```

??? example "Expected output"

    ```{.text .no-copy}

    +-------------------+-------+
    | Variable_name     | Value |
    +-------------------+-------+
    | wsrep_cluster_size| 3     |
    +-------------------+-------+
    ```

**wsrep_cluster_status**
Indicates the overall cluster state. The value should be "Primary" for normal operations.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_cluster_status';
```

??? example "Expected output"

    ```{.text .no-copy}

    +-----------------------------+-------------+
    | Variable_name               | Value       |
    +-----------------------------+-------------+
    | wsrep_cluster_status        | Primary     |
    +-----------------------------+-------------+
    ```

### Node readiness and connectivity

**wsrep_ready**
Shows whether the node can accept write operations. The value must be "ON" for the node to handle write requests.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_ready';
```

??? example "Expected output"

    ```{.text .no-copy}

    +---------------+-------+
    | Variable_name | Value |
    +---------------+-------+
    | wsrep_ready   | ON    |
    +---------------+-------+
    ```

**wsrep_connected**
Indicates whether the node maintains connection to the cluster. The value should be "ON" for normal operations.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_connected';
```

??? example "Expected output"

    ```{.text .no-copy}

    +------------------+-------+
    | Variable_name    | Value |
    +------------------+-------+
    | wsrep_connected  | ON    |
    +------------------+-------+
    ```

**wsrep_local_state_comment**
Describes the current state of the node within the cluster. The value should be "Synced" for normal operations.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_local_state_comment';
```

??? example "Expected output"

    ```{.text .no-copy}

    +-----------------------------+-----------+
    | Variable_name               | Value     |
    +-----------------------------+-----------+
    | wsrep_local_state_comment   | Synced    |
    +-----------------------------+-----------+
    ```

Common state values include:
* `Synced`: Node operates normally and processes transactions
* `Joining`: Node synchronizes with the cluster using SST or IST
* `Donor`: Node provides data to a joining node
* `Disconnected`: Node lost connection to the cluster

## Monitor replication performance

### Analyze replication lag and flow control

**wsrep_local_recv_queue_avg**
Shows the average length of the receive queue. This metric indicates how well the node processes incoming replication events. Values consistently above 0.0 suggest potential performance issues.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_local_recv_queue_avg';
```

??? example "Expected output"

    ```{.text .no-copy}

    +----------------------------+------------+
    | Variable_name              | Value      |
    +----------------------------+------------+
    | wsrep_local_recv_queue_avg | 0.00000000 |
    +----------------------------+------------+
    ```

**wsrep_flow_control_paused**
Indicates how often the node pauses replication due to flow control. High values suggest the node struggles to keep up with replication traffic.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_paused';
```

??? example "Expected output"

    ```{.text .no-copy}

    +----------------------------+-------+
    | Variable_name              | Value |
    +----------------------------+-------+
    | wsrep_flow_control_paused  | 0.00  |
    +----------------------------+-------+
    ```

### Check outbound replication performance

**wsrep_local_send_queue_avg**
Shows the average number of transactions queued for replication. Values well above 0.0 indicate replication delays.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_local_send_queue_avg';
```

??? example "Expected output"

    ```{.text .no-copy}

    +----------------------------+------------+
    | Variable_name              | Value      |
    +----------------------------+------------+
    | wsrep_local_send_queue_avg | 0.20414418 |
    +----------------------------+------------+
    ```

## Troubleshoot common issues

### High send queue values

When `wsrep_local_send_queue_avg` remains elevated, the node cannot replicate transactions quickly enough. This situation typically causes replication delays and cluster performance issues.

To reduce the backlog, consider these actions:

* Check network connectivity between nodes. Look for latency, dropped packets, or limited bandwidth that might slow replication. Use tools like `iperf`, `ping`, and `traceroute` to identify weak network links or routing problems.

* Monitor the load on each node. High CPU usage or disk I/O, either locally or on connected peers, can delay replication. Use system monitoring tools to review performance and reduce competing workloads when necessary.

* Adjust replication thread settings. If the application workload allows, increasing the `wsrep_slave_threads` value can improve parallel replication and help reduce queue size.

* Analyze flow control behavior. Review `wsrep_flow_control_sent` and `wsrep_flow_control_paused` metrics to determine whether this node slows down replication due to pressure from slower peers.

* Review cluster node balance. Avoid running nodes with significantly different hardware or network performance in the same cluster. Imbalanced environments often result in persistent replication lag.

* Enable compression if available. Compressing replication traffic can reduce network usage and improve performance, particularly in environments with slower or congested network links.

### Node state issues

When a node shows unexpected states in `wsrep_local_state_comment`, follow these troubleshooting steps:

* Check the database error log (/var/log/mysql/error.log)

* Verify sufficient disk space exists

* Confirm that donor nodes remain reachable and healthy

* Test network connectivity by checking ping and port access, especially TCP 4567, 4568, and 4444

* Review firewall rules and hostname resolution

* Restart the node if needed, then monitor the log for progress

## Monitor node status

The following table provides a comprehensive overview of key monitoring variables:

<table>
  <thead>
    <tr>
      <th style="width: 260px;">Variable</th>
      <th>What the variable indicates</th>
      <th>Example values</th>
      <th>Typical cause when not OK</th>
      <th>Troubleshooting tips</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>wsrep_ready</code></td>
      <td>Node readiness to handle queries</td>
      <td><code>ON</code>, <code>OFF</code></td>
      <td>SST in progress, Donor state, Desynced, local issues (disk, memory)</td>
      <td>
        * Check <code>wsrep_local_state_comment</code> (should be <code>Synced</code>)<br>
        * Wait for SST/IST to finish<br>
        * Review logs and system resource usage
      </td>
    </tr>
    <tr>
      <td><code>wsrep_local_state_comment</code></td>
      <td>Node's current role/state</td>
      <td><code>Joining</code>, <code>Donor</code>, <code>Synced</code>, etc.</td>
      <td>Node synchronizes or recovers</td>
      <td>
        * Monitor SST/IST progress<br>
        * Wait until state changes to <code>Synced</code><br>
        * Avoid restarting other nodes during SST
      </td>
    </tr>
    <tr>
      <td><code>wsrep_local_cert_failures</code></td>
      <td>Number of failed certifications</td>
      <td>0, 1, 5, ...</td>
      <td>Conflicting transactions or replication failures</td>
      <td>
        * Investigate frequent conflicts<br>
        * Review queries or retry failed operations
      </td>
    </tr>
    <tr>
      <td><code>wsrep_local_bf_aborts</code></td>
      <td>Transactions aborted due to brute-force conflict resolution</td>
      <td>0, 10, 100, ...</td>
      <td>High contention on hot rows or large transactions</td>
      <td>
        * Identify conflicting queries<br>
        * Break up large writes<br>
        * Reduce contention through schema or logic changes
      </td>
    </tr>
  </tbody>
</table>

## Check cluster state

### Verify cluster health

Connect to the MySQL server using a MySQL client or command-line tool to access one of the nodes in the cluster.

Verify that `wsrep_cluster_status` equals "Primary." This status indicates whether the cluster remains stable. If the status is not "Primary," the cluster may experience issues, such as a split-brain scenario or insufficient nodes to form a quorum.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_cluster_status';
```

??? example "Expected output"

    ```{.text .no-copy}

    +-----------------------------+-------------+
    | Variable_name               | Value       |
    +-----------------------------+-------------+
    | wsrep_cluster_status        | Primary     |
    +-----------------------------+-------------+
    ```

### Verify node connectivity

Check the `wsrep_connected` and `wsrep_ready` variables to ensure both equal "ON."

The `wsrep_connected` variable indicates whether the node connects to the cluster. If this value is not "ON," the node disconnects and cannot participate in cluster operations.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_connected';
```

??? example "Expected output"

    ```{.text .no-copy}

    +------------------+-------+
    | Variable_name    | Value |
    +------------------+-------+
    | wsrep_connected  | ON    |
    +------------------+-------+
    ```

## Monitor replication conflicts

### Track certification failures

The `wsrep_local_cert_failures` variable tracks the number of certification failures during the replication process. Certification failures occur when a node attempts to apply a write operation that conflicts with another operation already applied to the cluster. A high number of certification failures can indicate frequent write conflicts, leading to performance issues and increased latency.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_local_cert_failures';
```

??? example "Expected output"

    ```{.text .no-copy}

    +-----------------------------+-------+
    | Variable_name               | Value |
    +-----------------------------+-------+
    | wsrep_local_cert_failures   | 0     |
    +-----------------------------+-------+
    ```

### Track brute-force aborts

The `wsrep_local_bf_aborts` variable tracks the number of aborts due to conflicts with write sets being processed. These conflicts typically happen when multiple nodes attempt to write to the same data simultaneously, resulting in conflicts that require one operation to abort.

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_local_bf_aborts';
```

??? example "Expected output"

    ```{.text .no-copy}

    +-----------------------------+-------+
    | Variable_name               | Value |
    +-----------------------------+-------+
    | wsrep_local_bf_aborts       | 0     |
    +-----------------------------+-------+
    ```

## Monitor flow control

### Track flow control messages

You can identify excessive flow control messages by monitoring the `wsrep_flow_control_sent` and `wsrep_flow_control_recv` variables.

Flow control messages are signals used in the cluster to manage the flow of replication traffic between nodes. These messages help prevent a situation where a node becomes overwhelmed with incoming data, especially if the node lags behind in processing transactions. By regulating the flow of data, these messages ensure that all nodes can keep up with the replication process without losing data integrity.

The `wsrep_flow_control_sent` variable counts the number of flow control messages sent by the node to manage replication traffic. Conversely, the `wsrep_flow_control_recv` variable tracks the number of flow control messages received by the node, indicating how often the node has to pause or slow down its processing to accommodate the flow control mechanism.

Monitoring these variables allows you to assess the frequency of flow control messages in the cluster. A high number of these messages may indicate performance issues, such as nodes struggling to keep up with replication, prompting you to investigate and optimize the cluster's performance.

### Monitor replication queues

Large replication queues indicate a backlog of transactions waiting to be processed in a database cluster. You can identify these queues by monitoring the `wsrep_local_recv_queue` variable. When the replication queue grows significantly, the system struggles to keep up with incoming changes, which can lead to delays in data synchronization across nodes. This situation may result in increased latency for read and write operations, potential data inconsistencies, and a negative impact on overall system performance. Addressing large replication queues is crucial for maintaining efficient database operations and ensuring timely data availability.

## Gather cluster metrics

Gathering cluster metrics for long-term analysis and visualization plays a crucial role in maintaining the health and performance of a database cluster. Consistent collection of specific performance data over time allows for the creation of graphs that facilitate monitoring and evaluation. Tracking these metrics enables the identification of trends, early detection of issues, and informed decision-making to optimize overall cluster performance.

### Essential metrics to collect

* Queue Sizes:

  The `wsrep_local_recv_queue` and `wsrep_local_send_queue` variables provide insights into the sizes of the local receive and send queues in a database cluster.
  
  * The `wsrep_local_recv_queue` tracks the number of transactions waiting to be processed by the local node. A large receive queue may indicate that the node struggles to keep up with incoming replication traffic, potentially leading to delays in data synchronization and increased latency for read and write operations.
  
  * The `wsrep_local_send_queue` monitors the number of transactions that the local node has sent to other nodes but have not yet been acknowledged. A large send queue can suggest that other nodes are unable to process incoming changes quickly enough, which may also contribute to replication delays and affect overall system performance.
  
  Understanding these queue sizes is essential for diagnosing performance issues and ensuring efficient data replication across the cluster.

* Flow control metrics:

  The `wsrep_flow_control_sent` and `wsrep_flow_control_recv` variables provide important information about the flow control mechanism in a database cluster.
  
  * The `wsrep_flow_control_sent` variable indicates the number of flow control messages sent by the local node to other nodes. Flow control messages help manage the rate of data replication, ensuring that nodes do not become overwhelmed with incoming transactions. A high number of sent flow control messages may suggest that the local node frequently needs to slow down the replication process to maintain stability.
  
  * The `wsrep_flow_control_recv` variable tracks the number of flow control messages received by the local node from other nodes. This metric reflects how often the local node must pause or slow down its operations due to requests from other nodes. A high count of received flow control messages can indicate that the local node is experiencing pressure from its peers, which may lead to delays in processing transactions.
  
  Monitoring these flow control metrics is essential for understanding the dynamics of data replication within the cluster and for identifying potential performance bottlenecks.

* Replication metrics:

  The `wsrep_replicated` and `wsrep_received` variables provide critical insights into the replication process within a database cluster.
  
  * The `wsrep_replicated` variable indicates the total number of transactions that the local node has successfully replicated to other nodes in the cluster. This metric reflects the effectiveness of the replication process and helps assess how much data has been shared across the cluster. A high value for `wsrep_replicated` suggests that the node actively participates in the replication process and contributes to data consistency across all nodes.
  
  * The `wsrep_received` variable tracks the total number of transactions that the local node has received from other nodes. This metric shows how many transactions have been sent to the local node for processing. A high value for `wsrep_received` indicates that the node is receiving a significant amount of data from its peers, which can impact its performance if the incoming transaction rate exceeds its processing capacity.
  
  Understanding these replication metrics is essential for evaluating the health and efficiency of the replication process in the cluster. Monitoring both `wsrep_replicated` and `wsrep_received` helps identify potential issues related to data synchronization and performance bottlenecks.

* Replication byte metrics:

  The `wsrep_replicated_bytes` and `wsrep_received_bytes` variables provide important information about the volume of data involved in the replication process within a database cluster.
  
  * The `wsrep_replicated_bytes` variable indicates the total number of bytes that the local node has successfully replicated to other nodes in the cluster. This metric reflects the amount of data shared across the cluster and helps assess the efficiency of the replication process. A high value for `wsrep_replicated_bytes` suggests that the node actively participates in data replication, contributing to overall data consistency.
  
  * The `wsrep_received_bytes` variable tracks the total number of bytes that the local node has received from other nodes. This metric shows the volume of data sent to the local node for processing. A high value for `wsrep_received_bytes` indicates that the node is receiving a significant amount of data from its peers, which can affect performance if the incoming data rate exceeds the node's processing capacity.
  
  Understanding these replication metrics is essential for evaluating the health and efficiency of the replication process in the cluster. Monitoring both `wsrep_replicated_bytes` and `wsrep_received_bytes` helps identify potential issues related to data synchronization and performance bottlenecks.

## Use Percona Monitoring and Management

[Percona Monitoring and Management](https://www.percona.com/doc/percona-monitoring-and-management/index.html) includes two dashboards to monitor PXC:

1. PXC/Galera Cluster Overview:

    ![image](_static/pmm.pxc-galera-cluster-overview.png)

2. PXC/Galera Graphs:

    ![image](_static/pmm.pxc-galera-graphs.png)

    These dashboards are available from the menu:

    ![image](_static/pmm.menu.ha.png)

Please refer to the [official documentation](https://www.percona.com/doc/percona-monitoring-and-management/index.html) for details on Percona Monitoring and Management installation and setup.