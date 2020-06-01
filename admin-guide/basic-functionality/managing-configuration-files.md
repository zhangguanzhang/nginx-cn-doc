# Creating NGINX Plus and NGINX Configuration Files

NGINX和NGINX Plus与其他服务类似，因为它们使用以特定格式编写的基于文本的配置文件。 默认情况下，该文件名为 **nginx.conf** ，对于NGINX Plus，该文件位于 **/etc/nginx** 目录。(对于开源版NGINX，其位置取决于用于安装NGINX的软件包系统和操作系统。它通常是 **/usr/local/nginx/conf**, **/etc/nginx**, or **/usr/local/etc/nginx** 其中之一)

### Directives(指令)

The configuration file consists of _directives_ and their parameters. Simple \(single‑line\) directives each end with a semicolon. Other directives act as “containers” that group together related directives, enclosing them in curly braces \( `{}` \); these are often referred to as _blocks_. Here are some examples of simple directives.

```text
user             nobody;
error_log        logs/error.log notice;
worker_processes 1;
```

### Feature-Specific Configuration Files

为了使配置更易于维护，我们建议您将其拆分为一组功能特定的文件，这些文件存储在 **/etc/nginx/conf.d** 目录中，并在 **nginx.conf** 中使用[`include`](https://nginx.org/en/docs/ngx_core_module.html#include)指令以引用特定于功能的文件的内容。

```text
include conf.d/http;
include conf.d/stream;
include conf.d/exchange-enhanced;
```

### Contexts(上下文)

一些顶级指令（称为 _contexts_ ）将适用于不同流量类型的指令分组在一起：

* [`events`](https://nginx.org/en/docs/ngx_core_module.html#events) – 常规连接处理
* [`http`](https://nginx.org/en/docs/http/ngx_http_core_module.html#http) – HTTP流
* [`mail`](https://nginx.org/en/docs/mail/ngx_mail_core_module.html#mail) – Mail流
* [`stream`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#stream) – TCP and UDP流

放在这些上下文之外的指令被称为在 _main_ 上下文中。

#### Virtual Servers(虚拟主机)

在每个流量处理上下文中，都包含一个或多个`server`块来定义 _virtual servers_ ，它们控制请求的处理。 您可以包含在`server`上下文中的指令取决于流量类型.

对于HTTP流量(`http` 上下文中), 每一个[`server`](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) 指令控制特定资源请求的处理 域或IP地址。 服务器上下文中的一个或多个[`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)上下文定义了如何处理特定的URI集.

对于mail和TCP/UDP流([`mail`](https://nginx.org/en/docs/mail/ngx_mail_core_module.html)和[`stream`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html) 上下文)， `server` 指令分别控制到达特定 TCP 端口或者 UNIX socket 的通信处理.

#### 带有多个上下文的示例配置文件

The following configuration illustrates the use of contexts.

```text
user nobody; # 一个在 'main' context的Directives

events {
    # 连接处理的配置
}

http {
    # Configuration specific to HTTP and affecting all virtual servers  

    server {
        # 配置一个HTTP virtual server
        location /one {
            # configuration for processing URIs starting with '/one'
        }
        location /two {
            # configuration for processing URIs starting with '/two'
        }
    } 
    
    server {
        # 配置第二个HTTP virtual server
    }
}

stream {
    # 配置TCP/UDP的virtual servers
    server {
        # configuration of TCP virtual server 1 
    }
}
```

#### Inheritance

通常 一个 _child_ context – 包含在另一个 context (它的 _parent_) – 继承父级包含的指令的设置. 一些指令可以出现在多个上下文中，在这种情况下，您可以通过在子上下文中包含指令来重写从父级继承的设置. 有关示例，请参见[`proxy_set_header`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header) directive.

### Reloading Configuration

要使对配置文件的更改生效，必须重新加载配置文件. 你可以重启 `nginx` 进程或者发送 `reload` 信号来应用新的配置文件，而不会中断当前请求的处理. For details, see [Controlling NGINX Processes at Runtime](https://docs.nginx.com/nginx/admin-guide/basic-functionality/runtime-control/).

使用NGINX Plus, 您可以跨上游组中的服务器动态地重新配置[load balancing](https://docs.nginx.com/nginx/admin-guide/load-balancer/dynamic-configuration-api/)而无需重新加载配置。您还可以使用NGINX Plus API和key‑value store来动态控制访问，例如[based on client IP address](https://docs.nginx.com/nginx/admin-guide/security-controls/blacklisting-ip-addresses/).