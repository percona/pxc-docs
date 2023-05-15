# Non-Blocking Operations (NBO) method for Online Scheme Upgrades (OSU)

An [Online Schema Upgrade](online-schema-upgrade.md#online-schema-upgrade) can be a daily issue in an environment with accelerated development and deployment. The task becomes more difficult as the data grows. An `ALTER TABLE` statement is a multi-step operation and must run until it is complete. Aborting the statement may be more expensive than letting it complete.

The Non-Blocking Operations (NBO) method is similar to the `TOI` method (see [Online Schema Upgrade](online-schema-upgrade.md#online-schema-upgrade) for more information on the available types of online schema upgrades). Every replica processes the DDL statement at the same point in the cluster transaction stream, and other transactions cannot commit during the operation. The `NBO` method provides a more efficient locking strategy and avoids the `TOI` issue of long-running DDL statements blocking cluster updates.

In the NBO method, the supported DDL statement acquires a metadata lock on the table or schema at a late stage of the operation. The [`lock_wait_timeout`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_lock_wait_timeout) system variable defines the timeout, measured in seconds, to acquire metadata locks. The default value, 3153600, could cause `infinite` waits and should not be used with the `NBO` method.

Attempting a State Snapshot Transfer (SST) fails during the NBO operation.

To dynamically set the `NBO` mode in the client, run the following statement:

```sql
SET SESSION wsrep_OSU_method='NBO';
```

## Supported DDL statements

The NBO method supports the following DDL statements:

* `ALTER TABLE`

* `ALTER INDEX`

* `CREATE INDEX`

* `DROP INDEX`

## Limitations

The `NBO` method does not support the following:

* Running two DDL statements with conflicting locks on the same table. For example, you cannot run two `ALTER TABLE` statements for an `employees` table.

* Modifying a table changed during the NBO operation. However, you can modify other tables and execute NBO queries on other tables.

See the [Percona XtraDB Cluster 8.0.25-15.1](../release-notes/Percona-XtraDB-Cluster-8.0.25-15.1.md#pxc-8-0-25-15-1) Release notes for the latest information.
