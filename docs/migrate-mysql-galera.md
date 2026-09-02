# Migrate from MySQL Galera Cluster to Percona XtraDB Cluster

Migrate an existing MySQL Galera Cluster (Codership MySQL-wsrep build) to Percona XtraDB Cluster (PXC).

Schema, system tables, and user accounts typically carry over with the data directory.

--8<--- "get-help-snip.md"

## Prerequisites

Complete the following tasks before you start:

* Stop and start MySQL services

* Edit `my.cnf` or the equivalent file on your system

* Create and restore backups

* Use Galera [`wsrep`](glossary.md#wsrep) status variables, [State Snapshot Transfer (SST)](glossary.md#sst), and [quorum](glossary.md#quorum)

## What changes and what does not

| Area | Result |
| ---- | ------ |
| Database engine | MySQL-compatible. PXC runs on Percona Server for MySQL. Applications and schemas work without a rewrite. A return to Oracle MySQL after you use Percona features is not a simple reverse of this migration. |
| Schema and data | Carry over. Recreate schemas or reload data only for a restore-based cutover. |
| User accounts and privileges | Carry over with the data directory. |
| Clustering model | Same Galera [write-set](glossary.md#write-set) replication model on the Percona Galera fork. |
| Configuration | Update Galera library paths, [SST](glossary.md#sst) settings, and PXC defaults for traffic encryption and [Strict Mode](glossary.md#strict-mode). |
| Tools | Use [Percona XtraBackup :octicons-link-external-16:](https://docs.percona.com/percona-xtrabackup/), [Percona Monitoring and Management :octicons-link-external-16:](https://docs.percona.com/percona-monitoring-and-management/), and [Percona Toolkit :octicons-link-external-16:](https://docs.percona.com/percona-toolkit/). |

!!! warning "Do not change major versions during the migration"

    Match PXC to your MySQL major version.

    For example, migrate MySQL Galera 8.0 to PXC 8.0 first.

    After the cluster is stable on PXC, follow [Upgrade Percona XtraDB Cluster](upgrade-guide.md) for an upgrade to 8.4.

    A migration and a major-version change in one step raise risk and make troubleshooting harder.

## Migration method

Use a [maintenance-window cutover](#migrate-during-a-maintenance-window).

Stop the cluster, replace packages on all nodes, then [bootstrap](glossary.md#bootstrap) Percona XtraDB Cluster.

!!! warning "Rolling and in-place migration are not supported"

    You cannot replace one MySQL Galera node with PXC and rejoin that node to a mixed cluster.

    Work to enable rolling and in-place migration is tracked in [PXC-5286](https://perconadev.atlassian.net/browse/PXC-5286).

    That work is planned for a future 8.4 and 9.7 release.

    Do not attempt a rolling package swap on a production cluster.

Preparation for the maintenance-window cutover includes the following:

* Inventory

* Staging rehearsal

* Verified backup

## Before you begin

Complete these items before any production change:

* Root or `sudo` access on every cluster node

* A maintenance plan and a tested rollback path (verified backup or an untouched source cluster)

* Open network ports for Galera and MySQL: `3306`, `4444`, `4567`, `4568`

* Disk space for a full backup and temporary [SST](glossary.md#sst) traffic

* Application owners ready to validate reads and writes after cutover

!!! note

    Plan for a full cluster outage during the maintenance window.

    See [Get started with Percona XtraDB Cluster](get-started-cluster.md) for cluster sizing guidance after the migration.

## Task 1: Inventory your current cluster

Record the cluster state.

Use the record to choose a compatible PXC release and to keep required settings.

{.power-number}

1. **Every node.** Record the MySQL and Galera versions:

    ```sql
    SELECT VERSION();
    SHOW GLOBAL STATUS LIKE 'wsrep_provider_version';
    SHOW GLOBAL STATUS LIKE 'wsrep_protocol_version';
    ```

2. **Every node.** Record cluster membership and health:

    ```sql
    SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size';
    SHOW GLOBAL STATUS LIKE 'wsrep_cluster_status';
    SHOW GLOBAL STATUS LIKE 'wsrep_local_state_comment';
    SHOW GLOBAL STATUS LIKE 'wsrep_ready';
    ```

    ??? example "Expected output on a healthy three-node cluster"

        ```{.text .no-copy}
        +----------------------------+----------+
        | Variable_name              | Value    |
        +----------------------------+----------+
        | wsrep_cluster_size         | 3        |
        | wsrep_cluster_status       | Primary  |
        | wsrep_local_state_comment  | Synced   |
        | wsrep_ready                | ON       |
        +----------------------------+----------+
        ```

    **Failure criteria:** Any node that is not `Primary`, `Synced`, and `ON` is not ready for migration planning.

3. Export the Galera configuration you must keep:

    ```sql
    SHOW VARIABLES LIKE 'wsrep%';
    SHOW VARIABLES LIKE 'pxc%';
    ```

    Save a copy of each node configuration file.

    Common paths include `/etc/my.cnf` and `/etc/mysql/mysql.conf.d/mysqld.cnf`.

4. Record topology details:

    * Hostnames and IP addresses for every node

    * Galera arbitrator use ([garbd](garbd-howto.md)), if any

    * Load balancer or proxy in front of the cluster (ProxySQL or HAProxy)

    * SST method and credentials

    * Replication traffic encryption status

    * Async replicas that read from a Galera node

5. Confirm the target PXC version matches your MySQL major version.

    Review [Important changes in Percona XtraDB Cluster {{vers}}](upgrade-guide.md#important-changes-in-percona-xtradb-cluster-84) before any later upgrade to {{vers}}.

## Task 2: Plan for PXC defaults that differ from MySQL Galera

PXC enables security and safety settings that many MySQL Galera deployments leave off.

Plan for these settings before the first package swap.

### Traffic encryption

In PXC {{vers}}, [`pxc_encrypt_cluster_traffic`](encrypt-traffic.md#encrypt-pxc-traffic) is enabled by default.

A PXC node that expects encrypted replication traffic cannot join a cluster that uses unencrypted Galera traffic.

For a maintenance-window cutover, distribute identical certificates to every node before you bootstrap PXC.

Enable `pxc_encrypt_cluster_traffic` when you start the PXC cluster.

See [Encrypt PXC traffic](encrypt-traffic.md).

### PXC Strict Mode

[PXC Strict Mode](glossary.md#strict-mode) defaults to `ENFORCING` and blocks unsupported operations.

If the workload uses features that Strict Mode rejects, start migrated nodes with `PERMISSIVE` or `MASTER`.

Validate the application.

Then set Strict Mode to `ENFORCING`.

For configuration details, see [Percona XtraDB Cluster strict mode](strict-mode.md).

### Galera library path and SST method

Update the following settings when you move to PXC:

* Set `wsrep_provider` to the Percona Galera 4 library:

    * Debian or Ubuntu: `/usr/lib/galera4/libgalera_smm.so`

    * Red Hat Enterprise Linux (RHEL) derived systems: `/usr/lib64/galera4/libgalera_smm.so`

* Set `wsrep_sst_method=xtrabackup-v2`.

    PXC uses Percona XtraBackup for [SST](glossary.md#sst).

    Configure SST authentication as described in [Percona XtraBackup SST configuration](xtrabackup-sst.md).

## Task 3: Build a staging cluster and rehearse

Do not run the first migration attempt on production.

{.power-number}

1. Build a staging cluster that mirrors production node count, MySQL and Galera version, disk layout, and key `wsrep_*` settings.

2. Restore a recent production backup into staging.

    You can also clone production with an approved method.

3. Rehearse the full maintenance-window cutover on staging.

4. Run basic application tests:

    * Logins

    * Writes

    * Schema changes you run in production

    * Backup and restore

    * Failover of one node

5. Time each step.

    Use those times to size the production maintenance window.

6. Practice rollback.

    Restore from backup, or return to the pre-migration packages and data, until the procedure is reliable.

## Task 4: Create a verified backup

!!! warning "Required: Create a backup before migrating"

    Create and verify a backup before you remove any MySQL Galera packages.

{.power-number}

1. Create a full backup with [Percona XtraBackup :octicons-link-external-16:](https://docs.percona.com/percona-xtrabackup/) or another proven physical backup method.

2. Store the backup off the cluster nodes when possible.

3. Restore the backup to a scratch host.

    Confirm that MySQL starts and that critical schemas are present.

4. Keep this backup until the migrated PXC cluster runs cleanly in production for an agreed observation period.

## Task 5: Prepare the PXC configuration

Prepare a configuration that keeps your cluster identity and matches PXC defaults.

Complete this work before you install PXC packages on a node.

{.power-number}

1. Copy the MySQL Galera configuration to a backup file:

    ```shell
    sudo cp /etc/my.cnf /etc/my.cnf.galera.bak
    ```

    Use the path for your distribution if the path differs.

2. Keep these values consistent across the migration:

    * `wsrep_cluster_name`

    * `wsrep_cluster_address` (all node addresses)

    * `wsrep_node_name` and `wsrep_node_address` (per node)

    * `server-id` (unique per node when you use async replication)

3. Update Galera and PXC settings.

    The following fragment is an example only.

    Adjust paths, addresses, and options for each node and OS:

    ```text
    [mysqld]
    binlog_format=ROW
    default_storage_engine=InnoDB
    innodb_autoinc_lock_mode=2

    wsrep_provider=/usr/lib64/galera4/libgalera_smm.so
    wsrep_cluster_name=my-cluster
    wsrep_cluster_address=gcomm://192.168.70.61,192.168.70.62,192.168.70.63
    wsrep_node_name=node1
    wsrep_node_address=192.168.70.61

    wsrep_sst_method=xtrabackup-v2
    # Configure SST user credentials per xtrabackup-sst.md

    pxc_strict_mode=PERMISSIVE
    ```

    On Debian or Ubuntu, a common `wsrep_provider` path is `/usr/lib/galera4/libgalera_smm.so`.

4. If you enable encryption, place identical certificates on every node.

    Configure the certificates as described in [Encrypt PXC traffic](encrypt-traffic.md).

    Store certificates outside the data directory.

    One common path is `/etc/mysql/certs`.

See [Configure nodes for write-set replication](configure-nodes.md) for a full configuration procedure.

---

## Migrate during a maintenance window

Plan a full cluster outage for this cutover.

Do not delete the data directory when you remove MySQL Galera packages.

### Estimated downtime

The cluster is unavailable from the moment you stop application writes until validation passes and you return traffic to the proxy.

| Phase | Typical duration | Notes |
| ----- | ---------------- | ----- |
| Stop writes and stop MySQL on all nodes | 5 to 15 minutes | Depends on connection drain and clean shutdown |
| Package removal and PXC install on all nodes | 15 to 45 minutes per node if done in sequence; less if you prepare repositories in advance | Parallel work on nodes is fine while MySQL is stopped |
| Bootstrap first node and join remaining nodes | 10 to 60 minutes or more | Join time grows with data size when a node needs [SST](glossary.md#sst) |
| Validation and application checks | 15 to 60 minutes | Include failover and acceptance tests |

Use staging rehearsal timings for your data size and hardware.

Do not use these ranges as a production SLA.

Add buffer for rollback if a [checkpoint](#checkpoints-and-failure-criteria) fails.

### Where each step runs

This procedure marks the scope of every step:

| Label | Meaning |
| ----- | ------- |
| **Once** | Run on one host or one time for the whole cutover |
| **Every node** | Repeat on each cluster node before you continue |
| **Bootstrap node only** | Run only on the node you chose to initialize the cluster |
| **Joiner node** | Run on each remaining node, one node at a time |

Do not bootstrap more than one node.

### Adapt commands to your platform

!!! important

    Shell commands, service unit names, configuration paths, and package names vary by operating system and by how MySQL Galera was installed.

    The commands in this section are examples.

    They are not one complete, tested path for every platform.

    Build your runbook from the following sources:

    * Your inventory from [Task 1](#task-1-inventory-your-current-cluster)

    * The PXC install procedure for your OS: [Debian or Ubuntu](apt.md) or [Red Hat Enterprise Linux](yum.md)

    * [Bootstrap the first node](bootstrap.md) and [Configure nodes for write-set replication](configure-nodes.md)

    Rehearse the exact commands on staging before production.

### Checkpoints and failure criteria

At each checkpoint, compare results to the pass criteria.

| Checkpoint | Pass criteria | Failure criteria | Action on failure |
| ---------- | ------------- | ---------------- | ----------------- |
| C1: Pre-cutover health | Every node: `wsrep_cluster_status=Primary`, `wsrep_local_state_comment=Synced`, `wsrep_ready=ON` | Any node not `Synced` or not `Primary` | Do not start the outage. Fix cluster health first. |
| C2: MySQL stopped | Every node: MySQL service is inactive; data directory still present | Service still running, or data directory missing | Do not remove packages. Stop the service or restore the data directory from backup. |
| C3: Packages converted | Every node: PXC packages installed; prepared `my.cnf` in place; data directory unchanged | Install failed, wrong major version, or config missing | Fix the node before bootstrap. Do not bootstrap a partial set of nodes. |
| C4: Bootstrap node ready | Bootstrap node only: `Synced`, `Primary`, `wsrep_cluster_size=1`, `wsrep_ready=ON` | Node not `Synced`, not `Primary`, or service crash | Check the error log. Do not start joiner nodes. See [Rollback](#rollback). |
| C5: Each joiner synced | After each joiner start: that node `Synced` and `ON`; cluster size increases by one | Joiner stuck in donor/desync, SST error, or size does not increase | Stop adding nodes. Fix SST, encryption, or config. See [Troubleshooting checklist](#troubleshooting-checklist). |
| C6: Cluster complete | Every node: `Primary`, `Synced`, `ON`; `wsrep_cluster_size` equals node count | Size mismatch or any node not `Synced` | Do not return application traffic. |
| C7: Validation passed | Replication check, failover check, and application acceptance succeed | Any validation failure | Keep traffic off the cluster. Fix or roll back. |

Continue to the next phase only when the current checkpoint passes.

### Stop write traffic on the cluster

{.power-number}

1. **Once.** Announce the maintenance window and stop application writes.

    You can also stop the application.

2. **Every node.** Confirm cluster health before the outage.

    ```sql
    SHOW STATUS LIKE 'wsrep_cluster_status';
    SHOW STATUS LIKE 'wsrep_local_state_comment';
    SHOW STATUS LIKE 'wsrep_ready';
    SHOW STATUS LIKE 'wsrep_cluster_size';
    ```

    ??? example "Expected output on a healthy three-node cluster"

        ```{.text .no-copy}
        +----------------------------+----------+
        | Variable_name              | Value    |
        +----------------------------+----------+
        | wsrep_cluster_status       | Primary  |
        | wsrep_local_state_comment  | Synced   |
        | wsrep_ready                | ON       |
        | wsrep_cluster_size         | 3        |
        +----------------------------+----------+
        ```

    **Checkpoint C1:** Continue only when every node matches the pass criteria.

3. **Once.** Create a final backup if the previous backup does not meet your recovery point objective.

    Verify the backup before you stop MySQL.

4. **Every node.** Stop MySQL.

    Start with non-primary writers if your runbook requires an order.

    Example service command:

    ```shell
    sudo systemctl stop mysql
    ```

    If your service unit name differs (for example `mysqld`), use that name.

    Confirm the service is stopped:

    ```shell
    sudo systemctl is-active mysql
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        inactive
        ```

    Confirm the data directory still exists (common path `/var/lib/mysql`).

    **Checkpoint C2:** Continue only when MySQL is stopped on every node and the data directory remains.

### Convert every node to PXC

**Scope: Every node.** Complete all steps on node 1, then node 2, then node 3 (or in parallel while MySQL is stopped).

Do not start MySQL until every node finishes this phase and [Checkpoint C3](#checkpoints-and-failure-criteria) passes.

{.power-number}

1. **Every node.** Copy the configuration to a backup file and note the data directory path.

    A common path is `/var/lib/mysql`.

2. **Every node.** List the installed MySQL Galera packages and record the exact names.

    === "Debian or Ubuntu"

        ```shell
        dpkg -l | grep -E 'mysql|galera|wsrep'
        ```

    === "Red Hat Enterprise Linux"

        ```shell
        rpm -qa | grep -Ei 'mysql|galera|wsrep'
        ```

3. **Every node.** Remove only the MySQL Galera server packages that your inventory lists.

    Keep the data directory.

    Prefer package removal that does not delete configuration backups.

    Example (replace `PACKAGE_NAME` with names from your inventory):

    === "Debian or Ubuntu"

        ```shell
        sudo apt-get remove PACKAGE_NAME PACKAGE_NAME
        ```

        Do not run `purge` against the data directory until configuration backups exist.

    === "Red Hat Enterprise Linux"

        ```shell
        sudo rpm -e --nodeps PACKAGE_NAME PACKAGE_NAME
        ```

        Do not delete `/var/lib/mysql`.

4. **Every node.** Install Percona XtraDB Cluster for your target major version.

    Follow the full install procedure for your OS:

    * [Install on Debian or Ubuntu](apt.md)

    * [Install on Red Hat Enterprise Linux](yum.md)

    Select the Percona repository that matches your MySQL major version.

    For example, migrate MySQL Galera 8.0 to a PXC 8.0 repository before any later upgrade to {{vers}}.

    !!! note

        On RHEL-derived systems, the installer may save the previous configuration as `/etc/my.cnf.rpmsave` and install a default `/etc/my.cnf`.

        Restore your prepared PXC settings before you start the node.

5. **Every node.** Install your prepared PXC configuration.

    Keep `wsrep_cluster_name` and node addresses consistent across nodes.

    Set a unique `wsrep_node_name` and `wsrep_node_address` on each node.

6. **Every node.** Place identical encryption certificates on the node if you enable `pxc_encrypt_cluster_traffic`.

    **Checkpoint C3:** Continue only when every node has PXC installed, the prepared configuration is in place, and the data directory is intact.

    Do not bootstrap until C3 passes on all nodes.

### Bootstrap and form the PXC cluster

{.power-number}

1. **Once.** Choose the node with the most current data.

    Any node that was `Synced` when you stopped the cluster is a valid choice.

    That node is the [bootstrap](glossary.md#bootstrap) node.

    Record which host you chose.

2. **Bootstrap node only.** Bootstrap the first node.

    Example:

    ```shell
    sudo systemctl start mysql@bootstrap.service
    ```

    See [Bootstrap the first node](bootstrap.md) for the supported procedure on your platform.

3. **Bootstrap node only.** Verify the bootstrap node.

    ```sql
    SHOW STATUS LIKE 'wsrep_local_state_comment';
    SHOW STATUS LIKE 'wsrep_cluster_status';
    SHOW STATUS LIKE 'wsrep_cluster_size';
    SHOW STATUS LIKE 'wsrep_ready';
    SHOW STATUS LIKE 'wsrep_connected';
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        +----------------------------+----------+
        | Variable_name              | Value    |
        +----------------------------+----------+
        | wsrep_local_state_comment  | Synced   |
        | wsrep_cluster_status       | Primary  |
        | wsrep_cluster_size         | 1        |
        | wsrep_ready                | ON       |
        | wsrep_connected            | ON       |
        +----------------------------+----------+
        ```

    **Checkpoint C4:** Continue only when the bootstrap node matches this output.

    Do not start other nodes if C4 fails.

4. **Joiner node.** Start the next node with a normal start (not bootstrap).

    Example:

    ```shell
    sudo systemctl start mysql
    ```

    Watch the error log on the joiner until the node reaches `Synced`.

    ```sql
    SHOW STATUS LIKE 'wsrep_local_state_comment';
    SHOW STATUS LIKE 'wsrep_ready';
    SHOW STATUS LIKE 'wsrep_cluster_size';
    ```

    ??? example "Expected output after the first joiner succeeds on a three-node cluster"

        ```{.text .no-copy}
        +----------------------------+----------+
        | Variable_name              | Value    |
        +----------------------------+----------+
        | wsrep_local_state_comment  | Synced   |
        | wsrep_ready                | ON       |
        | wsrep_cluster_size         | 2        |
        +----------------------------+----------+
        ```

    **Checkpoint C5:** Continue to the next joiner only when this joiner is `Synced` and cluster size increased by one.

5. **Joiner node.** Repeat step 4 for each remaining node.

    Start one joiner at a time.

6. **Bootstrap node only.** After the cluster has at least two members, stop the bootstrap unit and start the node normally.

    The node then uses the standard `wsrep_cluster_address`.

    Example:

    ```shell
    sudo systemctl stop mysql@bootstrap.service
    sudo systemctl start mysql
    ```

    Confirm the former bootstrap node returns to `Synced`.

7. **Every node.** Confirm final cluster membership.

    ```sql
    SHOW STATUS LIKE 'wsrep_cluster_status';
    SHOW STATUS LIKE 'wsrep_local_state_comment';
    SHOW STATUS LIKE 'wsrep_ready';
    SHOW STATUS LIKE 'wsrep_cluster_size';
    ```

    ??? example "Expected output on every node of a three-node cluster"

        ```{.text .no-copy}
        +----------------------------+----------+
        | Variable_name              | Value    |
        +----------------------------+----------+
        | wsrep_cluster_status       | Primary  |
        | wsrep_local_state_comment  | Synced   |
        | wsrep_ready                | ON       |
        | wsrep_cluster_size         | 3        |
        +----------------------------+----------+
        ```

    **Checkpoint C6:** Continue to validation only when every node matches the expected cluster size and health values.

8. **Once.** Keep `pxc_strict_mode=PERMISSIVE` until [Task 6](#task-6-validate-the-migrated-cluster) passes.

    Then set `pxc_strict_mode` to `ENFORCING` on every node if your workload allows it.

---

## Task 6: Validate the migrated cluster

Run these checks before you return application traffic.

Do not mark the migration complete until **Checkpoint C7** passes.

{.power-number}

1. **Every node.** Confirm cluster health with the same queries as Checkpoint C6.

2. **Every node.** Confirm the nodes run PXC binaries:

    ```sql
    SELECT VERSION();
    ```

    ??? example "Expected output (version string varies by release)"

        ```{.text .no-copy}
        +-----------------------------------+
        | VERSION()                         |
        +-----------------------------------+
        | 8.4.x-x.x (Percona XtraDB Cluster)|
        +-----------------------------------+
        ```

    The version string must identify Percona XtraDB Cluster or Percona Server for MySQL.

    **Failure criteria:** A MySQL-wsrep or other non-PXC version string means the wrong binary is running. Stop and fix packages before you continue.

3. **Once** for the write, then **every other node** for the read.

    [Verify replication](verify-replication.md) with a small write on one node and a read on the other nodes.

    **Failure criteria:** Missing rows on any node means replication failed. Do not return traffic.

4. **Once.** Test failover:

    * Remove one node from the proxy pool

    * Stop MySQL on that node

    * Confirm the application still works against the remaining nodes

    * Start the node and wait for `Synced`

    **Failure criteria:** Application outage during the test, or the stopped node fails to return to `Synced`.

5. **Once.** Run application acceptance tests against the proxy endpoint.

    Do not test only against a single node.

6. **Once.** Create a post-migration backup and verify the backup.

7. **Every node.** Watch error logs and flow control after cutover.

    Include peak traffic when possible.

**Checkpoint C7:** Return application traffic only when steps 1 through 5 pass.

If any step fails, keep traffic off the cluster and use [Rollback](#rollback) or [Troubleshooting checklist](#troubleshooting-checklist).

## Task 7: Update related systems

Complete these steps after **Checkpoint C7** passes and application traffic is stable.

{.power-number}

1. **Once** (monitoring server). Configure monitoring for the nodes.

    [Percona Monitoring and Management :octicons-link-external-16:](https://docs.percona.com/percona-monitoring-and-management/) is the recommended option for PXC.

    See also [Monitor the cluster](monitoring.md).

2. **Once.** Update backup jobs to use Percona XtraBackup versions that match your PXC series.

3. **Once** (proxy host). If you use ProxySQL, confirm health checks and users still work.

    PXC {{vers}} defaults to `caching_sha2_password`.

    Use ProxySQL 2.6.2 or later.

    See [Load balance with ProxySQL](load-balance-proxysql.md).

4. **Once.** Enable async replicas only after you verify replication compatibility with the migrated [donor](glossary.md#donor-node).

5. **Every node.** Remove MySQL Galera packages and vendor repositories from the hosts.

    Later upgrades must pull only Percona packages.

## Rollback

Select a rollback method during planning.

Keep the method available until the observation period ends.

| Situation | Rollback approach |
| --------- | ----------------- |
| Migration not yet started | No action. Keep the verified backup. |
| Cutover failed during the maintenance window | Restore the verified pre-migration backup to MySQL Galera nodes, or return to the untouched source cluster if you migrated from a parallel environment. |
| Data was written on PXC after cutover | A package downgrade does not keep those writes. Restore from a backup taken after the writes, or use a planned data recovery process. |

!!! warning

    Package downgrades after data dictionary or privilege table changes are unsafe.

    Prefer restore from backup over ad-hoc package reversals.

## Troubleshooting checklist

| Symptom | Likely cause | What to check |
| ------- | ------------ | ------------- |
| Node fails to join after package swap | Encryption mismatch | `pxc_encrypt_cluster_traffic`, identical certificates on all nodes |
| Node starts as a standalone cluster | Wrong `wsrep_cluster_address` or accidental [bootstrap](glossary.md#bootstrap) | Configuration file. Avoid `mysql@bootstrap` except for the first node of a cluster. |
| [SST](glossary.md#sst) fails | XtraBackup or SST user privileges | [XtraBackup SST configuration](xtrabackup-sst.md), [donor](glossary.md#donor-node) error log |
| Application errors after migration | Strict Mode rejects statements | `pxc_strict_mode`, [Strict Mode](glossary.md#strict-mode), error log |
| Configuration missing after yum or dnf install | RHEL replaced `my.cnf` | Restore from `my.cnf.rpmsave` or your `.galera.bak` copy |
| Cluster size does not recover | [Quorum](glossary.md#quorum) loss | Bring back enough nodes. See [Cluster failover](failover.md) and [Emergency quorum recovery](emergency-quorum-recovery.md). |

## After a successful migration

* Keep the pre-migration backup until you pass your production observation period.

* Schedule the next maintenance window for any major-version upgrade that remains (for example 8.0 to {{vers}}).

    Use [Upgrade Percona XtraDB Cluster](upgrade-guide.md).

* Review [Percona XtraDB Cluster limitations](limitation.md) with your application team.

    Remove unsupported patterns on purpose.

For help with migration scope or high availability review, see [Get help from Percona](get-help.md).

!!! admonition "See also"

    * [How to Migrate from MySQL Galera Cluster to Percona XtraDB Cluster :octicons-link-external-16:](https://www.percona.com/blog/migrate-mysql-galera-cluster-to-percona-xtradb-cluster/)

    * [Upgrade Percona XtraDB Cluster](upgrade-guide.md)

    * [Get started with Percona XtraDB Cluster](get-started-cluster.md)
