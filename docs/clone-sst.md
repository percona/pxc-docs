# State Snapshot Transfer method with the Clone plugin

## Overview

Clone State Snapshot Transfer (SST) is available in Percona XtraDB Cluster (PXC) 8.4.4-4 and later.

Clone SST uses the MySQL Clone plugin to copy data from a Donor node to a Joiner node.

Clone SST transfers data at the file level.

Clone SST uses fewer resources than `xtrabackup` or `rsync`.

## Limitations

See [Clone plugin limitations :octicons-link-external-16:](https://docs.percona.com/percona-server/{{vers}}/clone-plugin.html).

## Key features

| Feature | Description |
|---------|-------------|
| File-level transfer | The Clone plugin copies data files with less overhead |
| Consistency | The Donor and Joiner keep consistent data |
| Shorter sync time | Node synchronization takes less time |
| Native MySQL support | The method uses MySQL. External tools are not required |

## Prerequisites

Clone SST requires the following:

* Percona XtraDB Cluster 8.4.4-4 or later

* Enough disk space and network bandwidth for the data transfer

* A configured PXC cluster with at least one Donor node

* The NetCat package

* Port `4444` open for SST data transfer by default

## Best practices

| Recommendation | Description |
|----------------|-------------|
| Select a suitable Donor | Choose a Donor with low load and enough resources |
| Monitor resource use | Track CPU, memory, and disk use during SST |
| Test before production | Run Clone SST in a staging environment first |

## Process outline

The following diagram shows the Clone SST process:

<div style="text-align: center;">
  <img src="./_static/clone-sst-process.png" alt="Clone SST process" width="200" />
</div>

## Enable the Clone SST method

### Donor and Joiner

Set [`wsrep_sst_allowed_methods`](wsrep-system-index.md#wsrep_sst_allowed_methods) to include `clone` on the Donor and the Joiner.

From PXC 8.4.4-4, the default value of `wsrep_sst_allowed_methods` includes `clone`.

In most cases, you do not need to set this option manually.

```text
[mysqld]
wsrep_sst_allowed_methods = xtrabackup-v2,clone
```

### Joiner

On the Joiner, set [`wsrep_sst_method`](wsrep-system-index.md#wsrep_sst_method) to `clone` in `my.cnf`.

`clone` is the only accepted value for Clone SST.

```text
[mysqld]
wsrep_sst_method = clone
```

### Read-only SST variables

`wsrep_sst_allowed_methods` and `wsrep_sst_method` are read-only at runtime.

Set these variables in `my.cnf` before you start the server.

A change while the server runs causes errors and can break node synchronization.

## Enable SSL for Clone SST

Store SSL certificates outside the data directory.

The Clone process changes the data directory.

Set the SSL certificates in `my.cnf`:

```text
[client]
ssl-ca = /<PATH>/ca.pem
ssl-cert = /<PATH>/client-cert.pem
ssl-key = /<PATH>/client-key.pem
[mysqld]
ssl-ca = /<PATH>/ca.pem
ssl-cert = /<PATH>/server-cert.pem
ssl-key = /<PATH>/server-key.pem
```

You can also set Clone-specific SSL options on the Joiner:

```text
[mysqld]
clone_ssl_ca = /<PATH>/ca.pem
clone_ssl_cert = /<PATH>/client-cert.pem
clone_ssl_key = /<PATH>/client-key.pem
```

Do not set `<PATH>` to the data directory.

## Clone SST temporary password

When `wsrep_sst_method` is `clone`, the Joiner creates a temporary password for the short-lived `clone_sst` account.

The Donor uses the same password to connect back to the Joiner.

If [`validate_password` :octicons-link-external-16:](https://dev.mysql.com/doc/refman/{{vers}}/en/validate-password.html) is enabled, the password must meet the policy on the Joiner and the Donor.

Configure password adjustments on the Joiner.

The password may use only characters that are safe for the SST credential handshake (`user:password@host:port`).

### Default password

The default password needs no extra configuration.

The default password includes the following:

* Length: 36 characters

* At least one uppercase letter

* At least one lowercase letter

* At least one digit

* At least one special character (`.`)

This default password meets most basic `validate_password` settings.

### Adjust the password for stricter validate_password rules

For a stricter policy, add a suffix in `my.cnf` on the Joiner.

Examples of stricter rules include a longer minimum length or higher mixed-case, digit, or special-character counts.

```text
[sst]
sst-password-suffix=AAA..//2344
```

The suffix is appended to the default generated password.

Both `sst-password-suffix` and `sst_password_suffix` are accepted.

Allowed characters in the suffix are the following:

* Letters (`A-Z`, `a-z`)

* Digits (`0-9`)

* Special characters: `.` `_` `/` `-`

Other characters cause SST to fail early.

Example policy:

```text
validate_password.length = 40
validate_password.mixed_case_count = 7
validate_password.number_count = 3
validate_password.special_char_count = 2
```

Example suffix for that policy:

```text
[sst]
sst-password-suffix=AAA..//2344
```

Match the suffix to your site policy.

The final password must pass validation on the Donor and the Joiner.

Configure `sst-password-suffix` on the Joiner only.

The Donor receives the final password from the Joiner during SST.

## Variables

### SST variables

Set the following variables for SST:

| Variable | Description | Link |
|----------|-------------|------|
| `sst_idle_timeout` | Maximum idle time in seconds before SST fails. Set this option in the `[sst]` section of `my.cnf`. | |
| `wsrep_sst_donor` | Preferred Donor for SST. If unset, the cluster selects a Donor. | [Learn more](wsrep-system-index.md#wsrep_sst_donor) |
| `wsrep_sst_method` | SST method. Only one value is allowed. | [Learn more](wsrep-system-index.md#wsrep_sst_method) |
| `wsrep_sst_receive_address` | IP address and port on the Joiner that receives SST data. | [Learn more](wsrep-system-index.md#wsrep_sst_receive_address) |

### Timeout and password options

Clone SST uses the following wait points between the Joiner and the Donor.

You can also set an optional password suffix for `validate_password`.

| Option | Default | Description |
|--------|---------|-------------|
| `joiner-timeout-wait-donor-message` | 60 | Seconds the Joiner waits for the Donor SST or Incremental State Transfer (IST) message. The process stops when the timeout is reached. |
| `joiner-timeout-clone-instance` | 90 | Seconds the Joiner waits for the clone recipient to accept connections. The process stops when the timeout is reached. |
| `donor-timeout-wait-joiner` | 200 | Seconds the Donor waits for the Joiner clone instance. Increase this value on slower systems. The Donor log shows a countdown and a timeout message. |
| `sst-password-suffix` | empty | Optional suffix appended to the default Clone SST temporary password. Use this option so the final password meets stricter `validate_password` rules. Configure this option on the Joiner only. Both `sst-password-suffix` and `sst_password_suffix` are accepted. |

Example configuration:

```text
[sst]
joiner-timeout-wait-donor-message=60
donor-timeout-wait-joiner=200
joiner-timeout-clone-instance=90
sst-password-suffix=AAA..//2344
```

## Debug the process

To write more debug output, set the following option in `my.cnf`:

```text
[sst]
wsrep-debug=true
```

The default SST port is 4444.

## Monitor the process

The Joiner log reports Clone progress as a percent of data transfer complete.

Use the following query:

```shell
SELECT FORMAT(((data/estimate)*100),2) 'completed%' FROM performance_schema.clone_progress WHERE stage LIKE 'FILE_COPY';
```

To check status and errors, use the following query:

```shell
SELECT STATE, ERROR_NO, ERROR_MESSAGE FROM performance_schema.clone_status;
```

## Troubleshoot

| Problem | Possible cause | Solution |
|---------|----------------|----------|
| Clone operation fails | Network interruptions | Confirm a stable network and enough bandwidth between nodes |
| Clone operation times out | Timeout values are too low | Increase the timeout values in the `[sst]` section of `my.cnf` |
| "No space left on device" error | Not enough disk space | Confirm that the Donor and Joiner each have free disk space of at least 1.5 times the database size |
| Permission denied errors | Incorrect MySQL user privileges | Grant `CLONE_ADMIN` to the MySQL user on both nodes |
| Connection refused on port 4444 | Firewall blocks traffic | Allow traffic on port 4444 between cluster nodes |
| Certificate validation failure | Incorrect SSL configuration | Confirm that SSL certificates are valid and stored outside the data directory |
| Clone plugin not found | Plugin is not installed | Install the plugin with `INSTALL PLUGIN clone SONAME 'mysql_clone.so'` |
| Data inconsistency after clone | Interrupted clone process | Check the MySQL error logs and restart the clone process |
| Password validation failure during Clone SST | `validate_password` policy is stricter than the default temporary password | Set `sst-password-suffix` on the Joiner so the final password meets the policy on both nodes. Use only allowed suffix characters. |
