# Compile and install from source code

For routine deployments, install from [official packages or a binary tarball](tarball.md). Compile here only when you package PXC yourself, maintain local patches, or develop the server.

Clone [GitHub :octicons-link-external-16:](https://github.com/percona/percona-xtradb-cluster) and add every package from the table.

| Package   | apt                 | yum               |
| --------- | ------------------- | ----------------- |
| Git       | `git`               | `git`             |
| SCons     | `scons`             | `scons`           |
| GCC       | `gcc`               | `gcc`             |
| g++       | `g++`               | `gcc-c++`         |
| OpenSSL   | `openssl`           | `openssl`         |
| Check     | `check`             | `check`           |
| CMake     | `cmake`             | `cmake`           |
| Bison     | `bison`             | `bison`           |
| Boost     | `libboost-all-dev`  | `boost-devel`     |
| Asio      | `libasio-dev`       | `asio-devel`      |
| Async I/O | `libaio-dev`        | `libaio-devel`    |
| ncurses   | `libncurses5-dev`   | `ncurses-devel`   |
| Readline  | `libreadline-dev`   | `readline-devel`  |
| PAM       | `libpam-dev`        | `pam-devel`       |
| socat     | `socat`             | `socat`           |
| curl      | `libcurl-dev`       | `libcurl-devel`   |

## Install dependencies

Install anything still missing with one of these commands:

=== "on Debian or Ubuntu"

    ```shell
    sudo apt install -y git scons gcc g++ openssl check cmake bison \
    libboost-all-dev libasio-dev libaio-dev libncurses5-dev libreadline-dev \
    libpam-dev socat libcurl-dev
    ```

=== "on Red Hat Enterprise Linux"

    ```shell
    sudo yum install -y git scons gcc gcc-c++ openssl check cmake bison \
    boost-devel asio-devel libaio-devel ncurses-devel readline-devel pam-devel \
    socat libcurl-devel
    ```

### glibc version

XtraBackup archives stamp glibc into the filename; treat `glibc2.xx` as a placeholder and substitute your real suffix (for example `glibc2.28`). Choose archives whose glibc tag is compatible with the host where you unpack and run XtraBackup (usually the machine that builds PXC). Check that host with:

```shell
ldd --version
```

If the compile host and the runtime host differ, match CPU architecture and glibc, or build on a machine that resembles production.

## Compile

Compile Percona XtraDB Cluster from source:
{.power-number}

1. Clone a fresh tree—stale checkouts can break the build scripts:

    ```shell
    git clone https://github.com/percona/percona-xtradb-cluster.git
    ```

2. Check out the `{{vers}}` branch and pull submodules:

    ```shell
    cd percona-xtradb-cluster
    git checkout {{vers}}
    git submodule update --init --recursive
    ```


3. Download Percona XtraBackup {{vers}} and Percona XtraBackup 8.4 `*.tar.gz` bundles for your OS from [Percona Software Downloads :octicons-link-external-16:](https://www.percona.com/downloads/).

    The repository does not ship XtraBackup; `build-binary.sh` copies it out of `pxc-build/pxc_extra`. It looks for two separate install trees with fixed names—`pxb-{{vers}}` powers backup and restore, `pxb-8.4` powers cross-version State Snapshot Transfer (SST):

    * Percona XtraBackup {{vers}} backs up and restores data through the integrated tooling.

    * Percona XtraBackup 8.4 drives SST so a 9.7 node can join a cluster that still has 8.4 members.

    The snippets target `Linux-x86_64`. On ARM64, rewrite filenames for your chip (`aarch64`, `arm64`, …); never reuse paths that disagree with your download.

    Stage everything under `pxc-build/pxc_extra`, then rename to match the `mv` targets in the snippet. Map `glibc2.xx`, `{{vers}}.x`, and `8.4.x` to the strings in your real filenames (for example `{{release}}` and `8.4.0-3`). Aim each `tar` and `mv` at the top-level folder that unpack creates.

    ```shell
    mkdir -p ./pxc-build/pxc_extra

    tar -xvf percona-xtrabackup-{{vers}}-Linux-x86_64.glibc2.xx.tar.gz -C ./pxc-build
    mv ./pxc-build/percona-xtrabackup-{{vers}}-Linux-x86_64.glibc2.xx ./pxc-build/pxc_extra/pxb-{{vers}}

    tar -xvf percona-xtrabackup-8.4.x-Linux-x86_64.glibc2.xx.tar.gz -C ./pxc-build
    mv ./pxc-build/percona-xtrabackup-8.4.x-Linux-x86_64.glibc2.xx ./pxc-build/pxc_extra/pxb-8.4
    ```

4. Run `./build-ps/build-binary.sh` and pass an output directory (the script defaults to the current directory):

    ```shell
    mkdir -p ./pxc-build
    ./build-ps/build-binary.sh ./pxc-build
    ```

A clean build drops a fresh archive in `pxc-build`, typically `Percona-XtraDB-Cluster_<version-number>-Linux.x86_64.glibc2.xx.tar.gz` (full and minimal builds can both appear). Harvest `<version-number>` and `glibc2.xx` from the name the script prints.

!!! note

    Published version strings, glibc tags, and archive names vary by release.

## Install and next steps

Deploy the tarball like any [binary tarball](tarball.md#install-from-binary-tarball): one directory tree holds binaries, libraries, and helpers—see the version and archive table on that page.

1. Unpack it on every node at the path you will treat as `basedir`.

2. Install runtime OS packages from the Debian, Ubuntu, or Red Hat lists in [Install from Binary Tarball](tarball.md).

3. Create `datadir` and a MySQL option file, point `basedir` and `datadir` at your paths, and merge [Galera and cluster settings](configure-nodes.md#configure-nodes-for-write-set-replication) into the file.

4. Initialize the data directory before the first boot (for example `./bin/mysqld --initialize` from the unpacked tree, using the account and permissions your policy requires).

5. Start the cluster: [bootstrap the first node](bootstrap.md#bootstrap-the-first-node), then [join the rest](add-node.md#add-nodes-to-cluster).

Package installs default to `/usr` and systemd; tarball or hand-built trees use other paths, but bootstrap and join order stay the same. See [Get started with Percona XtraDB Cluster](get-started-cluster.md#get-started-with-percona-xtradb-cluster) and [Install Percona XtraDB Cluster](install-index.md#install-percona-xtradb-cluster) for end-to-end context.
