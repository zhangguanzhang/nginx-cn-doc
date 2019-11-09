# OpenTracing

Instrument NGINX with OpenTracing-compliant requests for a range of distributed tracing services, such as Zipkin, Jaeger and Datadog. Note that the opentracing module provides the framework for recording traces; you will also need to install a service-specific tracing module. This module \(“tracer”\) pushes traces to the collector and analyser provided by that service.

### Installation Instructions

1. Install the OpenTracing module.

   For Amazon Linux, CentOS, Oracle Linux, and RHEL:

   ```text
   $ yum install nginx-plus-module-opentracing
   ```

   For Debian and Ubuntu:

   ```text
    $ apt-get install nginx-plus-module-opentracing
   ```

2. Put the following directive in the top‑level \(“main”\) context or the main NGINX Plus configuration file, /etc/nginx/nginx.conf:

   ```text
   load_module modules/ngx_http_opentracing_module.so;
   ```

3. Perform additional configuration as required by the [module](https://github.com/opentracing-contrib/nginx-opentracing). You will also need to [install a tracer](https://github.com/opentracing-contrib/nginx-opentracing#building-from-source) \(“portable binary plugin”\) for your selected service
4. Reload NGINX Plus to enable the module:

   ```text
   $ nginx -t && nginx -s reload
   ```

