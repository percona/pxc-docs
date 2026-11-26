> For the complete documentation index, see [llms.txt](https://docs.percona.com/percona-xtradb-cluster/9.7/llms.txt).

# Verify replication

Use the following procedure to verify replication by creating a new database on the second node, creating a table for that database on the third node, and adding some records to the table on the first node.

1. Create a new database on the second node:

   ```sql
   CREATE DATABASE percona;
   ```

   The following output confirms that a new database has been created:

   Expected output

   ```text
   Query OK, 1 row affected (0.01 sec)
   ```

1. Switch to a newly created database:

   ```sql
   USE percona;
   ```

   The following output confirms that a database has been changed:

   Expected output

   ```text
   Database changed
   ```

1. Create a table on the third node:

   ```sql
   CREATE TABLE example (node_id INT PRIMARY KEY, node_name VARCHAR(30));
   ```

   The following output confirms that a table has been created:

   Expected output

   ```text
   Query OK, 0 rows affected (0.05 sec)
   ```

1. Insert records on the first node:

   ```sql
   INSERT INTO percona.example VALUES (1, 'percona1');
   ```

   The following output confirms that the records have been inserted:

   Expected output

   ```text
   Query OK, 1 row affected (0.02 sec)
   ```

1. Retrieve rows from that table on the second node:

   ```sql
   SELECT * FROM percona.example;
   ```

   The following output confirms that all the rows have been retrieved:

   Expected output

   ```text
   +---------+-----------+
   | node_id | node_name |
   +---------+-----------+
   |       1 | percona1  |
   +---------+-----------+
   1 row in set (0.00 sec)
   ```

## Next steps

- Consider installing [ProxySQL](https://www.proxysql.com/) on client nodes for efficient workload management across the cluster without any changes to the applications that generate queries. This is the recommended high-availability solution for Percona XtraDB Cluster. For more information, see [Load balancing with ProxySQL](https://docs.percona.com/percona-xtradb-cluster/9.7/load-balance-proxysql.md#load-balance-with-proxysql).
- [Percona Monitoring and Management](https://www.percona.com/monitoring/) is the best choice for managing and monitoring Percona XtraDB Cluster performance. It provides visibility for the cluster and enables efficient troubleshooting.
