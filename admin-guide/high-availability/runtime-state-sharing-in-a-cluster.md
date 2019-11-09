# Runtime State Sharing in a Cluster

This chapter describes how to use NGINX Plus to synchronize shared memory zones across NGINX cluster nodes including sticky learn session persistence, requests limiting, and key-value store data.

* [Introduction](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#intro)
* [Prerequisites](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#prerequisites)
* [Configuring zone synchronization](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#config)
  * [Fine-tuning synchronization](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#config_finetune)
* [Managing cluster nodes](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#manage)
  * [Starting a node](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#manage_start)
  * [Stopping a node](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#manage_stop)
  * [Removing a node](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#manage_remove)
* [Using synchronization in a cluster](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#sync)
  * [Sticky learn zone synchronization](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#sync_sticky_learn)
  * [Request limits zone synchronization](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#sync_limit_req)
  * [Key-value storage zone synchronization](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#sync_keyval)
* [Monitoring Cluster State](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#monitor)
  * [Configuring the API](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#monitor_api_config)
  * [Polling sync status with the API](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/#monitor_api_use)

### Introduction

If several NGINX Plus instances are organized in a cluster, they can share some state data between them, including:

* [sticky learn](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/#sticky) session persistence
* [requests limiting](https://docs.nginx.com/nginx/admin-guide/security-controls/controlling-access-proxied-http/#limit_req)
* [key-value storage](https://nginx.org/en/docs/http/ngx_http_keyval_module.html#keyval_zone)

All NGINX Plus instances can exchange state data with all other members in a cluster, provided that the shared memory zone has the same name on all cluster members.

### Prerequisites

* [NGINX Plus R16](https://docs.nginx.com/nginx/releases/#r16) for sharing limits of requests processing and key-value data across the cluster
* [NGINX Plus R15](https://docs.nginx.com/nginx/releases/#r15) for sharing limits of sticky learn data across the cluster

### Configuring zone synchronization

For each NGINX instance in a cluster, open the NGINX configuration file and perform the following steps:

1. Enable synchronization between cluster nodes: in the top-level [`stream` `{}`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#stream) block, create a [`server` `{}`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#server) with the [`zone_sync`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync) directive:

   ```text
    stream {
       #...

        server {
            zone_sync;
            #...
        }
    }
   ```

2. Specify all other NGINX instances in a cluster with the [`zone_sync_server`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_server) directive. Cluster nodes can be added dynamically using the DNS service if the [`resolver`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#resolver) is used:

   ```text
    stream {

        resolver 10.0.0.53 valid=20s;

        server {
            zone_sync;
            zone_sync_server nginx-cluster.example.com:9000 resolve;
        }
    }
   ```

   Otherwise, each cluster node can be added statically as a separate line of the [`zone_sync_server`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_server) directive:

   ```text
    stream {

        server {
            zone_sync;
            zone_sync_server nginx-node1.example.com:9000;
            zone_sync_server nginx-node2.example.com:9000;
            zone_sync_server nginx-node3.example.com:9000;
        }
    }
   ```

3. Enable SSL by specifying the `ssl` parameter of the [`listen`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#listen) directive for the TCP server:

   ```text
   stream {

       resolver 10.0.0.53 valid=20s;

       server {
           listen 10.0.0.1:9000 ssl;
           #...

           zone_sync;
           zone_sync_server nginx-cluster.example.com:9000 resolve;
           #...
       }
   }
   ```

4. Specify the path to the certificates with the [`ssl_certificate`](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_certificate) directive, and to the private key with the [`ssl_certificate_key`](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_certificate_key) directive. Both the certificate and key must be in the PEM format:

   ```text
   stream {

       resolver 10.0.0.53 valid=20s;

       server {
           listen 10.0.0.1:9000 ssl;

           ssl_certificate_key /etc/ssl/nginx-1.example.com.key.pem;
           ssl_certificate     /etc/ssl/nginx-1.example.com.server_cert.pem;

           zone_sync;
           zone_sync_server    nginx-cluster.example.com:9000 resolve;
           #...

       }
   }
   ```

5. Enable SSL connections between cluster servers with the [`zone_sync_ssl`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_ssl) directive:

   ```text
    stream {

        resolver 10.0.0.53 valid=20s;

        server {
            listen 10.0.0.1:9000 ssl;

            ssl_certificate_key /etc/ssl/nginx-1.example.com.key.pem;
            ssl_certificate     /etc/ssl/nginx-1.example.com.server_cert.pem;

            zone_sync;
            zone_sync_server    nginx-cluster.example.com:9000 resolve;

            zone_sync_ssl;
            #...
        }
    }
   ```

6. Enable SSL certificate authentication on another cluster server with the [`zone_sync_ssl_certificate`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_ssl_certificate) and [`zone_sync_ssl_certificate_key`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_ssl_certificate_key) directives:

   ```text
    stream {

        resolver 10.0.0.53 valid=20s;

        server {
            listen 10.0.0.1:9000 ssl;

            ssl_certificate_key /etc/ssl/nginx-1.example.com.key.pem;
            ssl_certificate     /etc/ssl/nginx-1.example.com.server_cert.pem;

            zone_sync;
            zone_sync_server    nginx-cluster.example.com:9000 resolve;

            zone_sync_ssl;
            zone_sync_ssl_certificate     localhost.crt;
            zone_sync_ssl_certificate_key localhost.key;
            #...

        }
    }
   ```

   Additionally, you can enable verification of another cluster server certificate with [`zone_sync_ssl_verify`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_ssl_verify), [`zone_sync_ssl_verify_depth`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_ssl_verify_depth), and [`zone_sync_ssl_trusted_certificate`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_ssl_trusted_certificate):

   ```text
    stream {

        resolver 10.0.0.53 valid=20s;

        server {
            listen 10.0.0.1:9000 ssl;

            ssl_certificate_key /etc/ssl/nginx-1.example.com.key.pem;
            ssl_certificate     /etc/ssl/nginx-1.example.com.server_cert.pem;

            zone_sync;
            zone_sync_server    nginx-cluster.example.com:9000 resolve;

            zone_sync_ssl;
            zone_sync_ssl_certificate     localhost.crt;
            zone_sync_ssl_certificate_key localhost.key;

            zone_sync_ssl_verify              on;
            zone_sync_ssl_verify_depth        2;
            zone_sync_ssl_trusted_certificate /etc/ssl/ca_chain.crt.pem;
        }
    }
   ```

#### Fine-tuning Synchronization

Generally you do not have to tune sync options, but in some cases it can be useful to adjust some of these values:

```text
#...
zone_sync;
zone_sync_server nginx-cluster.example.com:9000 resolve;

zone_sync_buffers                256 4k;
zone_sync_connect_retry_interval 1s;
zone_sync_connect_timeout        5s;
zone_sync_interval               1s;
zone_sync_timeout                5s;
#...
```

[`zone_sync_buffers`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_buffers) - controls the number of buffers and their size. Increasng the number of buffers will increase the number of information stored in them.

[`zone_sync_connect_retry_interval`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_connect_retry_interval) - sets the timeout between connection attempts to a cluster node.

[`zone_sync_connect_timeout`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_connect_timeout) - sets the time required to connect to a cluster node.

[`zone_sync_interval`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_interval) - sets an interval for polling updates in a shared memory zone. Increasing this value may lead to high consumption of cpu and memory resources, decreasing the value may result in data inconsistency between cluster nodes.

[`zone_sync_timeout`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_timeout) - sets the lifetime of a TCP stream between cluster nodes. If a TCP stream is idle for longer than the value, the connection will be closed.

### Managing Cluster Nodes

#### Starting a Node

To start a new node:

* in case of DNS, update a DNS record of a cluster hostname with the IP address of the new node and start an instance
* in case of statically added nodes, add the node’s address to nginx configuration file and reload all other nodes

When the node is started, it discovers other nodes from DNS or static configuration and starts sending updates. Other nodes eventually discover the new node using DNS and start pushing updates to it.

#### Stopping a Node

To stop a node, send the ‘QUIT’ signal:

```text
nginx -s quit
```

As soon as the node receives the signal, it finishes zone synchronization and gracefully closes open connections.

#### Removing a Node

To remove a node:

* in case of DNS, update a DNS record of a cluster hostname and remove the node’s IP address
* in case of statically added nodes, remove the node’s address from nginx configuration file on each node and reload each node.

When the node is removed, other nodes close connections to the removed node and will no longer try to connect to it. After the node is removed, it can be stopped.

### Using synchronization in a cluster

#### Sticky learn zone synchronization

If your existing configuration already uses the [sticky learn](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/#sticky) feature, existing state can be simply shared across a cluster by adding the [sync](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky_learn_sync) parameter to the existing `sticky` directive in the configuration file of each NGINX instance in a cluster. Note that the zone name must be the same in all other NGINX nodes in the cluster:

```text
upstream my_backend {
    zone my_backend 64k;

        server backends.example.com resolve;
            sticky learn​ zone=sessions:1m
            create=$upstream_cookie_session
            lookup=$cookie_session
            sync;
}

server {
    listen 80;
        location / {
            proxy_pass http://my_backend;
        }
}
```

See [Enabling Session Persistence](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/#sticky) for information how to configure the “sticky learn” session persistence method.

#### Request limits zone synchronization

If your existing configuration already uses rate limiting, these limits can be applied across a cluster by simply adding the [`sync`](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req_zone) parameter to the [`limit_req_zone`](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req_zone) directive in the configuration file of each NGINX instance in a cluster:

```text
limit_req_zone $remote_addr zone=req:1M rate=100r/s sync;

server {
    listen 80;
    location / {
        limit_req zone=req;

        proxy_pass http://my_backend;
    }
}
```

The zone name also must be the same in all other NGINX nodes in the cluster.

See [Limiting the Request Rate](https://docs.nginx.com/nginx/admin-guide/security-controls/controlling-access-proxied-http/#limit_req) for more information.

#### Key-value storage zone synchronization

Similar to rate limiting and sticky learn, the contents of the key-value shared memory zone can be shared across NGINX machines in a cluster with the `sync` parameter of the [`keyval_zone`](https://nginx.org/en/docs/http/ngx_http_keyval_module.html#keyval_zone) directive:

```text
keyval_zone zone=one:32k state=one.keyval sync;
keyval $arg_text $text zone=one;
#...
server {
    #...
    location / {
        return 200 $text;
    }

    location /api {
        api write=on;
    }
}
```

See [Dynamic Blacklisting of IP Addresses](https://docs.nginx.com/nginx/admin-guide/security-controls/blacklisting-ip-addresses/) for information how to configure and manage the key-value storage.

### Monitoring Cluster State

Cluster state data can be monitored with [NGINX Plus API metrics](https://nginx.org/en/docs/ngx_http_api_module#stream_zone_sync_):

* names of shared memory zones
* total number of records on node
* number of records that needs to be sent
* sync status per each node in the cluster

### Configuring the API

In order to get access to API metrics, you will need to configure the API:

1. Enable the NGINX Plus API in read‑write mode with the [`api`](https://nginx.org/en/docs/http/ngx_http_api_module.html#api) directive:

   ```text
   # ...
    server {
        listen 80;
        server_name www.example.com;

        location /api {
            api write=on;
        }
    }
   ```

2. It is highly recommended to [restrict access](https://docs.nginx.com/nginx/admin-guide/high-availability/controlling-access-proxied-http/#restrict) to this location, for example by allowing access only from localhost \(`127.0.0.1`\), and by restricting access to `PATCH`, `POST`, and `DELETE` methods to some users with HTTP basic authentication:

   ```text
   # ...
    server {
        listen 80;
        server_name www.example.com;

        location /api {
            limit_except GET {
                auth_basic "NGINX Plus API";
                auth_basic_user_file /path/to/passwd/file;
            }

            api   write=on;

            allow 127.0.0.1;
            deny  all;
        }
    }
   ```

See [Using the API for Dynamic Configuration](https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/#api_use) for instructions how to configure and use NGINX Plus API.

### Polling Sync Status with the API

To get the synchronization status of the shared memory zone, send the API command, for example, with `curl`:

```text
$ curl -s '127.0.0.1/api/5/stream/zone_sync' | jq
```

The output will be:

```text
{
  "zones" : {
    "zone1" : {
      "records_pending" : 2061,
      "records_total" : 260575
    },
    "zone2" : {
      "records_pending" : 0,
      "records_total" : 14749
    }
  },
  "status" : {
    "bytes_in" : 1364923761,
    "msgs_in" : 337236,
    "msgs_out" : 346717,
    "bytes_out" : 1402765472,
    "nodes_online" : 15
  }
}
```

If all nodes have approximately the same number of records \(`records_total`\) and almost empty outgoing queue \(`records_pending`\), the cluster may be considered healthy.  


