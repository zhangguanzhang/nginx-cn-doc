# Perl

Implement location and variable handlers in Perl, and insert Perl calls into Server Side Includes \(SSI\).

### Installation Instructions

1. Install the Perl module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-perl
   ```

   For Debian/Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-perl
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-perl
   ```

2. Put the following directive in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ngx_http_perl_module.so;
   ```

3. Perform additional configuration as required by the [module](https://nginx.org/en/docs/http/ngx_http_perl_module.html).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

