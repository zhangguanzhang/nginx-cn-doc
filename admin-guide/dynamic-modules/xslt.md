# XSLT

Transform XML responses using one or more XSLT stylesheets.

### Installation Instructions

1. Install the XSLT module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-xslt
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-xslt
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-xslt
   ```

2. Put the following directive in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ngx_http_xslt_module.so;
   ```

3. Perform additional configuration as required by the [module](https://nginx.org/en/docs/http/ngx_http_xslt_module.html).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

