# Glossary

### ACID

Set of properties that guarantee database transactions are processed reliably. Stands for [`Atomicity`](glossary.md#atomicity), [`Consistency`](glossary.md#consistency), [`Isolation`](glossary.md#isolation), [`Durability`](glossary.md#durability).

### ACID

Atomicity means that database operations are applied following a "all or nothing" rule. A transaction is either fully applied or not at all.

### Consistency

Consistency means that each transaction that modifies the database takes it from one consistent state to another.

### Durability

Once a transaction is committed, it will remain so.

### Foreign Key

A referential constraint between two tables. Example: A purchase order in the purchase_orders table must have been made by a customer that exists in the customers table.

### Isolation

The Isolation requirement means that no transaction can interfere with another.

### InnoDB A

[`Storage Engine`](glossary.md#storage-engine) for MySQL and derivatives ([`Percona Server`](glossary.md#percona-server), [`MariaDB`](glossary.md#mariadb)) originally written by Innobase Oy, since acquired by Oracle. It provides [`ACID`](glossary.md#acid) compliant storage engine with [`foreign key`](glossary.md#foreign-key) support. InnoDB is the default storage engine on all platforms.

### Jenkins

[Jenkins](https://www.jenkins-ci.org) is a continuous integration system that we use to help ensure the continued quality of the software we produce. It helps us achieve the aims of:
* no failed tests in trunk on any platform,
* aid developers in ensuring merge requests build and test on all platforms,
* no known performance regressions (without a damn good explanation).

### LSN

Log Serial Number. A term used in relation to the [`InnoDB`](glossary.md#innodb) or [`XtraDB`](glossary.md#xtradb) storage engines. There are System-level LSNs and Page-level LSNs. The System LSN represents the most recent LSN value assigned to page changes. Each InnoDB page contains a Page LSN which is the max LSN for that page for changes that reside on the disk. This LSN is updated when the page is flushed to disk.

### MariaDB

A fork of [`MySQL`](glossary.md#mysql) that is maintained primarily by Monty Program AB. It aims to add features, fix bugs while maintaining 100% backwards compatibility with MySQL.

### MyISAM

A [`MySQL`](glossary.md#mysql) [`Storage Engine`](glossary.md#storage-engine) that was the default until MySQL 5.5. It doesn't fully support transactions but in some scenarios may be faster than [`InnoDB`](glossary.md#innodb). Each table is stored on disk in 3 files: [`.frm`](glossary.md#frm),i `.MYD`, `.MYI`.

### MySQL

An open source database that has spawned several distributions and forks. MySQL AB was the primary maintainer and distributor until bought by Sun Microsystems, which was then acquired by Oracle. As Oracle owns the MySQL trademark, the term MySQL is often used for the Oracle distribution of MySQL as distinct from the drop-in replacements such as [`MariaDB`](glossary.md#mariadb) and [`Percona Server`](glossary.md#percona-server).

### NUMA

Non-Uniform Memory Access ([`NUMA`](http://en.wikipedia.org/wiki/Non-Uniform_Memory_Access)) is a computer memory design used in multiprocessing, where the memory access time depends on the memory location relative to a processor. Under NUMA, a processor can access its own local memory faster than non-local memory, that is, memory local to another processor or memory shared between processors. The whole system may still operate as one unit, and all memory is basically accessible from everywhere, but at a potentially higher latency and lower performance.

### Percona Server for MySQL

Percona's branch of [`MySQL`](glossary.md#mysql) with performance and management improvements.

### Percona Server

See [`Percona Server for MySQL`](glossary.md#percona-server-for-mysql)

### Storage Engine

A [`Storage Engine`](glossary.md#storage-engine) is a piece of software that implements the details of data storage and retrieval for a database system. This term is primarily used within the [`MySQL`](glossary.md#mysql) ecosystem due to it being the first widely used relational database to have an abstraction layer around storage. It is analogous to a Virtual File System layer in an Operating System. A VFS layer allows an operating system to read and write multiple file systems (for example, FAT, NTFS, XFS, ext3) and a Storage Engine layer allows a database server to access tables stored in different engines (e.g. [`MyISAM`](glossary.md#myisam), InnoDB).

### InnoDB

Storage engine which provides ACID-compliant transactions and foreign key support, among others improvements over [`MyISAM`](glossary.md#myisam). It is the default engine for MySQL as of the 5.5 series.

### GTID

Global Transaction ID, in Percona XtraDB Cluster it consists of [`UUID`](glossary.md#uuid) and an ordinal sequence number which denotes the position of the change in the sequence.

### HAProxy

[`HAProxy`](https://www.haproxy.com/) is a free, very fast and reliable solution offering high availability, load balancing, and proxying for TCP and HTTP-based applications. It is particularly suited for web sites crawling under very high loads while needing persistence or Layer7 processing. Supporting tens of thousands of connections is clearly realistic with todays hardware. Its mode of operation makes its integration into existing architectures very easy and riskless, while still offering the possibility not to expose fragile web servers to the net.

### IST

Incremental State Transfer. Functionality which instead of whole state snapshot can catch up with the group by receiving the missing writesets, but only if the writeset is still in the donor's writeset cache.

### SST

State Snapshot Transfer is the full copy of data from one node to another.  It's used when a new node joins the cluster, it has to transfer data from an existing node.

Percona XtraDB Cluster: uses the `xtrabackup` program for this purpose. `xtrabackup` does not require `READ LOCK` for the entire syncing process - only for syncing the MySQL system tables and writing the information about the binlog, galera and replica information (same as the regular Percona XtraBackup backup).

The SST method is configured with the [`wsrep_sst_method`](wsrep-system-index.md#wsrep_sst_method) variable.
  
In PXC 8.0, the **mysql-upgrade** command is now run
automatically as part of [`SST`](glossary.md#sst). You do not have to run it
manually when upgrading your system from an older version.

### UUID

Universally Unique IDentifier which uniquely identifies the state and the sequence of changes node undergoes. 128-bit UUID is a classic DCE UUID Version 1 (based on current time and MAC address). Although in theory this UUID could be generated based on the real MAC-address, in the Galera it is always (without exception) based on the generated pseudo-random addresses ("locally administered" bit in the node address (in the UUID structure) is  always equal to unity).
  
Complete structure of the 128-bit UUID field and explanation for its generation are as follows:

| From| To| Length| Content|
| --- | --- | --- | --- |  
|  |  |  |  |  |  |  |  |  |
| 0| 31| 32| Bits 0-31 of Coordinated Universal Time (UTC) as a count of 100-nanosecond intervals since 00:00:00.00, 15 October 1582 encoded as big-endian 32-bit number.|
| 32| 47| 16| Bits 32-47 of UTC as a count of 100-nanosecond intervals since 00:00:00.00, 15 October 1582, encoded as big-endian 16-bit number.|
| 48| 59| 12| Bits 48-59 of UTC as a count of 100-nanosecond intervals since 00:00:00.00, 15 October 1582, encoded as big-endian 16-bit number.|
| 60| 63| 4| UUID version number: always equal to 1 (DCE UUID).
| 64| 69| 6| most-significants bits of random number, which generated from the server process PID and Coordinated Universal Time (UTC) as a count of 100-nanosecond intervals since 00:00:00.00, 15 October 1582.|
| 70| 71| 2| UID variant: always equal to binary 10 (DCE variant).|
| 72| 79| 8| 8 least-significant bits of  random number, which generated from the server process PID and Coordinated Universal Time (UTC) as a count of 100-nanosecond intervals since 00:00:00.00, 15 October 1582.|
| 80| 80| 1| Random bit (“unique node identifier”).
| 81| 81| 1| Always equal to the one (“locally administered MAC address”).|
| 82| 127| 46| Random bits (“unique node identifier”): readed from the `/dev/urandom` or (if `/dev/urandom` is unavailable) generated based on the server process PID, current time and bits of the default “zero node identifier” (entropy data).|

### XtraBackup

*Percona XtraDB* is an enhanced version of the InnoDB storage engine, designed to better scale on modern hardware, and including a variety of other features useful in high performance environments. It is fully backwards compatible, and so can be used as a drop-in replacement for standard InnoDB. See [`The Percona XtraDB Storage Engine`](https://docs.percona.com/percona-server/8.0/).

### XtraDB Cluster

Percona XtraDB Cluster is a high availability solution for MySQL.

### Percona XtraDB Cluster

Percona XtraDB Cluster (PXC) is a high availability solution for MySQL.

### my.cnf

This file refers to the database server’s main configuration file. Most Linux distributions place it as `/etc/mysql/my.cnf` or
`/etc/my.cnf`, but the location and name depends on the particular
installation. Note that this is not the only way of configuring the
server, some systems does not have one even and rely on the command
options to start the server and its defaults values.

### cluster replication

Normal replication path for cluster members. Can be encrypted (not by
default) and unicast or multicast (unicast by default). Runs on tcp port 4567 by default.

### datadir

The directory in which the database server stores its databases. Most Linux distribution use `/var/lib/mysql` by default.

### donor node

The node elected to provide a state transfer (SST or IST).

### ibdata

Default prefix for tablespace files, e.g., `ibdata1` is a 10MB
autoextendable file that *MySQL* creates for the shared tablespace by
default.

### joiner node

The node joining the cluster, usually a state transfer target.

### node

A cluster node – a single mysql instance that is in the cluster.

### primary cluster

A cluster with [quorum](glossary.md#quorum). A non-primary cluster will not allow any
operations and will give `Unknown command` errors on any clients
attempting to read or write from the database.

### quorum

A majority (> 50%) of nodes. In the event of a network partition, only the cluster partition that retains a quorum (if any) will remain Primary by default.

### split brain

Split brain occurs when two parts of a computer cluster are disconnected, each part believing that the other is no longer running. This problem can lead to data inconsistency.

### .frm

For each table, the server will create a file with the `.frm`
extension containing the table definition (for all storage engines).

### mysql.pxc.internal.session

This user is used by the SST process to run the SQL commands needed for [`SST`](glossary.md#sst), such as creating the `mysql.pxc.sst.user` and assigning it the role [`mysql.pxc.sst.role`](glossary.md#mysql-pxc-sst-role).

**Difference from previous major version 5.7**

The `wsrep-sst-auth` variable is no longer needed and has been removed.

### mysql.pxc.sst.role

This role has all the privileges needed to run xtrabackup to create a backup on the donor node.

### mysql.pxc.sst.user

This user (set up on the donor node) is assigned the [`mysql.pxc.sst.role`](glossary.md#mysql-pxc-sst-role) and runs the XtraBackup to make backups. The password for this is randomly generated for each SST. The password is generated automatically for each [`SST`](glossary.md#sst).