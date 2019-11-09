# Lua

Integrate Lua co‑routines into the NGINX event‑processing model.

### Installation Instructions

1. Install the Lua module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-lua
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-lua
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-lua
   ```

2. Put both of the following directives in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ndk_http_module.so;
   load_module modules/ngx_http_lua_module.so;
   ```

   **Note:** The directives must be in this order.

3. Perform additional configuration as required by the [module](https://github.com/openresty/lua-nginx-module).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

