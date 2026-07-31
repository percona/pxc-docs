# Migrate from MySQL Galera Cluster to Percona XtraDB Cluster

Percona XtraDB Cluster (PXC) uses the same [write-set](glossary.md#write-set) replication model as MySQL Galera Cluster, so a migration replaces the binaries and keeps the [datadir](glossary.md#datadir). Schemas, data, user accounts, and privileges carry over.

The path you use depends on the Galera Cluster version. From Galera Cluster 8.4.11 or 9.7.2, a PXC node can join a running Galera Cluster, so you can migrate one node at a time with no cluster outage. Earlier versions require a short full-cluster outage.

--8<--- "get-help-snip.md"

!!! warning "Back up your data before you start"

    Create and verify a physical backup with [Percona XtraBackup :octicons-link-external-16:](https://docs.percona.com/percona-xtrabackup/) before you change any node. A backup is the only reliable way back if a step fails.

## Before you start

* An N-node MySQL Galera Cluster where every node is `Synced` and the cluster is `Primary`.

* PXC packages for your platform. See [Install on Debian or Ubuntu](apt.md) or [Install on Red Hat Enterprise Linux](yum.md). Use the same version as your Galera Cluster when you can. A different version works, but it adds risk.

* Root or `sudo` access on every node.

Keep your [`wsrep`](glossary.md#wsrep) cluster identity settings the same when you write the PXC configuration: `wsrep_cluster_name`, `wsrep_cluster_address`, and the per-node `wsrep_node_name` and `wsrep_node_address`.

!!! note "Traffic encryption"

    PXC enables [`pxc_encrypt_cluster_traffic`](encrypt-traffic.md#encrypt-pxc-traffic) by default. If your Galera Cluster does not encrypt replication traffic, set `pxc_encrypt_cluster_traffic=OFF` on the PXC nodes. Otherwise, distribute identical certificates to every node. See [Encrypt PXC traffic](encrypt-traffic.md).

## Migrate from Galera Cluster earlier than 8.4.11 or 9.7.2

These versions cannot form a cluster with a PXC node, so you bootstrap a new cluster from an existing datadir during a maintenance window.
{.power-number}

1. Stop application writes, then stop MySQL on all Galera Cluster nodes.

2. Using the datadir of the last node you stopped, [bootstrap](glossary.md#bootstrap) a new cluster with the PXC binaries:

    ```shell
    sudo systemctl start mysql@bootstrap.service
    ```

    See [Bootstrap the first node](bootstrap.md).

3. Delete the `grastate.dat` file from the datadir of every other node. This forces those nodes to join with [SST](glossary.md#sst).

4. Start the other nodes one at a time with a normal start. Wait for each node to reach `Synced` before you start the next one.

    ```shell
    sudo systemctl start mysql
    ```

5. Stop the bootstrap unit on the first node and start it normally so it uses `wsrep_cluster_address`:

    ```shell
    sudo systemctl stop mysql@bootstrap.service
    sudo systemctl start mysql
    ```

## Migrate from Galera Cluster 8.4.11 or later, or 9.7.2 or later

A PXC node can join a running Galera Cluster of these versions. Choose either scenario. Both keep the cluster available throughout.

### Scenario 1: Convert each node in place

The node keeps its datadir, so it rejoins with [IST](glossary.md#ist) instead of a full data copy. This scenario needs no extra hardware.
{.power-number}

1. Stop one Galera Cluster node.

2. Install PXC on that host, write the PXC configuration, and start the node. It joins the cluster with IST.

3. Wait for the node to reach `Synced`, then repeat steps 1 and 2 on the next node.

### Scenario 2: Add PXC nodes and retire Galera Cluster nodes

Use this scenario when you also move to new hosts. Each new node joins with [SST](glossary.md#sst), so plan for the transfer time and the load on the [donor](glossary.md#donor-node).
{.power-number}

1. Start a fresh PXC node and join it to the existing Galera Cluster. The node joins with SST.

2. Wait for the new node to reach `Synced`.

3. Decommission one Galera Cluster node.

4. Repeat steps 1 through 3 until every Galera Cluster node is replaced.

## Verify the migration

Run these checks on every node:

```sql
SELECT VERSION();
SHOW STATUS LIKE 'wsrep_cluster_status';
SHOW STATUS LIKE 'wsrep_cluster_size';
SHOW STATUS LIKE 'wsrep_local_state_comment';
```

`VERSION()` must identify Percona XtraDB Cluster, `wsrep_cluster_status` must be `Primary`, `wsrep_local_state_comment` must be `Synced`, and `wsrep_cluster_size` must equal your node count.

Then [verify replication](verify-replication.md) with a write on one node and a read on the others.

## After you migrate

* PXC uses Percona XtraBackup for SST. Set `wsrep_sst_method=xtrabackup-v2` and configure the SST user. See [Percona XtraBackup SST configuration](xtrabackup-sst.md). Do not carry over `rsync`, MariaBackup, or a custom SST script.

* [PXC Strict Mode](glossary.md#strict-mode) defaults to `ENFORCING` and blocks unsupported operations. If your workload needs it, start with `PERMISSIVE`, validate the application, then move to `ENFORCING`. See [Percona XtraDB Cluster strict mode](strict-mode.md).

* Replace a Codership `garbd` arbitrator with the matching Percona package. See [Set up Galera arbitrator](garbd-howto.md).

* Point backup jobs and monitoring at the new cluster. [Percona Monitoring and Management :octicons-link-external-16:](https://docs.percona.com/percona-monitoring-and-management/) is the recommended option. See [Monitor the cluster](monitoring.md).

* Keep the pre-migration backup until the cluster runs cleanly in production for an agreed observation period.

To change major versions, complete the migration first, then use [Upgrade Percona XtraDB Cluster](upgrade-guide.md).

## Troubleshooting

| Symptom | Likely cause | What to check |
| ------- | ------------ | ------------- |
| Node fails to join | Encryption mismatch | `pxc_encrypt_cluster_traffic` must match on every node. Certificates must be identical. See [Encrypt PXC traffic](encrypt-traffic.md). |
| Node starts its own cluster | Wrong `wsrep_cluster_address` or an accidental bootstrap | Bootstrap only one node, and only when you form a new cluster. |
| SST fails | Unsupported SST method or SST user privileges | `wsrep_sst_method` must be `xtrabackup-v2` or `clone`. See [`wsrep_sst_allowed_methods`](wsrep-system-index.md#wsrep_sst_allowed_methods) and [XtraBackup SST configuration](xtrabackup-sst.md). |
| Client login fails | Authentication plugin mismatch | Accounts keep the plugin stored in `mysql.user`. Older connectors need TLS or RSA key exchange for `caching_sha2_password`. With ProxySQL, use 2.6.2 or later. See [Load balance with ProxySQL](load-balance-proxysql.md). |
| Application errors after migration | Strict Mode rejects statements | `pxc_strict_mode` and the error log. See [Percona XtraDB Cluster strict mode](strict-mode.md). |
| Configuration missing after a yum or dnf install | RHEL replaced `my.cnf` | Restore your settings from `/etc/my.cnf.rpmsave`. |
| Cluster size does not recover | [Quorum](glossary.md#quorum) loss | See [Cluster failover](failover.md) and [Emergency quorum recovery](emergency-quorum-recovery.md). |

## Further reading

!!! admonition "See also"

    * [How to Migrate from MySQL Galera Cluster to Percona XtraDB Cluster :octicons-link-external-16:](https://www.percona.com/blog/migrate-mysql-galera-cluster-to-percona-xtradb-cluster/)

    * [Configure nodes for write-set replication](configure-nodes.md)

    * [Upgrade Percona XtraDB Cluster](upgrade-guide.md)

    * [Percona XtraDB Cluster limitations](limitation.md)
