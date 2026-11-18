> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-xtradb-cluster/8.4/llms.txt).

# Install Percona XtraDB Cluster

Install Percona XtraDB Cluster on all hosts that you are planning to use as cluster nodes and ensure that you have root access to the MySQL server on each one.

We gather [Telemetry data](https://docs.percona.com/percona-xtradb-cluster/8.4/telemetry.html) in the Percona packages and Docker images.

Review [Get more help](https://docs.percona.com/percona-xtradb-cluster/8.4/get-help.html) for ways that we can work with you.

We recommended installing Percona XtraDB Cluster from official Percona software repositories using the appropriate package manager for your system:

- [Debian or Ubuntu](https://docs.percona.com/percona-xtradb-cluster/8.4/apt.html#install-from-repository)
- [Red Hat Enterprise Linux](https://docs.percona.com/percona-xtradb-cluster/8.4/yum.html#install-from-percona-software-repository)

Important

After installing Percona XtraDB Cluster the `mysql` service is *stopped* but *enabled* so that it may start the next time the system is restarted. The service starts if the the grastate.dat file exists and the value of `seqno` is not **-1**.

For more information

See the Galera state information in [Index of files created by PXC grastat.dat](https://docs.percona.com/percona-xtradb-cluster/8.4/wsrep-files-index.html#index-of-files-created-by-pxc)

## Installation alternatives

Percona also provides a generic tarball with all required files and binaries for manual installation:

- [Installing Percona XtraDB Cluster from Binary Tarball](https://docs.percona.com/percona-xtradb-cluster/8.4/tarball.html#install-from-binary-tarball)

If you want to build Percona XtraDB Cluster from source, see [Compiling and Installing from Source Code](https://docs.percona.com/percona-xtradb-cluster/8.4/compile.html#compile).

If you want to run Percona XtraDB Cluster using Docker, see [Run in a Docker container](https://docs.percona.com/percona-xtradb-cluster/8.4/docker.html).
