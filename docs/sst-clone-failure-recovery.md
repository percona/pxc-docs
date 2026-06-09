# SST/Clone failure recovery (when nodes refuse to join)

A node starts but never reaches `Synced`—SST or Clone fails, or the joiner is stuck. Most restarts after an unclean failure (OOM, power loss, crash) fall here; `grastate.dat` is often inconsistent and the "clean" IST path does not apply.

First step: Use Clone SST if you are not already. Set `wsrep_sst_method=clone` and meet the [Clone SST prerequisites](clone-sst.md#prerequisites); for many nodes, that alone fixes the join. If the node still refuses to join, work through the sections below and then [Environmental blockers](environmental-blockers.md) (AppArmor and systemd are the most common causes on Ubuntu 22.04).

## Use Clone SST first (recommended for small-to-mid datasets)

In PXC 8.4, the Clone plugin is the main stability improvement for rejoining: set `wsrep_sst_method=clone` and the joiner gets a full copy from a donor with a single restart—no grastate.dat editing, no xtrabackup-v2 scripts, and far fewer failure modes. For full setup, SSL, and options, see [State Snapshot Transfer (SST) Method using Clone plugin](clone-sst.md).

1. On every node (donor and joiner), set [`wsrep_sst_method`](wsrep-system-index.md#wsrep_sst_method) and [`wsrep_sst_allowed_methods`](wsrep-system-index.md#wsrep_sst_allowed_methods) in the configuration file (read-only at runtime; must be in `my.cnf` before startup):

   ```ini
   [mysqld]
   wsrep_sst_method = clone
   wsrep_sst_allowed_methods = xtrabackup-v2,clone
   ```

2. Meet the [Clone SST prerequisites](clone-sst.md#prerequisites) (Clone plugin, privileges, disk space, SSL if used). See [Enable the Clone SST Method](clone-sst.md#enable-the-clone-sst-method).

3. Restart the joiner. If the node will do a full SST, increase the [systemd start timeout](environmental-blockers.md#systemd-timeout-the-silent-killer) first. For many restarts, the node joins without grastate surgery or security tweaks.

If the node still refuses to join or your dataset or environment makes Clone unsuitable, use the sections below and [Environmental blockers](environmental-blockers.md).

## Before you start the joiner: PID file, port, and recovery

When Clone SST is configured, rejoining after an unclean exit is usually: ensure no process holds the MySQL port or data directory, start the service, and let Clone handle the transfer. When Clone is not in use or the node still fails, follow these steps.

The PID file may still be present after an unclean exit—e.g. MySQL crash, OOM, kill, or `systemctl stop mysql` timing out during a heavy buffer pool flush (systemd sends SIGKILL before the process can remove the file). On systemd systems the file also often remains because systemd is still tracking a stalled mysqld or a hung child (e.g. socat from a failed SST). Do not treat removing the PID file as the fix. If you remove it without verifying that no process holds the MySQL port (typically 3306) or the data directory, you can start a second instance while the first is still running and corrupt the datadir.

1. Check what is still running: `systemctl status mysql` (or `mysqld` on RHEL), `ps aux | grep mysqld`, and which process holds the port: `ss -tlnp | grep 3306` or `lsof -i :3306`. If a process is listening on the port, it must exit before you start MySQL again.
2. If mysqld or a child is still running, stop with `systemctl stop mysql` and wait. If it does not exit, check systemd and the cause (see [Environmental blockers](environmental-blockers.md)) and use a controlled kill; do not remove the PID file and start while anything holds the port or datadir.
3. Only after no process is listening on the port and no mysqld (or SST child) is running should you consider removing a stale PID file, then start the service.

After any unclean exit, the next start is not "remove PID file and go": InnoDB runs crash recovery and Galera may rebuild state (including GCache). That can take minutes or much longer. Allow for it before concluding the node is stuck; increase the [systemd start timeout](environmental-blockers.md#systemd-timeout-the-silent-killer) if the unit is killed before recovery or SST completes.

Finding the cause: Do not assume the MySQL error log has the answer. On Ubuntu 22.04, the most common causes—AppArmor killing SST children and systemd timing out during SST—do not appear there. Check `journalctl -u mysql`, then [Environmental blockers](environmental-blockers.md), then the [Troubleshooting cheat sheet](#troubleshooting-cheat-sheet-grep-the-error-log) for MySQL/WSREP-level failures.

Invalid configuration (e.g. syntax error in `my.cnf`) prevents MySQL and the WSREP provider from loading; the node cannot request or perform an SST. Fix the config and start again; the node may then need an SST to rejoin because it never left cleanly.

## Pre-flight: configuration validation

Before starting a joiner, validate config on every node. A single character difference in [`wsrep_cluster_address`](wsrep-system-index.md#wsrep_cluster_address) or a missing [`wsrep_node_address`](wsrep-system-index.md#wsrep_node_address) on a multi-homed host can prevent rejoin.

* wsrep_cluster_address — Identical across all nodes: same list, same order, no extra or missing commas or spaces.
* wsrep_node_address (multi-homed hosts) — Set to the IP that other nodes use to reach this node; otherwise the node may bind to 127.0.0.1 and the handshake fails.

Run e.g. `grep -E "wsrep_cluster_address|wsrep_node_address" /etc/mysql/my.cnf` on each node and fix any mismatch.

## Verify gcache coverage before restarting (planned restart only)

After an unclean failure the node is down and you cannot run `SHOW STATUS` on it; assume a full SST. For a planned restart while the node is still running: check whether the node's last position is still in a donor's [gcache](wsrep-provider-index.md#gcachesize). If the node's `wsrep_last_committed` is greater than or equal to a donor's `wsrep_local_cached_downto`, the IST window is open and the node can rejoin with IST; otherwise plan for a full SST (Clone, systemd timeout, disk space). See [Diagnose whether the joiner will use IST or SST](#diagnose-whether-the-joiner-will-use-ist-or-sst) for the exact steps.

## Disk space before SST

A full SST can fill the joiner's disk. Ensure free space on the joiner is at least as large as the donor's *used* data size (or ~1.5× the data directory size). Monitor free space during SST; if it drops to a few percent, the node can hang or corrupt the datadir.

## Do not remove sst_in_progress

Removing `sst_in_progress` does not fix a failed SST; it only bypasses the check and can lead to starting with a corrupted or incomplete datadir. Fix the underlying cause (environmental blockers, disk space, connectivity) and retry, or clear/restore the datadir and perform a fresh SST.

## PXC 8.4: authentication, SSL, and Clone

Auth plugins, SSL for cluster and SST, and the Clone plugin must be consistent. Mismatches cause "access denied" or handshake failures. Clone SST certificates must not be in the data directory (they are overwritten during SST). See [Enable SSL for Clone SST](clone-sst.md#enable-ssl-for-clone-sst) and [Clone SST prerequisites](clone-sst.md#prerequisites). All nodes should run the same PXC 8.4.x release.

## Troubleshooting cheat sheet: grep the error log

When the failure is at the MySQL or WSREP level, use these patterns (adjust the log path):

| Grep command | What it points to |
|--------------|-------------------|
| `grep -E "SST script failed|SST failed|sst.*fail" /var/log/mysql/error.log` | SST failed (script, permissions, donor/joiner). |
| `grep -iE "access denied|auth.*fail|authentication" /var/log/mysql/error.log` | SST or cluster auth (credentials, SSL). |
| `grep -iE "gcomm|handshake|gcs.*timeout|connection refused" /var/log/mysql/error.log` | GCOMM handshake, connectivity, wrong `wsrep_cluster_address`. |
| `grep -iE "uuid.*mismatch|different uuid|seqno.*mismatch" /var/log/mysql/error.log` | UUID/state mismatch, wrong bootstrap. |
| `grep -i "evicted" /var/log/mysql/error.log` | Node evicted (timeout or cluster decision). |

If you use Clone SST and the error log is empty or unhelpful, see [When the error log is silent: Clone SST](#when-the-error-log-is-silent-clone-sst).

## When the error log is silent: Clone SST

When Clone SST fails at the OS or network layer (firewall, port 4444, security policy, timeout), the MySQL error log on the joiner often has little useful information. Check: `performance_schema.clone_status` on the joiner (`SELECT STATE, ERROR_NO, ERROR_MESSAGE FROM performance_schema.clone_status;`), donor and joiner error logs, and system logs / firewall / connectivity on the SST port (default 4444). Verify [Environmental blockers](environmental-blockers.md) are not blocking Clone.

## Recover actual position after a crash: mysqld --wsrep-recover

After a crash, do not rely on `grastate.dat` for the node's last position—it is often wrong. Run `mysqld --wsrep-recover` (as the MySQL data directory owner, with MySQL stopped). This forces InnoDB to scan the redo logs and report the actual last committed transaction (`Recovered position: UUID:seqno`). Use that for IST-vs-SST decisions and, when the whole cluster is down, for choosing the bootstrap node; see [Crash recovery](crash-recovery.md).

## Diagnose whether the joiner will use IST or SST

On a donor, run `SHOW STATUS LIKE 'wsrep_last_committed'` and `SHOW STATUS LIKE 'wsrep_local_cached_downto'`. On the joiner (stopped), get the last position: after a crash use `mysqld --wsrep-recover` (see above); if the node left cleanly you can read `seqno` from `grastate.dat`. If the joiner's seqno is below the donor's `wsrep_local_cached_downto`, the joiner will need a full SST. Before starting the joiner, set the [systemd start timeout](environmental-blockers.md#systemd-timeout-the-silent-killer) so systemd does not kill the process during SST.

## Clone SST in detail

Full setup (SSL, timeouts, prerequisites) is in [State Snapshot Transfer (SST) Method using Clone plugin](clone-sst.md).

## PXC 8.4: applier threads and replica timers

Applier thread behavior and replica-related timers in PXC 8.4 are common friction points during recovery. Explicitly check the following.

Applier threads: In 8.4 the number of threads that apply replicated transactions is controlled by [`wsrep_applier_threads`](wsrep-system-index.md#wsrep_applier_threads) (the deprecated variable is `wsrep_slave_threads`). The default is 1. When a node is in JOINED state and catching up to SYNCED, a single applier thread can be a bottleneck; increasing `wsrep_applier_threads` on the joiner (e.g. to match or approach the donor) can speed catch-up. You can change it at runtime. If you see replication consistency issues after a recovery, try setting it back to 1 to isolate the cause. See [Percona XtraDB Cluster threading model](threading-model.md) and the [wsrep_applier_threads](wsrep-system-index.md#wsrep_applier_threads) description.

Replica timers: The option `wsrep_allow_replica_timers` (or the equivalent in your PXC 8.4 build) controls whether replica/applier timing behavior is enabled. Inconsistent or unsuitable settings across nodes can cause join or catch-up problems during recovery. Ensure this option is set consistently on all nodes or as required for your topology; if a joiner fails to reach SYNCED or behaves unexpectedly after IST/SST, verify replica timer settings in `my.cnf` and in `wsrep_provider_options` (or the relevant wsrep system variable index for your version).

## Other common causes when a node cannot join

* grastate.dat and bootstrap order — After a crash, `safe_to_bootstrap` is 0. Compare `seqno` (use `mysqld --wsrep-recover` when unsure) and bootstrap from the most advanced node; see [Crash recovery](crash-recovery.md) and [Bootstrap the first node](bootstrap.md).
* Network and bind address — Set [`wsrep_node_address`](wsrep-system-index.md#wsrep_node_address) to the IP other nodes use to reach this node; otherwise the node may bind to 127.0.0.1 and the handshake fails.

For more support options, see [Get help from Percona](get-help.md).
