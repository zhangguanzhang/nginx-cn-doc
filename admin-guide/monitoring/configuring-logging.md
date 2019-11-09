# Configuring Logging

This section describes how to configure logging of errors and processed requests in NGINX Open Source and NGINX Plus.

### Setting Up the Error Log

NGINX writes information about encountered issues of different severity levels to the error log. The [`error_log`](https://nginx.org/en/docs/ngx_core_module.html#error_log) directive sets up logging to a particular file, `stderr`, or `syslog` and specifies the minimal severity level of messages to log. By default, the error log is located at **logs/error.log** \(the absolute path depends on the operating system and installation\), and messages from all severity levels above the one specified are logged.

The configuration below changes the minimal severity level of error messages to log from `error` to `warn`:

```text
error_log logs/error.log warn;
```

In this case, messages of `warn`, `error` `crit`, `alert`, and `emerg` levels are logged.

The default setting of the error log works globally. To override it, place the [`error_log`](https://nginx.org/en/docs/ngx_core_module.html#error_log) directive in the main \(top-level\) configuration context. Settings in the main context are always inherited by other configuration levels. The `error_log` directive can be also specified at the [`http`](https://nginx.org/en/docs/http/ngx_http_core_module.html#http), [`stream`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#stream), `server` and [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) levels and overrides the setting inherited from the higher levels. In case of an error, the message is written to only one error log, the one closest to the level where the error has occurred. However, if several `error_log` directives are specified on the same level, the message are written to all specified logs.

> **Note:** The ability to specify multiple `error_log` directives on the same configuration level was added in open source NGINX version [1.5.2](https://nginx.org/en/CHANGES).

### Setting Up the Access Log

NGINX writes information about client requests in the access log right after the request is processed. By default, the access log is located at **logs/access.log**, and the information is written to the log in the predefined **combined** format. To override the default setting, use the [`log_format`](https://nginx.org/en/docs/http/ngx_http_log_module.html#log_format) directive to change the format of logged messages, as well as the [`access_log`](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log) directive to specify the location of the log and its format. The log format is defined using variables.

The following examples define the log format that extends the predefined **combined** format with the value indicating the ratio of gzip compression of the response. The format is then applied to a virtual server that enables compression.

```text
http {
    log_format compression '$remote_addr - $remote_user [$time_local] '
                           '"$request" $status $body_bytes_sent '
                           '"$http_referer" "$http_user_agent" "$gzip_ratio"';

    server {
        gzip on;
        access_log /spool/logs/nginx-access.log compression;
        ...
    }
}
```

Another example of the log format enables tracking different time values between NGINX and an upstream server that may help to diagnose a problem if your website experience slowdowns. You can use the following variables to log the indicated time values:

* [`$upstream_connect_time`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_connect_time) – The time spent on establishing a connection with an upstream server
* [`$upstream_header_time`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_header_time) – The time between establishing a connection and receiving the first byte of the response header from the upstream server
* [`$upstream_response_time`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_response_time) – The time between establishing a connection and receiving the last byte of the response body from the upstream server
* [`$request_time`](https://nginx.org/en/docs/http/ngx_http_log_module.html#var_request_time) – The total time spent processing a request

All time values are measured in seconds with millisecond resolution.

```text
http {
    log_format upstream_time '$remote_addr - $remote_user [$time_local] '
                             '"$request" $status $body_bytes_sent '
                             '"$http_referer" "$http_user_agent"'
                             'rt=$request_time uct="$upstream_connect_time" uht="$upstream_header_time" urt="$upstream_response_time"';

    server {
        access_log /spool/logs/nginx-access.log upstream_time;
        ...
    }
}
```

Here are some rules how to read the resulting time values:

* When a request is processed through several servers, the variable contains several values separated by commas
* When there is an internal redirect from one upstream group to another, the values are separated by semicolons
* When a request is unable to reach an upstream server or a full header cannot be received, the variable contains “0” \(zero\)
* In case of internal error while connecting to an upstream or when a reply is taken from the cache, the variable contains “-” \(hyphen\)

Logging can be optimized by enabling the buffer for log messages and the cache of descriptors of frequently used log files whose names contain variables. To enable buffering use the `buffer` parameter of the [`access_log`](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log) directive to specify the size of the buffer. The buffered messages are then written to the log file when the next log message does not fit into the buffer as well as in some other [cases](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log).

To enable caching of log file descriptors, use the [`open_log_file_cache`](https://nginx.org/en/docs/http/ngx_http_log_module.html#open_log_file_cache) directive.

Similar to the `error_log` directive, the `access_log` directive defined on a particular configuration level overrides the settings from the previous levels. When processing of a request is completed, the message is written to the log that is configured on the current level, or inherited from the previous levels. If one level defines multiple access logs, the message is written to all of them.

### Enabling Conditional Logging

Conditional logging allows excluding trivial or unimportant log entries from the access log. In NGINX, conditional logging is enabled by the `if` parameter to the [`access_log`](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log) directive.

This example excludes requests with HTTP status codes `2xx` \(Success\) and `3xx` \(Redirection\):

```text
map $status $loggable {
    ~^[23]  0;
    default 1;
}

access_log /path/to/access.log combined if=$loggable;
```

### Logging to Syslog

The `syslog` utility is a standard for computer message logging and allows collecting log messages from different devices on a single syslog server. In NGINX, logging to syslog is configured with the `syslog:` prefix in [`error_log`](https://nginx.org/en/docs/ngx_core_module.html#error_log) and [`access_log`](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log) directives.

Syslog messages can be sent to a `server=` which can be a domain name, an IP address, or a UNIX-domain socket path. A domain name or IP address can be specified with a port to override the default port, 514. A UNIX-domain socket path can be specified after the `unix:` prefix:

```text
error_log server=unix:/var/log/nginx.sock debug;
access_log syslog:server=[2001:db8::1]:1234,facility=local7,tag=nginx,severity=info;
```

In the example, NGINX error log messages are written to a UNIX domain socket at the `debug` logging level, and the access log is written to a syslog server with an IPv6 address and port 1234.

The `facility=` parameter specifies the type of program that is logging the message. The default value is `local7`. Other possible values are: `auth`, `authpriv`, `daemon`, `cron`, `ftp`, `lpr`, `kern`, `mail`, `news`, `syslog`, `user`, `uucp`, `local0 ... local7`.

The `tag=` parameter applies a custom tag to syslog messages \(`nginx` in our example\).

The `severity=` parameter sets the severity level of syslog messages for access log. Possible values in order of increasing severity are: `debug`, `info`, `notice`, `warn`, `error` \(default\), `crit`, `alert`, and `emerg`. Messages are logged at the specified level and all more severe levels. In our example, the severity level `error` also enables `crit`, `alert`, and `emerg` levels to be logged.

### Live Activity Monitoring

NGINX Plus provides a real-time live activity monitoring interface that shows key load and performance metrics of your [HTTP](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/) and [TCP](https://docs.nginx.com/nginx/admin-guide/load-balancer/tcp-udp-load-balancer/) upstream servers. See the [Live Activity Monitoring](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/) article for more information.

To learn more about NGINX Plus, please visit the [Products](https://www.nginx.com/products/) page.

