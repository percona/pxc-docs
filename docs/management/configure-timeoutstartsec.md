# Configure Systemd TimeoutStartSec 

A node may fail to load with `$ systemctl start mysql.service` if the node has a large buffer pool or must synchronize a large amount of data. 

A resolution is to either increase the setting for the `mysql` service `TimeoutStartSec` option to a higher number or disable the option.

Check the current timeout with the following:

```shell
$ systemctl show mysql.service -p TimeoutStartUSec
```

??? example "Expected output"
```{.text .no-copy}
TimeoutStartUSec=10min
```

The following command opens an editor and allows you to edit the value:

```shell
$ sudo systemctl edit mysql.service
```

In the text editor, add the following to overwrite the `TimeoutStartSec` configuration:

```text
[Service]
TimeoutStartSec=900s
```

After systemctl is reloaded, the option value changes to 15 minutes.

??? example "Expected output"
```{.text .no-copy}
TimeoutStartUSec=15min
```

To disable the option, change the value to `0`:

```text
[Service]
TimeoutStartSec=0
```

After systemctl is reloaded, the option is disabled.

??? example "Expected output"
```{.text .no-copy}
TimeoutStartUSec=infinity
```

For the edited change to take effect, run the following command:

```shell
$ sudo systemctl daemon-reload
```

After `systemctl` has been reloaded, verify your change with `systemctl show mysql.service -p TimeoutStartUSec`.

