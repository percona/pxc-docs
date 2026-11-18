> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-xtradb-cluster/8.4/llms.txt).

# Upgrade cluster from packages

Make a full backup (or dump if possible) of your database. Move the database configuration file, `my.cnf`, to another direction to save it. Stop the server with `/etc/init.d/mysql stop`.

**Debian-derived distribution**

1. Remove the installed packages with their dependencies: `sudo apt autoremove percona-server percona-client`

1. Do the required modifications in the database configuration file `my.cnf`.

1. Download the following packages for your architecture:

   - `percona-server-server`
   - `percona-server-client`
   - `percona-server-common`
   - `libperconaserverclient21`

   The following example downloads Percona Server for MySQL 8.4.11-12 packages for Debian 11.0:

   ```
   wget https://downloads.percona.com/downloads/Percona-Server-8.4/Percona-Server-8.4.11-12/binary/debian/bullseye/x86_64/Percona-Server-8.4.11-12-rd76e81f4-bullseye-x86_64-bundle.tar
   ```

1. Unpack the bundle to get the packages: `tar xvf Percona-Server-8.4.11-12-rd76e81f4-bullseye-x86_64-bundle.tar`.

   After you unpack the bundle, you should see the following packages:

   ```
   ls *.deb
   ```

   Expected output

   ```
   llibperconaserverclient21-dev_8.4.11-12.bullseye_amd64.deb
   percona-server-dbg_8.4.11-12.bullseye_amd64.deb
   libperconaserverclient21_8.4.11-12.bullseye_amd64.deb
   percona-server-rocksdb_8.4.11-12.bullseye_amd64.deb
   percona-mysql-router_8.4.11-12.bullseye_amd64.deb
   percona-server-server_8.4.11-12.bullseye_amd64.deb
   percona-server-client_8.4.11-12.bullseye_amd64.deb
   percona-server-source_8.4.11-12.bullseye_amd64.deb
   percona-server-common_8.4.11-12.bullseye_amd64.deb
   percona-server-test_8.4.11-12.bullseye_amd64.deb
   ```

1. Install Percona Server for MySQL:

   ```
   sudo dpkg -i *.deb
   ```

   This command installs the packages from the bundle. Another option is to download or specify only the packages you need for running Percona Server for MySQL installation (`libperconaserverclient21-dev_8.4.11-12.bullseye_amd64.deb`, `percona-server-client-8.4.11-12.bullseye_amd64.deb`, `percona-server-common-8.4.11-12.bullseye_amd64.deb`, and `percona-server-server-8.4.11-12.bullseye_amd64.deb`.

   Warning

   When installing packages manually, you must resolve all the dependencies and install missing packages yourself. At least the following packages should be installed before installing Percona Server for MySQL 8.4.11-12: * `libmecab2`, * `libjemalloc1`, * `zlib1g-dev`, * `libaio1`.

1. Running the upgrade:

   The mysqld binary automatically runs the upgrade process. To find more information, see [MySQL Upgrade Process Upgrades](https://dev.mysql.com/doc/refman/8.4.11-12/en/upgrading-what-is-upgraded.html)

1. Restart the service with `service mysql restart`. After the service has been successfully restarted use the new Percona Server for MySQL 8.4.11-12.

**Red Hat-derived distributions**

1. Check the installed packages:

   ```
   rpm -qa | grep percona-server
   ```

   Expected output

   ```
   percona-server-shared-8.4.11-12.el9.x86_64
   percona-server-shared-compat-8.4.11-12.el9.x86_64
   percona-server-client-8.4.11-12.el9.x86_64
   percona-server-server-8.4.11-12.el9.x86_64
   ```

   You may have the `shared-compat` package, which is required for compatibility.

1. Remove the packages without dependencies with `rpm -qa | grep percona-server | xargs rpm -e --nodeps`.

   It is important that you remove the packages without dependencies as many packages may depend on these (as they replace `mysql`) and will be removed if omitted.

   To remove the listed packages, run:

   ```
   rpm -qa | grep '^mysql-'| xargs rpm -e --nodeps`
   ```

1. Download the packages of the desired series for your architecture from the [Percona Software Downloads](https://www.percona.com/downloads) . The easiest way is to download the bundle which contains all the packages. The following example downloads Percona Server for MySQL 8.4.11-12 packages for CentOS 9:

   ```
   wget https://downloads.percona.com/downloads/Percona-Server-8.4/Percona-Server-8.4.11-12/binary/redhat/9/x86_64/Percona-Server-8.4.11-12-rd76e81f4-el9-x86_64-bundle.tar
   ```

1. Unpack the bundle to get the packages

   ```
   tar xvf Percona-Server-8.4.11-12-rd76e81f4-el9-x86_64-bundle.tar
   ```

   After you unpack the bundle, you should see the following packages: `ls \*.rpm`

1. Install Percona Server for MySQL:

   ```
   sudo rpm -ivh percona-server-server-8.4.11-12.el9.x86_64.rpm \
   percona-server-client-8.4.11-12.el9.x86_64.rpm \
   percona-server-shared-8.4.11-12.el9.x86_64.rpm \
   percona-server-shared-compat-8.4.11-12.el9.x86_64.rpm
   ```

   This command installs only packages required to run the Percona Server for MySQL 8.4.11-12.

1. You can install all the packages (for debugging, testing, etc.) with `sudo rpm -ivh \*.rpm`.

   Note

   When manually installing packages, you must resolve all the dependencies and install missing ones.

1. Modify your configuration file, `my.cnf`, and install the plugins if necessary.

   RHEL/CentOS automatically backs up the previous configuration file to `/etc/my.cnf.rpmsave` and installs the default `my.cnf`. After the upgrade/install process completes you can move the old configuration file back (after you remove all the unsupported system variables).

1. The schema of the grant table has changed, the server must be started without reading the grants. Add a line to my.cnf in the [mysqld] section,

   ```
   [mysqld]
   skip-grant-tables
   ```

   Restart the mysql server with `service mysql start`.

1. Running the upgrade:

   The mysqld binary automatically runs the upgrade process. To find more information, see [MySQL Upgrade Process Upgrades](https://dev.mysql.com/doc/refman/8.4/en/upgrading-what-is-upgraded.html)

1. Restart the server with `service mysql restart`. After the service has been successfully restarted you can use the new Percona Server for MySQL 8.4.11-12.
