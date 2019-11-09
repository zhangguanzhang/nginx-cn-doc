# Creating NGINX Plus and NGINX Configuration Files

NGINX and NGINX Plus are similar to other services in that they use a text‑based configuration file written in a particular format. By default the file is named **nginx.conf** and for NGINX Plus is placed in the **/etc/nginx** directory. \(For open source NGINX, the location depends on the package system used to install NGINX and the operating system. It is typically one of **/usr/local/nginx/conf**, **/etc/nginx**, or **/usr/local/etc/nginx**.\)

### Directives

The configuration file consists of _directives_ and their parameters. Simple \(single‑line\) directives each end with a semicolon. Other directives act as “containers” that group together related directives, enclosing them in curly braces \( `{}` \); these are often referred to as _blocks_. Here are some examples of simple directives.

```text
user             nobody;
error_log        logs/error.log notice;
worker_processes 1;
```

### Feature-Specific Configuration Files

To make the configuration easier to maintain, we recommend that you split it into a set of feature‑specific files stored in the **/etc/nginx/conf.d** directory and use the [`include`](https://nginx.org/en/docs/ngx_core_module.html#include) directive in the main **nginx.conf** file to reference the contents of the feature‑specific files.

```text
include conf.d/http;
include conf.d/stream;
include conf.d/exchange-enhanced;
```

### Contexts\(上下文\)

A few top‑level directives, referred to as _contexts_, group together the directives that apply to different traffic types:

* [`events`](https://nginx.org/en/docs/ngx_core_module.html#events) – General connection processing
* [`http`](https://nginx.org/en/docs/http/ngx_http_core_module.html#http) – HTTP traffic
* [`mail`](https://nginx.org/en/docs/mail/ngx_mail_core_module.html#mail) – Mail traffic
* [`stream`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#stream) – TCP and UDP traffic

Directives placed outside of these contexts are said to be in the _main_ context.

#### Virtual Servers

In each of the traffic‑handling contexts, you include one or more `server` blocks to define _virtual servers_ that control the processing of requests. The directives you can include within a `server` context vary depending on the traffic type.

For HTTP traffic \(the `http` context\), each [`server`](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) directive controls the processing of requests for resources at particular domains or IP addresses. One or more [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) contexts in a `server` context define how to process specific sets of URIs.

For mail and TCP/UDP traffic \(the [`mail`](https://nginx.org/en/docs/mail/ngx_mail_core_module.html) and [`stream`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html) contexts\) the `server` directives each control the processing of traffic arriving at a particular TCP port or UNIX socket.

#### Sample Configuration File with Multiple Contexts

The following configuration illustrates the use of contexts.

```text
user nobody; # a directive in the 'main' context

events {
    # configuration of connection processing
}

http {
    # Configuration specific to HTTP and affecting all virtual servers  

    server {
        # configuration of HTTP virtual server 1       
        location /one {
            # configuration for processing URIs starting with '/one'
        }
        location /two {
            # configuration for processing URIs starting with '/two'
        }
    } 
    
    server {
        # configuration of HTTP virtual server 2
    }
}

stream {
    # Configuration specific to TCP/UDP and affecting all virtual servers
    server {
        # configuration of TCP virtual server 1 
    }
}
```

#### Inheritance

In general, a _child_ context – one contained within another context \(its _parent_\) – inherits the settings of directives included at the parent level. Some directives can appear in multiple contexts, in which case you can override the setting inherited from the parent by including the directive in the child context. For an example, see the [`proxy_set_header`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header) directive.

### Reloading Configuration

For changes to the configuration file to take effect, it must be reloaded. You can either restart the `nginx` process or send the `reload` signal to upgrade the configuration without interrupting the processing of current requests. For details, see [Controlling NGINX Processes at Runtime](https://docs.nginx.com/nginx/admin-guide/basic-functionality/runtime-control/).

With NGINX Plus, you can dynamically reconfigure [load balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/) across the servers in an upstream group without reloading the configuration. You can also use the NGINX Plus API and key‑value store to dynamically control access, for example [based on client IP address](https://docs.nginx.com/nginx/admin-guide/security-controls/blacklisting-ip-addresses/).

