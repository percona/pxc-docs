# Binary log contents across replication modes

## Overview

This document provides a concise reference for the events recorded in the MySQL and Percona Server binary log (binlog) across different replication configurations and modes.

It covers:

- Asynchronous replication (GTID and non-GTID)
- Semi-synchronous replication
- Group replication
- Binlog format differences (ROW, STATEMENT, MIXED)

This document is intended for users familiar with MySQL replication concepts who need a quick reference for binlog contents across configurations.

!!! note
    Examples are simplified excerpts of `mysqlbinlog` output for clarity.

## Asynchronous replication

### Non-GTID

#### Configuration

```sql
gtid_mode=OFF
log_bin=ON
```

#### Binlog contents

* Event header (timestamp, server-id, event-type, size)

* Statement-based (SBR) or row-based (RBR) events

* No GTID event

* Transaction boundaries:
  
  - BEGIN / COMMIT (statement-based)

  - Xid event (row-based)

#### Example

```text
# at 4
#210101 12:00:00 server id 1  end_log_pos 123 CRC32 0xabcdef12  Query ...
SET TIMESTAMP=1622505600;
BEGIN
# at 123
#210101 12:00:00 server id 1  end_log_pos 456 CRC32 0x1234abcd  Table_map ...
# at 456
#210101 12:00:00 server id 1  end_log_pos 789 CRC32 0xdeadbeef  Write_rows ...
# at 789
#210101 12:00:01 server id 1  end_log_pos 1020 CRC32 0xf00dba11  Xid COMMIT
```

#### Key notes

* In RBR, the Xid event is the true transaction delimiter

* BEGIN may not always appear in row-based logging

### GTID

#### Configuration

```sql
gtid_mode=ON
enforce_gtid_consistency=ON
log_bin=ON
```

#### Binlog contents

* All events present in non-GTID mode

* A `GTID` event precedes each transaction (`<server_uuid>:<sequence_number>`)

* Transaction commit still marked by `Xid` or `COMMIT`

* Replicas track progress using executed GTIDs instead of log positions

* In some configurations (for example, GTID transitions), `ANONYMOUS_GTID` events may appear

#### Example

```text
# at 4
#210101 12:00:00 server id 1  end_log_pos 123 CRC32 0xabcdef12  GTID last_committed=0 sequence_number=1
# gtid: 3E11FA47-71CA-11E1-9E33-C80AA9429562:1
# at 123
#210101 12:00:00 server id 1  end_log_pos 456 CRC32 0x1234abcd  Query ...
SET TIMESTAMP=1622505600;
BEGIN
# ... row events ...
# at 789
#210101 12:00:01 server id 1  end_log_pos 1020 CRC32 0xf00dba11  Xid COMMIT
```

#### Key notes

* `last_committed` enables dependency tracking for parallel replication

* The `GTID` identifies the transaction, but `Xid` still marks commit boundaries

## Semi-synchronous replication

Semi-synchronous replication does not modify binlog contents.

* Binlog format is identical to asynchronous replication (GTID or non-GTID)

* Only commit behavior changes:

  - A transaction is acknowledged as committed after at least one replica confirms receipt of the binlog event (not necessarily that it has been applied)

## Group replication

### Configuration

* GTID must be enabled

### Binlog contents

* GTID event (standard format)

* Row-based events (Group Replication requires `binlog_format=ROW`)

* Writeset-related metadata (used for conflict detection and certification)

* `Xid` event marking transaction commit

### Example

```text
# at 4
#210101 12:00:00 server id 1  end_log_pos 123 CRC32 0xabcdef12  GTID ...
# gtid: 3E11FA47-71CA-11E1-9E33-C80AA9429562:1
# at 123
# ... row events ...
# at 789
#210101 12:00:01 server id 1  end_log_pos 1020 CRC32 0xf00dba11  <writeset metadata>
#210101 12:00:01 server id 1  end_log_pos 1050 CRC32 0xdeadbeef  Xid COMMIT
```

### Key notes

* GTID format is the same as standard GTID replication

* Writeset metadata enables conflict detection in multi-primary setups

* Additional metadata events may appear depending on version and configuration

## Binlog format differences

### Row

#### Binlog contents

* Table_map

* Write_rows, Update_rows, Delete_rows

* Xid event for commit

#### Key notes

* DML is logged as row events; some statements (such as DDL) are still logged as Query events

* BEGIN may not always appear

* Provides deterministic replication

### STATEMENT

#### Binlog contents

* Query events containing SQL statements

* BEGIN / COMMIT markers

#### Key notes

* Compact, but may be non-deterministic in some cases

### MIXED

#### Binlog contents

* MySQL dynamically chooses between statement-based and row-based logging

* Mix of:

  - Query events (DDL, safe statements)

  - Row events (non-deterministic DML)

#### Example

```text
# Query event (DDL)
CREATE TABLE t1 (id INT PRIMARY KEY);

# Row events (DML)
Table_map ...
Write_rows ...
```

#### Key notes

* DDL typically uses statement-based logging

* Row events are used when statement-based replication is unsafe

## Summary: non-GTID vs GTID

| Aspect                 | Non-GTID                       | GTID                           |
|------------------------|--------------------------------|--------------------------------|
| Transaction identifier | Position-based (`BEGIN`/`Xid`) | Explicit GTID `<UUID>:<seq>`   |
| Replica tracking       | `source_log_file` + position   | `Executed_Gtid_Set`            |
| Binlog structure       | No GTID event                  | GTID event associated with each transaction |
| Commit marker          | `Xid` / `COMMIT`               | `Xid` / `COMMIT`               |
| Group replication      | Not supported                  | Required (uses writeset metadata) |

## Minimal reproducible examples

### Non-GTID server

!!! note
    
    Some settings (such as `log_bin`) are typically configured at server startup and may not be dynamically changeable.

```sql
-- Enable binary logging and set format
SET GLOBAL log_bin = ON;
SET GLOBAL binlog_format = ROW;

-- Generate activity
USE test1;
INSERT INTO Persons1 (id, name) VALUES (1, 'Alice');

-- Inspect binlog
SHOW BINLOG EVENTS IN 'mysql-bin.000001' FROM 4;
```

??? example "Expected output (simplified)"

    ```text
    Table_map → Write_rows → Xid
    ```

## GTID-enabled server

!!! note
    
    Some settings (such as `log_bin`) are typically configured at server startup and may not be dynamically changeable.

```sql
-- Enable GTID (simplified example)
SET GLOBAL gtid_mode = ON;
SET GLOBAL enforce_gtid_consistency = ON;
SET GLOBAL log_bin = ON;
SET GLOBAL binlog_format = ROW;

-- Generate activity
INSERT INTO Persons2 (id, name) VALUES (2, 'Bob');

-- Inspect binlog
SHOW BINLOG EVENTS;
```

!!! note

    Enabling GTID on an existing system requires a controlled transition process and should not be done directly in production.

??? example "Expected output (simplified)"

    ```text
    GTID → BEGIN → row events → Xid
    ```
