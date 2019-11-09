# Image-Filter

Crop, resize, rotate, and perform other transformations on GIF, JPEG, and PNG images.

### Installation Instructions

1. Install the Image-Filter module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-image-filter
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-image-filter
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-image-filter
   ```

2. Put the following directive in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ngx_http_image_filter_module.so;
   ```

3. Perform additional configuration as required by the [module](https://nginx.org/en/docs/http/ngx_http_image_filter_module.html).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

