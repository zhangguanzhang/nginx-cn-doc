# Phusion Passenger Open Source

Deploy and administer applications written in Node.js, Python, and Ruby.

### Installation Instructions

1. Install the Phusion Passenger Open Source module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-passenger
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-passenger
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-passenger
   ```

2. Put the following directive in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ngx_http_passenger_module.so;
   ```

3. Perform additional configuration as required by the [module](https://www.phusionpassenger.com/library/install/nginx/).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

