# Restricting Access by Geographical Location



* [Introduction](https://docs.nginx.com/nginx/admin-guide/security-controls/controlling-access-by-geoip/#intro)
* [Prerequisites](https://docs.nginx.com/nginx/admin-guide/security-controls/controlling-access-by-geoip/#prereq)
* [Configuring GeoIP2 in NGINX Plus](https://docs.nginx.com/nginx/admin-guide/security-controls/controlling-access-by-geoip/#config)

### Introduction

NGINX Plus can differentiate users based on their geographical location. For example, you can have different website content for different countries, or you can restrict content distribution to a particular country or city.

NGINX Plus uses third-party MaxMind databases to match the IP address of the user and its location. As soon as the geoposition is known, it is then possible to use geoip-based variables in the [`map`](https://nginx.org/en/docs/http/ngx_http_map_module.html) or the [`split_clients`](https://nginx.org/en/docs/http/ngx_http_split_clients_module.html) module.

**Note** MaxMind GeoLite Legacy databases are currently [discontinued](https://support.maxmind.com/geolite-legacy-discontinuation-notice/), MaxMind GeoIP2 or GeoLite2 databases and NGINX Plus [GeoIP2 module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/geoip2/) should be used instead.

Restricting by geographical location works both for HTTP and TCP/UDP protocols.

### Prerequisites

* NGINX Plus [GeoIP2 dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/geoip2/)
* [GeoIP2](https://www.maxmind.com/en/geoip2-databases) or [GeoLite2](https://dev.maxmind.com/geoip/geoip2/geolite2/) databases from MaxMind

### Configuring GeoIP2 in NGINX Plus

1. Install the GeoIP2 dynamic module for NGINX Plus:

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-geoip2
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-geoip2
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-geoip2
   ```

2. Enable the GeoIP2 dynamic module in the NGINX Plus configuration file with the [`load_module`](https://nginx.org/en/docs/ngx_core_module.html#load_module) directive specified in the `main` configuration level:

   ```text
   load_module modules/ngx_http_geoip2_module.so;
   load_module modules/ngx_stream_geoip2_module.so;

   http {
       # ...
   }
   ```

3. Obtain and unzip GeoIP2 or GeoLite2 databases from the [MaxMind download page](https://www.maxmind.com/en/geoip2-databases):

   In this example, the GeoLite2 free downloadable databases are used:

   ```text
   $ wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-Country.mmdb.gz
   $ wget http://geolite.maxmind.com/download/geoip/database/GeoLite2-City.mmdb.gz
   $ gunzip GeoLite2-Country.mmdb.gz
   $ gunzip GeoLite2-City.mmdb.gz
   ```

4. Add the paths to the country and city databases to the NGINX configuration with the `geoip2` directive for [`http`](https://nginx.org/en/docs/http/ngx_http_core_module.html#http), [`stream`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#stream), or both:

   ```text
   http {
       #...
       geoip2 GeoIP2/GeoLite2-Country.mmdb;
       geoip2 GeoIP2/GeoLite2-City.mmdb;
       #...
    }
   
   stream {
       #...
       geoip2 GeoIP2/GeoLite2-Country.mmdb;
       geoip2 GeoIP2/GeoLite2-City.mmdb;
       #...
   }
   ```

5. Use the standard variables from the GeoIP2 module to pass data to the [`map`](https://nginx.org/en/docs/http/ngx_http_map_module.html) or [`split_clients`](https://nginx.org/en/docs/http/ngx_http_split_clients_module.html) directive.

   For example, using the `geoip2_data_country_code` variable and the [`map`](https://nginx.org/en/docs/http/ngx_http_map_module.html) module, you can create another variable whose value will be the closest server basing on a continent location:

   ```text
   #...
   map $geoip2_data_country_code $nearest_server {
       default default {};
       EU      eu;
       NA      na;
       AS      as;
       AF      af;
   #...
   ```

   Then you can choose an upstream server basing on the value passed in the `$nearest_server` variable:

   ```text
   #...
   server {
       listen 12346;
       proxy_pass $nearest_server;
   }
    upstream eu {
       server eu1.example.com:12345;
       server eu2.example.com:12345;
   }
   upstream na {
       server na1.example.com:12345;
       server na2.example.com:12345;
   }
   #...
   ```

   If the continent is Europe, then the value of the `$nearest_server` will be `eu`, and the connection will be passed to the `eu` upstream via the [`proxy_pass`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) directive.

### Complete Example

This example can be applied in both the [`http`](https://nginx.org/en/docs/http/ngx_http_core_module.html#http) and [`stream`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#stream) contexts.

```text
 # can be either "http {" or "stream {"
    #...
    geoip2 GeoIP2/GeoLite2-Country.mmdb;
    geoip2 GeoIP2/GeoLite2-City.mmdb;
    map $geoip2_data_country_code $nearest_server {
        default default {};
        EU      eu;
        NA      na;
        AS      as;
        AF      af;
    server {
        listen 12346;
        proxy_pass $nearest_server;
    }
     upstream eu {
        server eu1.example.com:12345;
        server eu2.example.com:12345;
    }
    upstream na {
        server na1.example.com:12345;
        server na2.example.com:12345;
    }
}
```

In this example, the IP address will be checked in the `GeoLite2-Country.mmdb` and `GeoLite2-City.mmdb` databases, the result will be written to the `geoip2_data_country_code` variable. NGINX Plus will match the value of the variable against values in the [`map`](https://nginx.org/en/docs/http/ngx_http_map_module.html#map) directive and white the result in the custom variable, in our example `$nearest_server`. Basing on the value of the `$nearest_server`, the [`proxy_pass`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) directive will choose a corresponding upstream server.  


