# Index of files created by Percona XtraDB Cluster

Percona XtraDB Cluster can create the following files:

### `GRA_*.log`

`GRA_*.log` files contain row-based binary log events for a transaction that failed to apply.

The replica thread does not apply that transaction.

Each file has a matching warning or error in the MySQL error log.

Some errors are harmless. One example is a Data Definition Language (DDL) statement that drops a table that does not exist.

Check these logs to determine what failed.

To decode a GRA log file, add a binary log header. Use an instance with `binlog_checksum` set to `NONE`. Extract the first 123 bytes from a binary log file.

The following commands create `GRA_HEADER`, combine the files, and decode the result:

```bash
head -c 123 mysqld-bin.000001 > GRA_HEADER
cat GRA_HEADER > /var/lib/mysql/GRA_1_2-bin.log
cat /var/lib/mysql/GRA_1_2.log >> /var/lib/mysql/GRA_1_2-bin.log
mysqlbinlog -vvv /var/lib/mysql/GRA_1_2-bin.log
```

??? example "mysqlbinlog output"

    ```{.text .no-copy}
    /*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
    /*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
    DELIMITER /*!*/;
    # at 4
    #160809  16:04:05 server id 3  end_log_pos 123     Start: binlog v 4, server v 8.0-log created 160809 16:04:05 at startup
    # Warning: this binlog is either in use or was not closed properly.
    ROLLBACK/*!*/;
    BINLOG '
    nbGpVw8DAAAAdwAAAHsAAAABAAQANS43LjEyLTVyYzEtbG9nAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    AAAAAAAAAAAAAAAAAACdsalXEzgNAAgAEgAEBAQEEgAAXwAEGggAAAAICAgCAAAACgoKKioAEjQA
    ALfQ8hw=
    '/*!*/;
    # at 123
    #160809  16:05:49 server id 2  end_log_pos 75     Query    thread_id=11    exec_time=0    error_code=0
    use `test`/*!*/;
    SET TIMESTAMP=1470738949/*!*/;
    SET @@session.pseudo_thread_id=11/*!*/;
    SET @@session.foreign_key_checks=1, @@session.sql_auto_is_null=0, @@session.unique_checks=1, @@session.autocommit=1/*!*/;
    SET @@session.sql_mode=1436549152/*!*/;
    SET @@session.auto_increment_increment=1, @@session.auto_increment_offset=1/*!*/;
    /*!\C utf8 *//*!*/;
    SET @@session.character_set_client=33,@@session.collation_connection=33,@@session.collation_server=8/*!*/;
    SET @@session.lc_time_names=0/*!*/;
    SET @@session.collation_database=DEFAULT/*!*/;
    drop table t
    /*!*/;
    SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
    DELIMITER ;
    # End of log file
    /*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
    /*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;
    ```

Use the decoded statement and timestamp to find the matching error in the MySQL error log.

??? example "Error message"

    ```{.text .no-copy}
    160805  9:33:37 8:52:21 [ERROR] Slave SQL: Error 'Unknown table 'test'' on query. Default database: 'test'. Query: 'drop table test', Error_code: 1051
    160805  9:33:37 8:52:21 [Warning] WSREP: RBR event 1 Query apply warning: 1, 3
    ```

In this example, a `DROP TABLE` statement ran against a table that does not exist.

### `gcache.page`

