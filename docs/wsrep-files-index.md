# Index of files created by PXC

- `GRA_\*.log`
    These files contain binlog events in ROW format representing the failed
    transaction. That means that the replica thread was not able to apply one of
    the transactions. For each of those file, a corresponding warning or error
    message is present in the mysql error log file. Those error can also be
    false positives like a bad `DDL` statement (dropping  a table that doesn’t
    exists for example) and therefore nothing to worry about. However it’s
    always recommended to check these log to understand what’s is happening.
    To be able to analyze these files binlog header needs to be added to the log
    file. To create the `GRA_HEADER` file you need an instance running with `binlog_checksum` set to `NONE` and extract first 120 bytes from the binlog file:
    This information can be used for checking the MySQL error log for the corresponding error message.
    ??? example "Error message"
  ```
    ```text
    160805  9:33:37 8:52:21 [ERROR] Slave SQL: Error 'Unknown table 'test'' on query. Default database: 'test'. Query: 'drop table test', Error_code: 1051
    160805  9:33:37 8:52:21 [Warning] WSREP: RBR event 1 Query apply warning: 1, 3
    ```
  ```
    In this example `DROP TABLE` statement was executed on a table that doesn’t exist.
- `gcache.page`
    See `[gcache.page_size](wsrep-provider-index.md#gcachepage_size)`
    !!! admonition "See also"
  ```
    [Percona Database Performance Blog: All You Need to Know About GCache (Galera-Cache) :octicons-link-external-16:](https://www.percona.com/blog/2016/11/16/all-you-need-to-know-about-gcache-galera-cache/)
  ```
- `galera.cache`
    This file is used as a main writeset store. It’s implemented as a permanent
    ring-buffer file that is preallocated on disk when the node is initialized.
    File size can be controlled with the variable `[gcache.size](wsrep-provider-index.md#gcachesize)`. If this value is bigger, more writesets are cached and chances are better that the re-joining node will get [IST](glossary.md#ist) instead of [SST](glossary.md#sst). Filename can be changed with the `[gcache.name](wsrep-provider-index.md#gcachename)` variable.

### `## grastate.dat`

This file contains the Galera state information.

- `version` - grastate version
- `uuid` - a unique identifier for the state and the sequence of changes it undergoes.For more information on how UUID is generated see [UUID](glossary.md#uuid).
- `seqno` - Ordinal Sequence Number, a 64-bit signed integer used to denote the position of the change in the sequence. `seqno` is `0` when no writesets have been generated or applied on that node, i.e., not applied/generated across the lifetime of a `grastate` file. `-1` is a special value for the `seqno` that is kept in the `grastate.dat` while the server is running to allow Galera to distinguish between a clean and an unclean shutdown. Upon a clean shutdown, the correct `seqno` value is written to the file. So, when the server is brought back up, if the value is still `-1` , this means that the server did not shut down cleanly. If the value is greater than `0`, this means that the shutdown was clean. `-1` is then written again to the file in order to allow the server to correctly detect if the next shutdown was clean in the same manner.
- `cert_index` - cert index restore through grastate is not implemented yet

Examples of this file look like this:

In case server node has this state when not running it means that that node crashed during the transaction processing.

```shell
GALERA saved state
version: 2.1
uuid:    1917033b-7081-11e2-0800-707f5d3b106b
seqno:   -1
cert_index:
```

In case server node has this state when not running it means that the node
was gracefully shut down.

```shell
GALERA saved state
version: 2.1
uuid:    1917033b-7081-11e2-0800-707f5d3b106b
seqno:   5192193423942
cert_index:
```

In case server node has this state when not running it means that the node crashed during the DDL. The same zero UUID and `seqno` of `-1` also appear after the cluster votes the node out for a data inconsistency: Galera marks the state corrupt so you can investigate while the evicted node is still running.

```shell
GALERA saved state
version: 2.1
uuid:    00000000-0000-0000-0000-000000000000
seqno:   -1
cert_index:
```

A corrupt `grastate.dat` does not force a full [SST](glossary.md#sst) on the next start. Restart recovery reads a usable position from InnoDB and the node can rejoin with [IST](glossary.md#ist), which leaves the inconsistent dataset in place. To force SST after an inconsistency eviction, enable `[repl.force_sst_after_inconsistency](wsrep-provider-index.md#replforce_sst_after_inconsistency)` (the default is `no`, which leaves the file in place for inspection) or remove `grastate.dat` manually before restart. When that option is `yes`, the node deletes the file when it marks the state corrupt; copy the file first if you still need it.

- `gvwstate.dat`
   This file is used for Primary Component recovery feature. This file is
   created once primary component is formed or changed, so you can get the
   latest primary component this node was in. And this file is deleted when the
   node is shutdown gracefully.
   First part contains the node [UUID](glossary.md#uuid) information. Second part contains
   the view information. View information is written between `#vwbeg` and
   `#vwend`. View information consists of:
  - view_id: [view_type] [view_uuid] [view_seq]. - `view_type` is always `3` which means primary view. `view_uuid` and `view_seq` identifies a unique view, which could be perceived as identifier of this primary component.
  - bootstrap: [bootstarp_or_not]. - it could be `0` or `1`, but it does not affect primary component recovery process now.
  - member: [node’s uuid] [node’s segment]. - it represents all nodes in this primary component.
    ??? example "Example of the file"
  ```
    ```text
    my_uuid: c5d5d990-30ee-11e4-aab1-46d0ed84b408
    #vwbeg
    view_id: 3 bc85bd53-31ac-11e4-9895-1f2ce13f2542 2 
    bootstrap: 0
    member: bc85bd53-31ac-11e4-9895-1f2ce13f2542 0
    member: c5d5d990-30ee-11e4-aab1-46d0ed84b408 0
    #vwend
    ```
  ```

