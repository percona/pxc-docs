# Install Percona XtraDB Cluster

Install Percona XtraDB Cluster on all hosts that you are planning to use as cluster nodes
and ensure that you have root access to the MySQL server on each one.

We gather [Telemetry data] in the Percona packages and Docker images.

## Ports required

Open specific ports for the Percona XtraDB Cluster to function correctly. 

* Port 3306 is the default port for MySQL. This port
 facilitates communication and data transfer between nodes and applications.

* Port 4567 is used for Galera replication traffic, which is vital for synchronizing data across the cluster nodes. 

* Port 4568 is used for Incremental State Transfer (IST), allowing nodes to
 transfer only the missing blocks of data. 

* Port 4444 is for State Snapshot Transfer (SST), which involves a complete data snapshot 
transfer from one node to another. 

* Port 9200 if you use Percona Monitoring and Management (PMM)
 for cluster monitoring.

## Recommendations

We recommend installing Percona XtraDB Cluster from official Percona software repositories
using the corresponding package manager for your system:

* [Debian or Ubuntu](apt.md#apt)

* [Red Hat or CentOS](yum.md#yum)

!!! important

    After installing Percona XtraDB Cluster, the ``mysql`` service is stopped but enabled so that it may start the next time you restart the system. The service starts if the the `grastate.dat` file exists and the value of ``seqno`` is not **-1**.

    !!! admonition "See also"

        More information about Galera state information in [Index of files created by PXC grastat.dat](wsrep-files-index.md#wsrep-file-index)

## Installation alternatives

Percona also provides a generic tarball with all required files and binaries
for manual installation:

* [Installing Percona XtraDB Cluster from Binary Tarball](tarball.md#tarball)

If you want to build Percona XtraDB Cluster from source, see [Compiling and Installing from Source Code](compile.md#compile).

If you want to run Percona XtraDB Cluster using Docker, see [Running Percona XtraDB Cluster in a Docker Container](docker.md#docker).


[Telemetry data]: telemetry.md
