# Installing Percona XtraDB Cluster on Debian or Ubuntu

Specific information on the supported platforms, products, and versions is described in [Percona Software and Platform Lifecycle](https://www.percona.com/services/policies/percona-software-platform-lifecycle#mysql).

The packages are available in the official Percona software repository
and on the [download page](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/LATEST/).
It is recommended to install Percona XtraDB Cluster from the official repository
using **apt**.

## Prerequisites

You need to have root access on the node where you will be installing
Percona XtraDB Cluster (either logged in as a user with root privileges or be able
to run commands with **sudo**).

Make sure that the following ports are not blocked by firewall or used
by other software. Percona XtraDB Cluster requires them for communication.


* 3306


* 4444


* 4567


* 4568

!!! note

    To view the listening ports, enter the following command:

    ```shell
    $ sudo ss -tunlp
    ```

### If *MySQL* Is Installed

If you previously had MySQL installed on the server, there might be an
[AppArmor](https://help.ubuntu.com/community/AppArmor) profile
which will prevent Percona XtraDB Cluster nodes from communicating with each other.
The best solution is to remove the `apparmor` package entirely:

```shell
$ sudo apt remove apparmor
```

If you need to have AppArmor enabled due to security policies or for
other reasons, it is possible to disable or extend the MySQL profile.

### Dependencies on Ubuntu

When installing on a Ubuntu system, make sure that the `universe`
repository is enabled to satisfy all essential dependencies.

!!! admonition "See also"

    [Ubuntu Documentation: Repositories](https://help.ubuntu.com/community/Repositories/Ubuntu)

## Installing from Repository

1. Configure Percona repositories as described in [Percona Software Repositories Documentation](https://www.percona.com/doc/percona-repo-config/index.html).

2. Install the Percona XtraDB Cluster server package:

    ```shell
    $ sudo apt install percona-xtradb-cluster-57
    ```

    !!! note

        Alternatively, you can install the `percona-xtradb-cluster-full-57` meta package, which contains the following additional packages:

         * `percona-xtradb-cluster-test-5.7`

         * `percona-xtradb-cluster-5.7-dbg`

         * `percona-xtradb-cluster-garbd-3.x`

         * `percona-xtradb-cluster-galera-3.x-dbg`

         * `percona-xtradb-cluster-garbd-3.x-dbg`

         * `libmysqlclient18`


    During installation, you will be prompted to provide a password for the `root` user on the database node.

3. Stop the `mysql` service:

    ```shell
    $ sudo service mysql stop
    ```

    !!! note

        All Debian-based distributions start services as soon as the corresponding package is installed. Before starting a Percona XtraDB Cluster node, it needs to be properly configured. For more information, see [Configuring Nodes for Write-Set Replication](../configure.md#configure).

## Next Steps

After you install Percona XtraDB Cluster and stop the `mysql` service,
configure the node according to the procedure described in [Configuring Nodes for Write-Set Replication](../configure.md#configure).
