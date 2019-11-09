# Configuring Dynamic Load Balancing with the NGINX Plus API

This chapter describes how to configure upstream servers and upstream server groups dynamically \(on-the-fly\) with the NGINX Plus REST API.

* [Overview](https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/#overview)
* [Prerequisites](https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/#prereq)
* [Enabling Dynamic Configuration](https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/#api_setup)
* [Using the API for Dynamic Configuration](https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/#api_use)
  * [Interactive Example](https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/#example)
* [Configuring Persistence of Dynamic Configuration](https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/#state)

### Overview

With NGINX Plus, configuration of upstream servers in a server group can be modified dynamically with the NGINX Plus REST API interface. You can send an API command to view all servers or a particular server in a group, modify parameters of a particular server, add or remove servers.

**Note:** In NGINX Plus Release 12 \([R12](https://docs.nginx.com/nginx/releases/#r12)\) and earlier, dynamic configuration was performed with the `upstream_conf` handler. That API \(and the extended `status` API\) are now deprecated in favor of the NGINX Plus API.

### Prerequisites

Prior to using the dynamic configuration feature, make sure that you have the following environment:

1. NGINX Plus [R13](https://docs.nginx.com/nginx/releases/#r13) or later
2. You have created upstream groups of application or web servers, as described in [HTTP Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/) and [TCP/UDP Load Balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/tcp-udp-load-balancer/)
3. Upstream server groups reside in the shared memory zone, as described in [Sharing Data with Multiple Worker Processes](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/)

### Enabling Dynamic Configuration

1. Create an upstream server group as described in [Proxying Traffic to a Group of Servers](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/#proxying-http-traffic-to-a-group-of-servers).

   ```text
   http {
       # ...
       upstream appservers {          
           server appserv1.example.com      weight=5;
           server appserv2.example.com:8080 fail_timeout=5s;
           server reserve1.example.com:8080 backup;
           server reserve2.example.com:8080 backup;
       }
    
       server {
       # Location that proxies requests to the upstream group
           location / {
               proxy_pass http://appservers;
               health_check;
            }
       }
   }
   ```

2. Include the [`zone`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone) directive in the `upstream` block. The `zone` directive configures a zone in the shared memory and sets the zone name and size. The configuration of the server group is kept in this zone, so all worker processes use the same configuration:

   ```text
   http {
       # ...
       upstream appservers {
           zone appservers 64k;
        
           server appserv1.example.com      weight=5;
           server appserv2.example.com:8080 fail_timeout=5s;
           server reserve1.example.com:8080 backup;
           server reserve2.example.com:8080 backup;
       }
   }
   ```

3. Enable the NGINX API in read‑write mode by including the [`api`](https://nginx.org/en/docs/http/ngx_http_api_module.html#api) directive in a dedicated [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) block in a [`server`](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) block.

   We strongly recommend restricting access to the location and to `PATCH`/`POST`/`DELETE` methods. This example uses the [`allow` and `deny` directives](https://nginx.org/en/docs/http/ngx_http_access_module.html) to grant access from the `localhost` address \(`127.0.0.1`\) and deny access from all other addresses. It also restricts access to `PATCH`/`POST`/`DELETE` methods with [`HTTP basic authentication`](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html):

   ```text
   server {
       location /api {
           limit_except GET {
               auth_basic "NGINX Plus API";
               auth_basic_user_file /path/to/passwd/file;
           }
           api write=on;
           allow 127.0.0.1;
           deny  all;
       }
   }
   ```

The complete configuration looks like this:

```text
http {
    # ...
    # Configuration of the server group
    upstream appservers {
        zone appservers 64k;
        
        server appserv1.example.com      weight=5;
        server appserv2.example.com:8080 fail_timeout=5s;
        server reserve1.example.com:8080 backup;
        server reserve2.example.com:8080 backup;
    }
    server {
        # Location that proxies requests to the upstream group
        location / {
            proxy_pass http://appservers;
            health_check;
        }
        
        # Location for dynamic configuration requests
        location /api {
            limit_except GET {
                auth_basic "NGINX Plus API";
                auth_basic_user_file /path/to/passwd/file;
            }
            api write=on;
            allow 127.0.0.1;
            deny  all;
        }
    }
}
```

### Using the API for Dynamic Configuration

The NGINX Plus REST API supports the following HTTP methods:

* `GET` – Display information about an upstream group or individual server in it
* `POST` – Add a server to the upstream group
* `PATCH` – Modify the parameters of a particular server
* `DELETE` – Delete a server from the upstream group

The endpoints and methods for the NGINX Plus API are described in detail in the [reference documentation](https://nginx.org/en/docs/http/ngx_http_api_module.html). In addition, the API has a built‑in a Swagger specification that can be used to explore the API and understand the capabilities of each resource. The Swagger documentation can be accessed at `http://_NGINX-host_/swagger-ui/`.

To change the configuration of an upstream group dynamically, send an HTTP request with the appropriate API method. The following examples use the `curl` command, but any mechanism for making HTTP requests is supported. All request bodies and responses are in JSON format.

The URI specifies the following information in this order:

* The hostname or IP address of the node that handles the request \(in the following examples, `127.0.0.1`\)
* The location where the `api` directive appears \(`api`\)
* The API version \(`4`\)
* The name of the upstream group, complete its place in the NGINX Plus configuration hierarchy represented as a slash&\#8209separated path \(`http/upstreams/appservers`\)

For example, to add a new server to the `appservers` upstream group, send the following `curl` command:

```text
curl -X POST -d '{ \
   "server": "10.0.0.1:8089", \
   "weight": 4, \
   "max_conns": 0, \
   "max_fails": 0, \
   "fail_timeout": "10s", \
   "slow_start": "10s", \
   "backup": true, \
   "down": true \
 }' -s 'http://127.0.0.1/api/5/http/upstreams/appservers/servers'
```

To remove a server from the upstream group:

```text
curl -X DELETE -s 'http://127.0.0.1/api/5/http/upstreams/appservers/servers/0'
```

To set the `down` parameter for the first server in the group \(with ID `0`\):

```text
curl -X PATCH -d '{ "down": true }' -s 'http://127.0.0.1/api/5/http/upstreams/appservers/servers/0'
```

#### Interactive Example

You can explore the Swagger interface to the NGINX Plus API in read‑only mode at [http://demo.nginx.com/swagger-ui/](https://demo.nginx.com/swagger-ui/).

### Configuring Persistence of Dynamic Configuration

With the basic configuration in [Enabling the API](https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/#api_setup), changes made with the API are stored only in the shared memory zone. The changes are discarded when the NGINX Plus configuration file is reloaded.

To make the changes persist across configuration reloads, move the list of upstream servers from the `upstream` block to a special file for storing server state, defined with the [`state`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#state) directive. The recommended path for Linux distributions is `/var/lib/nginx/state/`, and for FreeBSD distributions is `/var/db/nginx/state/`.

```text
http {
    # ...
    upstream appservers {
        zone appservers 64k;
        state /var/lib/nginx/state/appservers.conf;
        
        # All servers are defined in the state file
        # server appserv1.example.com      weight=5;
        # server appserv2.example.com:8080 fail_timeout=5s;
        # server reserve1.example.com:8080 backup;
        # server reserve2.example.com:8080 backup;
    }
}
```

Keep in mind that the state file can be modified only with configuration commands from the [API](https://nginx.org/en/docs/http/ngx_http_api_module.html) interface; do not modify the file directly \(for example, using a text editor\).

