# njs Scripting Language

Integrate [njs](https://nginx.org/en/docs/njs/) code into the NGINX event‑processing model for HTTP and TCP/UDP traffic.

### Installation Instructions

1. Install the NGINX JavaScript module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-njs
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-njs
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-njs
   ```

2. Put both of the following directives in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ngx_http_js_module.so;
   load_module modules/ngx_stream_js_module.so;
   ```

3. Perform additional configuration as required by the [module](https://www.nginx.com/blog/introduction-nginscript/).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

