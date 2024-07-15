# Glossary

## .frm

For each table, the server will create a file with the `.frm`
extension containing the table definition (for all storage engines).

## ACID

An acronym for [`Atomicity`](glossary.md#atomicity), [`Consistency`](glossary.md#consistency), [`Isolation`](glossary.md#isolation), [`Durability`](glossary.md#durability).

## Asynchronous replication

Asynchronous replication is a technique where data is first written to the primary node. After the primary acknowledges the write, the data is written to secondary nodes.

## Atomicity

This property guarantees that all updates of a transaction occur in the database or no updates occur. This guarantee also applies with a server exit. If a transaction fails, the entire operation rolls back.

## Cluster

A group of interconnected computers or servers that work together as a single system to ensure high availability and scalability of applications and databases.

## Cluster replication

Normal replication path for cluster members. Can be encrypted (not by
default) and unicast or multicast (unicast by default). Runs on tcp port 4567 by default.

## Consistency

This property guarantees that each transaction that modifies the database takes it from one consistent state to another. Consistency is implied with [Isolation](#isolation).

## Data definition language (DDL)

A subset of SQL commands used to define, alter, and manage database schema objects like tables, indexes, and columns.

## Data inconsistencies

Situations where data is not uniform across different nodes or parts of a system, potentially leading to errors or incorrect results.

## Data integrity

The accuracy and consistency of data stored in a database, ensuring that data is reliable and correct over its lifecycle.

## Database Schema

The structure that defines the organization of data in a database, including tables, columns, and relationships.

## datadir

The directory in which the database server stores its databases. Most Linux distribution use `/var/lib/mysql` by default.

## Deadlocks

A situation where two or more transactions are unable to proceed because each is waiting for the other to release resources, causing a standstill.

## donor node

The node elected to provide a state transfer (SST or IST).

## Downtime

The period during which a system or service is unavailable or not operational.

## Durability

Once a transaction is committed, it will remain so and is resistant to a server exit.

## Flow control

A mechanism to manage the rate of data replication in a cluster.

## Foreign Key

A referential constraint between two tables. Example: A purchase order in the purchase_orders table must have been made by a customer that exists in the customers table.

## General availability (GA)

A finalized version of the product which is made available to the general public. It is the final stage in the software release cycle.

## GTID

Global Transaction ID, in Percona XtraDB Cluster it consists of [`UUID`](glossary.md#uuid) and an ordinal sequence number which denotes the position of the change in the sequence.

## HAProxy

[`HAProxy`](https://www.haproxy.com/) is a free, very fast and reliable solution offering high availability, load balancing, and proxying for TCP and HTTP-based applications. It is particularly suited for web sites crawling under very high loads while needing persistence or Layer7 processing. Supporting tens of thousands of connections is clearly realistic with todays hardware. Its mode of operation makes its integration into existing architectures very easy and riskless, while still offering the possibility not to expose fragile web servers to the net.

## ibdata

Default prefix for tablespace files, e.g., `ibdata1` is a 10MB
auto-extendable file that MySQL creates for the shared tablespace by
default.

## Isolation

The Isolation guarantee means that no transaction can interfere with another. When transactions access data in a session, they also lock that data to prevent other operations on that data by other transaction.

## IST

Incremental State Transfer. Functionality which instead of whole state snapshot can catch up with the group by receiving the missing writesets, but only if the writeset is still in the donor's writeset cache.

## InnoDB

[`Storage Engine`](glossary.md#storage-engine) for MySQL and derivatives ([`Percona Server`](glossary.md#percona-server), [`MariaDB`](glossary.md#mariadb)) originally written by Innobase Oy, since acquired by Oracle. It provides [`ACID`](glossary.md#acid) compliant storage engine with [`foreign key`](glossary.md#foreign-key) support. InnoDB is the default storage engine on all platforms.

## Isolation

Temporarily removing a node from the cluster to perform operations without affecting the rest of the cluster.

## joiner node

The node joining the cluster, usually a state transfer target.

## LSN

Log Serial Number. A term used in relation to the [`InnoDB`](glossary.md#innodb) or [`XtraDB`](glossary.md#xtradb) storage engines. There are System-level LSNs and Page-level LSNs. The System LSN represents the most recent LSN value assigned to page changes. Each InnoDB page contains a Page LSN which is the max LSN for that page for changes that reside on the disk. This LSN is updated when the page is flushed to disk.

## MariaDB

A fork of [`MySQL`](glossary.md#mysql) that is maintained primarily by Monty Program AB. It aims to add features, fix bugs while maintaining 100% backwards compatibility with MySQL.

## Metadata lock

A type of lock in a database that prevents changes to the structure of database objects while certain operations are being performed.

## my.cnf

This file refers to the database server’s main configuration file. Most Linux distributions place it as `/etc/mysql/my.cnf` or
`/etc/my.cnf`, but the location and name depends on the particular
installation. Note that this is not the only way of configuring the
server, some systems does not have one even and rely on the command
options to start the server and its defaults values.

## MyISAM

A [`MySQL`](glossary.md#mysql) [`Storage Engine`](glossary.md#storage-engine) that was the default until MySQL 5.5. It doesn't fully support transactions but in some scenarios may be faster than [`InnoDB`](glossary.md#innodb). Each table is stored on disk in 3 files: [`.frm`](glossary.md#frm),i `.MYD`, `.MYI`.

## MySQL

An open source database that has spawned several distributions and forks. MySQL AB was the primary maintainer and distributor until bought by Sun Microsystems, which was then acquired by Oracle. As Oracle owns the MySQL trademark, the term MySQL is often used for the Oracle distribution of MySQL as distinct from the drop-in replacements such as [`MariaDB`](glossary.md#mariadb) and [`Percona Server`](glossary.md#percona-server).

## mysql.pxc.internal.session

This user is used by the SST process to run the SQL commands needed for [`SST`](glossary.md#sst), such as creating the `mysql.pxc.sst.user` and assigning it the role [`mysql.pxc.sst.role`](glossary.md#mysql-pxc-sst-role).

## mysql.pxc.sst.role

This role has all the privileges needed to run xtrabackup to create a backup on the donor node.

## mysql.pxc.sst.user

This user (set up on the donor node) is assigned the [`mysql.pxc.sst.role`](glossary.md#mysql-pxc-sst-role) and runs the XtraBackup to make backups. The password for this is randomly generated for each SST. The password is generated automatically for each [`SST`](glossary.md#sst).

## node

An individual server or computer within a cluster that stores and processes data.

## Non-production environment

A testing environment where new features or changes are tested before being deployed to the live production environment. It simulates the production environment but does not impact actual users or live data.

## Non-transactional

Operations that cannot be rolled back once executed.

## NUMA

Non-Uniform Memory Access ([`NUMA`](http://en.wikipedia.org/wiki/Non-Uniform_Memory_Access)) is a computer memory design used in multiprocessing, where the memory access time depends on the memory location relative to a processor. Under NUMA, a processor can access its own local memory faster than non-local memory, that is, memory local to another processor or memory shared between processors. The whole system may still operate as one unit, and all memory is basically accessible from everywhere, but at a potentially higher latency and lower performance.

## Percona Server for MySQL

Percona's branch of [`MySQL`](glossary.md#mysql) with performance and management improvements.

## Percona XtraDB Cluster

Percona XtraDB Cluster (PXC) is a high availability solution for MySQL.

## primary cluster

A cluster with [quorum](glossary.md#quorum). A non-primary cluster will not allow any
operations and will give `Unknown command` errors on any clients
attempting to read or write from the database.

## Production environment

The live environment where applications and databases are used by end-users, containing real data and running actual workloads.

## quorum

A majority (> 50%) of nodes. In the event of a network partition, only the cluster partition that retains a quorum (if any) will remain Primary by default.

## Rejoining the cluster

The process of reintegrating an isolated node back into the cluster after completing an update or maintenance task.

## Replica

A copy of a database node in a cluster that keeps the data synchronized with other nodes.

## Replication

The process of copying data from one database to another to ensure consistency and redundancy across multiple nodes.

## Rolling Schema Upgrade (RSU)

 A method for altering the structure of a database schema without bringing down the entire cluster. It updates one database at a time, reducing downtime but potentially causing temporary differences in database structures across the cluster.

## Schema change

Modifications to the database schema, such as adding, removing, or altering tables and columns.

## split brain

Split brain occurs when two parts of a computer cluster are disconnected, each part believing that the other is no longer running. This problem can lead to data inconsistency.

## State Snapshot Transfer (SST)

State Snapshot Transfer (SST) is a process that copies all data from one node to another in a database cluster. It happens when a new node joins the cluster and needs to get data from an existing node.

Percona XtraDB Cluster uses a program called `xtrabackup` for State Snapshot Transfer. This program has some advantages:

* It doesn't need to lock the entire database for reading during the sync process

* It only needs to lock the database briefly to sync MySQL system tables

* It also locks briefly to write information about the binlog, galera, and replica

These brief locks are similar to what happens during a regular Percona XtraBackup backup.

The SST process involves several steps:

1. The new node (joiner) contacts an existing node (donor) in the cluster

2. The donor node starts the `xtrabackup` program

3. `xtrabackup` creates a consistent copy of all the data

4. This data is transferred to the joiner node

5. The joiner node applies the received data

6. Once finished, the joiner node becomes an active part of the cluster

SST ensures that all nodes in the cluster have the same data. This is crucial for maintaining consistency across the database cluster.
  
## Storage Engine

A Storage Engine manages how data is stored and retrieved in a database system. It's a key component in the [MySQL](glossary.md#mysql) database ecosystem.

Storage Engines handle:

* Data storage on disk

* Data retrieval from disk

* Index management

* Transaction processing

* Caching mechanisms

MySQL introduced the pluggable storage engines, allowing users to choose different engines for different tables. This flexibility lets developers optimize their database performance based on specific needs.

Common MySQL Storage Engines include:

* [InnoDB](glossary.md#innodb): The default engine since MySQL 5.5

* [MyISAM](glossary.md#myisam): An older engine, still used in some scenarios

* Memory: Stores data in RAM for fast access

Each Storage Engine has its own strengths and weaknesses. For example:

* InnoDB supports transactions and foreign keys

* MyISAM is faster for read-heavy workloads but doesn't support transactions

* Memory is extremely fast, but data is lost when the server restarts

When creating a table in MySQL, you can specify which Storage Engine to use:

```sql
CREATE TABLE example_table (
    id INT PRIMARY KEY,
    name VARCHAR(50)
) ENGINE = InnoDB;
```


## Synchronization Point

A synchronization point in a database cluster represents a specific moment when all nodes in the cluster have reached the same state. This state includes:

* The same data content

* The same schema structure

* The same transaction history

Synchronization points serve several essential purposes in a cluster:

* Ensuring consistency across all nodes

* Facilitating node recovery after failures

* Enabling new nodes to join the cluster seamlessly

* Supporting backup operations

The replication system creates synchronization points automatically at regular intervals. These points are also known as Global Transaction IDs (GTIDs).

When a node reaches a synchronization point, the following occurs:

1. The node saves its current state

2. The node communicates this state to other nodes in the cluster

3. Other nodes confirm they have reached the same state

Synchronization points allow a cluster to maintain strong consistency. If a node fails or becomes disconnected, the cluster can use the most recent synchronization point to bring that node back into sync when the node rejoins the cluster.

## Synchronous cluster

A synchronous cluster in a Percona XtraDB Cluster database cluster is a group of servers that work together to maintain consistent data across all nodes. In this type of cluster:

* All nodes contain the same data at all times.

* When a write operation occurs on one node, the change is immediately replicated to all other nodes before the transaction is considered complete.

* Read operations can be performed on any node in the cluster, as they all have identical data.

* The cluster ensures strong consistency, meaning that any read operation will always return the most up-to-date data, regardless of which node processes the request.

* If a node fails or becomes unavailable, the cluster continues to operate with the remaining nodes, maintaining data integrity and availability.

* New nodes can be added to the cluster dynamically, and they will automatically synchronize with the existing nodes to obtain the current data set.

* The cluster uses a certification-based replication mechanism to ensure that conflicting writes are detected and resolved across all nodes.

Synchronous clusters provide high availability and data consistency but may have slightly higher latency for write operations than asynchronous replication methods. This trade-off is often acceptable for applications requiring strong data consistency and cannot tolerate data loss or inconsistencies between nodes.

## Tech preview 

A tech preview item can be a feature, a variable, or a value within a variable. The term designates that the item is not yet ready for production use and is not included in support by SLA. A tech preview item is included in a release so that users can provide feedback. The item is either updated and released as [general availability(GA)](#general-availability-ga) or removed if not useful. The item’s functionality can change from tech preview to GA.

## Total Order Isolation (TOI)

A method for performing schema changes, where the entire cluster is taken offline to apply the changes, resulting in complete downtime during the upgrade.

## Transaction stream

A sequence of transactions processed by a database cluster.

## Up-front locking

A mechanism to prevent other operations from modifying data while a specific operation is being executed, to avoid conflicts.

## UUID

Universally Unique IDentifier which uniquely identifies the state and the sequence of changes node undergoes. 128-bit UUID is a classic DCE UUID Version 1 (based on current time and MAC address). Although in theory this UUID could be generated based on the real MAC-address, in the Galera it is always (without exception) based on the generated pseudo-random addresses ("locally administered" bit in the node address (in the UUID structure) is  always equal to unity).
