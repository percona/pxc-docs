# Understand version numbers

A version number identifies the product release. The product contains the latest Generally Available (GA) features at the time of that release.

<style>
    table {
        border-collapse: collapse;
        width=100%;
    }
    table td {
        border: 2px solid black;
        padding: 8px;
        text-align: center;
    }
    tr:nth-child(even){
        background-color:#f5f5f5
    }
</style>

| 8.4.4| -4. | 2 |
|---|---|---|
| Base version | Minor build | Custom build |

Percona uses semantic version numbering, which follows the pattern of base version, minor build, and optional custom build. For each minor build release, Percona assigns unique, non-negative integers in increasing order. The version number combines the base Percona Server for MySQL version number, the minor build version, and the custom build version, if needed.

The version numbers for Percona XtraDB Cluster 8.4.4-4.2 define the following information:

* Base version - the leftmost set of numbers that indicate the Percona Server for MySQL version used as a base. Increasing the base version resets the minor build version and the custom build version to 0. 

* Minor build version - an internal number that increases with every Percona XtraDB Cluster release, and the custom build number is reset to 0.

* Custom build version - an optional number assigned to custom builds used for bug fixes. The features don't change unless the fixes include those features. For example, Percona XtraDB Cluster 8.4.4-4.1, 8.4.4-4.2, and 8.4.4-4.3 are based on the same Percona Server for MySQL and minor build versions, but are custom-built.
