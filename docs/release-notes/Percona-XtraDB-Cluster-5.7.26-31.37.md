# Percona XtraDB Cluster 5.7.26-31.37

Percona is glad to announce the release of Percona XtraDB Cluster 5.7.26-31.37 on
June 26, 2019.  Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/) or from our
[software repositories](../install/index.md#install).

Percona XtraDB Cluster 5.7.26-31.37 is now the current release, based on the following:

<!-- The versions of Galera are probably not correct -->

* [Percona Server for MySQL 5.7.26-29](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.26-29.html)


* Galera Replication library 3.26


* Galera/Codership WSREP API Release 5.7.25

## Bugs Fixed


* [#2480](https://jira.percona.com/browse/PXC-2480): In some cases, Percona XtraDB Cluster could not replicate `CURRENT_USER()`
used in the `ALTER` statement. `USER()` and `CURRENT_USER()` are no
longer allowed in any `ALTER` statement since they fail when replicated.


* [#2487](https://jira.percona.com/browse/PXC-2487): The case when a DDL or DML action was in progress from one
client and the provider was updated from another client could result in a race
condition.


* [#2490](https://jira.percona.com/browse/PXC-2490): Percona XtraDB Cluster could crash when `binlog_space_limit` was set to a value other than zero during
`wsrep_recover` mode.


* [#2491](https://jira.percona.com/browse/PXC-2491): SST could fail if the donor had encrypted undo logs.


* [#2537](https://jira.percona.com/browse/PXC-2537): Nodes could crash after an attempt to set a password using
`mysqladmin`


* [#2497](https://jira.percona.com/browse/PXC-2497): The user can set the preferred donor by setting the `wsrep_sst_donor` variable. An IP address is not valid as the value of
this variable. If the user still used an IP address, an error message was
produced that did not provide sufficient information. The error message has been
improved to suggest that the user check the value of the `wsrep_sst_donor` for an IP address.

<!-- 2560 is not public yet -->
**Other bugs fixed**:
[#2276](https://jira.percona.com/browse/PXC-2276),
[#2292](https://jira.percona.com/browse/PXC-2292),
[#2476](https://jira.percona.com/browse/PXC-2476),
[#2560](https://jira.percona.com/browse/PXC-2560)
