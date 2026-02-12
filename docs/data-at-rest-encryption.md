# Data at Rest Encryption

## Introduction

Data at rest encryption refers to encrypting data stored on a disk on a server. If an unauthorized user accesses the data files from the file system, encryption ensures the user cannot read the file contents. Percona Server allows you to enable, disable, and apply encryptions to the following objects:

* File-per-tablespace table

* Schema

* General tablespace

* System tablespace

* Temporary table

* Binary log files

* Redo log files

* Undo tablespaces

* Doublewrite buffer files

The transit data is defined as data that is transmitted to another node or client. Encrypted transit data uses an SSL connection.

Percona XtraDB Cluster {{vers}} supports all data at rest generally-available encryption features available from Percona Server for MySQL {{vers}}.

## Use the component_keyring_file

### Configuration

Percona XtraDB Cluster inherits the Percona Server for MySQL behavior to configure the `component_keyring_file`. The following example illustrates using the component. Review [Use the keyring vault component :octicons-link-external-16:](https://docs.percona.com/percona-server/{{vers}}/use-keyring-vault-component.html) for the latest information on keyring components.

!!! note

    The component_keyring_file component should not be used for regulatory compliance. 

Install the component using a manifest file and add the following options in the configuration file:

Create a manifest file named `mysqld.my` in the installation directory:

```{.text .no-copy}
{
 "read_local_manifest": false,
 "components": "file://component_keyring_file"
}
```

Add the following options to your configuration file:

```{.text .no-copy}
[mysqld]
component_keyring_file_data=<PATH>/keyring
```

The `SHOW COMPONENTS` statement checks if the component has been successfully loaded:

```sql
SHOW COMPONENTS;
```

??? example "Expected output"

    ```{.text .no-copy}
    +----------------------------------------+
    | Component_id                           |
    +----------------------------------------+
    | file://component_keyring_file          |
    +----------------------------------------+
    ```

!!! note

    PXC recommends the same configuration on all cluster nodes, and all nodes should have the keyring configured. A mismatch in the keyring configuration does not allow the JOINER node to join the cluster.

If the user has a bootstrapped node with keyring enabled, then upcoming cluster nodes inherit the keyring (the encrypted key) from the DONOR node.

#### Usage

XtraBackup re-encrypts the data using a transition-key and the JOINER node re-encrypts the data using a newly generated master-key.

Keyring (or, more generally, the Percona XtraDB Cluster SST process) is backward compatible, as in higher-version JOINER can join from lower-version DONOR, but the reverse is not supported.

Percona XtraDB Cluster does not allow the combination of nodes with encryption and nodes without encryption to maintain data consistency. For example, the user creates node-1 with encryption (keyring) enabled and node-2 with encryption (keyring) disabled. If the user attempts to create a table with encryption on node-1, the creation fails on node-2, causing data inconsistency. A node fails to start if the node fails to load the keyring component.

!!! note

    If the user does not specify the keyring parameters, the node does not know that the node must load the keyring. The JOINER node may start, but the node eventually shuts down when the DML level inconsistency with encrypted tablespace is detected.

If a node does not have an encrypted tablespace, the keyring file exists but is empty. Creating an encrypted table on the node populates the keyring file with the necessary encryption keys.

In an operation that is local to the node, you can rotate the key as needed. The `ALTER INSTANCE ROTATE INNODB MASTER KEY` statement is not replicated on cluster.

The JOINER node generates its keyring.

### Compatibility

The Percona XtraDB Cluster SST process with keyring support is backward compatible. A higher-version JOINER can join from a lower-version DONOR, but the reverse is not supported.

## Configure PXC to use component_keyring_vault component

### component_keyring_vault

The `component_keyring_vault` stores the master encryption key in a HashiCorp Vault server instead of a local file like `component_keyring_file`.

### Configuration

Configuration options are the same as [Percona Server for MySQL :octicons-link-external-16:](https://docs.percona.com/percona-server/{{vers}}/use-keyring-vault-component.html).

Create a manifest file named `mysqld.my` in the installation directory:

```{.text .no-copy}
{
 "read_local_manifest": false,
 "components": "file://component_keyring_vault"
}
```

Create a configuration file `component_keyring_vault.cnf` in JSON format:

```{.text .no-copy}
{
 "timeout": 15,
 "vault_url": "https://vault.public.com:8202",
 "secret_mount_point": "secret",
 "secret_mount_point_version": "AUTO",
 "token": "{randomly-generated-alphanumeric-string}",
 "vault_ca": "/data/keyring_vault_confs/vault_ca.crt"
}
```

The `secret_mount_point_version` parameter defaults to `AUTO` and controls whether the Vault KV Secrets Engine is version 1 (kv) or version 2 (kv-v2). Using the wrong KV version can cause silent failures during keyring operations.

!!! warning

    Token Security: Avoid embedding long-lived tokens directly in configuration files. Consider using [Vault's AppRole authentication :octicons-link-external-16:](https://developer.hashicorp.com/vault/docs/auth/approle) or dynamic token retrieval mechanisms for enhanced security.

The detailed description of these options can be found in the [Percona Server for MySQL keyring vault component documentation :octicons-link-external-16:](https://docs.percona.com/percona-server/{{vers}}/use-keyring-vault-component.html).

Vault-server is an external server, so make sure the PXC node can reach the server.

!!! warning

    SST Limitation: The rsync tool does not support the `component_keyring_vault`. Any rsync-SST on a joiner is aborted if the `component_keyring_vault` is configured.

Uniform Component Configuration: Percona XtraDB Cluster strongly recommends using the same keyring component type on all cluster nodes. Mixing keyring component types is only recommended during controlled transitions from `component_keyring_file` to `component_keyring_vault` or the reverse. Inconsistent keyring configurations can lead to data inconsistency and cluster instability.

All nodes do not need to refer to the same vault server. Whatever vault server is used, the server must be accessible from the respective node. All nodes do not need to use the same mount point.

If the node is not able to reach or connect to the vault server, an error is notified during the server restart, and the node refuses to start:

??? example "The warning message"

    ```{.text .no-copy}
    2018-05-29T03:54:33.859613Z 0 [Warning] Component component_keyring_vault reported:
    'There is no vault_ca specified in component_keyring_vault's configuration file.
    Please make sure that Vault's CA certificate is trusted by the machine
    from which you intend to connect to Vault.'
    2018-05-29T03:54:33.977145Z 0 [ERROR] Component component_keyring_vault reported:
    'CURL returned this error code: 7 with error message : Failed to connect
    to 127.0.0.1 port 8200: Connection refused'
    ```

When vault server connectivity issues occur, only the affected nodes fail to start. For example, if node-1 can connect to the vault server but node-2 cannot, only node-2 will refuse to start.

If a server has encrypted objects but cannot connect to the vault server during restart, those encrypted objects become inaccessible.

When the vault server is reachable but authentication credentials are incorrect, the same behavior occurs:

??? example "The warning message"

    ```{.text .no-copy}
    2018-05-29T03:58:54.461911Z 0 [Warning] Component component_keyring_vault reported:
    'There is no vault_ca specified in component_keyring_vault's configuration file.
    Please make sure that Vault's CA certificate is trusted by the machine
    from which you intend to connect to Vault.'
    2018-05-29T03:58:54.577477Z 0 [ERROR] Component component_keyring_vault reported:
    'Could not retrieve list of keys from Vault. Vault has returned the
    following error(s): ["permission denied"]'
    ```

In case of an accessible vault-server with the wrong mount point, there is no error during server restart, but the node still refuses to start:

```sql
CREATE TABLE t1 (c1 INT, PRIMARY KEY pk(c1)) ENCRYPTION='Y';
```

??? example "Expected output"

    ```{.text .no-copy}
    ERROR 3185 (HY000): Can't find master key from keyring, please check keyring
    component is loaded.
    ... [ERROR] Component component_keyring_vault reported: 'Could not write key to Vault. ...
    ... [ERROR] Component component_keyring_vault reported: 'Could not flush keys to keyring'
    ```

## Mix keyring component types

With XtraBackup introducing transition-key logic, you can now mix and match keyring components. For example, node-1 can be configured to use the `component_keyring_file` component while node-2 uses `component_keyring_vault`.

!!! warning

    Percona strongly recommends the same keyring component configuration for all cluster nodes. Mixing keyring component types is only recommended during controlled transitions from one keyring type to another. Inconsistent configurations can cause data corruption and cluster failures.

## Migrate keys between keyring keystores

Percona XtraDB Cluster supports key migration between keystores. The migration can be performed offline or online using a migration server with specific configuration options.

### Offline migration

In offline migration, the node to migrate is shut down, and the migration server takes care of migrating keys for the said server to a new keystore.

For example, a cluster has three Percona XtraDB Cluster nodes, n1, n2, and n3. The nodes use the `component_keyring_file`. To migrate the n2 node to use `component_keyring_vault`, use the following procedure:

1. Shut down the n2 node.

2. Start the Migration Server (`mysqld` with a special option).

3. The Migration Server copies the keys from the n2 keyring file and adds them to the vault server.

4. Start the n2 node with the vault parameter, and the keys are available.

Run the migration server:

```shell
/dev/shm/pxc80/bin/mysqld --defaults-file=/dev/shm/pxc80/copy_mig.cnf \
--keyring-migration-source=component_keyring_file \
--component_keyring_file_data=/dev/shm/pxc80/node2/keyring \
--keyring-migration-destination=component_keyring_vault \
--component_keyring_vault_config=/dev/shm/pxc80/vault/component_keyring_vault.cnf &
```

??? example "Expected output"

    ```{.text .no-copy}
    ... [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use
        --explicit_defaults_for_timestamp server option (see documentation for more details).
    ... [Note] --secure-file-priv is set to NULL. Operations related to importing and
        exporting data are disabled
    ... [Warning] WSREP: Node is not a cluster node. Disabling pxc_strict_mode
    ... [Note] /dev/shm/pxc84/bin/mysqld (mysqld 8.4-debug) starting as process 5710 ...
    ... [Note] Keyring migration successful.
    ```

On a successful migration, the destination keystore receives additional migrated keys (pre-existing keys in the destination keystore are not touched or removed). The source keystore retains the keys as the migration performs a copy operation and not a move operation.

If the migration fails, the destination keystore is unchanged.

### Online migration

In online migration, the node to migrate is kept running, and the migration server takes care of migrating keys for the said server to a new keystore by connecting to the node.

For example, a cluster has three Percona XtraDB Cluster nodes, n1, n2, and n3. The nodes use the `component_keyring_file`. Migrate the n3 node to use `component_keyring_vault` using the following procedure:

1. Start the Migration Server (`mysqld` with a special option).

2. The Migration Server copies the keys from the n3 keyring file and adds them to the vault server.

3. Restart the n3 node with the vault parameter, and the keys are available.

```{.text .no-copy}
/dev/shm/pxc80/bin/mysqld --defaults-file=/dev/shm/pxc80/copy_mig.cnf \
--keyring-migration-source=component_keyring_vault \
--component_keyring_vault_config=/dev/shm/pxc80/component_keyring_vault3.cnf \
--keyring-migration-destination=component_keyring_file \
--component_keyring_file_data=/dev/shm/pxc80/node3/keyring \
--keyring-migration-host=localhost \
--keyring-migration-user=root \
--keyring-migration-port=16300 \
--keyring-migration-password='' &
```

On a successful migration, the destination keystore receives the additional migrated keys. Any pre-existing keys in the destination keystore are unchanged. The source keystore retains the keys as the migration performs a copy operation and not a move operation.

If the migration fails, the destination keystore is not changed.

### Migration server options

* `--keyring-migration-source`: The source keyring component that manages the keys to be migrated.

* `--keyring-migration-destination`: The destination keyring component to which the migrated keys are to be copied

    !!! note

        For offline migration, no additional key migration options are needed.

* `--keyring-migration-host`: The host where the running server is located. This host is always the local host.

* `--keyring-migration-user`, `--keyring-migration-password`: The username and password for the account used to connect to the running server.

* `--keyring-migration-port`: Used for TCP/IP connections, the running server’s port number used to connect.

* `--keyring-migration-socket`: Used for Unix socket file or Windows named pipe connections, the running server socket or named pipe used to connect.

Prerequisite for migration:

Make sure to pass required keyring options and other configuration parameters for the two keyring components. For example, if `component_keyring_file` is one of the components, you must explicitly configure the `component_keyring_file_data` system variable in the my.cnf file.

Other non-keyring options may be required as well. One way to specify these options is by using `--defaults-file` to name an option file that contains the required options.

```{.text .no-copy}
[mysqld]
basedir=/dev/shm/pxc80
datadir=/dev/shm/pxc80/copy_mig
log-error=/dev/shm/pxc80/logs/copy_mig.err
socket=/tmp/copy_mig.sock
port=16400
```

!!! admonition "See also"

    [Encrypt traffic documentation](encrypt-traffic.md)

    [Percona Server for MySQL Documentation: Data-at-Rest Encryption :octicons-link-external-16:](https://www.percona.com/doc/percona-server/{{vers}}/security/data-at-rest-encryption.html#data-at-rest-encryption)
