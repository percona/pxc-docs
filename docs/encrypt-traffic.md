# Encrypt PXC traffic

Percona XtraDB Cluster (PXC) encrypts two types of traffic:

* Client-server traffic: queries and results between client applications and cluster nodes.

* Replication traffic: data exchanged between nodes during State Snapshot Transfer (SST), Incremental State Transfer (IST), write-set replication, and service messages.

Encryption prevents unauthorized access to both types of traffic. You can configure encryption automatically or manually.

!!! important "Use identical certificates on every node"

    Every node must use the same key and certificate files. If certificates differ between nodes, encrypted connections between them fail. The node cannot join or stay in the cluster.

    Generate certificates one time, typically on the first node. Copy the certificates to every other node over a secure channel.

    This requirement applies to every encryption type on this page.

## File and path reference

The following table lists the certificate and key files that PXC uses. All examples on this page store these files in `/etc/mysql/certs/` and reference them with absolute paths.

| File | Purpose |
| --- | --- |
| `ca.pem` | Certificate Authority (CA) certificate. Verifies signatures on other certificates. |
| `server-key.pem` and `server-cert.pem` | Server key and certificate. Secure database server activity and encrypt write-set replication traffic. |
| `client-key.pem` and `client-cert.pem` | Client key and certificate. Required only if the node also acts as a MySQL client, for example during SST with `mysqldump`. |

During initialization, MySQL generates a default set of key and certificate files in the data directory, for example `/var/lib/mysql`. These default files work for evaluation.

For production, move the certificates out of the data directory. Reference the new location with an absolute path in every configuration option.

To move the default files to `/etc/mysql/certs/`, run:

```bash
mkdir -p /etc/mysql/certs
mv /var/lib/mysql/*.pem /etc/mysql/certs/
chown -R mysql:mysql /etc/mysql/certs
```

## Encrypt client-server communication

PXC uses the MySQL SSL/TLS framework to encrypt connections between client applications and cluster nodes.

In `/etc/mysql/mysql.conf.d/mysqld.cnf`, configure the server with the certificate files:

```ini
[mysqld]
ssl-key=/etc/mysql/certs/server-key.pem
ssl-cert=/etc/mysql/certs/server-cert.pem
ssl-ca=/etc/mysql/certs/ca.pem
```

The server reads these files at startup to encrypt client connections. A MySQL client needs only the CA certificate to connect over an encrypted channel. A client that authenticates with a client certificate also needs the client key and certificate.