See [`gcache.page_size`](wsrep-provider-index.md#gcachepage_size).

!!! admonition "See also"

    [Percona Database Performance Blog: All You Need to Know About GCache (Galera-Cache) :octicons-link-external-16:](https://www.percona.com/blog/2016/11/16/all-you-need-to-know-about-gcache-galera-cache/)

### `galera.cache`

`galera.cache` is the main store for write sets.

The file is a permanent ring buffer. The node allocates the file on disk at initialization.

The [`gcache.size`](wsrep-provider-index.md#gcachesize) variable sets the file size. A larger file caches more write sets. A joining node is more likely to receive [Incremental State Transfer (IST)](glossary.md#ist) instead of [State Snapshot Transfer (SST)](glossary.md#sst).

The [`gcache.name`](wsrep-provider-index.md#gcachename) variable sets the file name.

### `grastate.dat`

`grastate.dat` stores Galera state information.

The file has the following fields:

* `version`: Format version of `grastate.dat`

* `uuid`: Unique identifier for the state and the change sequence. See [UUID](glossary.md#uuid).

* `seqno`: Sequence number of the last change. The value is a 64-bit signed integer.

* `cert_index`: Unused. Certification index restore is not implemented.

`seqno` uses the following values:

* `0`: No write sets were generated or applied on that node for the life of the file

* `-1`: The server is running, or the last shutdown was not clean

* A value greater than `0`: The last shutdown was clean

While the server runs, Galera writes `-1` to `grastate.dat`. A clean shutdown writes the correct `seqno` value.

At the next start, `-1` means the previous shutdown was not clean. A value greater than `0` means the previous shutdown was clean. The server then writes `-1` again. The next shutdown uses the same detection.

The following examples show `grastate.dat` in three situations.

The node is not running. This state means the node crashed during a transaction:

```{.text .no-copy}
GALERA saved state
version: 2.1
uuid:    1917033b-7081-11e2-0800-707f5d3b106b
seqno:   -1
cert_index:
```

The node is not running. This state means the node shut down cleanly:

```{.text .no-copy}
GALERA saved state
version: 2.1
uuid:    1917033b-7081-11e2-0800-707f5d3b106b
seqno:   5192193423942
cert_index:
```

The node is not running. This state means the node crashed during a DDL statement:

```{.text .no-copy}
GALERA saved state
version: 2.1
uuid:    00000000-0000-0000-0000-000000000000
seqno:   -1
cert_index:
```

The same zero UUID and `seqno` of `-1` also appear after the cluster votes the node out for inconsistent data.

Galera marks the state as corrupt. You can inspect the node while the node still runs.

A corrupt `grastate.dat` file does not force a full SST at the next start. Restart recovery reads a usable position from InnoDB. The node can rejoin with IST. The local data stays inconsistent.

To force SST after the cluster votes the node out for inconsistent data, use one of the following methods:

* Enable [`repl.force_sst_after_inconsistency`](wsrep-provider-index.md#replforce_sst_after_inconsistency). The default is `no`. The default keeps the file for inspection.

* Remove `grastate.dat` before you restart the node.

If the option is `yes`, the node deletes the file after Galera marks the state as corrupt. Copy the file first if you still need the file.

### `gvwstate.dat`

The Primary Component is the subset of nodes that holds [quorum](glossary.md#quorum). Only that subset accepts writes.

Primary Component recovery restores that subset after a full cluster outage. Galera uses `gvwstate.dat` for this process when [`pc.recovery`](wsrep-provider-index.md#pcrecovery) is `true` (the default). After every node from the last saved Primary Component can communicate again, the cluster restores the Primary Component. A manual bootstrap is not required.

A planned full-cluster restart still requires an explicit bootstrap.

Galera creates or updates `gvwstate.dat` when the Primary Component forms or changes. The file records the latest Primary Component for the node. A clean shutdown deletes the file.

The first part stores the UUID of the node. The second part stores the view. The view is written between `#vwbeg` and `#vwend`.

The view record has the following fields:

* `view_id`: `[view_type] [view_uuid] [view_seq]`. `view_type` is always `3`. That value means a primary view. `view_uuid` and `view_seq` identify the view.

* `bootstrap`: `0` or `1`. This value does not change Primary Component recovery.

* `member`: UUID and segment of each node in the Primary Component.

??? example "Example gvwstate.dat file"

    ```{.text .no-copy}
    my_uuid: c5d5d990-30ee-11e4-aab1-46d0ed84b408
    #vwbeg
    view_id: 3 bc85bd53-31ac-11e4-9895-1f2ce13f2542 2 
    bootstrap: 0
    member: bc85bd53-31ac-11e4-9895-1f2ce13f2542 0
    member: c5d5d990-30ee-11e4-aab1-46d0ed84b408 0
    #vwend
    ```
