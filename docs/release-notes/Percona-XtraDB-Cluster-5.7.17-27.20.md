# Percona XtraDB Cluster 5.7.17-27.20

Percona is glad to announce the release of
*Percona XtraDB Cluster* 5.7.17-27.20 on March 16, 2017.
Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/)
or from our [software repositories](../install/index.md#install).

Percona XtraDB Cluster 5.7.17-27.20 is now the current release,
based on the following:

* [Percona Server 5.7.17-11](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.17-11.html)

* Galera Replication library 3.20

* wsrep API version 27

All Percona software is open-source and free.
Details of this release can be found in the
[5.7.17-27.20 milestone on Launchpad](https://launchpad.net/percona-xtradb-cluster/+milestone/5.7.17-27.20).

## Fixed Bugs

* [BLD-512](https://jira.percona.com/browse/BLD-512): Fixed startup of `garbd`
on Ubuntu 16.04.2 LTS (Xenial Xerus).

* [BLD-519](https://jira.percona.com/browse/BLD-519): Added the `garbd` debug package to the repository.

* [BLD-569](https://jira.percona.com/browse/BLD-569): Fixed `grabd` script to return non-zero
if it fails to start.

* [BLD-570](https://jira.percona.com/browse/BLD-570): Fixed service script for `garbd`
on Ubuntu 16.04.2 LTS (Xenial Xerus) and Ubuntu 16.10 (Yakkety Yak).

* [BLD-593](https://jira.percona.com/browse/BLD-593): Limited the use of `rm` and `chown`
by `mysqld_safe` to avoid exploits of the CVE-2016-5617 vulnerability.
  For more information, see [#1660265](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1660265).

Credit to Dawid Golunski ([https://legalhackers.com](https://legalhackers.com)).

* [BLD-610](https://jira.percona.com/browse/BLD-610): Added version number to the dependency requirements
of the full RPM package.

* [BLD-643](https://jira.percona.com/browse/BLD-643): Fixed `systemctl` to mark `mysql` process as inactive after it fails to start and not attempt to start it again.
  For more information, see [#1662292](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1662292).

* [BLD-644](https://jira.percona.com/browse/BLD-644): Added the `which` package to Percona XtraDB Cluster dependencies
on CentOS 7.
  For more information, see [#1661398](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1661398).

* [BLD-645](https://jira.percona.com/browse/BLD-645): Fixed `mysqld_safe` to support options
with a forward slash (`/`).
  For more information, see [#1652838](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1652838).

* [BLD-647](https://jira.percona.com/browse/BLD-647): Fixed `systemctl` to show correct status
for `mysql` on CentOS 7.
  For more information, see [#1644382](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1644382).
