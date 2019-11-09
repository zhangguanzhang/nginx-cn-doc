# Headers-More

Set and clear input and output headers, extending the NGINX core [Headers](https://nginx.org/en/docs/http/ngx_http_headers_module.html) module.

### Installation Instructions

1. Install the Headers-More module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-headers-more
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-headers-more
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-headers-more
   ```

2. Put the following directive in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ngx_http_headers_more_filter_module.so;
   ```

3. Perform additional configuration as required by the [module](https://github.com/openresty/headers-more-nginx-module).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

