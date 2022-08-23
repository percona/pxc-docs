# Compiling and Installing from Source Code

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

You will likely have all or most of the packages already installed. If you are
not sure, run one of the following commands to install any missing
dependencies:

* For Debian or Ubuntu:

    ```shell
    $ sudo apt install -y git scons gcc g++ openssl check cmake bison \
    libboost-all-dev libasio-dev libaio-dev libncurses5-dev libreadline-dev \
    libpam-dev socat libcurl-dev
    ```

* For Red Hat Enterprise Linux or CentOS:

    ```shell
    $ sudo yum install -y git scons gcc gcc-c++ openssl check cmake bison \
    boost-devel asio-devel libaio-devel ncurses-devel readline-devel pam-devel \
    socat libcurl-devel
    ```

To compile Percona XtraDB Cluster from source code:


1. Clone the Percona XtraDB Cluster repository:

    ```shell
    $ git clone https://github.com/percona/percona-xtradb-cluster.git
    ```
    
    !!! important

        Clone the latest repository or update it to the latest state.
        Old codebase may not be compatible with the build script.

2. Check out the `8.0` branch:

    ```shell
    $ cd percona-xtradb-cluster-galera
    $ git checkout 8.0
    ```

3. Initialize the submodule:

    ```shell
    $ git submodule init wsrep/src && git submodule update wsrep/src
    $ git submodule init percona-xtradb-cluster-galera && git submodule update percona-xtradb-cluster-galera
    $ cd  percona-xtradb-cluster-galera
    $ git submodule init wsrep/src && git submodule update wsrep/src
    & git submodule init && git submodule update
    $ cd ..
    ```

4. Run the build script `./build-ps/build-binary.sh`.
    By default, it attempts building into the current directory. Specify
    the target output directory, such as `./pxc-build`:

    ```shell
    $ mkdir ./pxc-build
    $ ./build-ps/build-binary.sh ./pxc-build
    ```

When the compilation completes, `pxc-build` contains a tarball, such as `Percona-XtraDB-Cluster-8.0.x86_64.tar.gz`, that you can deploy on your system.

!!! note

    The exact version and release numbers may differ.
