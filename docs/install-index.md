# Install Percona XtraDB Cluster

Install Percona XtraDB Cluster on all hosts that you are planning to use as cluster nodes
and ensure that you have root access to the MySQL server on each one.

We gather [Telemetry data] in the Percona packages and Docker images.

--8<--- "get-help-snip.md"

We recommended installing Percona XtraDB Cluster from official Percona software repositories using the appropriate package manager for your system:

* [Debian or Ubuntu](apt.md#install-from-repository)

* [Red Hat Enterprise Linux](yum.md#install-from-percona-software-repository)

!!! important

    After installing Percona XtraDB Cluster the ``mysql`` service is *stopped* but *enabled* so that it may start the next time the system is restarted. The service starts if the the grastate.dat file exists and the value of ``seqno`` is not **-1**.

    !!! admonition "For more information"

        See the Galera state information in [Index of files created by PXC grastat.dat](wsrep-files-index.md#index-of-files-created-by-pxc)

## Installation alternatives

Percona also provides a generic tarball with all required files and binaries
for manual installation:

* [Installing Percona XtraDB Cluster from Binary Tarball](tarball.md#install-from-binary-tarball)

If you want to build Percona XtraDB Cluster from source, see [Compiling and Installing from Source Code](compile.md#compile).

If you want to run Percona XtraDB Cluster using Docker, see [Run in a Docker container](docker.md).

[Telemetry data]: telemetry.md
