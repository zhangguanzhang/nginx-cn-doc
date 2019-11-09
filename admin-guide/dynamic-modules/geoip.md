# GeoIP



Create variables based on the client IP address, using the precompiled MaxMind GeoIP databases, for both HTTP and TCP/UDP traffic.

**Note** MaxMind GeoLite Legacy databases are currently [discontinued](https://support.maxmind.com/geolite-legacy-discontinuation-notice/), MaxMind GeoIP2 or Geolite2 databases and NGINX Plus [GeoIP2 module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/geoip2/) should be used instead.

### Installation Instructions

1. Install the GeoIP module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-geoip
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-geoip
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-geoip
   ```

2. Put both of the following directives in the top‑level \(“main”\) context or the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ngx_http_geoip_module.so;
   load_module modules/ngx_stream_geoip_module.so;

   http {
       # ...
   }
   ```

3. Perform additional configuration as required by the module \([HTTP](https://nginx.org/en/docs/http/ngx_http_geoip_module.html) or [TCP/UDP](https://nginx.org/en/docs/stream/ngx_stream_geoip_module.html)\).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

### More Info

* [GeoIP2 Module Installation Instructions](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/geoip2/)
* [Restricting Access by Geographical Location](https://docs.nginx.com/nginx/admin-guide/security-controls/controlling-access-by-geoip/)
* [`ngx_http_geoip_module` Module Reference](https://nginx.org/en/docs/http/ngx_http_geoip_module.html)
* [`ngx_stream_geoip_module` Module Reference](https://nginx.org/en/docs/stream/ngx_stream_geoip_module.html)

