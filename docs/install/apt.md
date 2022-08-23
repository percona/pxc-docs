# Installing Percona XtraDB Cluster on Debian or Ubuntu

Specific information on the supported platforms, products, and versions
is described in [Percona Software and Platform Lifecycle](https://www.percona.com/services/policies/percona-software-platform-lifecycle#mysql).

The packages are available in the official Percona software repository
and on the [download page](https://www.percona.com/downloads/Percona-XtraDB-Cluster-80/LATEST/).
It is recommended to install Percona XtraDB Cluster from the official repository
using **apt**.

## Prerequisites

* You need to have root access on the node where you will be installing Percona XtraDB Cluster (either logged in as a user with root privileges or be able to run commands with **sudo**).

* Make sure that the following ports are not blocked by firewall or used by other software. Percona XtraDB Cluster requires them for communication.

    * 3306


    * 4444


    * 4567


    * 4568

!!! admonition "See also"

    For more information, see [Enabling AppArmor](../security/apparmor.md#apparmor).

## Installing from Repository


1. Update the sytem:

    ```shell
    sudo apt update
    ```

2. Install the necessary packages:

    ```shell
    sudo apt install -y wget gnupg2 lsb-release curl
    ```

3. Download the repository package

    ```shell
    wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
    ```

4. Install the package with `dpkg`:

    ```shell
    sudo dpkg -i percona-release_latest.generic_all.deb
    ```

5. Refresh the local cache to update the package information:

    ```shell
    sudo apt update
    ```

6. Enable the `release` repository for *Percona XtraDB Cluster*:

    ```shell
    sudo percona-release setup pxc80
    ```

7. Install the cluster:

    ```shell
    sudo apt install -y percona-xtradb-cluster
    ```

During the installation, you are requested to provide a password for the `root` user on the database node.

!!! note

    If needed, you could also install the `percona-xtradb-cluster-full` meta-package, which includes the following additional packages:

    * `libperconaserverclient21`

    * `libperconaserverclient21-dev`

    * `percona-xtradb-cluster`

    * `percona-xtradb-cluster-client`

    * `percona-xtradb-cluster-common`

    * `percona-xtradb-cluster-dbg`

    * `percona-xtradb-cluster-full`

    * `percona-xtradb-cluster-garbd`

    * `percona-xtradb-cluster-garbd-debug`

    * `percona-xtradb-cluster-server`

    * `percona-xtradb-cluster-server-debug`

    * `percona-xtradb-cluster-source`

    * `percona-xtradb-cluster-test`


## Next Steps

After you install Percona XtraDB Cluster and stop the `mysql` service,
configure the node according to the procedure described in [Configuring Nodes for Write-Set Replication](../configure.md#configure).
