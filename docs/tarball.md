# Install from Binary Tarball

Percona provides generic tarballs with all required files and binaries
for manual installation.

You can download the appropriate tarball package from
[Percona Software Downloads](https://www.percona.com/downloads).

--8<--- "get-help-snip.md"

### Version updates

The version number in the tarball name must be substituted with
the appropriate version number for your system. To indicate that such a
substitution is needed in statements, we use `<version-number>`.

| Name| Type| Description|
| ---- | ---- | ---------- | 
| Percona-XtraDB-Cluster_<version-number>-Linux.x86_64.glibc2.17.tar.gz| Full| Contains binary files, libraries, test files, and debug symbols|
| Percona-XtraDB-Cluster_<version-number>-Linux.x86_64.glibc2.17.minimal.tar.gz| Minimal| Contains binary files and libraries but does not include test files, or debug symbols|

Check your system to make sure the packages that the PXC
version requires are installed.

### For Debian or Ubuntu:

```{.bash data-prompt="$"}
$ sudo apt-get install -y \
socat libdbd-mysql-perl \
libaio1 libc6 libcurl3 libev4 libgcc1 libgcrypt20 \
libgpg-error0 libssl1.1 libstdc++6 zlib1g libatomic1
```

### For Red Hat Enterprise Linux:

```{.bash data-prompt="$"}
$ sudo yum install -y openssl socat  \
procps-ng chkconfig procps-ng coreutils shadow-utils \
```
