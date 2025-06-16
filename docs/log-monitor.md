# Monitor the logs

Percona XtraDB Cluster, built on the Galera Cluster technology, offers robust database server logging features. Effective log monitoring helps you identify issues, troubleshoot problems, and maintain cluster health. This guide covers essential log monitoring techniques for both beginning DevOps users and senior DBAs.

## Why log monitoring matters

Log monitoring provides critical insights into cluster operations and helps you:

* Identify issues early: Detect problems before they impact users
* Troubleshoot problems: Understand what went wrong and why
* Monitor performance: Track cluster performance and resource usage
* Ensure security: Detect unauthorized access and security threats
* Maintain compliance: Meet regulatory requirements for audit trails
* Optimize performance: Identify bottlenecks and optimization opportunities

## Log file locations and types

### Default log file location

By default, Percona XtraDB Cluster logs errors to a `<hostname>.err` file located in the data directory. This logging behavior can be customized in the `my.cnf` configuration file by using the `log_error` option or by specifying the `--log-error` parameter.

```{.bash data-prompt="$"}
# Default error log location
/var/lib/mysql/hostname.err

# Custom error log location (configured in my.cnf)
log_error = /var/log/mysql/error.log
```

### Essential log types

Percona XtraDB Cluster generates several types of logs:

| Log Type | Purpose | Location | Configuration |
|----------|---------|----------|---------------|
| Error Log | Critical errors, warnings, and startup messages | `<hostname>.err` or custom | `log_error` |
| Slow Query Log | Queries exceeding time threshold | Custom location | `slow_query_log`, `slow_query_log_file` |
| General Query Log | All SQL statements | Custom location | `general_log`, `general_log_file` |
| Binary Log | Transaction events for replication | Data directory | `log_bin`, `log_bin_index` |
| Relay Log | Replication events from master | Data directory | `relay_log`, `relay_log_index` |
| Galera Log | Cluster-specific events | Custom location | `wsrep_log_conflicts` |

## Monitor error logs

### Check error log location

First, identify where your error logs are located:

```{.bash data-prompt="mysql>"}
mysql> SHOW VARIABLES LIKE 'log_error';
```

??? example "Expected output"

    ```{.text .no-copy}

    +---------------+---------------------------+
    | Variable_name | Value                     |
    +---------------+---------------------------+
    | log_error     | /var/log/mysql/error.log |
    +---------------+---------------------------+
    ```

### Monitor error log in real-time

Use `tail` to monitor the error log in real-time:

```{.bash data-prompt="$"}
tail -f /var/log/mysql/error.log
```

### Common error log patterns

Look for these patterns in your error logs:

#### Critical errors requiring immediate attention

```{.text .no-copy}
[ERROR] WSREP: Failed to apply write set
[ERROR] WSREP: SST failed
[ERROR] WSREP: Node not ready
[ERROR] WSREP: Connection refused
[ERROR] WSREP: Quorum lost
```

#### Warnings that may indicate issues

```{.text .no-copy}
[Warning] WSREP: Flow control paused
[Warning] WSREP: High replication lag
[Warning] WSREP: Certification failure
[Warning] WSREP: Brute force abort
```

#### Normal operational messages

```{.text .no-copy}
[Note] WSREP: Node joined cluster
[Note] WSREP: SST completed
[Note] WSREP: Node synced
[Note] WSREP: Flow control resumed
```

### Parse error logs with grep

Use grep to filter specific error types:

```{.bash data-prompt="$"}
# Find all WSREP errors
grep "WSREP.*ERROR" /var/log/mysql/error.log

# Find replication-related errors
grep -i "replication\|wsrep" /var/log/mysql/error.log

# Find errors from the last hour
grep "$(date -d '1 hour ago' '+%Y-%m-%d %H')" /var/log/mysql/error.log
```

## Monitor slow query logs

### Enable slow query logging

Configure slow query logging in your `my.cnf`:

```{.ini}
[mysqld]
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
log_queries_not_using_indexes = 1
```

### Monitor slow queries

View slow queries in real-time:

```{.bash data-prompt="$"}
tail -f /var/log/mysql/slow.log
```

### Analyze slow query patterns

Use `mysqldumpslow` to analyze slow query patterns:

```{.bash data-prompt="$"}
# Show top 10 slow queries
mysqldumpslow -s t -t 10 /var/log/mysql/slow.log

# Show queries by average time
mysqldumpslow -s at -t 10 /var/log/mysql/slow.log

# Show queries by count
mysqldumpslow -s c -t 10 /var/log/mysql/slow.log
```

