# Monitor the cluster

Monitoring a Percona XtraDB Cluster is essential for maintaining performance, reliability, and overall health. Effective monitoring helps you identify issues before they impact users, optimize performance, and ensure data consistency across all nodes.

## Why monitoring matters

Monitoring provides critical insights into cluster health and performance across several key areas:

| Category                        | Description                                                                                                                                                     |
|---------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Performance optimization         | Identify bottlenecks: Monitoring helps identify slow queries, resource contention, and other performance bottlenecks that can degrade the overall efficiency of the database. |
|                                 | Resource utilization: By tracking CPU, memory, and disk I/O usage, you can optimize resource allocation and ensure that the cluster operates within its capacity. |
| Availability and uptime         | Detect failures: Continuous monitoring allows for the early detection of node failures or network issues, enabling quick responses to minimize downtime.      |
|                                 | Health checks: Regular health checks of nodes ensure that all components are functioning correctly and can help prevent unexpected outages.                   |
| Data integrity and consistency   | Replication monitoring: In a clustered environment, monitoring replication lag and certification failures helps ensure that data remains consistent across all nodes. |
|                                 | Error detection: Monitoring can help identify data corruption or inconsistencies, allowing for timely corrective actions.                                      |
| Security                        | Access monitoring: Keeping track of user access and authentication attempts can help detect unauthorized access or potential security breaches.                |
|                                 | Audit trails: Monitoring changes to data and schema can provide an audit trail for compliance and security purposes.                                          |
| Capacity planning               | Trend analysis: Monitoring historical performance data helps in forecasting future resource needs and planning for scaling the cluster as demand grows.       |
|                                 | Usage patterns: Understanding usage patterns can inform decisions about when to scale up or down, optimizing costs and performance.                           |
| Troubleshooting and diagnostics  | Root cause analysis: When issues arise, monitoring data can provide insights into the root causes, facilitating faster resolution.                             |
|                                 | Alerting: Setting up alerts for specific thresholds allows for proactive management of potential issues before they escalate.                                 |
| Compliance and reporting        | Regulatory compliance: Many industries have regulations that require monitoring and reporting on data access and integrity.                                   |
|                                 | Performance reporting: Regular reports on database performance can help stakeholders understand the health of the system and justify resource allocation.      |
| User experience                 | Response time monitoring: Tracking query response times ensures that users have a positive experience when interacting with the database.                     |
|                                 | Load balancing: Monitoring can help in distributing workloads evenly across nodes, preventing any single node from becoming a performance bottleneck.         |

## Cluster monitoring approach

The absence of a centralized node in the cluster enhances resilience and scalability. Each node operates independently, allowing for a distributed approach to data management and processing. This design eliminates a single point of failure, ensuring that the failure of one node does not compromise the entire system.

Each node maintains a unique view of the cluster, enabling autonomous data processing and request handling. This independence provides greater flexibility and performance, as nodes operate in parallel without waiting for a central authority to coordinate actions.

To identify the source of issues, administrators monitor each node independently. This approach offers a comprehensive view of the cluster's health and performance, facilitating more effective troubleshooting and optimization.

## Essential monitoring areas

### Cluster health monitoring

Monitor these critical areas to ensure cluster stability:

* **Node connectivity**: Verify that all nodes remain connected to the cluster
* **Cluster state**: Check that the cluster maintains primary status
* **Node readiness**: Ensure all nodes can accept write operations
* **Replication status**: Monitor replication lag and flow control

### Performance monitoring

Track these metrics to optimize cluster performance:

* **Query performance**: Monitor slow queries and query execution times
* **Resource utilization**: Track CPU, memory, and disk I/O usage
* **Replication performance**: Monitor replication queues and flow control
* **Network performance**: Check network latency and bandwidth usage

### Data consistency monitoring

Ensure data remains consistent across all nodes:

* **Replication lag**: Monitor delays in data synchronization
* **Certification failures**: Track transaction conflicts and aborts
* **Data integrity**: Verify data consistency across nodes
* **Backup status**: Monitor backup completion and integrity

