# Cluster failover

## Cluster membership and size

Cluster membership depends on which nodes connect to the rest of the cluster; 
no configuration setting explicitly defines the list of all possible cluster 
nodes. Therefore, each time a node joins the cluster, the total size of the 
cluster increases, and when a node leaves gracefully, the size decreases.

## Influence of cluster size on quorum

The cluster size directly influences the votes required to achieve a quorum. 
Quorum refers to the minimum number of votes needed for a cluster to function 
effectively and make decisions. Each node in the cluster typically represents 
one vote. Achieving quorum ensures the cluster can operate safely and avoid 
split-brain scenarios, where different parts operate independently without a 
clear leader.

## Quorum voting process

A quorum vote occurs when the system suspects that one or more nodes are 
disconnected due to a lack of response. This situation triggers a no-response 
timeout defined by the `evs.suspect_timeout` setting in the 
`wsrep_provider_options`, with a default value of five seconds. If a node goes 
down ungracefully, write operations on the cluster block for slightly longer 
than this timeout.

Once the system identifies that a node or nodes are disconnected, the 
remaining nodes cast a quorum vote. If most nodes connected before the 
disconnect remain active, that partition continues to operate. However, in the 
event of a network partition, some nodes may remain alive and active on each 
side of the disconnect. In this case, only the partition that achieves quorum 
continues to function, while any partitions without quorum transition to a 
non-primary state.

## Challenges of automatic failover

As a consequence, safe automatic failover is not possible in a two-node 
cluster. If one node fails, the remaining node becomes non-primary, which 
prevents the cluster from functioning effectively. 

Additionally, any cluster with an even number of nodes, such as two nodes 
located on different switches, faces the risk of a split-brain scenario. 
In a split-brain scenario, a network partition occurs, isolating the nodes 
from each other. Neither partition can achieve quorum because they lack 
the majority of votes needed to make decisions. As a result, both partitions 
transition to a non-primary state, leading to potential data inconsistencies 
and operational issues. This situation highlights the importance of designing 
clusters with an odd number of nodes to ensure reliable automatic failover 
and maintain quorum during failures.

## Best practices for automatic failover in cluster environments

For automatic failover, the rule of threes is recommended. This rule applies 
at various levels of your infrastructure, depending on how far the cluster is 
spread out to avoid single points of failure. 

For example, the following guidelines illustrate best practices for automatic failover in cluster environments:

* Ensure three nodes on a single switch.

* Distribute nodes evenly across at least three switches.

* Cover at least three networks.

* Include at least three data centers.

These guidelines help prevent split-brain situations and ensure that 
automatic failover functions correctly.

## Use an arbitrator for split-brain protection

If adding a third node, switch, network, or data center proves too expensive, 
you should consider using an arbitrator. An arbitrator acts as a voting member 
of the cluster that helps maintain quorum without storing any data. It can 
receive and relay replication messages between nodes, ensuring that the 
cluster remains informed about the state of each node. Instead of running 
the standard `mysqld` daemon, the arbitrator operates its lightweight 
daemon designed explicitly for this purpose.

Adding an arbitrator in a third location provides split-brain protection for a cluster that spans only two nodes or locations. This setup enhances the cluster's ability to maintain quorum and ensures 
more reliable operation during failures.

## Managing automatic failover and recovery in cluster environments

The rule of threes applies exclusively to automatic failover. In a two-node 
cluster or during an outage that leaves a minority of nodes active, the 
failure of one node results in the other becoming non-primary and refusing 
operations.

The following command acts as a critical recovery tool in scenarios where 
the cluster loses its Primary Component, which is the subset of nodes that 
can accept write operations and maintain data consistency. When the cluster 
loses this component, it cannot process write requests, leading to potential 
data loss or inconsistency.

This situation typically arises under these conditions:

| Scenario | Description |
|----------|-------------|
| Graceful shutdowns or crashes | All nodes in the cluster shut down or crash gracefully. |
| Network partition | A majority of nodes become unreachable, leaving the remaining nodes in a Non-Primary state and unable to accept writes. |

Use the command with caution and only on one node at a time, as incorrect 
usage can lead to split-brain scenarios. This results in multiple independent 
clusters with differing data, causing inconsistencies and corruption.

To recover the node from the non-primary state, execute the following command:

```shell
SET GLOBAL wsrep_provider_options='pc.bootstrap=true';
```

This command enables the bootstrap process for the primary component, allowing
the node and all connected nodes to become a primary cluster. By setting this
option to true, a node can initialize itself as the primary node when it
starts up without any other nodes available. However, ensure that no other
partition is operating as primary to avoid divergence, which can lead to two
databases that are impossible to re-merge automatically.

For instance, consider two data centers, one designated as primary and the
other for disaster recovery, each with an equal number of nodes. When an
extra arbitrator node operates solely in the primary data center, the
following high availability features become available:

* Automatically fail over any single node or nodes within the primary or
secondary data center.

* Prevent the primary data center from going down if the secondary fails,
thanks to the arbitrator.

* Keep the secondary data center in a non-primary state if the primary
data center fails.

* Instruct the secondary data center to bootstrap itself with a single
command after executing a disaster-recovery failover, while maintaining
control over the disaster-recovery failover process.