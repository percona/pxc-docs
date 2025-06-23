# Install from binary tarball

Percona provides generic tarballs with all essential files and binaries for manual installation. Using these binary tarballs, you can extract and install Percona XtraDB Cluster at your convenience, offering control and flexibility to the installation procedure. This method benefits advanced users or those with unique requirements that a standard installation method may not accommodate.



## Download standard releases

In *Percona XtraDB Cluster* 5.7.31-31.45 and later, the multiple binary tarballs available in the **Linux - Generic** section are replaced with the following:

| Named | Type| Description|
| ----------- | ----------- | ----------- | 
| Percona-XtraDB-Cluster-5.7.xx-relxx-xx-Linux.x86_64.glibc2.12.tar.gz| Full| Contains binaries, libraries, test files, and debug symbols|
| Percona-XtraDB-Cluster-5.7.xx-relxx-xx-Linux.x86_64.glibc2.12-minimal.tar.gz| Minimal | Contains binaries, and libraries but does not include test files, or debug symbols|

Both binary tarballs support all distributions.

For installations before *Percona XtraDB Cluster* 5.7.31-31.45, the **Linux - Generic** section contains multiple tarballs which are based on the *OpenSSL* library available in your distribution:


* `ssl100`: for Debian before 9 and Ubuntu before 14.04 versions


* `ssl101`: for CentOS 6 and CentOS 7


* `ssl102`: for Debian 9 and Ubuntu versions starting from 14.04

!!! note

    In CentOS version 7.04 and later, the *OpenSSL* library is `ssl102`.

For example, you can use `curl` as follows:

```text
curl -O https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/Percona-XtraDB-Cluster-5.7.31-31.45/binary/tarball/Percona-XtraDB-Cluster-5.7.31-rel34-31.45.1.Linux.x86_64.glibc2.tar.gz
```

[MySQL 5.7 post-EOL support from Percona program]: https://www.percona.com/post-mysql-5-7-eol-support


