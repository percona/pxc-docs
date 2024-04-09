# Setting up Galera Arbitrator

[Galera Arbitrator](https://galeracluster.com/library/documentation/arbitrator.html) is a member of *Percona XtraDB Cluster* that is used for voting
in case you have a small number of servers (usually two)
and don’t want to add any more resources.
Galera Arbitrator does not need a dedicated server.
It can be installed on a machine running some other application.
Just make sure it has good network connectivity.

Galera Arbitrator is a member of the cluster that participates in the voting,
but not in actual replication
(although it receives the same data as other nodes).
Also, it is not included in flow control calculations.

This document will show how to add Galera Arbitrator node
to an existing cluster.

!!! note

    For more information on how to set up a cluster you can read in the
    [Configuring Percona XtraDB Cluster on Ubuntu](ubuntu_howto.md#ubuntu-howto) or [Configuring Percona XtraDB Cluster on CentOS](centos_howto.md#centos-howto) manuals.

## Installation

*Galera Arbitrator* can be installed from Percona’s repository by running:

```shell
root@ubuntu:~# apt install percona-xtradb-cluster-garbd-5.7
```

on Debian/Ubuntu distributions, or:

```shell
[root@centos ~]# yum install Percona-XtraDB-Cluster-garbd-57
```

on CentOS/RHEL distributions.

## Configuration

To configure *Galera Arbitrator* on *Ubuntu/Debian* you need to edit the
`/etc/default/garbd` file. On *CentOS/RHEL* configuration can be found in
`/etc/sysconfig/garb` file.

Configuration file should look like this after installation:

```text
# Copyright (C) 2012 Codership Oy
# This config file is to be sourced by garb service script.

# REMOVE THIS AFTER CONFIGURATION

# A comma-separated list of node addresses (address[:port]) in the cluster
# GALERA_NODES=""

# Galera cluster name, should be the same as on the rest of the nodes.
# GALERA_GROUP=""

# Optional Galera internal options string (e.g. SSL settings)
# see https://galeracluster.com/documentation-webpages/galeraparameters.html
# GALERA_OPTIONS=""

# Log file for garbd. Optional, by default logs to syslog
# Deprecated for CentOS7, use journalctl to query the log for garbd
# LOG_FILE=""
```

To set it up you’ll need to add the information about the cluster you’ve set
up. This example is using cluster information from the [Configuring Percona XtraDB Cluster on Ubuntu](ubuntu_howto.md#ubuntu-howto).

```text
# Copyright (C) 2012 Codership Oy
# This config file is to be sourced by garb service script.

# A comma-separated list of node addresses (address[:port]) in the cluster
GALERA_NODES="192.168.70.61:4567, 192.168.70.62:4567, 192.168.70.63:4567"

# Galera cluster name, should be the same as on the rest of the nodes.
GALERA_GROUP="my_ubuntu_cluster"

# Optional Galera internal options string (e.g. SSL settings)
# see https://galeracluster.com/documentation-webpages/galeraparameters.html
# GALERA_OPTIONS=""

# Log file for garbd. Optional, by default logs to syslog
# Deprecated for CentOS7, use journalctl to query the log for garbd
# LOG_FILE=""
```

!!! note

    Please note that you need to remove the `# REMOVE THIS AFTER
    CONFIGURATION` line before you can start the service.

You can now start the *Galera Arbitrator* daemon (`garbd`) by running:

* On Debian or Ubuntu:

  ```shell
  root@server:~# service garbd start
  ```

  ```text
  [ ok ] Starting /usr/bin/garbd: :.
  ```

* On Red Hat Enterprise Linux or CentOS:

  ```shell
  root@server:~# service garb start
  ```

  ```text
  [ ok ] Starting /usr/bin/garbd: :.
  ```

You can additionally check the `arbitrator` status by running:

* On Debian or Ubuntu:

  ```shell
  root@server:~# service garbd status
  ```

  ```text
  [ ok ] garb is running.
  ```

* On Red Hat Enterprise Linux or CentOS:

  ```shell
  root@server:~# service garb status
  ```

  ```text
  [ ok ] garb is running.
  ```
