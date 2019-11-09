# NGINX WAF

Protect against Layer 7 attacks such as SQLi, XSS, CSRF, LFI, RFI, and more. The NGINX web application firewall \(WAF\) is built on ModSecurity 3.0.

### Installation Instructions

1. Install the NGINX WAF module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-modsecurity
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-modsecurity
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-modsecurity
   ```

2. Put the following directive in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ngx_http_modsecurity_module.so;
   ```

3. Perform additional configuration as required by the [module](https://github.com/SpiderLabs/ModSecurity/wiki/Reference-Manual).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

