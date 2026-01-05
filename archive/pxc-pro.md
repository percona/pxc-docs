# Percona XtraDB Cluster Pro

--8<--- "pro-build-announcement.md"

Non-paying Percona software users can also benefit from Percona Pro Builds, but they'll have to [build them from the source code](compile.md) provided by Percona and available to everyone.

## Capabilities

The following capabilities have been tested for {{release}} and are available in Percona XtraDB Cluster Pro:

| Name                                | Available since | Description  | 
| ----------------------------------- | ------------- | -------------|
|[Reduced backup lock time](https://docs.percona.com/percona-xtrabackup/8.4/reduction-in-locks.html)|8.4.6-6|The Percona XtraDB Cluster Pro `xtrabackup` SST (State Snapshot Transfer) method, based on Percona XtraBackup Pro, now uses the [Reduced backup lock time](https://docs.percona.com/percona-xtrabackup/8.4/reduction-in-locks.html) feature. This enhancement minimizes blocking on the **Donor node** during SST process while the backup is being prepared. The Percona XtraBackup reduced lock feature is enabled by default. To modify this behavior, set the desired `lock_ddl` value in the [xtrabackup] section of the `my.cnf` configuration file. For more information about the `--lock-ddl` option and its available values, refer to the [xtrabackup command-line options](https://docs.percona.com/percona-xtrabackup/8.4/xtrabackup-option-reference.html#lock-ddl) documentation.|
| Available on [Amazon Linux 2023](install-pro.md) | 8.4.4-4 |  Amazon Linux 2023 is a purpose-built Linux distribution optimized for AWS. It’s designed for performance, security, and seamless integration with the broader AWS ecosystem. We support both AMD64 and ARM64 versions of Amazon Linux 2023. |
| [FIPS compliance](fips.md)| 8.4.3-3 | FIPS compliance allows commercial cloud service providers to expand their presence with US government entities. |

## What's in it for you?

* Save on deploying and maintaining build infrastructure as we do the build and testing for you 
* Longer support for older versions of operating systems.  

[Install Percona XtraDB Cluster Pro](install-pro.md){.md-button}
