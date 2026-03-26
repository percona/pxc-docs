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

    You must specify a cipher. This document uses `AES128-SHA256` as an example.
    Choose a cipher that matches your security policy and OpenSSL configuration.
    If no cipher is specified, startup can fail with a “Terminate called after
    throwing an instance of ‘gnu::NotSet’” message.

    For more information, see [socket.ssl_cipher :octicons-link-external-16:](https://mariadb.com/docs/galera-cluster/reference/wsrep-variable-details/wsrep_provider_options#socket.ssl_cipher)

When starting from the shell, you can set the parameters from the command line or edit the configuration file. This is an example of starting from the command line:

```shell
garbd --group=<CLUSTER_NAME> \
--address="gcomm://<NODE1_IP>:4567,<NODE2_IP>:4567,<NODE3_IP>:4567" \
--option="socket.ssl=YES; socket.ssl_key=<SSL_KEY_PATH>; \
socket.ssl_cert=<SSL_CERT_PATH>; \
socket.ssl_ca=<SSL_CA_PATH>; \
socket.ssl_cipher=AES128-SHA256"
```

To avoid entering the options each time you start `garbd`, edit the options in the configuration file. To configure *Galera Arbitrator* on *Ubuntu/Debian*, edit the `/etc/default/garb` file. On RedHat or derivative distributions, the configuration can be found in `/etc/sysconfig/garb` file.

Replace placeholder values such as `<CLUSTER_NAME>`, `<NODE1_IP>`,
`<NODE2_IP>`, `<NODE3_IP>`, `<SSL_KEY_PATH>`, `<SSL_CERT_PATH>`, and
`<SSL_CA_PATH>` with values from your environment.

After installation, before you add your cluster settings, **`/etc/default/garb`** (Debian/Ubuntu) from **Percona XtraDB Cluster** {{vers}} packages matches the following template:

```{.text .no-copy}
# Copyright (C) 2012 Codership Oy
# This config file is to be sourced by garb service script.

# REMOVE THIS AFTER CONFIGURATION

# A comma-separated list of node addresses (address[:port]) in the cluster
# GALERA_NODES=""

# Galera cluster name, should be the same as on the rest of the nodes.
# GALERA_GROUP=""

# Optional Galera internal options string (e.g. SSL settings)
# see http://galeracluster.com/documentation-webpages/galeraparameters.html
# GALERA_OPTIONS=""

# Log file for garbd. Optional, by default logs to syslog
# LOG_FILE=""
```

Add the parameter information about the cluster. For this document, we use the cluster information from [Configuring Percona XtraDB Cluster on Ubuntu](configure-cluster-ubuntu.md#configure-a-cluster-on-debian-or-ubuntu).

!!! note

    Please note that you need to remove the `# REMOVE THIS AFTER
    CONFIGURATION` line before you can start the service.

```{.text .no-copy}
This config file is to be sourced by garb service script.
A comma-separated list of node addresses (address[:port]) in the cluster
GALERA_NODES="<NODE1_IP>:4567,<NODE2_IP>:4567,<NODE3_IP>:4567"
Galera cluster name, should be the same as on the rest of the nodes.
GALERA_GROUP="<CLUSTER_NAME>"
Optional Galera internal options string (e.g. SSL settings)
see http://galeracluster.com/documentation-webpages/galeraparameters.html
GALERA_OPTIONS="socket.ssl_key=<SSL_KEY_PATH>;socket.ssl_cert=<SSL_CERT_PATH>;socket.ssl_ca=<SSL_CA_PATH>;socket.ssl_cipher=AES128-SHA256"
Log file for garbd. Optional, by default logs to syslog
LOG_FILE="/var/log/garbd.log"
```

You can now start the *Galera Arbitrator* daemon (`garbd`). Run the following commands as root.

!!! note

    The systemd service name is `garb`, while the daemon binary name is `garbd`.

=== "On Debian or Ubuntu"

    ```shell
    systemctl start garb
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        # No output on success
        ```

=== "On Red Hat Enterprise Linux"

    ```shell
    systemctl start garb
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        # No output on success
        ```

Additionally, you can check the `arbitrator` status by running:

=== "On Debian or Ubuntu"

    ```shell
    systemctl is-active garb
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        active
        ```

## Verify arbitrator participation

1. Check the arbitrator service state:

    ```shell
    systemctl is-active garb
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        active
        ```

2. Check recent arbitrator logs for successful cluster communication:

    ```shell
    journalctl -u garb --no-pager -n 20
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        ... WSREP: ...
        ... connected ...
        ```

3. On a Percona XtraDB Cluster node, confirm that the cluster remains in `Primary` state:

    ```shell
    mysql -e "SHOW STATUS LIKE 'wsrep_cluster_status';"
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        +----------------------+---------+
        | Variable_name        | Value   |
        +----------------------+---------+
        | wsrep_cluster_status | Primary |
        +----------------------+---------+
        ```

## Troubleshooting

* `gnu::NotSet` at startup: set `socket.ssl_cipher` in `GALERA_OPTIONS`.

* Arbitrator cannot join cluster: verify `GALERA_NODES` format (`IP:4567` entries separated by commas, no missing ports).

* No connectivity to cluster nodes: verify firewall and routing for port `4567` between arbitrator and all cluster nodes.

* Service starts then exits: review `journalctl -u garb --no-pager` and validate certificate file paths in `GALERA_OPTIONS`.

=== "On Red Hat Enterprise Linux"

    ```shell
    systemctl is-active garb
    ```

    ??? example "Expected output"

        ```{.text .no-copy}
        active
        ```