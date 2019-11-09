# Cookie-Flag

Set the flags `HttpOnly`, `SameSite`, and `secure` for cookies in `Set-Cookie` upstream response headers.

### Installation Instructions

1. Install the Cookie-Flag module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-cookie-flag
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-cookie-flag
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-cookie-flag
   ```

2. Put the following directive in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ngx_http_cookie_flag_filter_module.so;
   ```

3. Perform additional configuration as required by the [module](https://github.com/AirisX/nginx_cookie_flag_module).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

