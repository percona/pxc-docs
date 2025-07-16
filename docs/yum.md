# Install Percona XtraDB Cluster on Red Hat Enterprise Linux and CentOS

A list of the supported platforms by products and versions
is available in [Percona Software and Platform Lifecycle](https://www.percona.com/services/policies/percona-software-platform-lifecycle#mysql).

We gather [Telemetry data] in the Percona packages and Docker images.

--8<--- "get-help-snip.md"

You can install Percona XtraDB Cluster with the following methods:

* Use the official repository using YUM

* Download and manually install the Percona XtraDB Cluster packages from [Percona Product Downloads](https://www.percona.com/downloads).

* Use the Percona Software repositories 

This documentation describes using the Percona Software repositories.

## Prerequisites

Installing Percona XtraDB Cluster requires that you either are logged in as a user with root privileges or can run commands with sudo.

The following ports enable communication between cluster nodes and ensure proper replication and synchronization:

| Port  | Purpose                                                                 |
|-------|-------------------------------------------------------------------------|
| 3306  | Standard MySQL port for client connections and SST via `mysqldump`.    |
| 4444  | Used for SST via `rsync` and Percona XtraBackup.                        |
| 4567  | Handles write-set replication (TCP) and multicast replication (UDP).   |
| 4568  | Facilitates Incremental State Transfer (IST) for node synchronization. |

RHEL 8 and other EL8 systems enable the MySQL module by default. This module hides the Percona-provided packages and the module must be disabled to make these packages visible. The following command disables the module:

```{.bash data-prompt="$"}
$ sudo dnf module disable mysql
```

SELinux should be disabled during installation, but it can be [re-enabled](selinux.md) once the process is complete.

## Install from Percona Software Repository

For more information on the Percona Software repositories and configuring Percona Repositories with `percona-release`, see the [Percona Software Repositories Documentation](https://docs.percona.com/percona-software-repositories/index.html).

RHEL 8 and other EL8 systems enable the MySQL module by default. This module hides the Percona-provided packages and the module must be disabled to make these packages visible. The following command disables the module:

```{.bash data-prompt="$"}
$ sudo dnf module disable mysql
```

With the release of RHEL 8, `yum` has been replaced by `dnf`, representing the next major development of the package manager. Nevertheless, many systems that now use `dnf` still recognize and support `yum` commands.

=== "Install on Red Hat 7"

    ```{.bash data-prompt="$"}
    $ sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
    $ sudo percona-release enable-only pxc-80 release
    $ sudo percona-release enable tools release
    $ sudo yum install percona-xtradb-cluster
    ```

=== "Install on Red Hat 8 or later"

    ```{.bash data-prompt="$"}
    $ sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
    $ sudo percona-release setup pxc-80
    $ sudo yum install percona-xtradb-cluster
    ```

## After installation

After the installation, start the `mysql` service and find the temporary password using the `grep` command. 

```{.bash data-prompt="$"}
$ sudo service mysql start
$ sudo grep 'temporary password' /var/log/mysqld.log
```

Use the temporary password to log into the server:

```{.bash data-prompt="$"}
$ mysql -u root -p
```

Run an `ALTER USER` statement to change the temporary password, exit the client, and stop the service.

```{.bash data-prompt="$"}
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'rootPass';
mysql> exit
$ sudo service mysql stop
```


## Next steps

Configure the node according to the procedure described in [Configuring Nodes for Write-Set Replication](configure-nodes.md#configure-nodes-for-write-set-replication).

[Telemetry data]: telemetry.md
