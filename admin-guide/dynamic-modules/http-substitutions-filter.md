# HTTP Substitutions Filter

Make text substitutions in response bodies, using both regular expressions and fixed strings, in this filter module.

### Installation Instructions

1. Install the HTTP Substitutions Filter module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-subs-filter
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-subs-filter
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-subs-filter
   ```

2. Put the following directive in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ngx_http_subs_filter_module.so;
   ```

3. Perform additional configuration as required by the [module](https://github.com/yaoweibin/ngx_http_substitutions_filter_module).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

