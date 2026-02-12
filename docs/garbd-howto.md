# Set up Galera arbitrator

The size of a cluster increases when a node joins the cluster and decreases when a node leaves. A cluster reacts to replication problems with inconsistency voting. The size of the cluster determines the required votes to achieve a quorum. If a node no longer responds and is disconnected from the cluster the remaining nodes vote. The majority of the nodes that vote are considered to be in the cluster.

The arbitrator is important if you have an even number of nodes remaining in the cluster. The arbitrator keeps the number of nodes as an odd number, which avoids the split-brain situation.

A [Galera Arbitrator :octicons-link-external-16:](https://mariadb.com/docs/galera-cluster/high-availability/understanding-quorum-monitoring-and-recovery#the-galera-arbitrator-garbd)
is a lightweight member of a **Percona XtraDB Cluster**. This member can vote but does not do any replication and is not included in flow control calculations. The Galera Arbitrator is a separate daemon called `garbd`. You can start this daemon separately from the cluster and run this daemon either as a service or from the shell. You cannot configure this daemon using the `my.cnf` file.

!!! note

    For more information on how to set up a cluster you can read in the
    [Configuring Percona XtraDB Cluster on Ubuntu](configure-cluster-ubuntu.md#configure-a-cluster-on-debian-or-ubuntu) or [Configure on Red Hat Enterprise Linux](configure-cluster-rhel.md#configure-a-cluster-on-red-hat-based-distributions) manuals.

## Installation

Galera Arbitrator does not need a dedicated server and can be installed on a machine running other applications. The server must have good network connectivity.

Run the following commands as root.

*Galera Arbitrator* can be installed from Percona’s repository on Debian/Ubuntu distributions with the following command:

```shell
apt install percona-xtradb-cluster-garbd
```

*Galera Arbitrator* can be installed from Percona’s repository on RedHat or derivative distributions with the following command:

```shell
yum install percona-xtradb-cluster-garbd
```

On RHEL 8 and later, you can use `dnf install percona-xtradb-cluster-garbd` instead.

## Start `garbd` and configuration

!!! note 

    On **Percona XtraDB Cluster** {{vers}}, SSL is enabled by default. To run the Galera Arbitrator, you must copy the SSL certificates and configure `garbd` to use the certificates.

    It is necessary to specify the cipher. In this example, it is `AES128-SHA256`. If you do not specify the cipher, an error occurs with a “Terminate called after throwing an instance of ‘gnu::NotSet’” message.

    For more information, see [socket.ssl_cipher :octicons-link-external-16:](https://mariadb.com/docs/galera-cluster/reference/wsrep-variable-details/wsrep_provider_options#socket.ssl_cipher)

When starting from the shell, you can set the parameters from the command line or edit the configuration file. This is an example of starting from the command line:

```shell
garbd --group=my_ubuntu_cluster \
--address="gcomm://192.168.70.61:4567, 192.168.70.62:4567, 192.168.70.63:4567" \
--option="socket.ssl=YES; socket.ssl_key=/etc/ssl/mysql/server-key.pem; \
socket.ssl_cert=/etc/ssl/mysql/server-cert.pem; \
socket.ssl_ca=/etc/ssl/mysql/ca.pem; \
socket.ssl_cipher=AES128-SHA256"
```

To avoid entering the options each time you start `garbd`, edit the options in the configuration file. To configure *Galera Arbitrator* on *Ubuntu/Debian*, edit the `/etc/default/garb` file. On RedHat or derivative distributions, the configuration can be found in `/etc/sysconfig/garb` file.

The configuration file should look like this after the installation and before you have added your parameters:

```{.text .no-copy}
Copyright (C) 2013-2015 Codership Oy
This config file is to be sourced by garb service script.
REMOVE THIS AFTER CONFIGURATION
A comma-separated list of node addresses (address[:port]) in the cluster
GALERA_NODES=""
Galera cluster name, should be the same as on the rest of the nodes.
GALERA_GROUP=""
Optional Galera internal options string (e.g. SSL settings)
see http://galeracluster.com/documentation-webpages/galeraparameters.html
GALERA_OPTIONS=""
Log file for garbd. Optional, by default logs to syslog
Deprecated for CentOS7, use journalctl to query the log for garbd
LOG_FILE=""
```

Add the parameter information about the cluster. For this document, we use the cluster information from [Configuring Percona XtraDB Cluster on Ubuntu](configure-cluster-ubuntu.md#configure-a-cluster-on-debian-or-ubuntu).

!!! note

    Please note that you need to remove the `# REMOVE THIS AFTER
    CONFIGURATION` line before you can start the service.

```{.text .no-copy}
This config file is to be sourced by garb service script.
A comma-separated list of node addresses (address[:port]) in the cluster
GALERA_NODES="192.168.70.61:4567, 192.168.70.62:4567, 192.168.70.63:4567"
Galera cluster name, should be the same as on the rest of the nodes.
GALERA_GROUP="my_ubuntu_cluster"
Optional Galera internal options string (e.g. SSL settings)
see http://galeracluster.com/documentation-webpages/galeraparameters.html
GALERA_OPTIONS="socket.ssl_key=/etc/ssl/mysql/server-key.pem;socket.ssl_cert=/etc/ssl/mysql/server-cert.pem;socket.ssl_ca=/etc/ssl/mysql/ca.pem;socket.ssl_cipher=AES128-SHA256"
Log file for garbd. Optional, by default logs to syslog
Deprecated for CentOS7, use journalctl to query the log for garbd
LOG_FILE="/var/log/garbd.log"
```

You can now start the *Galera Arbitrator* daemon (`garbd`). Run the following commands as root.

=== "On Debian or Ubuntu"

    ```shell
    service garbd start
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        [ ok ] Starting /usr/bin/garbd: :.
        ```

    !!! note 

        On systems that run `systemd` as the default system and service manager, use `systemctl` instead of `service` to invoke the command. Currently, both are supported.

        ```shell
        systemctl start garb
        ```

=== "On Red Hat Enterprise Linux or CentOS"

    ```shell
    service garb start
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        [ ok ] Starting /usr/bin/garbd: :.
        ```

Additionally, you can check the `arbitrator` status by running:

=== "On Debian or Ubuntu"

    ```shell
    service garbd status
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        [ ok ] garb is running.
        ```

=== "On Red Hat Enterprise Linux or CentOS"

    ```shell
    service garb status
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        [ ok ] garb is running.
        ```