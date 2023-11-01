# Install Percona XtraDB Cluster on Red Hat Enterprise Linux and CentOS

A list of the supported platforms by products and versions
is available in [Percona Software and Platform Lifecycle](https://www.percona.com/services/policies/percona-software-platform-lifecycle#mysql).

We gather [Telemetry data] in the Percona packages and Docker images.

You can install Percona XtraDB Cluster with the following methods:

* Use the official repository using YUM

* Download and manually install the Percona XtraDB Cluster packages from [Percona Product Downloads](http://www.percona.com/downloads/Percona-XtraDB-Cluster-80/LATEST/).

* Use the Percona Software repositories 

This documentation describes using the Percona Software repositories.

## Prerequisites

Installing Percona XtraDB Cluster requires that you either are logged in as a user with root privileges or can run commands with sudo.

 Percona XtraDB Cluster requires the specific ports for communication. Make sure that the following ports are available:

* 3306

* 4444

* 4567

* 4568

For information on SELinux, see [Enabling SELinux](selinux.md#selinux).

## Install from Percona Software Repository

For more information on the Percona Software repositories and configuring Percona Repositories with `percona-release`, see the [Percona Software Repositories Documentation](https://docs.percona.com/percona-software-repositories/index.html).

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

Configure the node according to the procedure described in [Configuring Nodes for Write-Set Replication](configure-nodes.md#configure).

[Telemetry data]: telemetry.md