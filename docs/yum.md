# Install on Red Hat Enterprise Linux

A list of the supported platforms by products and versions
is available in [Percona Software and Platform Lifecycle :octicons-link-external-16:](https://www.percona.com/services/policies/percona-software-platform-lifecycle#mysql).

We gather [Telemetry data] in the Percona packages and Docker images.

--8<--- "get-help-snip.md"

You can install Percona XtraDB Cluster with the following methods:

* Use the official repository using YUM

* Download and manually install the Percona XtraDB Cluster packages from [Percona Software Downloads :octicons-link-external-16:](https://www.percona.com/downloads/).

* Use the Percona Software repositories 

This documentation describes using the Percona Software repositories.

## Prerequisites

Installing Percona XtraDB Cluster requires that you either be logged in as a user with root privileges or be able to run commands with sudo.

 Percona XtraDB Cluster requires specific ports for communication. Make sure that the following ports are available:

* 3306

* 4444

* 4567

* 4568

For information on SELinux, see [Enabling SELinux](selinux.md#enable-selinux).

## Install from Percona Software Repository

For more information on the Percona Software repositories and configuring Percona Repositories with `percona-release`, see the [Percona Software Repositories Documentation :octicons-link-external-16:](https://docs.percona.com/percona-software-repositories/index.html).

## Install on Red Hat 8

RHEL 8 and other EL8 systems enable the MySQL module by default. This module hides the Percona-provided packages and the module must be disabled to make these packages visible. The following command disables the module: 

```shell
sudo yum module disable mysql
```

```shell
sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
sudo percona-release setup {{pkg}}
sudo yum install percona-xtradb-cluster
```

## After installation

After the installation, start the `mysql` service and use the `grep` command to find the temporary password. 

```shell
sudo service mysql start
sudo grep 'temporary password' /var/log/mysqld.log
```

Use the temporary password to log into the server:

```shell
mysql -u root -p
```

Run an `ALTER USER` statement to change the temporary password, exit the client, and stop the service.

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'rootPass';
exit
sudo service mysql stop
```

## Install on Red Hat 9 or later

RHEL 9 and subsequent versions use the `dnf` package manager and its module system. The default `mysql` module must be disabled to prevent conflicts with Percona's packages.

```shell
sudo dnf module disable mysql
```

Next, install the Percona repository and the cluster.

```shell
sudo dnf install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
sudo percona-release setup {{pkg}}
sudo dnf install percona-xtradb-cluster
```

### After installation

After installation, start the `mysqld` service and locate the temporary password.

```shell
sudo systemctl start mysqld
sudo grep 'temporary password' /var/log/mysqld.log
```

Use the temporary password to log in, and then run an `ALTER USER` statement to change it.

```shell
mysql -u root -p
```

Enter the temporary password.

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY 'rootPass';
```

For security, exit the client and stop the service.

```sql
exit
```

```shell
sudo systemctl stop mysqld
```

## Next steps

Configure the node according to the procedure described in [Configuring Nodes for Write-Set Replication](configure-nodes.md#configure-nodes-for-write-set-replication).

[Telemetry data]: telemetry.md
