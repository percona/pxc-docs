# Download a {{eol}} binary tarball

Version {{release}} contains fixes as part of the [MySQL 5.7 post-EOL support from Percona program], available to customers. Community members can [compile and install from Source Code](compile.md#compile) from publicly available source code, which is released quarterly.

As a Percona customer, request access to the Percona 5.7 Post-EOL repository from [Percona Support](https://www.percona.com/services/support/mysql-support) and receive your `CLIENTID` and `TOKEN`. Use these credentials to download the appropriate binary tarball.

| Type    | Name                                                                |Description         |
|---------|---------------------------------------------------------------------|--------------------|
| Full    | Percona-XtraDB-Cluster-&lt;release&gt;/private/[CLIENTID]-[TOKEN]/Percona-XtraDB-Cluster-5.7/Percona-XtraDB-Cluster-&lt;release&gt;/binary/tarball/Percona-XtraDB-Cluster-&lt;release&gt;-Linux.x86_64.glibc2.17.tar.gz   | Contains binaries, libraries, test files, and debug symbols   |
| Minimal | Percona-XtraDB-Cluster-&lt;release&gt;/private/[CLIENTID]-[TOKEN]/Percona-XtraDB-Cluster-5.7/Percona-XtraDB-Cluster-&lt;release&gt;-Linux.x86_64.glibc2.12-minimal.tar.gz | Contains binaries, and libraries but does not include test files, or debug symbols. |

Fetch and extract the correct binary tarball using your credentials. For example, for Oracle Linux 9, use the following command:

```{.bash data-prompt="$"}
 $ wget https://repo.percona.com/private/[CLIENTID]-[TOKEN]/Percona-XtraDB-Cluster-5.7/Percona-XtraDB-Cluster-{{release}}/binary/tarball/Percona-XtraDB-Cluster-{{release}}-Linux.x86_64.glibc2.17.tar.gz 
```

[MySQL 5.7 post-EOL support from Percona program]: https://www.percona.com/post-mysql-5-7-eol-support