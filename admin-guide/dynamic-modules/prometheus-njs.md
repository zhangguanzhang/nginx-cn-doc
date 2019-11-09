# Prometheus-njs

Expose Prometheus metrics endpoint directly from NGINX Plus.

### Module Info

The `nginx-plus-module-prometheus` module is an [njs](https://nginx.org/en/docs/njs/) module written to convert miscellaneous NGINX status metrics exposed by the [API](https://nginx.org/en/docs/http/ngx_http_api_module.html) module to a Prometheus compliant format. The module uses subrequests to the `/api` endpoint to access the metrics.

### Exported Metrics

The following NGINX status metrics are exported to Prometheus:

* [Connections](https://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_connections): `/connections`
* [SSL](https://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_ssl_object): `/ssl`
* [HTTP](https://nginx.org/en/docs/http/ngx_http_api_module.html#http_): `/http/`
* [HTTP Server Zones](https://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_http_server_zone): `/http/server_zones/`
* [Stream Server Zones](https://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_stream_server_zone): `/stream/server_zones/`
* [HTTP Upstreams](https://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_http_upstream): `/http/upstreams/`
* [Stream Upstreams](https://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_stream_upstream): `/stream/upstreams/`

Note: the `state` metric values in [`/http/upstreams/`](https://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_http_upstream) and [`/stream/upstreams/`](https://nginx.org/en/docs/http/ngx_http_api_module.html#def_nginx_stream_upstream) are converted using the following rule:

| **NGINX** | **Prometheus** |
| :--- | :--- |
| “up” | 1 |
| “draining” | 2 |
| “down” | 3 |
| “unavail” | 4 |
| “checking” | 5 |
| “unhealthy” | 6 |

### Installation Instructions

1. Install the nginx-plus-module-prometheus module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-prometheus
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-prometheus
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-prometheus
   ```

   Note: the [nginx-plus-module-njs](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/nginscript/) module will also be installed together with the module.

2. After module installation, perform the following steps in NGINX configuration file \(**nginx.conf**\):

* Enable the [nginx-plus-module-njs](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/nginscript/) module in the top‑level \(“main”\) context:

  ```text
  load_module modules/ngx_http_js_module.so;

  http {
      # ...
  }
  ```

* [Include](https://nginx.org/en/docs/http/ngx_http_js_module.html#js_include) the `main.js` file:

  ```text
  http {
      # ...
      js_include /usr/share/nginx-plus-module-prometheus/main.js;
  }
  ```

* [Enable](https://nginx.org/en/docs/http/ngx_http_api_module.html#api) the API to be able to expose the `/metrics` endpoint from Prometheus:

  ```text
  location /api {
      api;
      #...
  }
  ```

* [Create](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) an endpoint `/metrics` for Prometheus metrics:

  ```text
  location = /metrics {
      js_content prometheus_metrics;
  }
  ```

* \[Optional, for dynamic configuration of upstream routing\] [Set](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#set) the `$prom_keyval` variable with the [keyval](https://nginx.org/en/docs/http/ngx_http_keyval_module.html) holding upstream replacements:

  ```text
  keyval_zone zone=upstream_keyval:32k;

  location = /metrics {
      set  $prom_keyval "upstream_keyval";
  }
  ```

  Note: you will need to fill the `upstream_keyval` keyval with a map of the form `{replacement : upstream-to-replace}`. See [Dynamic Configuration of Upstream Routing](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/prometheus-njs/#1) for details.

* Optional, in case of `too big subrequest response` error in error log, [increase](https://nginx.org/en/docs/http/ngx_http_core_module.html#subrequest_output_buffer_size) the size of the buffer used for storing the response body of a subrequest:

  ```text
  http {
      # ...
      subrequest_output_buffer_size 32k;
  }
  ```

### Use Case: Dynamic Configuration of Upstream Routing

When configuring NGINX upstreams, it is possible to use a combination of NGINX [key-value store](https://nginx.org/en/docs/http/ngx_http_keyval_module.html) and generic upstream names to dynamically configure upstream routing without having to reload NGINX. However, this means that the upstream names inside the NGINX configuration file will be generic and not useful for metric reporting:

```text
keyval_zone zone=upstream_keyval:128k;
keyval      $domain $placeholder zone=upstream_keyval;

upstream 0 {
    zone dynamic 64k;
}

upstream 1 {
    zone dynamic 64k;
}
#...

upstream n {
    zone dynamic 64k;
}

server ... {

    location / {
        proxy_pass $scheme://$placeholder;
    }
}
```

#### Setting Module Variables

You can set variables in NGINX using the [`set`](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#set) directive. Setting variables in the same [`server {}`](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) or [`location {}`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) block as the `nginx-plus-module-prometheus` module will adjust how it processes requests:

```text
location = /metrics {
    set  $prom_keyval "upstream_keyval";
}
```

Setting the `$prom_keyval` variable will allow you to specify a keyval containing a map of upstream replacements with the form `{replacement : upstream-to-replace}`. This will replace any upstreams in the Prometheus output matching the `upstream-to-replace` and convert them using the corresponding `replacement`.

To replace the upstream `0` with a more informative upstream name, add to the `upstream_keyval` map:

```text
curl -X POST -d '{"backend":"0"}' -s 'localhost/api/5/http/keyvals/upstream_keyval'
```

### Complete Example

```text
load_module modules/ngx_http_js_module.so;

worker_processes  auto;

events {
    worker_connections  1024;
}

http {

    js_include /usr/share/nginx-plus-module-prometheus/main.js;

    subrequest_output_buffer_size 32k;

    keyval_zone zone=upstream_keyval:32k;

    error_log  /var/log/nginx/error_log notice;
    access_log /var/log/nginx/access_log;

    upstream 0 {
        zone backend_zone 64k;
        server 127.0.0.1:8080;
        server 127.0.1.2:8080;
    }

    upstream backend2 {
        zone   backend_zone2 64k;
        server 127.0.1.1:80;
    }

    server {
        listen 80;

        location / {
            return 200 OK;
        }

        location /api {
            api;
        }

        location = /metrics {
            set        $prom_keyval "upstream_keyval";
            js_content prometheus_metrics;
        }

        location /0 {
            proxy_pass http://0;
        }

        status_zone backend_zone;
    }

    server {
        listen 8080;

        location / {
            return 200 OK_backend;
        }

        status_zone backend_zone2;
    }
}

stream {

    upstream   stream_backend {
        zone   backend_stream_zone 64k;
        server 127.0.0.3:8080;
    }

    server {
        listen      8081 udp;
        proxy_pass  stream_backend;

        status_zone backend_stream_zone;
    }
}
```

