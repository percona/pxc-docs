# Understand version numbers

A version number identifies the product release. The product contains the latest Generally Available (GA) features at the time of that release.

| 8.0.20| -11. | 2 |
|---|---|---|
| Base version | Minor build | Custom build |

Percona uses semantic version numbering, which follows the pattern of base version, minor build, and an optional custom build. Percona assigns unique, non-negative integers in increasing order for each minor build release. The version number combines the base Percona Server for MySQL version number, the minor build version, and the custom build version, if needed.

The version numbers for Percona XtraDB Cluster 8.0.20-11.2 define the following information:

* Base version - the leftmost set of numbers that indicate the Percona Server for MySQL version used as a base. An increase in the base version resets the minor build version and the custom build version to 0. 

* Minor build version - an internal number that increases with every Percona XtraDB Cluster release, and the custom build number is reset to 0.

* Custom build version - an optional number assigned to custom builds used for bug fixes. The features don't change unless the fixes include those features. For example, Percona XtraDB Cluster 8.0.20-11.1, 8.0.20-11.2, and 8.0.20-11.3 are based on the same Percona Server for MySQL version and minor build version but are custom build versions.