## Monitoring tools and approaches

### Built-in MySQL monitoring

Use standard MySQL monitoring commands to check cluster status:

```{.bash data-prompt="mysql>"}
mysql> SHOW GLOBAL STATUS LIKE 'wsrep_%';
```

This command displays all wsrep-related status variables, providing comprehensive information about cluster health and performance.

### Manual cluster monitoring with myq-tools

Manual cluster monitoring can be performed using [myq-tools](https://github.com/jayjanssen/myq-tools/). This toolset includes utilities for monitoring MySQL performance and status.

#### Install myq-tools

Download and install myq-tools on your monitoring system:

```{.bash data-prompt="$"}
git clone https://github.com/jayjanssen/myq-tools.git
cd myq-tools
sudo make install
```

#### Use myq_status utility

The `myq_status` utility offers iostat-like views of MySQL `SHOW GLOBAL STATUS` variables, providing insights into the performance and status of the MySQL environment.

```{.bash data-prompt="$"}
myq_status -h localhost -u root -p
```

??? example "Expected output"

    ```{.text .no-copy}

    MySQL Status 2024-01-15 10:30:45
    +------------------+--------+--------+--------+--------+
    | Variable         | Current| 1min   | 5min   | 15min  |
    +------------------+--------+--------+--------+--------+
    | wsrep_ready      | ON     | ON     | ON     | ON     |
    | wsrep_connected  | ON     | ON     | ON     | ON     |
    | wsrep_cluster_size| 3     | 3      | 3      | 3      |
    +------------------+--------+--------+--------+--------+
    ```

#### Monitor specific variables

Focus on specific wsrep variables for targeted monitoring:

```{.bash data-prompt="$"}
myq_status -h localhost -u root -p --filter wsrep_
```

This command filters the output to show only wsrep-related variables, making it easier to focus on cluster-specific metrics.

### Percona Monitoring and Management (PMM)

For comprehensive monitoring, use Percona Monitoring and Management, which provides specialized dashboards for Percona XtraDB Cluster monitoring.

#### Install PMM

Follow the [PMM installation guide](https://www.percona.com/doc/percona-monitoring-and-management/index.html) to set up monitoring for your cluster.

#### Configure cluster monitoring

Add each cluster node to PMM for comprehensive monitoring:

```{.bash data-prompt="$"}
pmm-admin add mysql --username=pmm --password=pmm --host=localhost --port=3306
```

#### Access monitoring dashboards

PMM provides two specialized dashboards for cluster monitoring:

1. **PXC/Galera Cluster Overview**: High-level cluster health and status
2. **PXC/Galera Graphs**: Detailed performance metrics and trends

## Set up monitoring alerts

### Create monitoring scripts

Create scripts to monitor critical cluster metrics and send alerts when thresholds are exceeded.

#### Basic cluster health check

```{.bash}
#!/bin/bash
# Basic cluster health monitoring script

MYSQL_HOST="localhost"
MYSQL_USER="root"
MYSQL_PASSWORD="password"

# Check cluster status
CLUSTER_STATUS=$(mysql -h $MYSQL_HOST -u $MYSQL_USER -p$MYSQL_PASSWORD -e "SHOW GLOBAL STATUS LIKE 'wsrep_cluster_status';" | grep -v Variable_name | awk '{print $2}')

if [ "$CLUSTER_STATUS" != "Primary" ]; then
    echo "ALERT: Cluster status is $CLUSTER_STATUS"
    # Send alert notification
fi

# Check node connectivity
CONNECTED=$(mysql -h $MYSQL_HOST -u $MYSQL_USER -p$MYSQL_PASSWORD -e "SHOW GLOBAL STATUS LIKE 'wsrep_connected';" | grep -v Variable_name | awk '{print $2}')

if [ "$CONNECTED" != "ON" ]; then
    echo "ALERT: Node is not connected to cluster"
    # Send alert notification
fi
```

#### Monitor replication performance

```{.bash}
#!/bin/bash
# Replication performance monitoring script

MYSQL_HOST="localhost"
MYSQL_USER="root"
MYSQL_PASSWORD="password"

# Check replication queue
RECV_QUEUE=$(mysql -h $MYSQL_HOST -u $MYSQL_USER -p$MYSQL_PASSWORD -e "SHOW GLOBAL STATUS LIKE 'wsrep_local_recv_queue_avg';" | grep -v Variable_name | awk '{print $2}')

# Alert if queue is too high
if (( $(echo "$RECV_QUEUE > 1.0" | bc -l) )); then
    echo "ALERT: High receive queue: $RECV_QUEUE"
    # Send alert notification
fi

# Check flow control
FLOW_CONTROL=$(mysql -h $MYSQL_HOST -u $MYSQL_USER -p$MYSQL_PASSWORD -e "SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_paused';" | grep -v Variable_name | awk '{print $2}')

if (( $(echo "$FLOW_CONTROL > 0.1" | bc -l) )); then
    echo "ALERT: High flow control: $FLOW_CONTROL"
    # Send alert notification
fi
```

### Configure alert thresholds

Set up appropriate alert thresholds for different monitoring metrics:

| Metric | Warning Threshold | Critical Threshold | Action |
|--------|------------------|-------------------|---------|
| wsrep_cluster_status | Not "Primary" | Not "Primary" | Check cluster configuration |
| wsrep_connected | "OFF" | "OFF" | Check network connectivity |
| wsrep_cluster_size | < Expected count | < Expected count | Check node status |
| wsrep_local_recv_queue_avg | > 0.5 | > 1.0 | Check node performance |
| wsrep_flow_control_paused | > 0.05 | > 0.1 | Check replication performance |
| wsrep_local_cert_failures | > 10/hour | > 50/hour | Check for conflicts |

## Troubleshooting common issues

### Node connectivity problems

When nodes lose connection to the cluster:

1. Check network connectivity between nodes
2. Verify firewall rules allow communication on ports 4567, 4568, and 4444
3. Review cluster configuration and node addresses
4. Check system resources and disk space
5. Review MySQL error logs for specific error messages

### Replication performance issues

When replication queues grow or flow control activates frequently:

1. Check network performance between nodes
2. Monitor disk I/O performance on all nodes
3. Review CPU usage and system load
4. Check for long-running transactions
5. Consider adjusting replication thread settings

### Cluster state problems

When the cluster loses primary status:

1. Verify that a majority of nodes remain online
2. Check network connectivity between all nodes
3. Review cluster configuration and quorum settings
4. Consider bootstrap procedures if necessary
5. Check for split-brain scenarios

## Best practices for cluster monitoring

### Establish monitoring baselines

* Monitor cluster performance during normal operations
* Document typical values for key metrics
* Set up baseline alerts for unusual patterns
* Track performance trends over time

### Implement comprehensive monitoring

* Monitor all cluster nodes, not just one
* Use multiple monitoring approaches for redundancy
* Set up both automated and manual monitoring
* Include both technical and business metrics

### Plan for scalability

* Design monitoring to scale with cluster growth
* Use centralized monitoring where possible
* Implement automated scaling based on metrics
* Plan for monitoring tool capacity

### Regular monitoring maintenance

* Review and update alert thresholds regularly
* Test monitoring systems and alerting procedures
* Update monitoring tools and configurations
* Document monitoring procedures and contacts

## Integration with existing monitoring

### Prometheus and Grafana

Integrate cluster monitoring with existing Prometheus and Grafana setups:

* Export MySQL metrics to Prometheus
* Create custom dashboards for cluster-specific metrics
* Set up alerting rules for cluster health
* Use Grafana for visualization and analysis

### Nagios and Zabbix

Integrate with existing monitoring infrastructure:

* Create custom checks for cluster health
* Set up service dependencies and escalation
* Use existing alerting and notification systems
* Leverage existing monitoring workflows

### Cloud monitoring services

Use cloud-native monitoring services:

* AWS CloudWatch for AWS deployments
* Azure Monitor for Azure deployments
* Google Cloud Monitoring for GCP deployments
* Custom metrics and dashboards for cluster health