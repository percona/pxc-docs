# Restart the cluster nodes

To restart a cluster node, shut down MySQL and restarting it.
The node should leave the cluster
(and the total vote count for [quorum](glossary.md#quorum) should decrement).

When it rejoins, the node should synchronize using [IST](glossary.md#ist).
If the set of changes needed for IST are not found in the `gcache` file
on any other node in the entire cluster,
then [SST](glossary.md#sst) will be performed instead.
Therefore, restarting cluster nodes for rolling configuration changes
or software upgrades is rather simple from the cluster’s perspective.

!!! note

    If you restart a node with an invalid configuration change that prevents MySQL from loading, Galera will drop the node’s state and force an SST for that node.

!!! note

    If MySQL fails for any reason, it will not remove its PID file (which is by design deleted only on clean shutdown). Obviously server will not restart if existing PID file is present. So in case of encountered MySQL failure for any reason with the relevant records in log, PID file should be removed manually.
