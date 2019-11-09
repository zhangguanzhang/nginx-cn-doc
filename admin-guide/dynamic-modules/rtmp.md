# RTMP

Stream Real-Time Messaging Protocol \(RTMP\), Apple HTTP Live Streaming \(HLS\), and Dynamic Adaptive Streaming over HTTP \(DASH\) video formats.

### Installation Instructions

1. Install the RTMP Media Streaming module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-rtmp
   ```

   For Debian and Ubuntu:

   ```text
   $ apt-get install nginx-plus-module-rtmp
   ```

   For SLES:

   ```text
   $ zypper install nginx-plus-module-rtmp
   ```

2. Put the following directive in the top-level \(“main”\) context of the main NGINX Plus configuration file, **/etc/nginx/nginx.conf**:

   ```text
   load_module modules/ngx_rtmp_module.so;
   ```

3. Perform additional configuration as required by the [module](https://github.com/arut/nginx-rtmp-module).
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

