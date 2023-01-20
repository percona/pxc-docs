# Install Percona XtraDB Cluster

Install Percona XtraDB Cluster on all hosts that you are planning to use as cluster nodes
and ensure that you have root access to the MySQL server on each one.

It is recommended to install Percona XtraDB Cluster from official Percona software repositories
using the corresponding package manager for your system:

* [Debian or Ubuntu](apt.md#apt)

* [Red Hat or CentOS](yum.md#yum)

!!! important

    After installing Percona XtraDB Cluster the ``mysql`` service is *stopped* but *enabled* so that it may start the next time the system is restarted. The service starts if the the grastate.dat file exists and the value of ``seqno`` is not **-1**.

    !!! admonition "See also"

        More information about Galera state information in [Index of files created by PXC grastat.dat](../wsrep-files-index.md#wsrep-file-index)

## Installation alternatives

Percona also provides a generic tarball with all required files and binaries
for manual installation:

* [Installing Percona XtraDB Cluster from Binary Tarball](tarball.md#tarball)

If you want to build Percona XtraDB Cluster from source, see [Compiling and Installing from Source Code](compile.md#compile).

If you want to run Percona XtraDB Cluster using Docker, see [Running Percona XtraDB Cluster in a Docker Container](docker.md#docker).

## Product version numbering

The version number in PXC releases contains the following components:

* The version of Percona Server for MySQL that the given PXC release is based on

* The sequence number which represents the PXC built.

For example, version number *8.0.18-9.3* means that this is the third PXC
build based on Percona Server for MySQL 8.0.18-9.
