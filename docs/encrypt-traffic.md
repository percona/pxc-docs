# Encrypt PXC traffic

Percona XtraDB Cluster processes two distinct types of traffic and applies encryption to ensure secure data exchanges across all connections.

* Client-server traffic: flows between client applications and cluster nodes. This type of communication allows clients to send queries and retrieve data from the cluster, supporting standard database operations while maintaining data integrity and security.

* Replication traffic: synchronizes replication traffic across cluster nodes using [State Snapshot Transfer (SST)](glossary.md#sst), [Incremental State Transfer (IST)](glossary.md#ist), write-set replication, and various service messages. These processes ensure data consistency and prevent loss.

Percona XtraDB Cluster encrypts all types of traffic to prevent unauthorized access and ensure data confidentiality. The system allows administrators to configure replication traffic encryption using either automated processes or manual setup, depending on security requirements and organizational policies. Implementing encryption enhances data protection, mitigates security risks, and maintains compliance with industry standards.

## Encrypt client-server communication

Percona XtraDB Cluster relies on MySQL’s built-in encryption framework to protect communication between client applications and cluster nodes. This mechanism ensures secure data exchanges, preventing unauthorized access and maintaining confidentiality.

MySQL automatically generates default key and certificate files during initialization and stores them in the data directory. Administrators can replace these files with custom-generated certificates to meet specific security requirements, as the [Generate keys and certificates manually](#generate-keys-and-certificates-manually) section outlines.

The automatically created files provide a convenient option for SSL configuration, but consistency across all cluster nodes is essential. Each node should use the same key and certificate files to maintain a secure, unified encryption setup.

Administrators must define the appropriate encryption settings for each node in the `my.cnf` configuration file. These settings establish secure connections and ensure encrypted data transmission within the cluster.

```text
[mysqld]
ssl-ca=/etc/mysql/certs/ca.pem
ssl-cert=/etc/mysql/certs/server-cert.pem
ssl-key=/etc/mysql/certs/server-key.pem

[client]
ssl-ca=/etc/mysql/certs/ca.pem
ssl-cert=/etc/mysql/certs/client-cert.pem
ssl-key=/etc/mysql/certs/client-key.pem
```

The system reads the stored key and certificate files to secure client communication when the node restarts. These files encrypt data exchanges, ensuring privacy and protection. MySQL clients only need the second part of the configuration to connect to cluster nodes.

MySQL generates key and certificate files by default and stores them in the data directory. Users can use these files or create new certificates for stronger security or custom encryption settings. Generating new certificates helps meet specific security policies and industry standards.

Refer to
[Generate keys and certificates manually](#generate-keys-and-certificates-manually) section for step-by-step instructions on key creation, certificate signing, and configuration. This guide provides the details to set up encryption according to security requirements.

## Encrypt replication traffic

Replication traffic refers to the inter-node traffic, which includes
[the SST traffic](glossary.md#sst), [IST traffic](glossary.md#ist), and replication traffic.

Each type of traffic uses a separate channel, so administrators must configure secure channels for all three variants to protect replication traffic fully.

Percona XtraDB Cluster provides a single configuration option to secure all replication traffic. This option, known as SSL automatic configuration, ensures encrypted communication across the cluster. Administrators can also enhance security by configuring each channel separately using independent parameters.

## Use consistent SSL certificates across all cluster nodes

Automatic SSL encryption relies on key and certificate files to secure communication. MySQL generates default key and certificate files during initialization and stores them in the data directory.

All cluster nodes must use the same SSL certificates to maintain a secure and consistent encryption setup. Using different certificates on individual nodes can create connectivity problems and weaken security. Configure each node with identical SSL credentials to ensure reliable and encrypted communication across the cluster.

### Configure SSL encryption

Percona XtraDB Cluster provides the pxc-encrypt-cluster-traffic variable to automate SSL encryption for replication traffic, including State Snapshot Transfer (SST), Incremental State Transfer (IST), and other replication processes. This configuration ensures that data exchanges between nodes remain encrypted, preventing unauthorized access and maintaining the cluster's security.

By default, [`pxc-encrypt-cluster-traffic`](wsrep-system-index.md#pxc_encrypt_cluster_traffic) enables a secure communication channel for replication. This variable is not dynamic, meaning administrators cannot modify the value while running the cluster. Any change to the encryption settings requires stopping the cluster, updating the configuration, and restarting the nodes.

When enabled, [`pxc-encrypt-cluster-traffic`](wsrep-system-index.md#pxc_encrypt_cluster_traffic) automatically applies encryption settings, including encrypt, ssl_key, ssl-ca, and ssl-cert. The system uses these parameters to create a secure data exchange between nodes.

Administrators can explicitly configure encryption in `my.cnf` configuration file by setting ```pxc-encrypt-cluster-traffic=ON```. 

This configuration applies the following settings:

```text
[mysqld]
wsrep_provider_options=”socket.ssl_key=server-key.pem;socket.ssl_cert=server-cert.pem;socket.ssl_ca=ca.pem”

[sst]
encrypt=4
ssl-key=server-key.pem
ssl-ca=ca.pem
ssl-cert=server-cert.pem
```

For [`wsrep_provider_options`](wsrep-system-index.md#wsrep_provider_options), only the mentioned options are affected (`socket.ssl_key`, `socket,ssl_cert`, and `socket.ssl_ca`), the rest is not modified.

#### Disable pxc-encrypt-cluster-traffic and assess security risks

The default setting for [`PCX`](wsrep-system-index.md#pxc_encrypt_cluster_traffic) significantly enhances your system's security by encrypting the traffic between nodes in a Percona XtraDB Cluster (PXC).

When the option for [`pxc-encrypt-cluster-traffic`](wsrep-system-index.md#pxc_encrypt_cluster_traffic) remains disabled, individuals with access to the network can connect to any PXC node, either as a client or as another node attempting to join the cluster. This unrestricted access allows these individuals to query sensitive data or obtain a complete copy of the data stored within the cluster.

In situations where disabling [`pxc-encrypt-cluster-traffic`](wsrep-system-index.md#pxc_encrypt_cluster_traffic) becomes necessary, follow these steps to ensure proper configuration. First, halt the cluster to prevent any disruptions during the update process. Next, locate the `[mysqld]` section of the configuration file for each node and modify the setting to `pxc-encrypt-cluster-traffic=OFF`. After making this change, restart the cluster to apply the new configuration. This process ensures that the cluster operates under the updated security settings.

## SSL automatic configuration

Automatic SSL encryption setup requires key and certificate files to secure communication. MySQL generates default key and certificate files and stores them in the data directory. These files support automatic SSL configuration, but all nodes must use the same key and certificate files to ensure a consistent security setup. Administrators can replace the auto-generated files with manually created certificates, as described in the [Generate keys and certificates manually](#generate-keys-and-certificates-manually) section.

When configuring encryption, the system first checks the SSL-ca, SSL, and SSL settings under [mysqld]. If the configuration does not include these options, the system actively searches the data directory for `ca.pem`, `server-cert.pem`, and `server-key.pem`. The system does not check the [sst] section for key and certificate files.

If the system locates all three required files, encryption is successfully configured. If any file is missing, the system generates a fatal error. Ensuring all necessary files are present prevents encryption failures and maintains secure communication within the cluster.

## SSL manual configuration

A user who needs encryption for a specific channel or wants to use different certificates can configure encryption manually. This approach provides greater flexibility, allowing users to customize security settings based on operational requirements.

To manually enable encryption, specify the locations of the required key and certificate files in the Percona XtraDB Cluster configuration. If the necessary files are unavailable, refer to [Generate keys and certificates manually](#generate-keys-and-certificates-manually) for instructions on creating them.

Encryption settings remain static during operation. Enabling encryption on a running cluster requires restarting the entire cluster.

You can enable encryption in three key areas of Percona XtraDB Cluster operation:

| Encryption Type           | Description |
|---------------------------|-------------|
| [Encrypt SST traffic](#encrypt-sst-traffic) | Encrypts [SST](glossary.md#sst) traffic during full data copy from one cluster node (donor) to the joining node (joiner). |
| [Encrypt replication traffic](#encrypt-replication-traffic) | Encrypts all replication-related traffic within the cluster. |
| [Encrypt IST traffic](#encrypt-replicationist-traffic) | Encrypts internal Percona XtraDB Cluster communication, including write-set replication, [IST](glossary.md#ist), and various service messages. |

### Encrypt SST traffic

When a new node (JOINER) joins the cluster, a full copy of the existing data is required to synchronize with the cluster. This process, known as State Snapshot Transfer (SST), ensures the new node receives a complete and accurate dataset from an existing node (DONOR). SST is a critical operation in Percona XtraDB Cluster because nodes start with a consistent state, avoiding data mismatches or inconsistencies.

To enable encryption during SST, administrators must configure the necessary settings in the cluster configuration file. When using the keyring_file plugin, encryption becomes mandatory. The system must transfer the keyring along with encrypted data files to ensure decryption. Without this step, the recipient node cannot access the transferred data.

For more information, see [State snapshot transfer](state-snapshot-transfer.md#state-snapshot-transfer).

The following options are to be set in `my.cnf` on all nodes:

```text
early-plugin-load=keyring_file.so
keyring-file-data=/path/to/keyring/file
```

The cluster will not work if keyring configuration across nodes is
different.

### xtrabackup

The only available SST method is xtrabackup-v2, which relies on Percona XtraBackup to transfer data securely between nodes. The wsrep_sst_method parameter is always set to xtrabackup-v2 and cannot be changed.

Encryption for this method is controlled by the encrypt option:

* `encrypt=0` is the default setting, meaning encryption is disabled.

* `encrypt=4` enables encryption using key and certificate files generated with OpenSSL. 

For details on creating custom encryption keys and certificates, see [Generating Keys and Certificates Manually](#generate-keys-and-certificates-manually)

To enable encryption for SST using XtraBackup, define the paths to the key and certificate files in each node’s configuration under the [sst] section. This operation ensures secure data transfer between the donor and joiner nodes. Properly configured encryption prevents unauthorized access and maintains the integrity of the transferred data.

An example configuration in `my.cnf`:

```text
[sst]
encrypt=4
ssl-ca=/etc/mysql/certs/ca.pem
ssl-cert=/etc/mysql/certs/server-cert.pem
ssl-key=/etc/mysql/certs/server-key.pem
```

SSL clients require DH parameters to be at least 1024 bits, due to the [logjam vulnerability](https://en.wikipedia.org/wiki/Logjam_(computer_security)).
However, versions of `socat` earlier than 1.7.3 use 512-bit parameters.
If a `dhparams.pem` file of required length is not found during SST in the data directory, a 2048-bit file is generated, which can take several minutes.

To avoid this delay, create the `dhparams.pem` file manually and place the file in the data directory before joining the node to the cluster.

An example command to generate the `dhparams.pem` file:

```{.bash data-prompt="$"}
$ openssl dhparam -out /path/to/datadir/dhparams.pem 2048
```

For more information, see [Percona XtraDB Cluster: “dh key too small” error during an SST using SSL](https://www.percona.com/blog/percona-xtradb-cluster-dh-key-too-small-error-during-an-sst-using-ssl/).

### Encrypt replication/IST traffic

Replication traffic in Percona XtraDB Cluster consists of the following key components:

| Feature                      | Description                                                                                          |
|-----------------------------|------------------------------------------------------------------------------------------------------|
| Write-set replication       | The cluster continuously replicates transactions from one node to all other nodes, ensuring data consistency across the system. |
| Incremental State Transfer (IST) | Transfers only missing transactions from the donor node to the joining node, reducing overhead.        |
| Service messages            | Synchronization signals exchanged between nodes to maintain cluster consistency.                     |

All replication traffic passes through the `gcomm` communication channel. Encrypting this channel secures [IST traffic](glossary.md#ist),
write-set replication, and service messages ensuring confidentiality and integrity. IST uses a separate channel, so administrators must configure both channels with the same encryption parameters.

To enable encryption for all these processes,
define the paths to the key, specify the paths to the key, certificate, and certificate authority files using the appropriate [wsrep provider options](wsrep-provider-index.md#wsrep-provider-index):


* [`socket.ssl_ca`](wsrep-provider-index.md#socket.ssl_ca)

* [`socket.ssl_cert`](wsrep-provider-index.md#socket.ssl_cert)

* [`socket.ssl_key`](wsrep-provider-index.md#socket.ssl_key)

To configure these encryption settings, define the required parameters in the configuration file using the [`wsrep_provider_options`](wsrep-system-index.md#wsrep_provider_options) variable. This ensures secure replication traffic across the cluster.

```shell
wsrep_provider_options="socket.ssl=yes;socket.ssl_ca=/etc/mysql/certs/ca.pem;socket.ssl_cert=/etc/mysql/certs/server-cert.pem;socket.ssl_key=/etc/mysql/certs/server-key.pem"
```

All nodes must share the same key and certificate files to maintain a secure and consistent encryption setup. Ideally, use the same files configured for [Encrypt client-server](#encrypt-client-server-communication) communication to simplify management and ensure compatibility across the cluster.

Check [upgrade-certificate](#upgrade-certificates) section on how to upgrade existing certificates.

## Generate keys and certificates manually

MySQL automatically generates default key and certificate files and stores them in the data directory during initialization. Administrators can create new sets of key and certificate files to replace the default ones, tailoring security settings to specific encryption requirements.

| Key and Certificate Type            | Purpose |
|--------------------------------------|---------|
| Certificate Authority (CA) key and certificate | Signs the server and client certificates to ensure authentication and trust. |
| Server key and certificate | Secures database server activity and encrypts write-set replication traffic within the cluster. |
| Client key and certificate | Encrypts client communication traffic to protect interactions with the database. |

Generate the required key and certificate files using OpenSSL to ensure secure encryption.

The `Common Name` value assigned to the server and client keys and certificates must be unique and different from the value used for the Certificate Authority (CA) certificate. This distinction helps maintain proper authentication and prevents conflicts in identity verification.

=== "Generate CA key and certificate"

    The Certificate Authority is used to verify the signature on certificates. These commands generate a Certificate Authority (CA) key and certificate:

    1. Generate the CA key file:

        ```{.bash data-prompt="$"}
        $ openssl genrsa 2048 > ca-key.pem
        ```
        
        The command does the following:
        
        * Generates a 2048-bit RSA private key and saves the key to `ca-key.pem`.

        * This key is essential for signing certificates.
        
    2. Generate the CA certificate file:

        ```{.bash data-prompt="$"}
        $ openssl req -new -x509 -nodes -days 3600
            -key ca-key.pem -out ca.pem
        ```
        
        The command does the following:
        
        * Generates a self-signed CA certificate valid for 3600 days.
        
        * Uses the previously created private key (ca-key.pem).
        
        * The `-nodes` flag ensures the private key is not encrypted.
        
        * Outputs the certificate to ca.pem.
        
    This CA certificate can then be used to sign other certificates for secure authentication within a system.

=== "Generate server key and certificate"

    1. Generate a new RSA key pair and certificate request:

        ```{.bash data-prompt="$"}
        $ openssl req -newkey rsa:2048 -days 3600 \
            -nodes -keyout server-key.pem -out server-req.pem
        ```
        
        The command does the following:
        
        * Creates a 2048-bit RSA private key (server-key.pem).
        
        * Generates a certificate signing request (CSR) (server-req.pem).
        
        * `-days 3600` sets the certificate validity period to 3600 days.
        
        * `-nodes` ensures the private key remains unencrypted.

    2. This command reads an RSA private key from the `server-key.pem` file, processes the key, and then writes the processed key back to the same file, removing the passphrase.

        ```{.bash data-prompt="$"}
        $ openssl rsa -in server-key.pem -out server-key.pem
        ```
        
        
    3. This command generates a signed certificate from a CSR using a specified CA certificate and its corresponding private key. The command also sets a defined validity period and serial number for the new certificate.

        ```{.bash data-prompt="$"}
        $ openssl x509 -req -in server-req.pem -days 3600 \
            -CA ca.pem -CAkey ca-key.pem -set_serial 01 \
            -out server-cert.pem
        ```
        
        The command does the following:
        
        * Processes a certificate signing request (CSR) from the `server-req.pem` file.
        
        * Sets the generated certificate's validity period to 3600 days, using the `-days 3600` option.
        
        * Specifies the Certificate Authority (CA) certificate, `ca.pem`, to sign the CSR with the -CA `ca.pem` option.
        
        * Provides the CA certificate's private key, `ca-key.pem`, for signing the CSR using the -CAkey `ca-key.pem` option.
        
        * Assigns a serial number of `01` to the newly created certificate using the `-set_serial 01` option.
        
        * Writes the resulting signed certificate to the `server-cert.pem` file, using the -out `server-cert.pem` option.
        
=== "Generate client key and certificate"

    1. Generate the client key file:

        ```{.bash data-prompt="$"}
        $ openssl req -newkey rsa:2048 -days 3600 \
            -nodes -keyout client-key.pem -out client-req.pem
        ```
        
        The command does the following:
        
        * Creates a 2048-bit RSA private key (server-key.pem).
        
        * Generates a certificate signing request (CSR) (server-req.pem).
        
        * `-days` 3600 sets the certificate validity period to 3600 days.
        
        * `-nodes` ensures the private key remains unencrypted.

    2. Remove the passphrase:

        ```{.bash data-prompt="$"}
        $ openssl rsa -in client-key.pem -out client-key.pem
        ```
        
        The command ensures the private key (server-key.pem) is stored without a passphrase.

    3. Sign the certificate request using the CA certificate and key:

        ```{.bash data-prompt="$"}
        $ openssl x509 -req -in client-req.pem -days 3600 \
           -CA ca.pem -CAkey ca-key.pem -set_serial 01 \
           -out client-cert.pem
        ```
        
       The command does the following:
       
       * Signs the CSR (server-req.pem) using the CA certificate (ca.pem) and CA key (ca-key.pem).
       
       * Generates a server certificate (server-cert.pem) valid for 3600 days.
       
       * Assigns a serial number (01) to the certificate. 

### Verify certificates

To check whether the server and client certificates are properly signed by the Certificate Authority (CA) certificate, run the following command:

```{.bash data-prompt="$"}
$ openssl verify -CAfile ca.pem server-cert.pem client-cert.pem
```

This command verifies that the server and client certificates are valid and trusted by the specified CA certificate (`ca.pem`). If the certificates are correctly signed, OpenSSL returns a confirmation message;  
otherwise, error details indicate issues with the certificate chain.

```text
server-cert.pem: OK
client-cert.pem: OK
```

### Failed validation caused by matching CN

An SSL configuration may fail if the certificate and CA files share the same attributes. To verify whether this is causing the issue, run the following openssl command and check that the CN field differs between the `Subject` and `Issuer` lines.

```{.bash data-prompt="$"}
$ openssl x509 -in server-cert.pem -text -noout
```

**Incorrect values**

```{.text .no-copy}
Certificate:
Data:
Version: 1 (0x0)
Serial Number: 1 (0x1)
Signature Algorithm: sha256WithRSAEncryption
Issuer: CN=www.percona.com, O=Database Performance., C=US
...
Subject: CN=www.percona.com, O=Database Performance., C=AU
...
```

To obtain a more compact output run `openssl` specifying `-subject` and `-issuer` parameters:

```{.bash data-prompt="$"}
$ openssl x509 -in server-cert.pem -subject -issuer -noout
```

??? example "Expected output"

    ```{.text .no-copy}
    subject= /CN=www.percona.com/O=Database Performance./C=AU
    issuer= /CN=www.percona.com/O=Database Performance./C=US
    ```

### Deploy keys and certificates

Use `scp` or `sftp` to send key and certificate files to each node securely.   
Store files in `/etc/mysql/certs/` or another accessible location.    
Set permissions to limit access, ensuring only `mysqld` has read rights.   

The following files are required:

| File Type                           | Description |
|--------------------------------------|-------------|
| Certificate Authority certificate (`ca.pem`) | Verifies signatures to authenticate and establish trust. |
| Server key and certificate (`server-key.pem`, `server-cert.pem`) | Secures database server activity and encrypts write-set replication traffic. |
| Client key and certificate (`client-key.pem`, `client-cert.pem`) | Requires usage only if the node functions as a MySQL client, such as during SST using `mysqldump`. |

Refer to the [Upgrade certificates](#upgrade-certificates) section for details on upgrading certificates when needed.

### Upgrade certificates

The following procedure outlines the steps to upgrade certificates used for securing replication traffic when operating with two nodes in the cluster.

1. Restart the first node with the [`socket.ssl_ca`](wsrep-provider-index.md#socket.ssl_ca) option set to a combination of the the old and new certificates in a single file.

    For example, you can merge contents of `old-ca.pem`
    and `new-ca.pem` into `upgrade-ca.pem` as follows:

    ```shell
    cat old-ca.pem > upgrade-ca.pem && \
    cat new-ca.pem >> upgrade-ca.pem
    ```

    Set the [`wsrep_provider_options`](wsrep-system-index.md#wsrep_provider_options) variable as follows:

    ```shell
    wsrep_provider_options="socket.ssl=yes;socket.ssl_ca=/etc/mysql/certs/upgrade-ca.pem;socket.ssl_cert=/etc/mysql/certs/old-cert.pem;socket.ssl_key=/etc/mysql/certs/old-key.pem"
    ```

2. Restart the second node with the [`socket.ssl_ca`](wsrep-provider-index.md#socket.ssl_ca), [`socket.ssl_cert`](wsrep-provider-index.md#socket.ssl_cert), and [`socket.ssl_key`](wsrep-provider-index.md#socket.ssl_cert) options set to the corresponding new certificate files.

    ```shell
    wsrep_provider_options="socket.ssl=yes;socket.ssl_ca=/etc/mysql/certs/new-ca.pem;socket.ssl_cert=/etc/mysql/certs/new-cert.pem;socket.ssl_key=/etc/mysql/certs/new-key.pem"
    ```

3. Restart the first node with the new certificate files, as in the previous step.

4. You can remove the old certificate files.
