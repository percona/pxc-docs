# Configure Systemd TimeoutStartSec 

If your MySQL service takes too long to start, it might be due to a large buffer pool or a significant amount of data synchronization. You can either increase the MySQL service `TimeoutStartSec` option or disable the option.

We recommend that yoou treat `TimeoutStartSec` as a diagnostic and management tool, not as a permanent fix for underlying performance problems.

The following are the benefits of using `TimeoutStartSec`:

* Prevents services from hanging indefinitely during startup

* Provides controlled startup time management

* Stops runaway processes from consuming system resources

* Offers flexibility in handling complex service initialization

* Helps detect and manage slow-starting database services

* Enables precise control over service launch duration

The following are the potential drawbacks of using `TimeoutStartSec`:

* Arbitrarily low settings might interrupt legitimate slow startups

* Requires careful tuning for different system configurations

* Can mask underlying performance or configuration issues

* Might hide complex initialization problems

* Demands ongoing monitoring and adjustment

You should test and use `TimeoutStartSec` in the following situations:

* Large databases with extensive initialization requirements

* Servers with limited resources

* Complex clustered database environments

* Systems with significant data synchronization needs

Use the following techniques to test the option:

* Start with generous timeout values

* Gradually reduce timeout as you understand your system's behavior

* Monitor service startup patterns

* Use logging to understand initialization bottlenecks

* Test configuration changes in staging environments

## Check the current timeout

Check the current timeout with the following:

```{.bash data-prompt="$"}
$ systemctl show mysql.service -p TimeoutStartUSec
```

??? example "Expected output"

    ```{.text .no-copy}
    TimeoutStartUSec=10min
    ```

## Edit the command

The following command opens an editor:

```{.bash data-prompt="$"}
$ sudo systemctl edit mysql.service
```

In the text editor, overwrite the `TimeoutStartSec` configuration:

```{.text .no-copy}
[Service]
TimeoutStartSec=900s
```

For the edited change to take effect, run the following command:

```{.bash data-prompt="$"}
$ sudo systemctl daemon-reload
```

After reloading the system, the value is 15 minutes.

??? example "Expected output"

    ```{.text .no-copy}
    TimeoutStartUSec=15min
    ```

## Disable the option

To disable the option, change the value to `0`:

```{.text .no-copy}
[Service]
TimeoutStartSec=0
```

For the edited change to take effect, run the following command:

```{.bash data-prompt="$"}
$ sudo systemctl daemon-reload
```

After reloading the system, the option is disabled.

??? example "Expected output"

    ```{.text .no-copy}
    TimeoutStartUSec=infinity
    ```

## Verify the change

After `systemctl` has been reloaded, verify your change. 

```{.bash data-prompt="$"}
$ systemctl show mysql.service -p TimeoutStartUSec
```