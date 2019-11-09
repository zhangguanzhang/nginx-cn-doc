# Encrypted-Session

This dynamic module provides encryption and decryption support for NGINX variables based on AES-256 with MAC. It is usually used with the [Set-Misc](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/set-misc/) dynamic module and the NGINX [rewrite](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html) module.

### Installation Instructions

1. Install the Encrypted-Session module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-encrypted-session
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-encrypted-session
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-encrypted-session
   ```

2. Put the following directive in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ndk_http_module.so;
   load_module modules/ngx_http_encrypted_session_module.so;
   ```

   **Note:** The directives must be in this order.

3. Perform additional configuration as required by the [module](https://github.com/openresty/encrypted-session-nginx-module).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

