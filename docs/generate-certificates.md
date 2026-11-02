# Generate SSL certificates for the cluster

Create the certificates one time.

Run the commands on an administration host, or on the first node.

Place the same certificate files on every node. Different certificate files stop a node from joining the cluster.

`ca-key.pem` signs the server certificate and the client certificate. Keep `ca-key.pem` off every cluster node. Delete `ca-key.pem` from a cluster node after you sign the certificates on that node.

The server Common Name and the client Common Name must differ from the Certificate Authority Common Name.

On the administration host, run the following commands. Replace each Common Name when your security policy requires a different value.

```shell
mkdir -p ~/pxc-certs
cd ~/pxc-certs
openssl genrsa -out ca-key.pem 2048
openssl req -new -x509 -nodes -days 3600 \
    -key ca-key.pem -out ca.pem \
    -subj "/CN=PXC-CA"
openssl req -newkey rsa:2048 -nodes -days 3600 \
    -keyout server-key.pem -out server-req.pem \
    -subj "/CN=PXC-Server"
openssl rsa -in server-key.pem -out server-key.pem
openssl x509 -req -in server-req.pem -days 3600 \
    -CA ca.pem -CAkey ca-key.pem -set_serial 01 \
    -out server-cert.pem
openssl req -newkey rsa:2048 -nodes -days 3600 \
    -keyout client-key.pem -out client-req.pem \
    -subj "/CN=PXC-Client"
openssl rsa -in client-key.pem -out client-key.pem
openssl x509 -req -in client-req.pem -days 3600 \
    -CA ca.pem -CAkey ca-key.pem -set_serial 01 \
    -out client-cert.pem
openssl verify -CAfile ca.pem server-cert.pem client-cert.pem
```

Type `y` when OpenSSL asks to sign a certificate.

The `openssl verify` command prints the following lines when the certificates are valid:

```text
server-cert.pem: OK
client-cert.pem: OK
```

Copy the following files to each node:

* `ca.pem`

* `server-cert.pem`

* `server-key.pem`

* `client-cert.pem`

* `client-key.pem`

Leave the following files on the administration host:

* `ca-key.pem`

* `server-req.pem`

* `client-req.pem`

The host names match the example in [Configure nodes for write-set replication](configure-nodes.md#configure-nodes-for-write-set-replication). Replace the host names with the names of your nodes.

On Ubuntu, AppArmor allows the `mysql` user to read certificates under `/etc/mysql/`. See [Enable AppArmor](apparmor.md#enable-apparmor).

Grant the `mysql` user ownership of `/etc/mysql/certs/`. Limit private keys to the `mysql` user.

Run the copy from the host that holds the certificate files. When that host is a cluster node, install the files locally and omit that host from the loop.

```shell
sudo mkdir -p /etc/mysql/certs
sudo cp ~/pxc-certs/ca.pem ~/pxc-certs/server-cert.pem ~/pxc-certs/server-key.pem ~/pxc-certs/client-cert.pem ~/pxc-certs/client-key.pem /etc/mysql/certs/
sudo chown -R mysql:mysql /etc/mysql/certs
sudo chmod 600 /etc/mysql/certs/*-key.pem
rm -f ~/pxc-certs/ca-key.pem
```

```shell
for host in pxc1 pxc2 pxc3
do
    ssh "$host" 'mkdir -p ~/pxc-certs'
    scp ca.pem server-cert.pem server-key.pem client-cert.pem client-key.pem "${host}:~/pxc-certs/"
    ssh "$host" 'sudo mkdir -p /etc/mysql/certs && sudo cp ~/pxc-certs/ca.pem ~/pxc-certs/server-cert.pem ~/pxc-certs/server-key.pem ~/pxc-certs/client-cert.pem ~/pxc-certs/client-key.pem /etc/mysql/certs/ && sudo chown -R mysql:mysql /etc/mysql/certs && sudo chmod 600 /etc/mysql/certs/*-key.pem && rm -f ~/pxc-certs/ca-key.pem'
done
```

Add the certificate paths on every node. See [Configure nodes for write-set replication](configure-nodes.md#configure-nodes-for-write-set-replication).

See [Generate keys and certificates manually](encrypt-traffic.md#generate-keys-and-certificates-manually).
