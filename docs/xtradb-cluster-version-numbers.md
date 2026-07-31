# Understand version numbers

A version number identifies the innovation product release. The product contains the latest features, improvements, and bug fixes at the time of that release.

| 9.7.1 | -1 |
|---|---|
| Base version | Minor build version |

Percona uses semantic version numbering, which follows the pattern of base version and build version. Percona assigns unique, non-negative integers in increasing order for each version release. The version number combines the base Percona Server for MySQL version number, the minor build version, and the custom build version, if needed.

The version numbers for Percona XtraDB Cluster {{release}} define the following information:

* Base version - the leftmost numbers indicate [MySQL {{vers}} :octicons-link-external-16:](https://dev.mysql.com/doc/relnotes/mysql/{{vers}}/en/) version used as a base. 

* Minor build version - an internal number that increases by one every time Percona XtraDB Cluster is released.