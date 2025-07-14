# Install Percona XtraDB Cluster Pro

This document provides guidelines how to install Pro packages of Percona XtraDB Cluster from Percona repositories. [Check files in packages built for Percona XtraDB Cluster Pro :material-arrow-right:](pro-files.md){.md-button}

## Version changes

Starting with Percona XtraDB Cluster 8.4.4-4 Pro, this build is also available for Amazon Linux 2023 (AL2023) platform. We support both AMD64 and ARM64 versions of Amazon Linux 2023.

## Prerequisites

--8<--- "pro-build-announcement.md"

* You need to have root access on the node where you will be installing Percona XtraDB Cluster (either logged in as a user with root privileges or be able to run commands with sudo).

* Make sure that the following ports are not blocked by firewall or used by other software. Percona XtraDB Cluster requires them for communication.

    * 3306


    * 4444


    * 4567


    * 4568

!!! admonition "See also"

    For more information, see [Enabling AppArmor](apparmor.md#enable-apparmor).

## Procedure

1. Request the access to the pro repository from Percona Support. You will receive the client ID and the access token which you use when downloading the packages.

2. Configure the repository and install Percona XtraDB Cluster packages

    === "On Debian or Ubuntu"

        1. Use the apt package manager to dowload `percona-release`

            ```{.bash .data-prompt="$"}
            $ sudo apt update
            ```

        2. Install the necessary packages

            ```{.bash .data-prompt="$"}
            $ sudo apt install -y wget gnupg2 lsb-release curl
            ```
        
        3. Download the `percona-release` repository package

            ```{.bash .data-prompt="$"}
            $  wget https://repo.percona.com/apt/percona-release_latest.generic_all.deb
            ```

        4. Install the package with `dpkg`:

            ```{.bash .data-prompt="$"}
            $ sudo dpkg -i percona-release_latest.generic_all.deb
            ```

        5. Refresh the local cache to update the package information

            ```{.bash .data-prompt="$"}
            $ sudo apt update
            ```
        
        6. Enable the specific percona-release product

            ```{.bash .data-prompt="$"}
            $ sudo percona-release setup pxc-84-pro --user_name=<Your PRO repository user name> --repo_token=<Your PRO repository token>
            ```
    
        7.  Install the cluster:

            ```{.bash .data-prompt="$"}
            $ sudo apt install -y percona-xtradb-cluster-pro-84
            ```

            Install other required packages. [Check files in the DEB package built for Percona XtraDB Cluster 8.4](pro-files.md#files-in-the-deb-package).

    === "On RHEL, Amazon Linux 2023, or their derivatives"

        ```{.bash data-prompt="$"}
        $ sudo yum install https://repo.percona.com/yum/percona-release-latest.noarch.rpm
        $ sudo percona-release setup pxc-84-pro --user_name=<Your PRO repository user name> --repo_token=<Your PRO repository token>
        $ sudo yum install percona-xtradb-cluster-pro-84
        ```
  
        ### After installation

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

## Upgrade to Percona XtraDB Cluster Pro

If you already use Percona XtraDB Cluster, you can upgrade to Percona XtraDB Cluster Pro.

First, install the latest version of Percona XtraDB Cluster, enable the pro repository, and install the pro packages.

## Next step

[Enable the FIPS mode :material-arrow-right:](fips.md){.md-button}
