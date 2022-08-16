# Installing Percona XtraDB Cluster

Install Percona XtraDB Cluster on all hosts that you are planning to use as cluster nodes
and ensure that you have root access to the MySQL server on each one.

It is recommended to install Percona XtraDB Cluster from official Percona software repositories
using the corresponding package manager for your system:


* [Debian or Ubuntu](apt.md#apt)


* [Red Hat or CentOS](yum.md#yum)

## Installation Alternatives

Percona also provides a generic tarball with all required files and binaries
for manual installation:

* [Installing Percona XtraDB Cluster from Binary Tarball](tarball.md#tarball)

If you want to build Percona XtraDB Cluster from source, see [Compiling and Installing from Source Code](compile.md#compile).

If you want to run Percona XtraDB Cluster using Docker, see [Running Percona XtraDB Cluster in a Docker Container](docker.md#pxc-docker-container-running).
