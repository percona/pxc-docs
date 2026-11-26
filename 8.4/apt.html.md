> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-xtradb-cluster/8.4/llms.txt).

# Install on Debian or Ubuntu

Specific information on the supported platforms, products, and versions is described in [Percona Software and Platform Lifecycle](https://www.percona.com/services/policies/percona-software-platform-lifecycle#mysql) .

The packages are available in the official Percona software repository and on the [Software Downloads](https://www.percona.com/downloads) . We recommended that you install Percona XtraDB Cluster from the official repository using APT.

We gather [Telemetry data](https://docs.percona.com/percona-xtradb-cluster/8.4/telemetry.md) in the Percona packages and Docker images.

Review [Get more help](https://docs.percona.com/percona-xtradb-cluster/8.4/get-help.md) for ways that we can work with you.

## Prerequisites

- You need to have root access on the node where you will be installing Percona XtraDB Cluster (either logged in as a user with root privileges or be able to run commands with sudo).

- Make sure that the following ports are not blocked by the firewall or used by other software. Percona XtraDB Cluster requires them for communication.

  - 3306
  - 4444
  - 4567
  - 4568

See also

For more information, see [Enabling AppArmor](https://docs.percona.com/percona-xtradb-cluster/8.4/apparmor.md#enable-apparmor).

## Install from Repository

The following steps install from the APT repository.

1. Update the system:

   ```shell
   sudo apt update
   ```

1. Install the necessary packages:

   ```shell
   sudo apt install -y wget gnupg2 lsb-release curl
   ```

1. Download the repository package

   ```shell
   wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
   ```

1. Install the package with `dpkg`:

   ```shell
   sudo dpkg -i percona-release_latest.generic_all.deb
   ```

1. Refresh the local cache to update the package information:

   ```shell
   sudo apt update
   ```

1. Enable the `release` repository for *Percona XtraDB Cluster*:

   ```shell
   sudo percona-release setup pxc-84-lts
   ```

1. Install the cluster:

   ```shell
   sudo apt install -y percona-xtradb-cluster
   ```

During the installation, you are requested to provide a password for the `root` user on the database node.

Note

If needed, you could also install the `percona-xtradb-cluster-full` meta-package, which includes the following additional packages:

- `libperconaserverclient21`
- `libperconaserverclient21-dev`
- `percona-xtradb-cluster`
- `percona-xtradb-cluster-client`
- `percona-xtradb-cluster-common`
- `percona-xtradb-cluster-dbg`
- `percona-xtradb-cluster-full`
- `percona-xtradb-cluster-garbd`
- `percona-xtradb-cluster-garbd-debug`
- `percona-xtradb-cluster-server`
- `percona-xtradb-cluster-server-debug`
- `percona-xtradb-cluster-source`
- `percona-xtradb-cluster-test`

## Next steps

After you install Percona XtraDB Cluster and stop the `mysql` service, configure the node according to the procedure described in [Configuring Nodes for Write-Set Replication](https://docs.percona.com/percona-xtradb-cluster/8.4/configure-nodes.md#configure-nodes-for-write-set-replication).
