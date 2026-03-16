# Environmental blockers (AppArmor, systemd, firewalls)

These often prevent nodes from joining and do not show up in the MySQL error log. If a node refuses to join, work through this section. See [Restart the cluster nodes](restarting-nodes.md) for the full recovery index.

## Security context (AppArmor, SELinux): the "SST jail"

On Ubuntu, the default AppArmor profile does not know about XtraBackup or socat and can block the donor from sending data and the joiner from running the SST method. The failure is often silent. Without fixing this, no amount of PID file removal or grastate.dat editing will make the node join.

Option A — Temporary bridge (confirm AppArmor is the cause): Put both profiles in complain mode and restart MySQL on the joiner. If the node joins, add proper rules and put the profiles back in enforce mode. Run as root or with `sudo`:

```shell
aa-complain /usr/sbin/mysqld
aa-complain /usr/bin/wsrep_sst_xtrabackup-v2
systemctl restart mysql
```

Option B — Keep enforce mode, allow SST: Add rules to allow the SST script to run xtrabackup and socat. Add to both `/etc/apparmor.d/usr.sbin.mysqld` and `/etc/apparmor.d/usr.bin.wsrep_sst_xtrabackup-v2` (adjust paths for your datadir and binaries):

```text
  /usr/bin/xtrabackup rix,
  /usr/bin/xbstream rix,
  /usr/bin/socat rix,
  /var/lib/mysql/ r,
  /var/lib/mysql/** rwk,
```

Then `systemctl reload apparmor` and `systemctl start mysql`. If the node joined after Option A, add rules like Option B (see [Enable AppArmor](apparmor.md)), then:

```shell
aa-enforce /usr/sbin/mysqld
aa-enforce /usr/bin/wsrep_sst_xtrabackup-v2
```

SELinux (RHEL / Rocky / Alma): Ensure the SST script and ports are allowed; see [SELinux](selinux.md).

## Systemd timeout: the "silent killer"

On Ubuntu 22.04, systemd manages MySQL. If an SST takes longer than the start timeout (often 90 seconds), systemd kills the process. The user thinks the node "refuses to join," but it was killed by the OS mid-SST. Check `journalctl -u mysql` for timeout or unit failure.

The fix: Create a drop-in so the mysql unit has no start timeout. Run as root or with `sudo`:

```shell
systemctl edit mysql.service
```

(On RHEL use `systemctl edit mysqld.service`.) In the editor, add and save:

```ini
[Service]
TimeoutStartSec=0
```

Then:

```shell
systemctl daemon-reload
systemctl start mysql
```

`TimeoutStartSec=0` disables the start timeout so systemd waits indefinitely for SST and recovery. If the node was already killed mid-SST, you may need to clear or restore the data directory and retry after increasing the timeout.

## Firewalls and network

Ensure the SST port (default 4444 for Clone, or the port used by your SST method) and the cluster communication ports are open between joiner and donor. Blocked ports cause silent join failures. See [Secure the network](secure-network.md) for cluster and client connectivity.

For more support options, see [Get help from Percona](get-help.md).