For steps to generate your own certificates, see [Generate keys and certificates manually](#generate-keys-and-certificates-manually).

### Authentication error with secure transport

On MySQL 8.4, replication setup can produce this error:

```text
Authentication requires secure connection
```

This error occurs when a replica connects to the source with an account that requires an encrypted connection, but the replica does not provide one. Accounts created with `REQUIRE SSL`, or a source with `require_secure_transport=ON`, both trigger this requirement.

To resolve the error on the replica, run:

```sql
STOP REPLICA;
CHANGE REPLICATION SOURCE TO SOURCE_SSL = 1;
START REPLICA;
```

## Encrypt replication traffic

Replication traffic includes SST, IST, write-set replication, and service messages sent over the `gcomm` channel.

You can encrypt all replication traffic with one variable, or configure each channel separately for more control.

### Configure automatic encryption

The [`pxc_encrypt_cluster_traffic`](wsrep-system-index.md#pxc_encrypt_cluster_traffic) variable encrypts all replication traffic, including SST, IST, and write-set replication.

This variable has two properties:

- Enabled by default.

- Static: a configuration change requires a full cluster restart.

When enabled, `pxc_encrypt_cluster_traffic` applies the `encrypt`, `ssl-key`, `ssl-cert`, and `ssl-ca` settings automatically, using the files listed in [File and path reference](#file-and-path-reference).

To set this explicitly:

```ini
[mysqld]
pxc_encrypt_cluster_traffic=ON
wsrep_provider_options="socket.ssl_key=/etc/mysql/certs/server-key.pem;socket.ssl_cert=/etc/mysql/certs/server-cert.pem;socket.ssl_ca=/etc/mysql/certs/ca.pem"

[sst]
encrypt=4
ssl-key=/etc/mysql/certs/server-key.pem
ssl-cert=/etc/mysql/certs/server-cert.pem
ssl-ca=/etc/mysql/certs/ca.pem
```

`wsrep_provider_options` changes only three options: `socket.ssl_key`, `socket.ssl_cert`, and `socket.ssl_ca`. All other provider options stay the same.

!!! warning "Use absolute paths"
    A bare filename, for example `server-key.pem`, resolves relative to the data directory. If you moved your certificates to `/etc/mysql/certs/` or another custom location, PXC cannot find them with a bare filename. Use an absolute path for every certificate option.

### How automatic certificate detection works

PXC checks for explicit `ssl-ca`, `ssl-cert`, and `ssl-key` settings under `[mysqld]` first.

If these settings are absent, PXC searches the data directory for three files: `ca.pem`, `server-cert.pem`, and `server-key.pem`. PXC does not search the `[sst]` section for these files.

The search produces one of two results:

* PXC finds all three files: encryption configures automatically.

* PXC finds fewer than three files: PXC generates a fatal error and does not start.

This automatic search applies only to files in the data directory. After you move certificates to `/etc/mysql/certs/` or another custom path, set `ssl-ca`, `ssl-cert`, and `ssl-key` explicitly.

### Disable automatic encryption

Disabling `pxc_encrypt_cluster_traffic` exposes replication traffic to interception on the network. Confirm that your network meets your security requirements before you disable this variable.

To disable `pxc_encrypt_cluster_traffic`:

1. Stop the cluster.

2. In the `[mysqld]` section of every node configuration file, set `pxc_encrypt_cluster_traffic=OFF`.

3. Restart the cluster.

## Configure encryption manually per channel

Configure encryption manually when you need encryption on one channel only, or when different channels require different certificates. Every node must still use identical certificate files.

Manual encryption settings are static. A change requires a full cluster restart.

| Encryption type | Covers |
| --- | --- |
| [Encrypt SST traffic](#encrypt-sst-traffic) | Data copied from a donor node to a joiner node during SST. |
| [Encrypt replication and IST traffic](#encrypt-replication-and-ist-traffic) | Write-set replication, IST, and service messages over `gcomm`. |

### Encrypt SST traffic

SST copies a full, consistent dataset from an existing node, the donor, to a new node, the joiner.

PXC 8.4 supports two SST methods: `xtrabackup-v2` and `clone`. Set `wsrep_sst_method` in the configuration file before you start the node; PXC does not allow this variable to change while the node is running.

This page covers encryption for the xtrabackup-v2 method. For the clone method, see [State Snapshot Transfer (SST) Method using Clone plugin](clone-sst.md).

The `encrypt` option controls SST encryption:

* `encrypt=0`: disabled. This is the default value.

* `encrypt=4`: enabled, using key and certificate files that OpenSSL generates.

Configure the `[sst]` section on every node:

```ini
[sst]
encrypt=4
ssl-ca=/etc/mysql/certs/ca.pem
ssl-cert=/etc/mysql/certs/server-cert.pem
ssl-key=/etc/mysql/certs/server-key.pem
```

For more information, see [State snapshot transfer](state-snapshot-transfer.md).

#### Diffie-Hellman parameters

SSL clients require Diffie-Hellman (DH) parameters of at least 1024 bits. This requirement addresses the [logjam vulnerability](https://en.wikipedia.org/wiki/Logjam_(computer_security)).

`socat` versions earlier than 1.7.3 default to 512-bit parameters, which do not meet this requirement.

If PXC does not find a `dhparams.pem` file of sufficient length in the data directory during SST, PXC generates a 2048-bit file automatically. Generation can take several minutes and delays the joining node.

To avoid this delay, generate `dhparams.pem` in advance. Place the file in the data directory before the node joins the cluster:

```bash
openssl dhparam -out /path/to/datadir/dhparams.pem 2048
```

For more information, see [PXC: dh key too small error during an SST using SSL](https://www.percona.com/blog/percona-xtradb-cluster-dh-key-too-small-error-during-an-sst-using-ssl/).

#### Encryption with the component keyring

If you encrypt data at rest with a keyring, SST encryption becomes mandatory. PXC transfers the keyring together with the encrypted data files, so the joining node can decrypt them.

Percona Server for MySQL 8.4 does not support the legacy `keyring_file` plugin. Use the `component_keyring_file` component instead, loaded through a manifest file rather than `early-plugin-load`.

For the full setup procedure, see the following pages in the Percona Server for MySQL documentation:

* [Keyring components overview](https://docs.percona.com/percona-server/8.4/keyring-components-plugins-overview.html)

* [Get started with component keyring](https://docs.percona.com/percona-server/8.4/quickstart-component-keyring.html)

Every node must use identical keyring configuration. A mismatch prevents the cluster from operating correctly.

!!! note "PXC does not replicate the keyring file"

    Galera replicates metadata and transactional state, not the keyring file itself. Copy the keyring file to every node manually, over a secure channel.

### Encrypt replication and IST traffic

Replication traffic includes three components:

| Component | Description |
| --- | --- |
| Write-set replication | Continuous replication of transactions to every node. |
| Incremental State Transfer (IST) | Transactions that a joining node is missing. IST transfers less data than a full SST. |
| Service messages | Signals that maintain cluster consistency between nodes. |

All three components use the `gcomm` communication channel. IST uses a separate connection from write-set replication and service messages. Configure both connections with the same encryption parameters.

Configure encryption with three [wsrep provider options](wsrep-provider-index.md), set through [`wsrep_provider_options`](wsrep-system-index.md#wsrep_provider_options):

* [`socket.ssl_ca`](wsrep-provider-index.md#socketssl_ca)

* [`socket.ssl_cert`](wsrep-provider-index.md#socketssl_cert)

* [`socket.ssl_key`](wsrep-provider-index.md#socketssl_key)

```ini
wsrep_provider_options="socket.ssl=yes;socket.ssl_ca=/etc/mysql/certs/ca.pem;socket.ssl_cert=/etc/mysql/certs/server-cert.pem;socket.ssl_key=/etc/mysql/certs/server-key.pem"
```

Where possible, reuse the certificate files that you configured for [client-server communication](#encrypt-client-server-communication).

To upgrade these certificates without downtime, see [Upgrade certificates](#upgrade-certificates).

## Generate keys and certificates manually

MySQL generates a default key and certificate set in the data directory during initialization. For production, generate your own set instead.

| Key or certificate | Purpose |
| --- | --- |
| Certificate Authority (CA) key and certificate | Signs server and client certificates to establish trust. |
| Server key and certificate | Secures database server activity and encrypts write-set replication traffic. |
| Client key and certificate | Encrypts client connections to the database. |

The Common Name (CN) on the server certificate and the client certificate must be unique. Each CN must differ from the CN on the CA certificate. A matching CN can cause certificate validation to fail. See [Failed validation caused by matching CN](#failed-validation-caused-by-matching-cn).

### Generate the CA key and certificate

1. Generate the CA private key:

    ```bash
    openssl genrsa 2048 > ca-key.pem
    ```

    This command generates a 2048-bit RSA private key and saves the key to `ca-key.pem`. This key signs every other certificate.

2. Generate the self-signed CA certificate:

    ```bash
    openssl req -new -x509 -nodes -days 3600 \
        -key ca-key.pem -out ca.pem
    ```

    This command produces the following results:

    * Creates a self-signed CA certificate valid for 3600 days.

    * Uses the private key from step 1, `ca-key.pem`.

    * Leaves the private key unencrypted, with the `-nodes` flag.

    * Writes the certificate to `ca.pem`.

### Generate the server key and certificate

1. Generate a private key and a certificate signing request (CSR):

    ```bash
    openssl req -newkey rsa:2048 -days 3600 \
        -nodes -keyout server-key.pem -out server-req.pem
    ```

    This command produces the following results:

    * Creates a 2048-bit RSA private key, `server-key.pem`.

    * Generates a CSR, `server-req.pem`.

    * Sets the certificate validity period to 3600 days, with `-days 3600`.

    * Leaves the private key unencrypted, with `-nodes`.

2. Remove the passphrase from the key:

    ```bash
    openssl rsa -in server-key.pem -out server-key.pem
    ```

    This command reads the private key from `server-key.pem` and writes the key back to the same file, without a passphrase.

3. Sign the CSR with the CA certificate and key:

    ```bash
    openssl x509 -req -in server-req.pem -days 3600 \
        -CA ca.pem -CAkey ca-key.pem -set_serial 01 \
        -out server-cert.pem
    ```

    This command produces the following results:

    * Signs the CSR, `server-req.pem`, with `ca.pem` and `ca-key.pem`.

    * Sets the validity period to 3600 days.

    * Assigns the serial number `01` to the certificate.

    * Writes the signed certificate to `server-cert.pem`.

### Generate the client key and certificate

1. Generate a private key and a CSR:

    ```bash
    openssl req -newkey rsa:2048 -days 3600 \
        -nodes -keyout client-key.pem -out client-req.pem
    ```

    This command produces the following results:

    * Creates a 2048-bit RSA private key, `client-key.pem`.

    * Generates a CSR, `client-req.pem`.

    * Sets the certificate validity period to 3600 days, with `-days 3600`.

    * Leaves the private key unencrypted, with `-nodes`.

2. Remove the passphrase from the key:

    ```bash
    openssl rsa -in client-key.pem -out client-key.pem
    ```

    This command stores the private key, `client-key.pem`, without a passphrase.

3. Sign the CSR with the CA certificate and key:

    ```bash
    openssl x509 -req -in client-req.pem -days 3600 \
        -CA ca.pem -CAkey ca-key.pem -set_serial 01 \
        -out client-cert.pem
    ```

    This command produces the following results:

    * Signs the CSR, `client-req.pem`, with `ca.pem` and `ca-key.pem`.

    * Generates a client certificate, `client-cert.pem`, valid for 3600 days.

    * Assigns the serial number `01` to the certificate.

### Verify certificates

To confirm that the CA certificate signed the server and client certificates, run:

```bash
openssl verify -CAfile ca.pem server-cert.pem client-cert.pem
```

A valid certificate chain produces this output:

```
server-cert.pem: OK
client-cert.pem: OK
```

Any other output indicates a problem with the certificate chain.

### Failed validation caused by matching CN

SSL configuration can fail when the certificate and the CA share the same subject attributes.

To check for this issue, compare the `Subject` and `Issuer` lines:

```bash
openssl x509 -in server-cert.pem -text -noout
```

Incorrect output, where `Subject` and `Issuer` use the same CN:

```
Issuer: CN=www.percona.com, O=Database Performance., C=US
Subject: CN=www.percona.com, O=Database Performance., C=AU
```

For a compact view, run:

```bash
openssl x509 -in server-cert.pem -subject -issuer -noout
```

Expected output, where the CN or another attribute, for example `C`, differs between `subject` and `issuer`:

```
subject= /CN=www.percona.com/O=Database Performance./C=AU
issuer= /CN=www.percona.com/O=Database Performance./C=US
```

### Deploy keys and certificates

Transfer key and certificate files to every node with `scp` or `sftp`.

Store the files in `/etc/mysql/certs/`, or another location that you use consistently across every node. Set permissions so only `mysqld` can read the files:

```bash
chown -R mysql:mysql /etc/mysql/certs
chmod 600 /etc/mysql/certs/*-key.pem
```

### Upgrade certificates

To upgrade replication certificates on a running two-node cluster without downtime:

1. Restart the first node with [`socket.ssl_ca`](wsrep-provider-index.md#socketssl_ca) set to a file that combines the old and new CA certificates.

    To merge `old-ca.pem` and `new-ca.pem` into `upgrade-ca.pem`, run:

    ```bash
    cat old-ca.pem > upgrade-ca.pem && \
    cat new-ca.pem >> upgrade-ca.pem
    ```

    Set `wsrep_provider_options` as follows:

    ```ini
    wsrep_provider_options="socket.ssl=yes;socket.ssl_ca=/etc/mysql/certs/upgrade-ca.pem;socket.ssl_cert=/etc/mysql/certs/old-cert.pem;socket.ssl_key=/etc/mysql/certs/old-key.pem"
    ```

2. Restart the second node with [`socket.ssl_ca`](wsrep-provider-index.md#socketssl_ca), [`socket.ssl_cert`](wsrep-provider-index.md#socketssl_cert), and [`socket.ssl_key`](wsrep-provider-index.md#socketssl_key) set to the new certificate files:

    ```ini
    wsrep_provider_options="socket.ssl=yes;socket.ssl_ca=/etc/mysql/certs/new-ca.pem;socket.ssl_cert=/etc/mysql/certs/new-cert.pem;socket.ssl_key=/etc/mysql/certs/new-key.pem"
    ```

3. Restart the first node again, with the new certificate files from step 2.

4. Remove the old certificate files.
