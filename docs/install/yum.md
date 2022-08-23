# Installing Percona XtraDB Cluster on Red Hat Enterprise Linux and CentOS

Specific information on the supported platforms, products, and versions
is described in [Percona Software and Platform Lifecycle](https://www.percona.com/services/policies/percona-software-platform-lifecycle#mysql).

The packages are available in the official Percona software repository
and on the [download page](http://www.percona.com/downloads/Percona-XtraDB-Cluster-80/LATEST/).
It is recommended to install Percona XtraDB Cluster from the official repository
using **yum**.

## Prerequisites

!!! note

    You need to have root access on the node where you will be installing Percona XtraDB Cluster (either logged in as a user with root privileges or be able to run commands with **sudo**).

!!! note

    Make sure that the following ports are not blocked by firewall or used by other software. Percona XtraDB Cluster requires them for communication.

    * 3306

    * 4444

    * 4567

    * 4568

    For information on SELinux, see [Enabling SELinux](../security/selinux.md#selinux).

## Installing from Percona Repository

1. Configure Percona repositories as described in [Percona Software Repositories Documentation](https://www.percona.com/doc/percona-repo-config/index.html).

2. Install the Percona XtraDB Cluster packages:

    ```shell
    $ sudo yum install percona-xtradb-cluster
    ```

    !!! note

        Alternatively you can install the `percona-xtradb-cluster-full` meta package, which contains the following additional packages:

        * `percona-xtraDB-cluster-shared`

        * `percona-xtraDB-cluster-shared-compat`

        * `percona-xtradb-cluster-client`

        * `percona-xtradb-cluster-debuginfo`

        * `percona-xtradb-cluster-devel`

        * `percona-xtradb-cluster-garbd`

        * `percona-xtradb-cluster-server`

        * `percona-xtradb-cluster-test`

3. Start the Percona XtraDB Cluster server:

    ```shell
    $ sudo service mysql start
    ```

4. Copy the automatically generated temporary password for the superuser account:

    ```shell
    $ sudo grep 'temporary password' /var/log/mysqld.log
    ```

5. Use this password to log in as `root`:

    ```shell
    $ mysql -u root -p
    ```

6. Change the password for the superuser account and log out. For example:

    ```sql
    mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'rootPass';
    ```

    The example of the output is the following:

    ```text
    Query OK, 0 rows affected (0.00 sec)

    mysql> exit
    Bye
    ```

7. Stop the `mysql` service:

    ```shell
    $ sudo service mysql stop
    ```

## Next Steps

After you install Percona XtraDB Cluster and change the superuser account password,
configure the node according to the procedure described in [Configuring Nodes for Write-Set Replication](../configure.md#configure).
