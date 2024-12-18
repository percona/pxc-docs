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

You may have already installed most of the packages. Run one of the following commands to install any missing
dependencies:

=== "on Debian or Ubuntu"

    ```{.bash data-prompt="$"}
    $ sudo apt install -y git scons gcc g++ openssl check cmake bison \
    libboost-all-dev libasio-dev libaio-dev libncurses5-dev libreadline-dev \
    libpam-dev socat libcurl-dev
    ```

=== "on Red Hat Enterprise Linux"

    ```{.bash data-prompt="$"}
    $ sudo yum install -y git scons gcc gcc-c++ openssl check cmake bison \
    boost-devel asio-devel libaio-devel ncurses-devel readline-devel pam-devel \
    socat libcurl-devel
    ```

### glibc version

The glibc (GNU C Library) version can differ across software builds due to several key factors:

| Reason                          | Description                                                                                                                                                                                                                   |
|---------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Operating System Variation      | When you build software on different Linux distributions or versions, each may ship with a different default glibc version. For example, Red Hat Enterprise Linux or Ubuntu might have distinct system library versions that impact compilation. |
| Backward Compatibility Considerations | Some applications are compiled to support multiple glibc versions. <br> - Developers often create builds that can run on older systems. <br> - This means intentionally targeting a slightly older glibc version for wider compatibility. |
| System Architecture Differences | 32-bit and 64-bit systems might require different glibc implementations. <br> - ARM, x86, and other processor architectures can have unique library requirements.                                                     |
| Security and Patch Levels       | Distributions backport security patches at different rates. <br> - A system's glibc version reflects its current security update status. <br> - Critical security updates can prompt version changes.                        |
| Compilation Environment         | The specific development environment and build tools used can directly influence which glibc version gets linked during compilation. Container environments, cross-compilation setups, and build servers might have unique library configurations. |

Practical Tip: Use `ldd --version` to check your current glibc version and understand potential compatibility constraints in your software ecosystem.



## Compile

To compile Percona XtraDB Cluster from source code:
{.power-number}

1. Clone the latest repository or update it to the latest state. The old codebase may not be compatible with the build script. Clone the Percona XtraDB Cluster repository:

    ```{.bash data-prompt="$"}
    $ git clone https://github.com/percona/percona-xtradb-cluster.git
    ```
    
2. Check out the `{{vers}}` branch and initialize submodules:

    ```{.bash data-prompt="$"}
    $ cd percona-xtradb-cluster
    $ git checkout {{vers}}
    $ git submodule update --init --recursive
    ```

3. Download the matching Percona XtraDB Cluster {{vers}} tarball (*.tar.gz) for your operating system from [Percona Software Downloads](https://www.percona.com/downloads/). The following example extracts the Percona XtraDB Cluster {{vers}} tar.gz file to the target directory `./pxc-build`:

    ```{.bash data-prompt="$"}
    $ tar -xvf percona-xtrabackup-{{vers}}-Linux-x86_64.glibc2.31.tar.gz -C ./pxc-build
    ```

4. Run the build script `./build-ps/build-binary.sh`. By default, it attempts to build into the current directory. Specify the target output directory, such as `./pxc-build`:

    ```{.bash data-prompt="$"}
    $ mkdir ./pxc-build
    $ ./build-ps/build-binary.sh ./pxc-build
    ```

When the compilation completes, `pxc-build` contains a tarball, such as `Percona-XtraDB-Cluster-{{vers}}.tar.gz`, that you can deploy on your system.

!!! note

    The exact version and release numbers may differ.
