# Install from Binary Tarball

Percona provides generic tarballs with all required files and binaries
for manual installation.

You can download the appropriate tarball package from
[Percona Software Downloads :octicons-link-external-16:](https://www.percona.com/downloads).

--8<--- "get-help-snip.md"

### Version updates

The version number in the tarball name must be substituted with
the appropriate version number for your system. To indicate that such a
substitution is needed in statements, we use `<version-number>`.

| Name| Type| Description|
| ---- | ---- | ---------- | 
| Percona-XtraDB-Cluster_&lt;version-number&gt;-Linux.x86_64.glibc2.17.tar.gz| Full| Contains binary files, libraries, test files, and debug symbols|
| Percona-XtraDB-Cluster_&lt;version-number&gt;-Linux.x86_64.glibc2.17.minimal.tar.gz| Minimal| Contains binary files and libraries but does not include test files, or debug symbols|

Check your system to make sure the packages that the PXC
version requires are installed.

```shell
# Debian 12, Ubuntu 22.04
sudo apt-get install -y socat libdbd-mysql-perl libaio1 libc6 libcurl4 libev4 \
libgcc-s1 libgcrypt20 libgpg-error0 libssl3 libstdc++6 zlib1g libatomic1

# Debian 13, Ubuntu 24.04, Ubuntu 26.04
sudo apt-get install -y socat libdbd-mysql-perl libaio1t64 libc6 libcurl4t64 libev4t64 \
libgcc-s1 libgcrypt20 libgpg-error0 libssl3t64 libstdc++6 zlib1g libatomic1

# Red Hat Enterprise Linux 8, 9, 10
sudo dnf install -y openssl socat procps-ng chkconfig coreutils shadow-utils
```
