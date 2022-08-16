# Percona XtraDB Cluster 5.7.18-29.20

Percona is glad to announce the release of
*Percona XtraDB Cluster* 5.7.18-29.20 on June 2, 2017.
Binaries are available from the [downloads section](https://www.percona.com/downloads/Percona-XtraDB-Cluster-57/)
or from our [software repositories](../install/index.md#install).

!!! note

    Due to new package dependency, Ubuntu/Debian users should use `apt-get dist-upgrade` or `apt-get install percona-xtradb-cluster-57` to upgrade.

Percona XtraDB Cluster 5.7.18-29.20 is now the current release,
based on the following:


* [Percona Server 5.7.18-15](https://www.percona.com/doc/percona-server/5.7/release-notes/Percona-Server-5.7.18-15.html)


* Galera Replication library 3.20


* wsrep API version 29

All Percona software is open-source and free.

## Fixed Bugs


* [PXC-749](https://jira.percona.com/browse/PXC-749): Fixed memory leak when running `INSERT` on a table without primary key defined
and `wsrep_certify_nonPK` disabled (set to `0`).

  !!! note

      It is recommended to have primary keys defined on all tables for correct write set replication.

* [PXC-812](https://jira.percona.com/browse/PXC-812): Fixed SST script to leave the DONOR keyring when JOINER clears the datadir.

* [PXC-813](https://jira.percona.com/browse/PXC-813): Fixed SST script to use UTC time format.

* [PXC-816](https://jira.percona.com/browse/PXC-816): Fixed hook for caching GTID events in asynchronous replication.
  For more information, see [#1681831](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1681831).

* [PXC-820](https://jira.percona.com/browse/PXC-820): Enabled querying of `pxc_maint_mode` by another client during the transition period.

* [PXC-823](https://jira.percona.com/browse/PXC-823): Fixed SST flow to gracefully shut down JOINER node if SST fails because DONOR leaves the cluster due to network failure. This ensures that the DONOR is then able to recover to synced state when network connectivity is restored
  For more information, see [#1684810](https://bugs.launchpad.net/percona-xtradb-cluster/+bug/1684810).

* [PXC-824](https://jira.percona.com/browse/PXC-824): Fixed graceful shutdown of Percona XtraDB Cluster node to wait until applier thread finishes.

## Other Improvements

* [PXC-819](https://jira.percona.com/browse/PXC-819): Added five new status variables
to expose required values from `wsrep_ist_receive_status` and `wsrep_flow_control_interval` as numbers, rather than strings that need to be parsed:

    * `wsrep_flow_control_interval_low`


    * `wsrep_flow_control_interval_high`


    * `wsrep_ist_receive_seqno_start`


    * `wsrep_ist_receive_seqno_current`


    * `wsrep_ist_receive_seqno_end`