## Monitor general query logs

### Enable general query logging

Configure general query logging in your `my.cnf`:

```{.ini}
[mysqld]
general_log = 1
general_log_file = /var/log/mysql/general.log
```

### Monitor all queries

View all queries in real-time:

```{.bash data-prompt="$"}
tail -f /var/log/mysql/general.log
```

### Filter specific query types

Use grep to filter specific query types:

```{.bash data-prompt="$"}
# Find all SELECT queries
grep "SELECT" /var/log/mysql/general.log

# Find all INSERT queries
grep "INSERT" /var/log/mysql/general.log

# Find queries from specific users
grep "root\[root\]" /var/log/mysql/general.log
```

## Monitor Galera-specific logs

### Enable Galera logging

Configure Galera logging in your `my.cnf`:

```{.ini}
[mysqld]
wsrep_log_conflicts = 1
wsrep_debug = 1
```

### Monitor Galera events

Look for these Galera-specific log entries:

#### Cluster membership events

```{.text .no-copy}
WSREP: Member 0.0 (node1) requested state transfer from '*any*'. Selected 1.0 (node2)
WSREP: Shifting SYNCED -> DONOR/DESYNCED
WSREP: Shifting DONOR/DESYNCED -> SYNCED
```

#### Replication events

```{.text .no-copy}
WSREP: Recovered position: 1234567890-1234567890
WSREP: Recovered cluster state: 1234567890
WSREP: Recovered group UUID: 12345678-1234-1234-1234-123456789012
```

#### Flow control events

```{.text .no-copy}
WSREP: Flow control paused
WSREP: Flow control resumed
WSREP: Flow control paused (throttled)
```

## Set up log monitoring automation

### Create log monitoring scripts

Create scripts to monitor logs and send alerts when issues occur.

#### Basic error log monitoring

```{.bash}
#!/bin/bash
# Basic error log monitoring script

ERROR_LOG="/var/log/mysql/error.log"
ALERT_EMAIL="admin@example.com"

# Check for critical errors
if grep -q "WSREP.*ERROR\|FATAL\|CRITICAL" "$ERROR_LOG"; then
    echo "Critical errors detected in MySQL error log" | mail -s "MySQL Alert" "$ALERT_EMAIL"
fi

# Check for replication errors
if grep -q "WSREP.*replication\|WSREP.*certification" "$ERROR_LOG"; then
    echo "Replication errors detected in MySQL error log" | mail -s "MySQL Replication Alert" "$ALERT_EMAIL"
fi
```

#### Advanced log analysis script

```{.bash}
#!/bin/bash
# Advanced log analysis script

ERROR_LOG="/var/log/mysql/error.log"
SLOW_LOG="/var/log/mysql/slow.log"
REPORT_FILE="/tmp/mysql_log_report.txt"

# Generate log analysis report
echo "MySQL Log Analysis Report - $(date)" > "$REPORT_FILE"
echo "========================================" >> "$REPORT_FILE"

# Count error types
echo "Error Summary:" >> "$REPORT_FILE"
grep -c "WSREP.*ERROR" "$ERROR_LOG" | xargs echo "WSREP Errors:" >> "$REPORT_FILE"
grep -c "FATAL" "$ERROR_LOG" | xargs echo "Fatal Errors:" >> "$REPORT_FILE"
grep -c "Warning" "$ERROR_LOG" | xargs echo "Warnings:" >> "$REPORT_FILE"

# Analyze slow queries
echo "" >> "$REPORT_FILE"
echo "Slow Query Summary:" >> "$REPORT_FILE"
if [ -f "$SLOW_LOG" ]; then
    mysqldumpslow -s t -t 5 "$SLOW_LOG" >> "$REPORT_FILE"
else
    echo "Slow query log not found" >> "$REPORT_FILE"
fi

# Send report
mail -s "MySQL Log Analysis Report" "$ALERT_EMAIL" < "$REPORT_FILE"
```

### Set up log rotation

Configure log rotation to prevent log files from growing too large:

```{.bash}
# Create logrotate configuration
sudo tee /etc/logrotate.d/mysql << EOF
/var/log/mysql/*.log {
    daily
    missingok
    rotate 7
    compress
    delaycompress
    notifempty
    create 640 mysql mysql
    postrotate
        /usr/bin/mysqladmin flush-logs
    endscript
}
EOF
```

