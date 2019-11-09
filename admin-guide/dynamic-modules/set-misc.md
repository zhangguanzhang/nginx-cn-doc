# Set-Misc

Implement numerous additional `set_*` directives, extending the NGINX core [Rewrite](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html) module\).

### Installation Instructions

1. Install the Set-Misc module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-set-misc
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-set-misc
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-set-misc
   ```

2. Put both of the following directives in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ndk_http_module.so;
   load_module modules/ngx_http_set_misc_module.so;
   ```

   **Note:** The directives must be in this order.

3. Perform additional configuration as required by the [module](https://github.com/openresty/set-misc-nginx-module).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

