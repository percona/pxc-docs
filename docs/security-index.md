# Security basics

By default, Percona XtraDB Cluster does not encrypt or protect stored data. To ensure the security of your deployment, you must take additional measures. Consider the following areas when securing a Percona XtraDB Cluster environment:

| Topic | Description |
|-------|-------------|
| [Securing the Network](secure-network.md#secure-network) | Anyone with access to your network can connect to any Percona XtraDB Cluster node either as a client or as another node joining the cluster. You should consider restricting access using VPN and filter traffic on ports used by Percona XtraDB Cluster. |
| [Encrypting PXC Traffic](encrypt-traffic.md#encrypt-traffic) | Unencrypted traffic can potentially be viewed by anyone monitoring your network. In Percona XtraDB Cluster {{vers}}, traffic encryption is enabled by default. |
| Data-at-rest encryption | Percona XtraDB Cluster supports tablespace encryption to provide at-rest encryption for physical tablespace data files. For more information, see [Percona Server for MySQL Data at Rest Encryption](https://docs.percona.com/percona-server/8.4/data-at-rest-encryption.html). |

## Security modules

Most modern distributions include security modules that actively control resource access for users and applications. By default, these modules often restrict communication between Percona XtraDB Cluster nodes.

The simplest solution is to disable or remove these modules, but this approach is unsuitable for production environments. Instead, configure the required security policies to allow proper communication for Percona XtraDB Cluster.

### SELinux

SELinux, or Security-Enhanced Linux, commonly enabled by default on Red Hat Enterprise Linux and its derivatives, enhances system security by enforcing mandatory access controls. This security mechanism restricts how processes interact with each other and with files, ensuring that only authorized actions occur within the system. 

By implementing a policy-based approach, SELinux helps protect against unauthorized access and potential vulnerabilities, thereby strengthening the operating system's overall security posture. 

This security module protects data in user home directories and offers the following key benefits:

* Prevents unauthorized users from exploiting the system

* Allows authorized users to access files

* Used as a role-based access control system

SELinux operates in one of two modes that determine how it applies security policies:

*	Enforcing mode (default in most RHEL-based systems): SELinux actively enforces its security policies. It blocks and logs unauthorized access based on those policies.

*	Permissive mode: SELinux does not enforce the policies. It logs violations as if it were enforcing them but allows the actions to proceed.

```{.bash data-prompt="$"}
$ setenforce 0
```
	
The `setenforce 0` command does the following:

*	Sets SELinux to permissive mode immediately (without reboot).

* Useful for troubleshooting or testing, because the system logs potential SELinux policy violations without preventing the actions.

This change is temporary. After a reboot, SELinux reverts to the mode defined in the configuration file (/etc/selinux/config).

To restore enforcing mode, run:

```{.bash data-prompt="$"}
$ setenforce 1
``` 

!!! admonition "See also"

    For more information, see [Enabling AppArmor](selinux.md#selinux)

### AppArmor

[AppArmor](https://wiki.apparmor.net/) is included
in Debian and Ubuntu. *Percona XtraDB Cluster* provides several AppArmor profiles to simplify maintenance. During installation and configuration, you can set the `mysqld` profile to `complain` mode to assist with troubleshooting.

!!! admonition "See also"

    For more information, see [Enabling AppArmor](apparmor.md#apparmor)