## Troubleshoot common log issues

### High error log volume

When error logs grow too quickly:

1. Check for recurring errors that indicate configuration issues
2. Review log level settings and adjust if necessary
3. Implement log rotation to manage file sizes
4. Investigate root causes of frequent errors

### Missing log entries

When expected log entries don't appear:

1. Verify log file locations and permissions
2. Check MySQL configuration settings
3. Ensure logging is enabled for the desired log types
4. Verify disk space and file system permissions

### Performance impact of logging

When logging affects performance:

1. Adjust log levels to reduce verbosity
2. Use asynchronous logging where possible
3. Implement log rotation to manage file sizes
4. Consider using external log aggregation tools

## Best practices for log monitoring

### Establish monitoring baselines

* Monitor log patterns during normal operations
* Document typical error rates and patterns
* Set up baseline alerts for unusual patterns
* Track log growth rates over time

### Implement comprehensive monitoring

* Monitor all log types, not just error logs
* Use multiple monitoring approaches for redundancy
* Set up both automated and manual monitoring
* Include both technical and business metrics

### Plan for scalability

* Design log monitoring to scale with cluster growth
* Use centralized log aggregation where possible
* Implement automated log analysis and alerting
* Plan for log storage and retention requirements

### Regular monitoring maintenance

* Review and update alert thresholds regularly
* Test monitoring systems and alerting procedures
* Update log monitoring tools and configurations
* Document log monitoring procedures and contacts

## Integration with monitoring tools

### ELK Stack (Elasticsearch, Logstash, Kibana)

Integrate MySQL logs with ELK stack for centralized monitoring:

```{.yaml}
# Logstash configuration for MySQL logs
input {
  file {
    path => "/var/log/mysql/*.log"
    type => "mysql"
  }
}

filter {
  if [type] == "mysql" {
    grok {
      match => { "message" => "%{MYSQL_LOG}" }
    }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "mysql-logs-%{+YYYY.MM.dd}"
  }
}
```

### Prometheus and Grafana

Export MySQL log metrics to Prometheus:

```{.bash}
# Create log monitoring exporter
#!/bin/bash
# Export log metrics to Prometheus

ERROR_COUNT=$(grep -c "ERROR" /var/log/mysql/error.log)
WARNING_COUNT=$(grep -c "Warning" /var/log/mysql/error.log)

echo "mysql_log_errors_total $ERROR_COUNT"
echo "mysql_log_warnings_total $WARNING_COUNT"
```

### Splunk integration

Configure Splunk to monitor MySQL logs:

```{.ini}
# inputs.conf
[monitor:///var/log/mysql/]
index = mysql
sourcetype = mysql:error

# props.conf
[mysql:error]
SHOULD_LINEMERGE = false
TIME_PREFIX = ^
TIME_FORMAT = %Y-%m-%d %H:%M:%S
```

## Log monitoring alerts

### Set up critical alerts

Configure alerts for critical log events:

| Event | Alert Level | Action |
|-------|-------------|---------|
| WSREP ERROR | Critical | Immediate notification |
| FATAL errors | Critical | Immediate notification |
| Replication failures | High | Alert within 5 minutes |
| Flow control pauses | Medium | Alert within 15 minutes |
| High error rates | Medium | Alert within 30 minutes |

### Create alert escalation procedures

* **Level 1**: Automated log analysis and basic alerts
* **Level 2**: Human review of critical events
* **Level 3**: Escalation to senior DBA team
* **Level 4**: Escalation to management and stakeholders

## Log monitoring checklist

### Daily monitoring tasks

* [ ] Check error logs for new critical errors
* [ ] Review slow query logs for performance issues
* [ ] Verify log rotation is working correctly
* [ ] Check disk space for log files
* [ ] Review alert notifications and responses

### Weekly monitoring tasks

* [ ] Analyze log patterns and trends
* [ ] Review and update alert thresholds
* [ ] Test log monitoring systems
* [ ] Update log monitoring documentation
* [ ] Review log retention policies

### Monthly monitoring tasks

* [ ] Comprehensive log analysis and reporting
* [ ] Review and optimize log monitoring performance
* [ ] Update log monitoring tools and configurations
* [ ] Conduct log monitoring training and knowledge sharing
* [ ] Review and update log monitoring procedures