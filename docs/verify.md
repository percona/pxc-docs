# Verifying Replication

Use the following procedure to verify replication
by creating a new database on the second node,
creating a table for that database on the third node,
and adding some records to the table on the first node.

* Create a new database on the second node:

  ```sql
  mysql@pxc2> CREATE DATABASE percona;
  ```
  
  The following output confirms that a new database has been created:

  ```text
  Query OK, 1 row affected (0.01 sec)
  ```

* Switch to a newly created database:

  ```sql
  mysql@pxc3> USE percona;
  ```

  The following output confirms that a database has been changed:

  ```text
  Database changed
  ```


* Create a table on the third node:

  ```sql
  mysql@pxc3> CREATE TABLE example (node_id INT PRIMARY KEY, node_name VARCHAR(30));
  ```

  The following output confirms that a table has been created:

  ```text
  Query OK, 0 rows affected (0.05 sec)
  ```

* Insert records on the first node:

  ```sql
  mysql@pxc1> INSERT INTO percona.example VALUES (1, 'percona1');
  ```

  The following output confirms that the records have been inserted:

  ```text
  Query OK, 1 row affected (0.02 sec)
  ```

* Retrieve rows from that table on the second node:

  ```sql
  mysql@pxc2> SELECT * FROM percona.example;
  ```

  The following output confirms that all the rows have been retrieved:

  ```text
  +---------+-----------+
  | node_id | node_name |
  +---------+-----------+
  |       1 | percona1  |
  +---------+-----------+
  1 row in set (0.00 sec)
  ```

## Next Steps

* Consider installing [ProxySQL](https://www.proxysql.com/) on client nodes
for efficient workload management across the cluster without any changes
to the applications that generate queries. This is the recommended high-availability solution for Percona XtraDB Cluster. For more information, see [Load balancing with ProxySQL](howtos/proxysql.md#load-balancing-with-proxysql).

* [Percona Monitoring and Management](https://www.percona.com/software/database-tools/percona-monitoring-and-management) is the best choice for managing and monitoring Percona XtraDB Cluster performance.
It provides visibility for the cluster and enables efficient troubleshooting.
