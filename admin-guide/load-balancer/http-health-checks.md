# HTTP Health Checks

This article describes how to configure and use HTTP health checks in NGINX Plus and NGINX open source.

* [Introduction](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-health-check/#intro)
* [Prerequisites](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-health-check/#prereq)
* [Passive Health Checks](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-health-check/#hc_passive)
  * [Server Slow Start](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-health-check/#slow_start)
* [Active Health Checks](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-health-check/#hc_active)
  * [Specifying the Requested URI](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-health-check/#hc_uri)
  * [Defining Custom Conditions](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-health-check/#hc_match)

### Introduction

NGINX and NGINX Plus can continually test your upstream servers, avoid the servers that have failed, and gracefully add the recovered servers into the load‑balanced group.

### Prerequisites

* For passive health checks, [NGINX open source](https://nginx.org/en/) or [NGINX Plus](https://www.nginx.com/products/nginx)
* For active health checks and the [live activity monitoring dashboard](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/), NGINX Plus
* A load‑balanced group of [HTTP upstream servers](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)

### Passive Health Checks

For passive health checks, NGINX and NGINX Plus monitor transactions as they happen, and try to resume failed connections. If the transaction still cannot be resumed, NGINX open source and NGINX Plus mark the server as unavailable and temporarily stop sending requests to it until it is marked active again.

The conditions under which an upstream server is marked unavailable are defined for each upstream server with parameters to the [`server`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server) directive in the `upstream` block:

* [`fail_timeout`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#fail_timeout) – Sets the time during which a number of failed attempts must happen for the server to be marked unavailable, and also the time for which the server is marked unavailable \(default is 10 seconds\).
* [`max_fails`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#max_fails) – Sets the number of failed attempts that must occur during the `fail_timeout` period for the server to be marked unavailable \(default is 1 attempt\).

In the following example, if NGINX fails to send a request to a server or does not receive a response from it 3 times in 30 seconds, it marks the server as unavailable for 30 seconds:

```text
upstream backend {
    server backend1.example.com;
    server backend2.example.com max_fails=3 fail_timeout=30s;
}
```

Note that if there is only a single server in a group, the `fail_timeout` and `max_fails` parameters are ignored and the server is never marked unavailable.

#### Server Slow Start

A recently recovered server can be easily overwhelmed by connections, which may cause the server to be marked as unavailable again. Slow start allows an upstream server to gradually recover its weight from zero to its nominal value after it has been recovered or became available. This can be done with the [`slow_start`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#slow_start) parameter of the the upstream [`server`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server) directive:

```text
upstream backend {
    server backend1.example.com slow_start=30s;
    server backend2.example.com;
    server 192.0.0.1 backup;
}
```

Note that if there is only a single server in a group, the `slow_start` parameter is ignored and the server is never marked unavailable. Slow start is exclusive to NGINX Plus.

### Active Health Checks

NGINX Plus can periodically check the health of upstream servers by sending special health‑check requests to each server and verifying the correct response.

To enable active health checks:

1. In the [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) that passes requests \([`proxy_pass`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)\) to an upstream group, include the [`health_check`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check) directive:

   ```text
   server {
       location / {
           proxy_pass http://backend;
           health_check;
       }
   }
   ```

   This snippet defines a server that passes all requests \(`location /`\) to the upstream group called `backend`. It also enables advanced health monitoring with the [`health_check`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check) directive: by default, every five seconds NGINX Plus sends a request for “**/**” to each server in the `backend` group. If any communication error or timeout occurs \(the server responds with a status code outside the range from `200` through `399`\) the health check fails. The server is marked as unhealthy, and NGINX Plus does not send client requests to it until it once again passes a health check.

   Optionally you can specify another port for health checks, for example, for monitoring health of many services on the same host. Specify a new port with the [`port`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check_port) parameter of the [`health_check`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check) directive:

   ```text
   server {
       location / {
           proxy_pass   http://backend;
           health_check port=8080;
       }
   }
   ```

2. In the upstream server group, define a shared memory zone with the [`zone`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone) directive:

   ```text
   http {
       upstream backend {
           zone backend 64k;
           server backend1.example.com;
           server backend2.example.com;
           server backend3.example.com;
           server backend4.example.com;
       }
   }
   ```

   The zone is shared among all worker processes and stores the configuration of the upstream group. This [enables](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-health-check/nginx/admin-guide/http-load-balancer#shared) the worker processes to use the same set of counters to keep track of responses from the servers in the group.

   The defaults for active health checks can be overridden with parameters to the `health_check` directive:

   ```text
   location / {
       proxy_pass http://backend;
       health_check interval=10 fails=3 passes=2;
   }
   ```

   Here, the [`interval`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check_interval) parameter increases the delay between health checks from the default 5 seconds to 10 seconds. The [`fails`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check_fails) parameter requires the server to fail three health checks to be marked as unhealthy \(up from the default one\). Finally, the [`passes`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check_passes) parameter means the server must pass two consecutive checks to be marked as healthy again instead of the default one.

#### Specifying the Requested URI

Use the [`uri`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check_uri) parameter of the [`health_check`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check) directive to set the URI to request in a health check:

```text
location / {
    proxy_pass http://backend;
    health_check uri=/some/path;
}
```

The specified URI is appended to the server domain name or IP address set for the server in the `upstream` block. For the first server in the sample `backend` group declared above, a health check requests the URI **http://backend1.example.com/some/path**.

#### Defining Custom Conditions

You can set custom conditions that the response must satisfy for the server to pass the health check. The conditions are defined in a [`match`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#match) block, which is referenced in the [`match`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check_match) parameter of the [`health_check`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check) directive.

1. On the `http {}` level, specify the `match` `{}` block and name it, for example, `server_ok`:

   ```text
   http {
       #...
       match server_ok {
           # tests are here
       }
   }
   ```

2. Refer to the block from the [`health_check`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check) directive by specifying the [`match`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check_match) parameter and the name of the [`match`](https://nginx.org/en/docs/http/ngx_stream_upstream_hc_module.html#match) block:

   ```text
   http {
       #...
       match server_ok {
           status 200-399;
           body !~ "maintenance mode";
       }
       server {
           #...
           location / {
               proxy_pass http://backend;
               health_check match=server_ok;
           }
       }
   }
   ```

   Here the health check is passed if the status code of the response is in the range `200`–`399`, and its body does not contain the string `maintenance mode`.

The `match` directive enables NGINX Plus to check the status code, header fields, and the body of a response. Using this directive it is possible to verify whether the status is in a specified range, whether a response includes a header, or whether the header or body matches a regular expression. The `match` directive can contain one status condition, one body condition, and multiple header conditions. A response must satisfy all conditions defined in `match` block for the server to pass the health check.

For example, the following `match` directive matches responses that have status code `200`, the exact value `text/html` in the `Content-Type` header, and the text `Welcome to nginx!` in the body:

```text
match welcome {
    status 200;
    header Content-Type = text/html;
    body ~ "Welcome to nginx!";
}
```

The following example uses the exclamation point \(`!`\) to define characteristics the response must not have to pass the health check. In this case, the health check passes when the status code is something other than `301`, `302`, `303`, or `307`, and there is no `Refresh` header.

```text
match not_redirect {
    status ! 301-303 307;
    header ! Refresh;
}
```

Health checks can also be enabled for non-HTTP protocols, such as [FastCGI](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html), [memcached](https://nginx.org/en/docs/http/ngx_http_memcached_module.html), [SCGI](https://nginx.org/en/docs/http/ngx_http_scgi_module.html), [uwsgi](https://nginx.org/en/docs/http/ngx_http_uwsgi_module.html), and also for [TCP and UDP](https://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html#health_check).

