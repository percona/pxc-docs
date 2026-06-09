# Emergency quorum recovery (when nodes are up but traffic is blocked)

Nodes are running and accept connections, but MySQL refuses writes (and often reads) with errors such as `WSREP has not yet prepared node for application use`. The cluster has lost quorum or the *primary component*.

Why this happens: Each node has one vote; the cluster needs a majority to form a *primary component*. Only a primary accepts SQL. If one node leaves cleanly, votes decrease and the remaining nodes can form a primary—but if a node crashes (power loss, kill, kernel panic), the others still expect its vote until they time out. In a 3-node cluster, two nodes left after a crash often cannot form quorum (they expect three votes). Likewise, with two nodes left after a planned restart, a network flicker between those two can cause both to drop to non-primary; the cluster is then "online" but refuses every query.

Recovery options:

1. Restore connectivity so the remaining nodes can see each other and re-form a primary.
2. Bring the missing node(s) back so the cluster can form quorum again.
3. Emergency override (when you have confirmed the other nodes are really down): force one node to form a new primary so it can serve traffic; then start the other nodes so they rejoin.

## Emergency override: force a primary when traffic is blocked

Run the following on one node that is still up (connected as a user with sufficient privileges):

```sql
SET GLOBAL wsrep_provider_options='pc.bootstrap=YES';
```

The [`pc.bootstrap`](wsrep-provider-index.md#pcbootstrap) option makes that node form a new primary component. After the command runs, the node accepts writes; the cluster is effectively that one node until the others are started and rejoin (via IST or SST). Then start the other nodes so they join this primary.

!!! warning "Only when the other nodes are down"

    Run this override only when you have confirmed that the other nodes are actually down or unreachable. If another node is still primary elsewhere (for example, in another datacenter after a split), setting `pc.bootstrap=YES` on a second node creates two separate clusters with diverging data (split-brain). See [Scenario 5: Two nodes disappear from the cluster](crash-recovery.md#scenario-5-two-nodes-disappear-from-the-cluster) and [Scenario 7: Split brain](crash-recovery.md#scenario-7-the-cluster-loses-its-primary-state-due-to-split-brain) in Crash recovery.

For more support options, see [Get help from Percona](get-help.md).
