# Live Activity Monitoring

This article describes how to configure and use runtime monitoring services in NGINX Plus: the interactive Dashboard and NGINX Plus REST API.

* [About Live Activity Monitoring](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#overview)
* [Prerequisites](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#prereq)
* [Gathering Data to Appear in Statistics](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#status_data)
* [Configuring the API](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#json_configure)
* [Using the Dashboard](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#dashboard)
  * [Accessing the Dashboard](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#dashboard_access)
  * [Tabs Overview](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#dashboard_tabs)
  * [Managing Upstream Servers from the Dashboard](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#dashboard_upstream)
  * [Configuring Dashboard Options](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#dashboard_options)
* [Using the REST API](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#json)
  * [Getting statistics with the API](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#json_get)
  * [Resetting the statistics](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#json_delete)
  * [Managing Upstream Servers with the API](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#api_use)
  * [Enabling the Swagger UI](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#swagger_enable)
  * [Using the Swagger UI](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#swagger_access)
  * [API Live Examples](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#json_example)

### About Live Activity Monitoring

NGINX Plus provides various monitoring tools for your server infrastructure:

* the interactive Dashboard page available since NGINX Plus [Release 9](https://docs.nginx.com/nginx/releases/#r9) - a real-time live activity monitoring interface that shows key load and performance metrics of your server infrastructure.
* NGINX REST API available since NGINX Plus [Release 14](https://docs.nginx.com/nginx/releases/#r14) - an interface that can obtain extended status information, reset statistics, manage upstream servers on-the-fly, and manage key-value store. With the API you can connect NGINX Plus status information with third-party tools that support the JSON interface, for example, NewRelic or your own dashboard.

  **Note**: Prior to NGINX Plus [R14](https://docs.nginx.com/nginx/releases/#r14), gathering statistics and management of upstream servers in the Dashboard was performed with the [`status`](https://nginx.org/en/docs/http/ngx_http_status_module.html#status) and [`upstream_conf`](https://nginx.org/en/docs/http/ngx_http_upstream_conf_module.html) modules. Now the extended [`status`](https://nginx.org/en/docs/http/ngx_http_status_module.html#status) and [`upstream_conf`](https://nginx.org/en/docs/http/ngx_http_upstream_conf_module.html) modules are superseded by the [`api`](https://nginx.org/en/docs/http/ngx_http_api_module.html) module. Starting from R16, the [`status`](https://nginx.org/en/docs/http/ngx_http_status_module.html#status) and [`upstream_conf`](https://nginx.org/en/docs/http/ngx_http_upstream_conf_module.html) modules will be removed and completely superseded with the [`api`](https://nginx.org/en/docs/http/ngx_http_api_module.html) module.

![](../../.gitbook/assets/image%20%282%29.png)

### Prerequisites

* NGINX Plus [R13](https://docs.nginx.com/nginx/releases/#r13) and later for NGINX Plus REST API and the Dashboard
* Data for statistics \(see [Gathering Data to Appear in Statistics](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#status_data)\)

### Gathering Data to Appear in Statistics

In order to collect data from virtual servers, upstream server groups, or cache zones, you will need to _enable shared memory zones_ for the objects you want to collect data for. A shared memory zone stores configuration and runtime state information referenced by NGINX worker processes.

* To make [HTTP](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/) and [TCP](https://docs.nginx.com/nginx/admin-guide/load-balancer/tcp-udp-load-balancer/) server to appear in statistics, specify the [`status_zone`](https://nginx.org/en/docs/http/ngx_http_api_module.html#status_zone) directive. The same zone name can be specified more than once for many [`server`](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) blocks. The [`status_zone`](https://nginx.org/en/docs/http/ngx_http_api_module.html#status_zone) directive can also be specified for [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) blocks - in this case, the statistics will be aggregated separately for servers and locations in the Dashboard:

  ```text
  server {
      # ...
      status_zone status_page;
      location / {
          proxy_pass http://backend;
          status_zone location_zone;
      }
  }
  ```

* To make an upstream server group to appear in statistics, specify the [`zone`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone) directive per each [`upstream`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream) block:

  ```text
  upstream backend {
      zone   backend 64k;
      server backend1.example.com;
      server backend2.example.com;
  }
  ```

* To make cache appear in statistics, make sure that caching is enabled in your configuration. A shared memory zone for caching is specified in the [`proxy_cache_path`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path), [`fastcgi_cache_path`](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_cache_path), [`scgi_cache_path`](https://nginx.org/en/docs/http/ngx_http_scgi_module.html#scgi_cache_path), or [`uwsgi_cache_path`](https://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_cache_path%22%3Euwsgi_cache_path) directive in the `keys_zone` parameter. See [NGINX Content Caching](https://docs.nginx.com/nginx/admin-guide/content-cache/content-caching/) for more information:

  ```text
  http {
      # ...
      proxy_cache_path /data/nginx/cache keys_zone=one:10m;
  }
  ```

* To make health checks appear in statistics, make sure that health checks are enabled with the [`health_check`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html) directive and the server group resides in the [shared memory](https://nginx.org/en/docs/http/ngx_http_api_module.html#status_zone). See [HTTP Health Checks](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-health-check/) and [TCP Health Checks](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/nginx/admin-guide/load-balancer/ttcp-health-check.md) for more information:

  ```text
  server {
      # ...
      status_zone status_page;
      location / {
          proxy_pass http://backend;
          health_check;
      }
  }
  ```

* To make cluster information appear in the Dashboard, make sure that NGINX Plus instances are organized in the cluster and zone synchronization is enabled on each instance. See [Runtime State Sharing in a Cluster](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/) for details.
* To make resolver statistics appear in the Dashboard, specify the [`status_zone`](https://nginx.org/en/docs/http/ngx_http_core_module.html#resolver_status_zone) parameter of the [`resolver`](https://nginx.org/en/docs/http/ngx_http_core_module.html#resolver) directive:

  ```text
  resolver 192.168.33.70 status_zone=resolver-zone1;

  server {
      # ...
      }
  ```

* When finished, save and exit configuration file.
* Test the configuration and reload NGINX Plus:

  ```text
  sudo nginx -t && sudo nginx -s reload
  ```

### Configuring the API

To enable the API:

* In the [`http`](https://nginx.org/en/docs/http/ngx_http_core_module.html#http) context, specify a [`server`](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) block that will be responsible for the API:

  ```text
  http {
      server {
          # your api configuration will be here
      }
  }
  ```

* Create a [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) for API requests and specify the [`api`](https://nginx.org/en/docs/http/ngx_http_api_module.html#api) directive in this location:

  ```text
  http {
      # ...
      server {
          listen 192.168.1.23;
          # ...
          location /api {
              api;
              # ...
          }
      }
  }
  ```

* In order to make changes with the API, such as [resetting statistics counters](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#json_delete), managing [upstream servers on-the-fly](https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/) or [key-value storage](https://docs.nginx.com/nginx/admin-guide/security-controls/blacklisting-ip-addresses/), managing upstream servers from the [Dashboard](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#dashboard_upstream), enable the read-write mode for the API by specifying the `write=on` parameter for the [`api`](https://nginx.org/en/docs/http/ngx_http_api_module.html#api) directive:

  ```text
  http {
      # ...
      server {
          listen 192.168.1.23;
          # ...
          location /api {
              api write=on;
              # ...
          }
      }
  }
  ```

* It is recommended restricting access to this location, for example, allow access only from local networks with [`allow`](https://nginx.org/en/docs/http/ngx_http_access_module.html#allow) and [`deny`](https://nginx.org/en/docs/http/ngx_http_access_module.html#deny) directives:

  ```text
  http {
      # ...
      server {
          listen 192.168.1.23;
          # ...
          location /api {
              api write=on;
              allow 192.168.1.0/24;
              deny  all;
          }
      }
  }
  ```

* When the API is enabled in the write mode, it is recommended restricting access to `PATCH`, `POST`, and `DELETE` methods to particular users. This can be done by implementing [`HTTP basic authentication`](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html):

  ```text
  http {
      ...
      server {
          listen 192.168.1.23;
          # ...
          location /api {
              limit_except GET {
                  auth_basic "NGINX Plus API";
                  auth_basic_user_file /path/to/passwd/file;
              }
              api   write=on;
              allow 192.168.1.0/24;
              deny  all;
          }
      }
  }
  ```

* Enable the [Dashboard](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#dashboard) by specifying the `/dashboard.html` location. By default the Dashboard is located in the root directory \(for example, `/usr/share/nginx/html`\) specified by the [root](https://nginx.org/en/docs/http/ngx_http_core_module.html#root) directive:

  ```text
  http {
      ...
      server {
          listen 192.168.1.23;
          # ...
          location /api {
              limit_except GET {
                  auth_basic "NGINX Plus API";
                  auth_basic_user_file /path/to/passwd/file;
              }
              api   write=on;
              allow 192.168.1.0/24;
              deny  all;
          }
          location = /dashboard.html {
              root   /usr/share/nginx/html;
          }
      }
  }
  ```

* Enable the [Swagger UI](https://docs.nginx.com/nginx/admin-guide/monitoring/live-activity-monitoring/#swagger_enable) by creating a [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) for it, for example, _/swagger-ui_. By default Swagger UI is located in the root directory specified by the [`root`](https://nginx.org/en/docs/http/ngx_http_core_module.html#root) directive, for example, _/usr/share/nginx/html_:

  ```text
  http {
      # ...
      server {
          listen 192.168.1.23;
          # ...
          location /api {
              limit_except GET {
                  auth_basic "NGINX Plus API";
                  auth_basic_user_file /path/to/passwd/file;
              }
              api   write=on;
              allow 192.168.1.0/24;
              deny  all;
          }
          location = /dashboard.html {
              root   /usr/share/nginx/html;
          }
          location /swagger-ui {
              root   /usr/share/nginx/html;
          }
      }
  }
  ```

### Using the Dashboard

NGINX Plus Dashboard provides a real-time live activity monitoring interface that shows key load and performance metrics of your server infrastructure.

#### Accessing the Dashboard

In the address bar of your browser, type-in the address that corresponds to your Dashboard page \(in our example `http://192.168.1.23/dashboard.html`\). This will display the Dashboard page located at `/usr/share/nginx/html` as specified in the `root` directive.

There is also a live demo page from NGINX available at [demo.nginx.com/dashboard.html](https://demo.nginx.com/dashboard.html):

![](../../.gitbook/assets/image%20%283%29.png)

#### Tabs Overview

All information in NGINX Plus Dashboard is represented in tabs.

![The row of tabs at the top of the window on the NGINX&#xA0;Plus dashboard make it easy to drill down to more detailed information about server zones, upstream groups, or the cache](https://www.nginx.com/wp-content/uploads/2019/09/dashboard-tabs.png)

The **HTTP Zones** tab gives detailed statistics on the frontend performance. Statistics are shown per each [`server`](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) or [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) in the [`http`](https://nginx.org/en/docs/http/ngx_http_core_module.html#http) context. TCP and UDP status zones appear on the **TCP/UDP Zones** tab. For NGINX Plus to collect information for each server, you must include the [`status_zone`](https://nginx.org/en/docs/http/ngx_http_api_module.html#status_zone) directive in each `server` or `location` configuration block.

![The &apos;HTTP zones&apos; tab in the NGINX&#xA0;Plus live activity monitoring dashboard displays information about NGINX&#xA0;Plus&apos; interaction with clients](https://www.nginx.com/wp-content/uploads/2019/09/dashboard_tab_http_zones.png)

The **HTTP Upstreams** tab provides information about each upstream group for HTTP and HTTPS traffic. TCP and UDP upstream groups appear on the **TCP/UDP Upstreams** tab. For NGINX Plus to collect information for an upstream group, you must include the [`zone`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone) directive in the [`upstream`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream) configuration block.

![The &apos;Upstreams&apos; tab on the NGINX&#xA0;Plus live activity monitoring dashboard provides information about the servers in each upstream group for HTTP/HTTPS traffic](https://www.nginx.com/wp-content/uploads/2019/09/dashboard_tab_upstreams.png)

The **Caches** tab provides statistics about the caches configured in NGINX Plus. For NGINX Plus to collect information for an upstream group, you must [configure cache](https://docs.nginx.com/nginx/admin-guide/content-cache/content-caching/).

![The &apos;Caches&apos; tab in the NGINX&#xA0;Plus live activity monitoring dashboard provides information about cache readiness, fullness, and hit ratio](https://www.nginx.com/wp-content/uploads/2019/09/dashboard_tab_caches.png)

The **Shared Zones** tab shows how much memory is used by each shared memory zone, including cache zones, SSL session cache, upstream zones, keyval zones, session log, sticky sessions, limit\_conn and limit\_req zones.

![The &apos;Shared Zones&apos; tab in the NGINX&#xA0;Plus live activity monitoring dashboard provides information about memory usage across all shared memory zones](https://www.nginx.com/wp-content/uploads/2019/09/dashboard_tab_shared_zones.png)

The **Cluster** tab provides the synchronization status of shared memory zones across all NGINX cluster nodes. See [Runtime State Sharing in a Cluster](https://docs.nginx.com/nginx/admin-guide/high-availability/zone_sync/) for details on how to organize NGINX instances in a cluster and configure synchronization between all cluster nodes.

![The &apos;Cluster&apos; tab in the NGINX&#xA0;Plus live activity monitoring dashboard provides synchronization information of shared memory zones of NGINX cluster nodes](https://www.nginx.com/wp-content/uploads/2019/09/dashboard_tab_cluster.png)

The **Resolvers** tab provides DNS server statistics of requests and responses per each DNS status zone. For NGINX Plus to collect information about your DNS servers, include the [`status_zone`](https://nginx.org/en/docs/http/ngx_http_core_module.html#resolver_status_zone) parameter in the [`resolver`](https://nginx.org/en/docs/http/ngx_http_core_module.html#resolver) directive.

![The &apos;Resolvers&apos; tab in the NGINX&#xA0;Plus live activity monitoring dashboard provides information about cache readiness, fullness, and hit ratio](https://www.nginx.com/wp-content/uploads/2019/09/dashboard_tab_resolvers.png)

#### Managing Upstream Servers from the Dashboard

You can add new or modify and remove upstream servers directly from the Dashboard interface. Note that you must previously enable the [`api`](https://nginx.org/en/docs/http/ngx_http_api_module.html#api) in the write mode.

In the **Upstreams** or **TCP/UDP Upstreams** tab, click the pencil icon next to the server name and choose between **Edit selected** and **Add server** buttons:

![In editing mode on the &apos;Upstreams&apos; tab in the NGINX&#xA0;Plus live activity monitoring dashboard, you can add, remove, or modify servers](https://cdn.wp.nginx.com/wp-content/uploads/2015/09/Screen-Shot-2015-09-08-at-10.31.14-AM.png)

To add an upstream server, click **Add server**:

![The &apos;Add server&apos; interface for adding servers to an upstream group in the NGINX&#xA0;Plus live activity monitoring dashboard](https://cdn.wp.nginx.com/wp-content/uploads/2015/09/Screen-Shot-2015-09-08-at-10.47.22-AM.png)

To remove or modify an upstream server, click the box to the left of each server’s name, then click **Edit selected**:

![The &apos;Edit selected&apos; interface for modifying or removing servers in an upstream group in the NGINX&#xA0;Plus live activity monitoring dashboard](https://www.nginx.com/wp-content/uploads/2016/07/dashboard-r7-edit-server-interface.png)

When finished, click the **Save** button to save the changes.

#### Configuring Dashboard Options

You can configure the threshold for Dashboard warnings and alerts by clicking the Gear button in the Tabs menu:

![The &apos;Dashboard configuration&apos; interface for modifying Dashboard settings](https://www.nginx.com/wp-content/uploads/2019/09/dashboard_options.png)

**Update every N sec** - updates the Dashboard data after the specified number of seconds, default is `1` second.

**4xx warnings threshold** - represents the ratio between the numbers of `Total` requests and `4xx` errors for `HTTP Upstreams` and `HTTP Zones`. Default is `30%`.

**Calculate hit ratio for the past N sec** - represents all cache hits within the specified number of seconds for `Caches`. Default is `300` seconds.

**Not synced data threshold** - represents the ratio between `Pending` records and `Total` records for `Clusters`. Default is `70%`.

**Resolver errors threshold** - represents the ratio between `Requests` and resolver errors within the time frame specified in **Update every N sec** for `Resolvers`. Default is `3%`.

### Using the REST API

With NGINX Plus, statistics of your server infrastructure can be managed with the REST API interface. The API is based on standard HTTP requests: statistics can be obtained with `GET` requests and reset with `DELETE` requests. Upstream servers can be added with `POST` requests and modified with `PATCH` requests. See [Managing Upstream Servers with the API](https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/) for more information.

The requests are sent in the JSON format that allows you to connect the stats to monitoring tools or dashboards that support JSON.

#### Getting statistics with the API

The status information of any element can be accessed with a slash-separated URL. The URL may look as follows:

[`http://demo.nginx.com/api/5/http/caches/http_cache?fields=expired,bypass`](https://demo.nginx.com/api/5/http/caches/http_cache?fields=expired,bypass)

where:

* `/api` is the location you have configured in the NGINX configuration file for the API
* `/5` is the API version, the current API version is `5`
* `/http/caches/http_cache` is the path to the resource
* `?fields=expired,bypass` is an optional argument that specifies which fields of the requested object will be output

The requested information is returned in the JSON data format.

To get the list of all available rootpoints, send the `GET` request with the ‘curl’ command in terminal \(in the example, JSON pretty print extension “json\_pp” is used\):

```text
curl -s 'https://demo.nginx.com/api/5/' | json_pp
```

The JSON data returned:

```text
[
   "nginx",
   "processes",
   "connections",
   "slabs",
   "http",
   "stream",
   "resolvers",
   "ssl"
]
```

To get the statistics for a particular endpoint, for example, obtain general information about NGINX, send the following `GET` request:

```text
curl -s 'https://demo.nginx.com/api/5/nginx' | json_pp
```

The JSON data returned:

```text
{
   "generation" : 3,
   "timestamp" : "2019-09-02T14:04:51.515Z",
   "pid" : 2201,
   "address" : "206.251.255.64",
   "build" : "nginx-plus-r19",
   "version" : "1.17.3",
   "load_timestamp" : "2019-09-01T10:00:00.114Z",
   "ppid" : 92033
}
```

You can specify which fields of the requested object will be output with the optional _fields_ argument in the request line. For example, to display only NGINX Plus version and build, specify the command:

```text
curl -s 'https://demo.nginx.com/api/5/nginx?fields=version,build' | json_pp
```

The JSON data returned:

```text
{
   "version" : "1.17.3",
   "build" : "nginx-plus-r19"
}
```

For a complete list of available endpoints and supported methods see [reference documentation](https://nginx.org/en/docs/http/ngx_http_api_module.html).

#### Resetting the statistics

Resetting the statistics is performed by sending an API command with the `DELETE` method to a target endpoint. Make sure that your API is configured in the read-write mode.

You can reset the following types of statistics:

* abnormally terminated and respawned child processes
* accepted and dropped client connections
* SSL handshakes and session reuses
* the `reqs` and `fails` metrics for each memory slot
* total client HTTP requests
* accepted and discarded requests, responses, received and sent bytes in a particular HTTP server zone
* cache hits and cache misses in a particular cache zone
* statistics for a particular http or stream upstream server in an upstream server group

For example, to reset the number of abnormally terminated and respawned child processes, you can perform the following command in the terminal via curl:

```text
curl -X DELETE -s 'http://192.168.1.23/api/5/processes'
```

To reset accepted and dropped client connections perform the following command:

```text
curl -X DELETE  -s 'http://192.168.1.23/api/5/connections'
```

### Managing Upstream Servers with the API

The NGINX Plus REST API supports `POST` and `PATCH` HTTP methods to dynamically add a server to the upstream group or modify server parameters.

To dynamically change the configuration of an upstream group, send an HTTP request with the appropriate API method. The following examples use the `curl` command, but any mechanism for making HTTP requests is supported. All request bodies and responses are in JSON format.

The URI specifies the following information in this order:

* The hostname or IP address of the node that handles the request \(in the following examples, **192.168.1.23**\)
* The location where the `api` directive appears \(**api**\)
* The API version \(**5**\)
* The name of the upstream group, complete its place in the NGINX Plus configuration hierarchy represented as a slash-separated path \(**http/upstreams/appservers**\)

For example, to add a new server to the **appservers** upstream group, send the following `curl` command:

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
 }' -s 'http://192.168.1.23/api/5/http/upstreams/appservers/servers'
```

To remove a server from the upstream group:

```text
curl -X DELETE -s 'http://192.168.1.23/api/5/http/upstreams/appservers/servers/0'
```

To set the `down` parameter for the first server in the group \(with ID `0`\):

```text
curl -X PATCH -d '{ "down": true }' -s 'http://192.168.1.23/api/5/http/upstreams/appservers/servers/0'
```

#### The Swagger UI

NGINX Plus contains an integrated Swagger UI page that allows you to explore the REST API documentation and send API commands with the graphical user interface.

#### Using the Swagger UI

To access the Swagger UI page:

* In the address bar of your browser, type-in the address of Swagger UI, in our example the address is _http://192.168.1.23/swagger-ui/_:

![Swagger UI](https://www.nginx.com/wp-content/uploads/2019/09/swagger-ui.png)

* If you have configured the HTTPS protocol for the Swagger UI page, you will need to choose the “HTTPS” scheme in the “Schemes” menu.
* Click on the operation you want to fulfil.
* Click **Try it out**.
* Fill in the obligatory fields, if required. Generally, the required field is the name of the shared memory zone.
* As an option you can display only particular fields. In the “Fields” line specify the fields you want to be displayed separated by commas. If no fields are specified, then all fields are displayed.
* Click **Execute**. The result and the corresponding HTTP error code will be displayed below the **Execute** command.

#### API Live Examples

NGINX provides live examples of JSON data and Swagger UI on a demo website.

Live example of JSON data is available at: [https://demo.nginx.com/api/5/](https://demo.nginx.com/api/5/)

You can send an API command with curl or with a browser:

```text
curl -s 'http://demo.nginx.com/api/5/'
curl -s 'http://demo.nginx.com/api/5/nginx?fields=version,build'
curl -s 'http://demo.nginx.com/api/5/http/caches/http_cache'
curl -s 'http://demo.nginx.com/api/5/http/upstreams/'
curl -s 'http://demo.nginx.com/api/5/http/upstreams/demo-backend'
curl -s 'http://demo.nginx.com/api/5/http/upstreams/demo-backend/servers/0'
```

The Swagger UI demo page is available at: [http://demo.nginx.com/swagger-ui/](https://demo.nginx.com/swagger-ui/)



Live examples operate in the read-only mode, resetting the statistics via the `DELETE` method and creating/modifying upstream servers with the `POST`/`PATCH` methods are not available. Also note that as the demo API is served over the HTTP protocol, it is required to choose the “HTTP” scheme in the “Schemes” menu on the [Swagger UI demo page](https://demo.nginx.com/swagger-ui/).

