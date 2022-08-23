# Encrypting PXC Traffic

There are two kinds of traffic in Percona XtraDB Cluster:


1. Client-Server traffic (the one between client applications and cluster
nodes).


2. Replication traffic, that includes [SST](../glossary.md#sst), [IST](../glossary.md#ist), write-set replication, and various service messages.

    Percona XtraDB Cluster supports encryption for all types of traffic. Replication traffic
    encryption can be configured either in automatic or in manual mode.

## Encrypting Client-Server Communication

Percona XtraDB Cluster uses the underlying MySQL encryption mechanism
to secure communication between client applications and cluster nodes.

Specify the following settings in the `my.cnf` configuration file
for each node:

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

After restart the node will use these files to encrypt communication with
clients. MySQL clients require only the second part of the configuration
to communicate with cluster nodes.

Starting from the version 5.7, *MySQL* generates default key and certificate
files and places them in data directory. You can either use them or generate
new certificates. For generation of new certificate please refer to
[Generating Keys and Certificates Manually](#generating-keys-and-certificates-manually) section.

## Encrypting Replication Traffic

Replication traffic refers to the inter-node traffic which includes
[SST](../glossary.md#sst) traffic, [IST](../glossary.md#ist) traffic, and replication traffic.

Traffic of each type is transferred via different channel, and so it is
important to configure secure channels for all 3 variants to completely
secure the replication traffic.

Starting from 5.7, PXC supports a single configuration option which helps to
secure complete replication traffic, and is often referred as Automatic
Configuration. User can also ignore this and configure security of
each channel by specifying independent parameters.

Section below will help, covering this aspect.

## SSL Automatic Configuration

### Enabling `pxc-encrypt-cluster-traffic`

Percona XtraDB Cluster includes the `pxc-encrypt-cluster-traffic` variable that
enables automatic configuration of SSL encryption there-by encrypting
[SST](../glossary.md#sst), [IST](../glossary.md#ist), and replication traffic.

This variable is not dynamic and so cannot be changed on runtime. To
enable automatic configuration of SSL encryption, set
`pxc-encrypt-cluster-traffic=ON` in the the `[mysqld]` section of the
`my.cnf` file, and restart the cluster (by default it is disabled
there-by using non-secured channel for replication).

!!! note
    
    Setting `pxc-encrypt-cluster-traffic=ON` has effect of applying the following settings in `my.cnf` configuration file:

    ```text
    [mysqld]
    wsrep_provider_options=”socket.ssl_key=server-key.pem;socket.ssl_cert=server-cert.pem;socket.ssl_ca=ca.pem”

    [sst]
    encrypt=4
    ssl-key=server-key.pem
    ssl-ca=ca.pem
    ssl-cert=server-cert.pem
    ```

    For [`wsrep_provider_options`](../wsrep-system-index.md#wsrep_provider_options), only the mentioned options are affected (`socket.ssl_key`, `socket,ssl_cert`, and `socket.ssl_ca`), the rest is not modified.

Automatic configuration of the SSL encryption needs key and certificate files.
Starting from the version 5.7, *MySQL* generates default key and certificate
files and places them in data directory. These auto-generated files are
suitable for automatic SSL configuration, but you should use the same key and
certificate files on all nodes. Also you can override auto-generated files with
manually created ones, as covered by the [Generating Keys and Certificates Manually](#generating-keys-and-certificates-manually) section.

Necessary key and certificate files are first searched at the `ssl-ca`,
`ssl-cert`, and `ssl-key` options under `[mysqld]`. If these options are
not set, it then looks in the data directory for `ca.pem`,
`server-cert.pem`, and `server-key.pem` files.

!!! note 

    The `[sst]` section is not searched.

If all three files are found, they are used to configure encryption.
If any of the files is missing, a fatal error is generated.

## SSL Manual Configuration

If user wants to enable encryption for specific channel only or
use different certificates or other mix-match, then user can opt for
manual configuration. This helps to provide more flexibility to end-users.

To enable encryption manually, the location of the required key and certificate
files shoud be specified in the Percona XtraDB Cluster configuration. If you do not have the
necessary files, see [Generating Keys and Certificates Manually](#generating-keys-and-certificates-manually).

!!! note 

    Encryption settings are not dynamic. To enable it on a running cluster, you need to restart the entire cluster.

There are three aspects of Percona XtraDB Cluster operation, where you can enable encryption:

* [Encrypting SST Traffic](#encrypting-sst-traffic)

  This refers to [SST](../glossary.md#sst) traffic during full data copy
  from one cluster node (donor) to the joining node (joiner).


* [Encrypting Replication Traffic](#encrypting-replication-traffic)


* [Encrypting IST Traffic](#encrypting-replicationist-traffic)

  This refers to all internal Percona XtraDB Cluster communication,
  such as, write-set replication, [IST](../glossary.md#ist), and various service messages.

### Encrypting SST Traffic

This refers to full data transfer
that usually occurs when a new node (JOINER) joins the cluster
and receives data from an existing node (DONOR).

For more information, see [State Snapshot Transfer](../manual/state_snapshot_transfer.md#state-snapshot-transfer).

!!! note 

    If `keyring_file` plugin is used, then SST encryption is mandatory: when copying encrypted data via SST, the keyring must be sent over with the files for decryption. In this case following options are to be set in `my.cnf` on all nodes:

    ```text
    early-plugin-load=keyring_file.so
    keyring-file-data=/path/to/keyring/file
    ```

    The cluster will not work if keyring configuration across nodes is
    different.

The following SST methods are available: `xtrabackup`, `rsync`, and `mysqldump`.

#### xtrabackup

This is the default SST method (the [`wsrep_sst_method`](../wsrep-system-index.md#wsrep_sst_method) is set to `xtrabackup-v2`), which uses [Percona XtraBackup](https://www.percona.com/software/mysql-database/percona-xtrabackup) to perform non-blocking transfer
of files. For more information, see [Percona XtraBackup SST Configuration](../manual/xtrabackup_sst.md#xtrabackup-sst).

Encryption mode for this method is selected using the [`encrypt`](../manual/xtrabackup_sst.md#cmdoption-arg-encrypt) option:


* `encrypt=0` is the default value, meaning that encryption is disabled.


* `encrypt=1`, `encrypt=2`, and `encrypt=3` have been deprecated.


* `encrypt=4` enables encryption based on key and certificate files generated with OpenSSL.
  For more information, see [Generating Keys and Certificates Manually](#generating-keys-and-certificates-manually).

  To enable encryption for SST using XtraBackup,
  specify the location of the keys and certificate files
  in the each node’s configuration under `[sst]`:

  ```text
  [sst]
  encrypt=4
  ssl-ca=/etc/mysql/certs/ca.pem
  ssl-cert=/etc/mysql/certs/server-cert.pem
  ssl-key=/etc/mysql/certs/server-key.pem
  ```

!!! note 

    SSL clients require DH parameters to be at least 1024 bits, due to the [logjam vulnerability](https://en.wikipedia.org/wiki/Logjam_(computer_security)).
    However, versions of `socat` earlier than 1.7.3 use 512-bit parameters.
    If a `dhparams.pem` file of required length
    is not found during SST in the data directory,
    it is generated with 2048 bits, which can take several minutes.
    To avoid this delay, create the `dhparams.pem` file manually
    and place it in the data directory before joining the node to the cluster:

    ```text
    openssl dhparam -out /path/to/datadir/dhparams.pem 2048
    ```

    For more information, see [this blog post](https://www.percona.com/blog/2017/04/23/percona-xtradb-cluster-dh-key-too-small-error-during-an-sst-using-ssl/).

#### rsync

This SST method does not support encryption.
Avoid using this method if you need to secure traffic
between DONOR and JOINER nodes. If you using keyring plugin then
keyring file needs to be send over from DONOR to JOINER. Avoid using
this method in such cases too.

#### mysqldump

This SST method dumps data from DONOR and imports it to JOINER.
Encryption in this case is performed using the same certificates
configured for [Encrypting Client-Server Communication](#encrypting-client-server-communication),
because `mysqldump` connects through the database client.

Here is how to enable encryption for SST using `mysqldump`
in a running cluster:


1. Create a user for SST on one of the nodes:

    ```sql
    mysql> CREATE USER 'sstuser'$'%' IDENTIFIED BY PASSWORD 'sst_password';
    ```

    !!! note 

        This user must have the same name and password on all nodes
        where you want to use `mysqldump` for SST.

2. Grant usage privileges to this user and require SSL:

    ```sql
    mysql> GRANT USAGE ON *.* TO 'sstuser' REQUIRE SSL;
    ```

3. To make sure that the SST user replicated across the cluster, run the following query on another node:

    ```sql
    mysql> SELECT User, Host, ssl_type FROM mysql.user WHERE User='sstuser';
    ```

    The example of the output is the following:
    
    ```sql
    +----------+------+----------+
    | User     | Host | ssl_type |
    +----------+------+----------+
    | sstuser | %    | Any      |
    +----------+------+----------+
    ```

    !!! note 

        If the [`wsrep_OSU_method`](../wsrep-system-index.md#wsrep_OSU_method) is set to ROI, you need to manually create the SST user on each node in the cluster.

4. Specify corresponding certificate files in both `[mysqld]` and `[client]` sections of the configuration file on each node:

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

    For more information, see [Encrypting Client-Server Communication](#encrypting-client-server-communication).


5. Also specify the SST user credentials in the [`wsrep_sst_auth`](../wsrep-system-index.md#wsrep_sst_auth) variable on each node:

    ```text
    [mysqld]
    wsrep_sst_auth = sstuser:sst_password
    ```

6. Restart the cluster with the new configuration.

    If you do everything correctly,
    `mysqldump` will connect to DONOR with the SST user,
    generate a dump file, and import it to JOINER node.

### Encrypting Replication/IST Traffic

Replication traffic refers to the following:


* Write-set replication which is the main workload of Percona XtraDB Cluster
(replicating transactions that execute on one node to all other nodes).


* Incremental State Transfer ([IST](../glossary.md#ist)) which
is copying only missing transactions from DONOR to JOINER node.


* Service messages which ensure that all nodes are synchronized.

All this traffic is transferred via the same underlying communication channel
(`gcomm`). Securing this channel will ensure that [IST](../glossary.md#ist) traffic,
write-set replication, and service messages are encrypted.
(For IST, a separate channel is configured using the same configuration
parameters, so 2 sections are described together).

To enable encryption for all these processes,
define the paths to the key, certificate and certificate authority files
using the following [wsrep provider options](../wsrep-provider-index.md#wsrep-provider-index):


* [`socket.ssl_ca`](../wsrep-provider-index.md#socket.ssl_ca)

* [`socket.ssl_cert`](../wsrep-provider-index.md#socket.ssl_cert)

* [`socket.ssl_key`](../wsrep-provider-index.md#socket.ssl_key)

To set these options, use the [`wsrep_provider_options`](../wsrep-system-index.md#wsrep_provider_options) variable in the configuration file:

```text
wsrep_provider_options="socket.ssl=yes;socket.ssl_ca=/etc/mysql/certs/ca.pem;socket.ssl_cert=/etc/mysql/certs/server-cert.pem;socket.ssl_key=/etc/mysql/certs/server-key.pem"
```

!!! note 

    You must use the same key and certificate files on all nodes, preferably those used for Encrypting Client-Server Communication.

Check the [Upgrading Certificates](#upgrading-certificates) section on how to upgrade existing certificates.

## Generating Keys and Certificates Manually

As mentioned above, *MySQL* generates default key and certificate
files and places them in data directory. If user wants to override these
certificates, the following new sets of files can be generated:


* *Certificate Authority (CA) key and certificate*
to sign the server and client certificates.


* *Server key and certificate*
to secure database server activity and write-set replication traffic.


* *Client key and certificate*
to secure client communication traffic.

These files should be generated using [OpenSSL](https://www.openssl.org/).

!!! note 

    The `Common Name` value used for the server and client keys and certificates must differ from that value used for the CA certificate.

### Generating CA Key and Certificate

The Certificate Authority is used to verify the signature on certificates.

1. Generate the CA key file:

    ```shell
    $ openssl genrsa 2048 > ca-key.pem
    ```

2. Generate the CA certificate file:

    ```shell
    $ openssl req -new -x509 -nodes -days 3600
        -key ca-key.pem -out ca.pem
    ```

### Generating Server Key and Certificate

1. Generate the server key file:

    ```shell
    $ openssl req -newkey rsa:2048 -days 3600 \
        -nodes -keyout server-key.pem -out server-req.pem
    ```

2. Remove the passphrase:

    ```shell
    $ openssl rsa -in server-key.pem -out server-key.pem
    ```

3. Generate the server certificate file:

    ```shell
    $ openssl x509 -req -in server-req.pem -days 3600 \
        -CA ca.pem -CAkey ca-key.pem -set_serial 01 \
        -out server-cert.pem
    ```

### Generating Client Key and Certificate

1. Generate the client key file:

    ```shell
    $ openssl req -newkey rsa:2048 -days 3600 \
        -nodes -keyout client-key.pem -out client-req.pem
    ```

2. Remove the passphrase:

    ```shell
    $ openssl rsa -in client-key.pem -out client-key.pem
    ```

3. Generate the client certificate file:

    ```shell
    $ openssl x509 -req -in client-req.pem -days 3600 \
       -CA ca.pem -CAkey ca-key.pem -set_serial 01 \
       -out client-cert.pem
    ```

### Verifying Certificates

To verify that the server and client certificates
are correctly signed by the CA certificate,
run the following command:

```shell
$ openssl verify -CAfile ca.pem server-cert.pem client-cert.pem
```

If the verification is successful, you should see the following output:

```text
server-cert.pem: OK
client-cert.pem: OK
```

### Failed validation caused by matching CN

Sometimes, an SSL configuration may fail if the certificate and the CA files contain the same .

To check if this is the case run `openssl` command as follows and verify that the **CN** field differs for the *Subject* and *Issuer* lines.

```shell
$ openssl x509 -in server-cert.pem -text -noout
```

To obtain a more compact output run `openssl` specifying -subject and -issuer parameters:

```shell
$ openssl x509 -in server-cert.pem -subject -issuer -noout
```

### Deploying Keys and Certificates

Use a secure method (for example, `scp` or `sftp`)
to send the key and certificate files to each node.
Place them under the `/etc/mysql/certs/` directory
or similar location where you can find them later.

!!! note 

    Make sure that this directory is protected with proper permissions.
    Most likely, you only want to give read permissions to the user running `mysqld`.

The following files are required:

* Certificate Authority certificate file (`ca.pem`)

  This file is used to verify signatures.

* Server key and certificate files (`server-key.pem` and `server-cert.pem`)

  These files are used to secure database server activity
  and write-set replication traffic.

* Client key and certificate files (`client-key.pem` and `client-cert.pem`)

  These files are required only if the node should act as a MySQL client.
  For example, if you are planning to perform SST using `mysqldump`.

!!! note 

    [Upgrading Certificates](#upgrading-certificates) subsection covers the details on upgrading certificates, if necessary.

### Upgrading Certificates

The following procedure shows how to upgrade certificates
used for securing replication traffic when there are two nodes in the cluster.


1. Restart the first node with the [`socket.ssl_ca`](../wsrep-provider-index.md#socket.ssl_ca) option set to a combination of the the old and new certificates in a single file.

    For example, you can merge contents of `old-ca.pem`
    and `new-ca.pem` into `upgrade-ca.pem` as follows:

    ```shell
    cat old-ca.pem > upgrade-ca.pem && \
    cat new-ca.pem >> upgrade-ca.pem
    ```

    Set the [`wsrep_provider_options`](../wsrep-system-index.md#wsrep_provider_options) variable as follows:

    ```shell
    wsrep_provider_options="socket.ssl=yes;socket.ssl_ca=/etc/mysql/certs/upgrade-ca.pem;socket.ssl_cert=/etc/mysql/certs/old-cert.pem;socket.ssl_key=/etc/mysql/certs/old-key.pem"
    ```

2. Restart the second node with the [`socket.ssl_ca`](../wsrep-provider-index.md#socket.ssl_ca), [`socket.ssl_cert`](../wsrep-provider-index.md#socket.ssl_cert), and [`socket.ssl_key`](../wsrep-provider-index.md#socket.ssl_cert) options set to the corresponding new certificate files.

    ```shell
    wsrep_provider_options="socket.ssl=yes;socket.ssl_ca=/etc/mysql/certs/new-ca.pem;socket.ssl_cert=/etc/mysql/certs/new-cert.pem;socket.ssl_key=/etc/mysql/certs/new-key.pem"
    ```

3. Restart the first node with the new certificate files, as in the previous step.

4. You can remove the old certificate files.
