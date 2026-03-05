# Compile and install from Source Code

If you want to compile Percona XtraDB Cluster, you can find the source code on
[GitHub](https://github.com/percona/percona-xtradb-cluster).
Before you begin, make sure that the following packages are installed:

|  | apt| yum|
| --- | ----- | --- | 
| Git| `git` | `git`|
| SCons | `scons` | `scons`|
| GCC| `gcc` | `gcc` |
| g++ | `g++` | `gcc-c++` |
| OpenSSL| `openssl` | `openssl`|
| Check| `check`| `check` |
| CMake| `cmake` | `cmake` |
| Bison| `bison` | `bison`|
| Boost | `libboost-all-dev` | `boost-devel` |
| Asio| `libasio-dev`| `asio-devel` |
| Async I/O| `libaio-dev` | `libaio-devel`|
| ncurses | `libncurses5-dev` | `ncurses-devel`|
| Readline| `libreadline-dev`| `readline-devel`|
| PAM | `libpam-dev`| `pam-devel`|
| socat| `socat` | `socat`|
| curl | `libcurl-dev` | `libcurl-devel`|

## Check packages

You will likely have all or most of the packages already installed. If you are
not sure, run one of the following commands to install any missing
dependencies:

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

## Compile

To compile Percona XtraDB Cluster from source code:
{.power-number}

1. Clone the latest repository or update it to the latest state. The old codebase may not be compatible with the build script. Clone the Percona XtraDB Cluster repository:

    ```shell
    git clone https://github.com/percona/percona-xtradb-cluster.git
    ```

2. Check out the `8.0` branch and initialize submodules:

    ```shell
    cd percona-xtradb-cluster
    git checkout 8.0
    git submodule update --init --recursive
    ```

3. Download the matching Percona XtraBackup 8.0 tarball (*.tar.gz) for your operating system from [Percona Downloads](https://www.percona.com/downloads/). 

    The following example extract the Percona XtraBackup 8.0.32-25 tar.gz file to the target directory `./pxc-build`:

    ```shell
    tar -xvf percona-xtrabackup-8.0.32-25-Linux-x86_64.glibc2.17.tar.gz -C ./pxc-build
    ```

3. Download the matching Percona XtraBackup tarballs (`*.tar.gz`) for your operating system from [Percona Downloads](https://www.percona.com/downloads/).

    For Percona XtraDB Cluster 8.0 source builds, both of the following are required:

    * Percona XtraBackup 8.0  
    * Percona XtraBackup 2.4  

    Create the target directory:

    ```shell
    mkdir -p ./pxc-build
    ```

    Extract each tarball into `./pxc-build`:

    ```shell
    tar -xvf percona-xtrabackup-8.0.32-25-Linux-x86_64.glibc2.17.tar.gz -C ./pxc-build
    tar -xvf percona-xtrabackup-2.4.xx-Linux-x86_64.glibc2.17.tar.gz -C ./pxc-build
    ```

4. Run the build script `./build-ps/build-binary.sh`.

    By default, it attempts building into the current directory. Specify
    the target output directory, such as `./pxc-build`:

    ```shell
    mkdir ./pxc-build
    ./build-ps/build-binary.sh ./pxc-build
    ```

When the compilation completes, `pxc-build` contains a tarball, such as `Percona-XtraDB-Cluster-8.0.x86_64.tar.gz`, that you can deploy on your system.

!!! note

    The exact version and release numbers may differ.
