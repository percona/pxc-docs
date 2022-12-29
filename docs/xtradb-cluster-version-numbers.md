# Percona XtraDB Cluster 5.7 version numbers

A version number identifies the product release. The product contains the latest Generally Available (GA) features at the time of that release. Percona XtraDB Cluster assigns a set of unique numbers for each version in increasing order. The numbers are a combination of the Percona Server for MySQL version and internal version numbers denoting the build.

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

| 5.7.40| 31 | 63 | 2 |
|---|---|---|---|
| Base version | WSREP API version | Minor build | Custom build |

Percona uses semantic version numbering, which follows the pattern of base version, minor build, and an optional custom build. Percona assigns unique, non-negative integers in increasing order for each minor build release. The version number combines the base Percona Server for MySQL version number, the minor build version, and the custom build version, if needed.

The version numbers for Percona XtraDB Cluster 5.7.40-31.63 define the following information:

* Base version - the leftmost set of numbers indicate the Percona Server for MySQL version used as a base. An increase in base version resets the minor build version and the custom build version to 0.

* WSREP API version - the version of the WSREP API

* Minor build version - an internal number that denotes which version. When this number increases by one, the custom build fix is reset to 0.

* Custom build version - an optional number assigned to custom builds used for bug fixes. The software features, unless they're included in the bug fix, don't change.
