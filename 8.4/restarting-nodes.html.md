> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-xtradb-cluster/8.4/llms.txt).

# Restart the cluster nodes

This page is the entry point for restarts and recovery when nodes are managed directly (systemd, bare metal, or VMs). In PXC 8.4, Clone SST (`wsrep_sst_method=clone`) is the default rejoin path—set it once and most restarts are “clear port/process, start the service, let Clone run.” For Kubernetes, use the [Percona Operator for MySQL](https://docs.percona.com/percona-operator-for-mysql/pxc/) (PXC-based); it automates bootstrap and recovery. For full-cluster down (all nodes stopped, bootstrap from scratch), see [Crash recovery](https://docs.percona.com/percona-xtradb-cluster/8.4/crash-recovery.md).

Which problem do you have?

| Scenario                                                                                                                                                                                 | Topic                                                                                                                                                              |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Nodes are up but traffic is blocked — Cluster accepts connections but refuses every SQL query (`WSREP has not yet prepared node for application use`). Quorum or primary component lost. | [Emergency quorum recovery](https://docs.percona.com/percona-xtradb-cluster/8.4/emergency-quorum-recovery.md)                                                      |
| Nodes refuse to join — Joiner won’t sync, SST/Clone fails, or node never reaches `Synced`.                                                                                               | [SST/Clone failure recovery](https://docs.percona.com/percona-xtradb-cluster/8.4/sst-clone-failure-recovery.md)                                                    |
| A node was voted out for data inconsistency — Restart used IST and left the inconsistent dataset in place.                                                                               | [Rejoin after an inconsistency eviction](https://docs.percona.com/percona-xtradb-cluster/8.4/sst-clone-failure-recovery.md#rejoin-after-an-inconsistency-eviction) |
| Environmental blockers — AppArmor, systemd killing the process mid-SST, or firewalls blocking SST/GCOMM.                                                                                 | [Environmental blockers](https://docs.percona.com/percona-xtradb-cluster/8.4/environmental-blockers.md)                                                            |

Each topic is a separate page with step-by-step procedures. Start with the one that matches your situation; the pages cross-link where needed (for example, SST/Clone failure recovery points to Environmental blockers when the cause is AppArmor or systemd).
