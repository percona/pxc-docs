> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-xtradb-cluster/8.4/llms.txt).

# Migrate from MySQL Galera Cluster to Percona XtraDB Cluster

Percona XtraDB Cluster (PXC) uses the same [write-set](https://docs.percona.com/percona-xtradb-cluster/8.4/glossary.html#write-set) replication model as MySQL Galera Cluster, so a migration replaces the binaries and keeps the [datadir](https://docs.percona.com/percona-xtradb-cluster/8.4/glossary.html#datadir). The schemas, data, user accounts, and privileges carry over.

Starting in PXC 8.4.11, a PXC node can join a running MySQL Galera Cluster. You can convert nodes one at a time, in place, so the cluster stays available throughout and no extra hardware is required.

You can use either of the following methods:

- Stop one node in an N-node Galera Cluster, replace the binaries with PXC, start PXC on the existing datadir, and let that node rejoin
- Add a new PXC node to the Galera Cluster

Both approaches keep the cluster available. Earlier PXC versions cannot form a mixed cluster with Galera Cluster, so those migrations need a short full-cluster outage.

Set these before you start a PXC node

- Use `wsrep_sst_method=xtrabackup-v2`. PXC requires this SST method.
- PXC encrypts the Galera channel by default. Set [`pxc_encrypt_cluster_traffic`](https://docs.percona.com/percona-xtradb-cluster/8.4/encrypt-traffic.html#encrypt-pxc-traffic) so it matches the Galera Cluster before you start the PXC node.

Review [Get more help](https://docs.percona.com/percona-xtradb-cluster/8.4/get-help.html) for ways that we can work with you.

Back up your data before you start

Create and verify a physical backup with [Percona XtraBackup](https://docs.percona.com/percona-xtrabackup/) before you change any node. A backup is the only reliable way back if a step fails.

## Before you start

- An N-node MySQL Galera Cluster where every node is `Synced` and the cluster is `Primary`.
- PXC packages for your platform. See [Install on Debian or Ubuntu](https://docs.percona.com/percona-xtradb-cluster/8.4/apt.html) or [Install on Red Hat Enterprise Linux](https://docs.percona.com/percona-xtradb-cluster/8.4/yum.html).
- Root or `sudo` access on every node.

Reuse the Galera Cluster `my.cnf`. Keep [`wsrep`](https://docs.percona.com/percona-xtradb-cluster/8.4/glossary.html#wsrep) cluster identity the same (`wsrep_cluster_name`, `wsrep_cluster_address`, and the per-node `wsrep_node_name` and `wsrep_node_address`), then change only what PXC requires:

- `wsrep_sst_method=xtrabackup-v2`. PXC uses Percona XtraBackup for SST. Do not carry over `rsync`, MariaBackup, or a custom SST script. See [Percona XtraBackup SST configuration](https://docs.percona.com/percona-xtradb-cluster/8.4/xtrabackup-sst.html).
- `wsrep_provider` must point at the PXC Galera library. See [Configure nodes for write-set replication](https://docs.percona.com/percona-xtradb-cluster/8.4/configure-nodes.html).
- [`pxc_encrypt_cluster_traffic`](https://docs.percona.com/percona-xtradb-cluster/8.4/encrypt-traffic.html#encrypt-pxc-traffic) must match the Galera Cluster. PXC enables it by default.

Traffic encryption

PXC enables [`pxc_encrypt_cluster_traffic`](https://docs.percona.com/percona-xtradb-cluster/8.4/encrypt-traffic.html#encrypt-pxc-traffic) by default. If your Galera Cluster does not encrypt replication traffic, set `pxc_encrypt_cluster_traffic=OFF` on the PXC nodes. Otherwise, distribute identical certificates to every node. See [Encrypt PXC traffic](https://docs.percona.com/percona-xtradb-cluster/8.4/encrypt-traffic.html).

## Migrate with PXC 8.4.11 or newer

Introduced in Percona XtraDB Cluster 8.4.11, you can join a running Galera Cluster. Choose either scenario. Both keep the cluster available throughout.

### Scenario 1: Convert each node in place

The node keeps its datadir, so it rejoins with [IST](https://docs.percona.com/percona-xtradb-cluster/8.4/glossary.html#ist) instead of a full data copy. This scenario needs no extra hardware.

1. Stop one Galera Cluster node.
1. Install PXC on that host. Keep the Galera Cluster `my.cnf`, apply the adjustments in [Before you start](#before-you-start), and start the node. It joins the cluster with IST.
1. Wait for the node to reach `Synced`, then repeat steps 1 and 2 on the next node.

### Scenario 2: Add PXC nodes and retire Galera Cluster nodes

Use this scenario when you also move to new hosts. Each new node joins with [SST](https://docs.percona.com/percona-xtradb-cluster/8.4/glossary.html#sst), so plan for the transfer time and the load on the [donor](https://docs.percona.com/percona-xtradb-cluster/8.4/glossary.html#donor-node).

1. Start a fresh PXC node and join it to the existing Galera Cluster. Copy the Galera Cluster `my.cnf`, apply the same adjustments, and start the node. It joins with SST.
1. Wait for the new node to reach `Synced`.
1. Decommission one Galera Cluster node.
1. Repeat steps 1 through 3 until every Galera Cluster node is replaced.

## Migrate with PXC earlier than 8.4.11

These PXC versions cannot form a cluster with Galera Cluster nodes, so you bootstrap a new cluster from an existing datadir during a maintenance window.

1. Stop application writes, then stop MySQL on all Galera Cluster nodes.

1. Using the datadir of the last node you stopped, [bootstrap](https://docs.percona.com/percona-xtradb-cluster/8.4/glossary.html#bootstrap) a new cluster with the PXC binaries:

   ```
   sudo systemctl start mysql@bootstrap.service
   ```

   See [Bootstrap the first node](https://docs.percona.com/percona-xtradb-cluster/8.4/bootstrap.html).

1. Delete the `grastate.dat` file from the datadir of every other node. This forces those nodes to join with [SST](https://docs.percona.com/percona-xtradb-cluster/8.4/glossary.html#sst).

1. Start the other nodes one at a time with a normal start. Wait for each node to reach `Synced` before you start the next one.

   ```
   sudo systemctl start mysql
   ```

1. Stop the bootstrap unit on the first node and start it normally so it uses `wsrep_cluster_address`:

   ```
   sudo systemctl stop mysql@bootstrap.service
   sudo systemctl start mysql
   ```

## Verify the migration

Run these checks on every node:

```
SELECT VERSION();
SHOW STATUS LIKE 'wsrep_cluster_status';
SHOW STATUS LIKE 'wsrep_cluster_size';
SHOW STATUS LIKE 'wsrep_local_state_comment';
```

`VERSION()` must identify Percona XtraDB Cluster, `wsrep_cluster_status` must be `Primary`, `wsrep_local_state_comment` must be `Synced`, and `wsrep_cluster_size` must equal your node count.

Then [verify replication](https://docs.percona.com/percona-xtradb-cluster/8.4/verify-replication.html) with a write on one node and a read on the others.

## After you migrate

- [PXC Strict Mode](https://docs.percona.com/percona-xtradb-cluster/8.4/glossary.html#strict-mode) defaults to `ENFORCING` and blocks unsupported operations. If your workload needs it, start with `PERMISSIVE`, validate the application, then move to `ENFORCING`. See [Percona XtraDB Cluster strict mode](https://docs.percona.com/percona-xtradb-cluster/8.4/strict-mode.html).
- Replace a Codership `garbd` arbitrator with the matching Percona package. See [Set up Galera arbitrator](https://docs.percona.com/percona-xtradb-cluster/8.4/garbd-howto.html).
- Point backup jobs and monitoring at the new cluster. [Percona Monitoring and Management](https://docs.percona.com/percona-monitoring-and-management/) is the recommended option. See [Monitor the cluster](https://docs.percona.com/percona-xtradb-cluster/8.4/monitoring.html).
- Keep the pre-migration backup until the cluster runs cleanly in production for an agreed observation period.

To change major versions, complete the migration first, then use [Upgrade Percona XtraDB Cluster](https://docs.percona.com/percona-xtradb-cluster/8.4/upgrade-guide.html).

## Troubleshooting

| Symptom                                          | Likely cause                                                                            | What to check                                                                                                                                                                                                                                                                                     |
| ------------------------------------------------ | --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Node fails to join                               | Encryption mismatch                                                                     | `pxc_encrypt_cluster_traffic` must match on every node. Certificates must be identical. See [Encrypt PXC traffic](https://docs.percona.com/percona-xtradb-cluster/8.4/encrypt-traffic.html).                                                                                                      |
| Node starts its own cluster                      | Wrong `wsrep_cluster_address` or an accidental bootstrap                                | Bootstrap only one node, and only when you form a new cluster.                                                                                                                                                                                                                                    |
| SST fails                                        | Unsupported SST method                                                                  | `wsrep_sst_method` must be `xtrabackup-v2`. See [`wsrep_sst_allowed_methods`](https://docs.percona.com/percona-xtradb-cluster/8.4/wsrep-system-index.html#wsrep_sst_allowed_methods) and [XtraBackup SST configuration](https://docs.percona.com/percona-xtradb-cluster/8.4/xtrabackup-sst.html). |
| Client login fails                               | Authentication plugin mismatch                                                          | Accounts keep the plugin stored in `mysql.user`. Older connectors need TLS or RSA key exchange for `caching_sha2_password`. With ProxySQL, use 2.6.2 or later. See [Load balance with ProxySQL](https://docs.percona.com/percona-xtradb-cluster/8.4/load-balance-proxysql.html).                  |
| Application errors after migration               | Strict Mode rejects statements                                                          | `pxc_strict_mode` and the error log. See [Percona XtraDB Cluster strict mode](https://docs.percona.com/percona-xtradb-cluster/8.4/strict-mode.html).                                                                                                                                              |
| Configuration missing after a yum or dnf install | RHEL replaced `my.cnf`                                                                  | Restore your settings from `/etc/my.cnf.rpmsave`.                                                                                                                                                                                                                                                 |
| Cluster size does not recover                    | [Quorum](https://docs.percona.com/percona-xtradb-cluster/8.4/glossary.html#quorum) loss | See [Cluster failover](https://docs.percona.com/percona-xtradb-cluster/8.4/failover.html) and [Emergency quorum recovery](https://docs.percona.com/percona-xtradb-cluster/8.4/emergency-quorum-recovery.html).                                                                                    |

## Further reading

See also

- [How to Migrate from MySQL Galera Cluster to Percona XtraDB Cluster](https://www.percona.com/blog/migrate-mysql-galera-cluster-to-percona-xtradb-cluster/)
- [Configure nodes for write-set replication](https://docs.percona.com/percona-xtradb-cluster/8.4/configure-nodes.html)
- [Upgrade Percona XtraDB Cluster](https://docs.percona.com/percona-xtradb-cluster/8.4/upgrade-guide.html)
- [Percona XtraDB Cluster limitations](https://docs.percona.com/percona-xtradb-cluster/8.4/limitation.html)
