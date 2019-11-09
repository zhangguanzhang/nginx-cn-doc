# GeoIP2

Create variables based on the client IP address, using the precompiled [MaxMind GeoIP2](https://www.maxmind.com/en/geoip2-databases) databases.

### Module Info

MaxMind’s GeoIP2 database provides contextual data for a comprehensive profile of IP addresses, including geolocation data \(region, state, city, postal code\) and extra data \(ISP, domain, connection type\). Basing on these data, NGINX Plus will be able to perform different user differentiation strategies, for example, provide different type of content depending on a country.

### Installation Instructions

1. Install the GeoIP2 module.

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

2. Put both of the following directives in the top‑level \(“main”\) context or the main NGINX Plus configuration file, **nginx.conf**:

   ```text
   load_module modules/ngx_http_geoip2_module.so;
   load_module modules/ngx_stream_geoip2_module.so;

   http {
       # ...
   }
   ```

3. Perform additional configuration as required by the [module](https://github.com/leev/ngx_http_geoip2_module#user-content-download-maxmind-geolite2-database-optional).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

### More Info

* [Restricting Access by Geographical Location](https://docs.nginx.com/nginx/admin-guide/security-controls/controlling-access-by-geoip/)
* [MaxMind GeoIP2 Databases](https://www.maxmind.com/en/geoip2-databases)
* [MaxMind Geolite2 Free Downloadable Databases](https://dev.maxmind.com/geoip/geoip2/geolite2/)

