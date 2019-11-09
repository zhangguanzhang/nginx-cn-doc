# Load Balancing Node.js Application Servers with NGINX Open Source and NGINX Plus



This deployment guide explains how to use NGINX Open Source and NGINX Plus to load balance HTTP and HTTPS traffic across a pool of Node.js application servers. The detailed instructions in this guide apply to both cloud‑based and on‑premises deployments of Node.js.

* [About NGINX and NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#about-nginx)
* [About Node.js](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#about-nodejs)
* [Prerequisites and System Requirements](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#prereqs)
* [Configuring an SSL/TLS Certificate for Client Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#tls-certificate)
* [Creating and Modifying Configuration Files](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#config-files)
* [Configuring Basic Load Balancing with NGINX or NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#basic)
  * [Configuring Virtual Servers for HTTP and HTTPS Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#virtual-servers)
  * [Configuring Basic Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#load-balancing-basic)
  * [Configuring Basic Session Persistence](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#session-persistence-basic)
  * [Configuring Load Balancing of WebSocket Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#websocket)
  * [Configuring Content Caching](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#caching)
  * [Configuring HTTP/2 Support](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#http2)
  * [Full Configuration for Basic Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#full-configuration-basic)
* [Configuring Enhanced Load Balancing with NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#enhanced)
  * [Configuring Advanced Session Persistence](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#session-persistence-advanced)
  * [Configuring Application Health Checks](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#health-checks)
  * [Enabling Live Activity Monitoring](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#live-activity-monitoring)
  * [Enabling Dynamic Reconfiguration of Upstream Groups](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#reconfiguration)
  * [Full Configuration for Enhanced Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#full-configuration-enhanced)
* [Resources](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#resources)

### About NGINX Open Source and NGINX Plus

[NGINX Open Source](https://nginx.org/en) is an open source web server and reverse proxy that has grown in popularity in recent years because of its scalability, outstanding performance, and small footprint. NGINX Open Source was first created to solve the C10K problem \(serving 10,000 simultaneous connections on a single web server\). NGINX Open Source’s features and performance have made it a staple of high‑performance sites – now powering [the majority of the 100,000 busiest websites in the world](https://w3techs.com/technologies/cross/web_server/ranking).

[NGINX Plus](https://www.nginx.com/products/nginx) is the commercially supported version of NGINX Open Source. NGINX Plus is a complete application delivery platform, extending the power of NGINX Open Source with a host of enterprise‑ready capabilities that enhance a Node.js deployment and are instrumental to building web applications at scale:

* [Full‑featured HTTP, TCP, and UDP load balancing](https://www.nginx.com/products/nginx/load-balancing/)
* [Intelligent session persistence](https://www.nginx.com/products/nginx/load-balancing/#session-persistence)
* [High‑performance reverse proxy](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/web-server/reverse-proxy.md)
* [Caching and offload of dynamic and static content](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/content-cache/content-caching.md)
* [Adaptive streaming to deliver audio and video to any device](https://www.nginx.com/products/nginx/streaming-media/)
* [Application-aware health checks](https://www.nginx.com/products/nginx/load-balancing/#health-checks) and [high availability](https://www.nginx.com/products/nginx/high-availability/)
* [Advanced activity monitoring available via a dashboard or API](https://www.nginx.com/products/nginx/live-activity-monitoring/)
* [Management and real‑time configuration changes with DevOps‑friendly tools](https://www.nginx.com/products/nginx/load-balancing/#load-balancing-api)

### About Node.js

[Node.js ](https://nodejs.org/en/about/)is a JavaScript runtime built on [Chrome’s V8 JavaScript engine](https://developers.google.com/v8/). Node.js uses an event‑driven, non‑blocking I/O model that makes it lightweight and efficient. The package ecosystem for Node.js, [`npm`](https://www.npmjs.com/), is the largest ecosystem of open source libraries in the world.

To download the Node.js software and get installation instructions, visit the [Node.js](https://nodejs.org/en/download/) website.

The information in this deployment guide applies equally to open source Node.js software and commercially supported Node.js frameworks.

### Prerequisites and System Requirements

* A Node.js application server installed and configured on a physical or virtual system.
* A Linux system to host NGINX Open Source or NGINX Plus. To avoid potential conflicts with other applications, we recommend you install NGINX Plus on a fresh physical or virtual system. For the list of Linux distributions supported by NGINX Plus, see [NGINX Plus Technical Specifications](https://docs.nginx.com/nginx/technical-specs/).
* NGINX Open Source or NGINX Plus installed on the physical or virtual system. Some features are available only with [NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#enhanced), including sophisticated session persistence, application health checks, live activity monitoring, and dynamic reconfiguration of upstream groups. For installation instructions for both products, see the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/installing-nginx/index.rst).

The instructions assume you have basic Linux system administration skills, including the following. Full instructions are not provided for these tasks.

* Configuring and deploying a Node.js application
* Installing Linux software from vendor‑supplied packages
* Editing configuration files
* Copying files between a central administrative system and Linux servers
* Running basic commands to start and stop services
* Reading log files

#### About Sample Values and Copying of Text

* `example.com` is used as a sample domain name \(in key names and configuration blocks\). Replace it with your organization’s name.
* Many NGINX Open Source and NGINX Plus configuration blocks in this guide list two sample Node.js application servers with IP addresses 192.168.33.11 and 192.168.33.12. Replace these addresses with the IP addresses of your Node.js servers. Include a line in the configuration block for each server if you have more or fewer than two.
* For readability reasons, some commands appear on multiple lines. If you want to copy and paste them into a terminal window, we recommend that you first copy them into a text editor, where you can substitute the object names that are appropriate for your deployment and remove any extraneous formatting characters that your browser might insert.
* We recommend that you do not copy text from the configuration snippets in this guide into your configuration files. For the recommended way to create configuration files, see [Creating and Modifying Configuration Files](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#config-files).

### Configuring an SSL/TLS Certificate for Client Traffic

If you plan to enable SSL/TLS encryption of traffic between NGINX Open Source or NGINX Plus and clients of your Node.js application, you need to configure a server certificate for NGINX Open Source or NGINX Plus.

* SSL/TLS support is enabled by default in all [NGINX Plus packages](https://cs.nginx.com/) and [NGINX Open Source binaries](https://nginx.org/en/linux_packages.html) provided by NGINX, Inc.
* If you are compiling NGINX Open Source from source, include the `--with-http_ssl_module` parameter to enable SSL/TLS support for HTTP traffic \(the corresponding parameter for TCP/UDP is `--with-stream_ssl_module`, and for email is `--with-;mail_ssl_module`, but this guide does not cover those protocol types\).
* If using binaries from other providers, consult the provider documentation to determine if they support SSL/TLS.

There are several ways to obtain a server certificate, including the following. For your convenience, step-by-step instructions are provided for the second and third options.

* If you already have an SSL certificate for NGINX Open Source or NGINX Plus installed on another UNIX or Linux system \(including systems running Apache HTTP Server\), copy it to the **/etc/nginx/ssl** directory on the NGINX Open Source or NGINX Plus server.
* Generate a self‑signed certificate as described in [Generating a Self‑Signed Certificate](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#certificate-self-signed) below. This is sufficient for testing scenarios, but clients of production deployments generally require a certificate signed by a certificate authority \(CA\).
* Request a new certificate from a CA or your organization’s security group, as described in [Generating a Certificate Request](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#certificate-request) below.

For more details on SSL/TLS termination, see the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/security-controls/terminating-ssl-http.md).

#### Generating a Self-Signed Certificate

Generate a public‑private key pair and a self‑signed server certificate in PEM format that is based on them.

1. Log in as the root user on a machine that has the `openssl` software installed.
2. Generate the key pair in PEM format \(the default\). To encrypt the private key, include the `-des3` parameter. \(Other encryption algorithms are available, listed on the man page for the [`genrsa`](https://www.openssl.org/docs/manmaster/man1/genrsa.html) command.\) You are prompted for the passphrase used as the basis for encryption.

   ```text
   root# openssl genrsa -des3 -out ~/private-key.pem 2048
   Generating RSA private key ...
   Enter pass phrase for private-key.pem:
   ```

3. Create a backup of the key file in a secure location. If you lose the key, the certificate becomes unusable.

   ```text
   root# cp ~/private-key.pem secure-dir/private-key.pem.backup
   ```

4. Generate the certificate. Include the `-new` and `-x509` parameters to make a new self‑signed certificate. Optionally include the `-days` parameter to change the key’s validity lifetime from the default of 30 days \(10950 days is about 30 years\). Respond to the prompts with values appropriate for your testing deployment.

   ```text
   root# openssl req -new -x509 -key ~/private-key.pem -out ~/self-cert.pem \
                      -days 10950
   ```

5. Copy or move the certificate file and associated key files to the **/etc/nginx/ssl** directory on the NGINX Open Source or NGINX Plus server.

#### Generating a Certificate Request

1. Log in as the root user on a machine that has the `openssl` software installed.
2. Create a private key to be packaged in the certificate.

   ```text
   root# openssl genrsa -out ~/example.com.key 2048
   ```

3. Create a backup of the key file in a secure location. If you lose the key, the certificate becomes unusable.

   ```text
   root# cp ~/example.com.key <SECURE-DIR>/example.com.key.backup
   ```

4. Create a Certificate Signing Request \(CSR\) file.

   ```text
   root# openssl req -new -sha256 -key ~/example.com.key -out ~/example.com.csr
   ```

5. Request a certificate from a CA or your internal security group, providing the CSR file \(**example.com.csr**\). As a reminder, never share private keys \(**.key** files\) directly with third parties.

   The certificate needs to be PEM format rather than in the Windows‑compatible PFX format. If you request the certificate from a CA website yourself, choose NGINX or Apache \(if available\) when asked to select the server platform for which to generate the certificate.

6. Copy or move the certificate file and associated key files to the **/etc/nginx/ssl** directory on the NGINX Plus server.

### Creating and Modifying Configuration Files

To reduce errors, this guide has you copy directives from files provided by NGINX, Inc. into your configuration files, instead of using a text editor to type in the directives yourself. Then you go through the sections in this guide \(starting with [Configuring Virtual Servers for HTTP and HTTPS Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#virtual-servers)\) to learn how to modify the directives as required for your deployment.

As provided, there is one file for basic load balancing \(with NGINX Open Source or NGINX Plus\) and one file for enhanced load balancing \(with NGINX Plus\). If you are installing and configuring NGINX Open Source or NGINX Plus on a fresh Linux system and using it only to load balance Node.js traffic, you can use the provided file as your main configuration file, which by convention is called **/etc/nginx/nginx.conf**.

We recommend, however, that instead of a single configuration file you use the scheme that is set up automatically when you install an NGINX Plus package, especially if you already have an existing NGINX Open Source or NGINX Plus deployment or plan to expand your use of NGINX Open Source or NGINX Plus to other purposes in future. In the conventional scheme, the main configuration file is still called **/etc/nginx/nginx.conf**, but instead of including all directives in it, you create separate configuration files for different functions and store the files in the **/etc/nginx/conf.d** directory. You then use the `include` directive in the appropriate contexts of the main file to read in the contents of the function‑specific files.

If you have just installed NGINX Open Source or NGINX Plus there is a default configuration file, **default.conf**, in the **/etc/nginx/conf.d** directory. This configuration defined there is not appropriate for the deployment described in this guide, but you want to leave a file with that name in the directory so it does not get replaced with a new version the next time you upgrade NGINX Open Source or NGINX Plus. To save a copy for future reference you can copy it to a new name without the **.conf** extension.

To download the complete configuration file for basic load balancing:

```text
root# cd /etc/nginx/conf.d
root# curl https://www.nginx.com/resource/conf/nodejs-basic.conf > nodejs-basic.conf
```

To download the complete configuration file for enhanced load balancing:

```text
root# cd /etc/nginx/conf.d
root# curl https://www.nginx.com/resource/conf/nodejs-enhanced.conf > nodejs-enhanced.conf
```

\(You can also access the URL in a browser and and copy the text into the indicated file.\)

**Note:** If you download both files, place only one of them in the **/etc/nginx/conf.d** directory.

To set up the conventional configuration scheme, add an `http` configuration block in the main **nginx.conf** file, if it does not already exist. \(The standard placement is below any global directives.\) Add this [`include`](https://nginx.org/en/docs/ngx_core_module.html#include) directive with the appropriate filename:

```text
http {
    include conf.d/nodejs-(basic|enhanced).conf;
}
```

Directive documentation: [`include`](https://nginx.org/en/docs/ngx_core_module.html#include)

You can also use wildcard notation to reference all files that pertain to a certain function or traffic type in the appropriate context block. For example, if you name all HTTP configuration files _function_-http.conf, this is an appropriate `include` directive:

```text
http {
    include conf.d/*-http.conf;
}
```

For reference purposes, the full configuration files are also provided in this document:

* [Full Configuration for Basic Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#full-configuration-basic)
* [Full Configuration for Enhanced Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#full-configuration-enhanced)

We recommend, however, that you do not copy text directly from this document. It does not necessarily use the same mechanisms for positioning text \(such as line breaks and white space\) as text editors do. In text copied into an editor, lines might run together and indenting of child statements in configuration blocks might be missing or inconsistent. The absence of formatting does not present a problem for NGINX Open Source or NGINX Plus, because \(like many compilers\) they ignore white space during parsing, relying solely on semicolons and curly braces as delimiters. The absence of white space does, however, make it more difficult for humans to interpret the configuration and modify it without making mistakes.

#### About Reloading Updated Configuration

We recommend that each time you complete a set of updates to the configuration, you run the `nginx -t` command to test the configuration file for syntactic validity.

```text
root# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

To tell NGINX Open Source or NGINX Plus to start using the new configuration, run one of the following commands:

```text
root# nginx -s reload
```

or

```text
root# service nginx reload
```

### Configuring Basic Load Balancing with NGINX Open Source or NGINX Plus

This section explains how to set up NGINX Open Source or NGINX Plus as a load balancer in front of two Node.js servers. The instructions in the first two sections are mandatory:

* [Configuring Virtual Servers for HTTP and HTTPS Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#virtual-servers)
* [Configuring Basic Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#load-balancing-basic)

The instructions in the remaining sections are optional, depending on the requirements of your application:

* [Configuring Basic Session Persistence](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#session-persistence-basic)
* [Configuring Load Balancing of WebSocket Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#websocket)
* [Configuring Content Caching](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#caching)
* [Configuring HTTP/2 Support](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#http2)

The complete configuration file appears in [Full Configuration for Basic Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#full-configuration-basic).

If you are using NGINX Plus, you can configure additional enhanced features after you complete the configuration of basic load balancing. See [Configuring Enhanced Load Balancing with NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#enhanced).

#### Configuring Virtual Servers for HTTP and HTTPS Traffic

These directives define virtual servers for HTTP and HTTPS traffic in separate `server` blocks in the top‑level `http` configuration block. All HTTP requests are redirected to the HTTPS server.

1. Configure a `server` block that listens for requests for **https://example.com** received on port 443.

   The `ssl_certificate` and `ssl_certificate_key` directives are required; substitute the names of the certificate and private key you chose in [Configuring an SSL/TLS Certificate for Client Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#tls-certificate).

   The other directives are optional but recommended.

   ```text
   # in the 'http' block
   server {
       listen 443 ssl;
       server_name example.com;
    
       ssl_certificate           /etc/nginx/ssl/<certificate-name>;
       ssl_certificate_key       /etc/nginx/ssl/<private-key>;
       ssl_session_cache         shared:SSL:1m;
       ssl_prefer_server_ciphers on;
    }
   ```

   Directive documentation: [`listen`](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen), [`server`](https://nginx.org/en/docs/http/ngx_http_core_module.html#server), [`server_name`](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_name), [`ssl_certificate`](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate), [`ssl_certificate_key`](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate_key), [`ssl_prefer_server_ciphers`](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_prefer_server_ciphers), [`ssl_session_cache`](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_session_cache)

2. Configure a `server` block that permanently redirects requests received on port 80 for **http://example.com** to the HTTPS server, which is defined in the next step.

   If you’re not using SSL/TLS for client connections, omit the `location` block. When instructed in the remainder of this guide to add directives to the `server` block for HTTPS traffic, add them to this block instead.

   ```text
   # in the 'http' block
   server {
        listen 80;
        server_name example.com;
    
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header Connection "";
         
        # Redirect all HTTP requests to HTTPS
        location / {
            return 301 https://$server_name$request_uri;
        }
   }   
   ```

   Directive documentation: [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location), [`proxy_http_version`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_http_version), [`proxy_set_header`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header), [`return`](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#return)

For more information on configuring SSL/TLS, see the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/security-controls/terminating-ssl-http.md) and the reference documentation for the HTTP [SSL/TLS module](https://nginx.org/en/docs/http/ngx_http_ssl_module.html).

#### Configuring Basic Load Balancing

To configure load balancing, you first create a named “upstream group,” which lists the backend servers. You then set up NGINX Open Source or NGINX Plus as a reverse proxy and load balancer by referring to the upstream group in one or more `proxy_pass` directives.

1. Configure an upstream group called **nodejs** with two Node.js application servers listening on port 8080, one on IP address 192.168.33.11 and the other on 192.168.33.12.

```text
# in the 'http' block
upstream nodejs {
    server 192.168.33.11:8080;
    server 192.168.33.12:8080;
}
```

Directive documentation: [`server`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server), [`upstream`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream)

1. In the `server` block for HTTPS traffic that we created in [Configuring Virtual Servers for HTTP and HTTPS Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#virtual-servers), include these two `location` blocks:
   * The first one matches HTTPS requests in which the path starts with **/webapp/**, and proxies them to the **nodejs** upstream group we created in the previous step.
   * The second one funnels all traffic to the first `location` block, by doing a temporary redirect of all requests for **http://example.com/**.

     ```text
     # in the 'server' block for HTTPS traffic
     location /webapp/ {
         proxy_pass http://nodejs;
     }

     location = / {
         return 302 /webapp/;
     }
     ```

     Directive documentation: [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location), [`proxy_pass`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass), [`return`](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#return)

Note that these blocks handle only standard HTTPS traffic. If you want to load balance WebSocket traffic, you need to add another `location` block as described in [Configuring Proxy of WebSocket Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#websocket).

By default, NGINX Open Source and NGINX Plus use the Round Robin algorithm for load balancing among servers. The load balancer runs through the list of servers in the upstream group in order, forwarding each new request to the next server. In our example, the first request goes to 192.168.33.11, the second to 192.168.33.12, the third to 192.168.33.11, and so on. For information about the other available load‑balancing algorithms, see the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/deployment-guides/admin-guide/load-balancer/http-load-balancer/#choosing-a-load-balancing-method).

In NGINX Plus, you can also set up dynamic reconfiguration of an upstream group when the set of backend servers changes, using the Domain Name System \(DNS\) or an API; see [Enabling Dynamic Reconfiguration of Upstream Groups](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#reconfiguration).

For more information on proxying and load balancing, see [Reverse Proxy](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/web-server/reverse-proxy.md) and [Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/load-balancer/http-load-balancer.md) in the NGINX Plus Admin Guide, and the reference documentation for the HTTP [Proxy](https://nginx.org/en/docs/http/ngx_http_proxy_module.html) and [Upstream](https://nginx.org/en/docs/http/ngx_http_upstream_module.html) modules.

#### Configuring Basic Session Persistence

If your application requires basic session persistence \(also known as _sticky sessions_\), you can implement it in NGINX Open Source with the IP Hash load‑balancing algorithm. \(NGINX Plus offers a more sophisticated form of session persistence, as described in [Configuring Advanced Session Persistence](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#session-persistence-advanced).\)

With the IP Hash algorithm, for each request NGINX calculates a hash based on the client’s IP address, and associates the hash with one of the upstream servers. It sends all requests with that hash to that server, thus establishing session persistence.

If the client has an IPv6 address, the hash is based on the entire address. If it has an IPv4 address, the hash is based on just the first three octets of the address. This is designed to optimize for ISP clients that are assigned IP addresses dynamically from a subnetwork \(/24\) range. However, it is not effective in these cases:

* The majority of the traffic to your site is coming from one forward proxy or from clients on the same /24 network, because in that case IP Hash maps all clients to the same server.
* A client’s IP address can change during the session, for example when a mobile client switches from a WiFi network to a cellular one.

To configure session persistence in NGINX, add the `ip_hash` directive to the `upstream` block created in [Configuring Basic Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#load-balancing-basic):

```text
# in the 'http' block
upstream nodejs {
    ip_hash;
    server 192.168.33.11:8080;
    server 192.168.33.12:8080;
}
```

Directive documentation: [`ip_hash`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#ip_hash)

You can also use the Hash load‑balancing method for session persistence, with the hash based on any combination of text and [NGINX variables](https://nginx.org/en/docs/varindex.html) you specify. For example, you can hash on full \(four‑octet\) client IP addresses with the following configuration.

```text
# in the 'http' block
upstream nodejs {
    hash $remote_addr;
    server 192.168.33.11:8080;
    server 192.168.33.12:8080;
}
```

Directive documentation: [`hash`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#hash)

#### Configuring Load Balancing of WebSocket Traffic

The WebSocket protocol \(defined in [RFC 6455](https://tools.ietf.org/html/rfc6455)\) enables simultaneous two‑way communication over a single TCP connection between clients and servers, where each side can send data independently from the other. To initiate the WebSocket connection, the client sends a handshake request to the server, upgrading the request from standard HTTP to WebSocket. The connection is established if the handshake request passes validation, and the server accepts the request. When a WebSocket connection is created, a browser client can send data to a server while simultaneously receiving data from that server.

The Node.js app server supports WebSocket out of the box, so no additional Node.js configuration is required. If you want to use NGINX Open Source or NGINX Plus to proxy WebSocket traffic to your Node.js application servers, add the directives discussed in this section.

NGINX Open Source and NGINX Plus by default use HTTP/1.0 for upstream connections. To be proxied correctly, WebSocket connections require HTTP/1.1 along with some other configuration directives that set HTTP headers:

```text
# in the 'http' block
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

# in the 'server' block for HTTPS traffic
location /wstunnel/ {
    proxy_pass http://nodejs;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
}
```

Directive documentation: [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location), [`map`](https://nginx.org/en/docs/http/ngx_http_map_module.html#map), [`proxy_http_version`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_http_version), [`proxy_pass`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass), [`proxy_set_header`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header)

The first `proxy_set_header` directive is needed because the `Upgrade` request header is hop-by-hop; that is, the HTTP specification explicitly forbids proxies from forwarding it. This directive overrides the prohibition.

The second `proxy_set_header` directive sets the `Connection` header to a value that depends on the test in the `map` block: if the request has an `Upgrade` header, the `Connection` header is set to `upgrade`; otherwise, it is set to `close`.

For more information about proxying WebSocket traffic, see [WebSocket proxying](https://nginx.org/en/docs/http/websocket.html) and [NGINX as a WebSocket Proxy](https://www.nginx.com/blog/websocket-nginx/).

#### Configuring Content Caching

Caching responses from your Node.js app servers can both improve response time to clients and reduce load on the servers, because eligible responses are served immediately from the cache instead of being generated again on the server. There are a variety of useful directives that can be used to fine‑tune caching behavior; for a detailed discussion, see [A Guide to Caching with NGINX](https://www.nginx.com/blog/nginx-caching-guide/).

To enable basic caching of responses from the Node.js app server, add the following configuration:

1. Include the `proxy_cache_path` directive to create the local disk directory **/tmp/NGINX\_cache/** for use as a cache. The `keys_zone` parameter allocates 10 megabytes \(MB\) of shared memory for a zone called **backcache**, which is used to store cache keys and metadata such as usage timers. A 1‑MB zone can store data for about 8,000 keys.

   ```text
   # in the 'http' block
   proxy_cache_path /tmp/NGINX_cache/ keys_zone=backcache:10m;
   ```

   Directive documentation: [`proxy_cache_path`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path)

2. In the `location` block that matches HTTPS requests in which the path starts with **/webapp/**, include the `proxy_cache` directive to reference the cache created in the previous step.

   ```text
   # in the 'server' block for HTTPS traffic
   location /webapp/ {
       proxy_pass http://nodejs;
       proxy_cache backcache;
   }
   ```

   Directive documentation: [`proxy_cache`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache), [`proxy_pass`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)

For more complete information on caching, refer to the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/content-cache/content-caching.md) and the reference documentation for the HTTP [Proxy](https://nginx.org/en/docs/http/ngx_http_proxy_module.html) module.

#### Configuring HTTP/2 Support

HTTP/2 is fully supported in both NGINX 1.9.5 and later, and NGINX Plus R7 and later. As always, we recommend you run the latest version of software to take advantage of improvements and bug fixes.

* If using NGINX Open Source, note that in version 1.9.5 and later the SPDY module is completely removed from the codebase and replaced with the [HTTP/2 module](https://nginx.org/en/docs/http/ngx_http_v2_module.html). After upgrading to version 1.9.5 or later, you can no longer configure NGINX Open Source to use SPDY. If you want to keep using SPDY, you need to compile NGINX Open Source from the sources in the [NGINX 1.8.x branch](https://nginx.org/en/download.html).
* If using NGINX Plus, in R11 and later the nginx-plus package supports HTTP/2 by default, and the nginx-plus-extras package available in previous releases is deprecated by separate [dynamic modules](https://www.nginx.com/products/nginx/modules/) authored by NGINX, Inc.

  In NGINX Plus R8 through R10, the nginx-plus and nginx-plus-extras packages support HTTP/2 by default.

  In NGINX Plus R8 and later, NGINX Plus supports HTTP/2 by default, and does not support SPDY.

  If using NGINX Plus R7, you must install the nginx-plus-http2 package instead of the nginx-plus or nginx-plus-extras package.

To enable HTTP/2 support, add the `http2` parameter to the [`listen`](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive in the `server` block for HTTPS traffic that we created in [Configuring Virtual Servers for HTTP and HTTPS Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#virtual-servers), so that it looks like this:

```text
# in the 'server' block for HTTPS traffic
listen 443 ssl http2;
```

Directive documentation: [`listen`](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen)

To verify that HTTP/2 translation is working, you can use the “HTTP/2 and SPDY indicator” plug‑in available for [Google Chrome](https://chrome.google.com/webstore/detail/http2-and-spdy-indicator/mpbpobfflnpcgagjijhmgnchggcjblin?hl=en) and [Firefox](https://addons.mozilla.org/en-US/firefox/addon/http2-indicator/).

#### Full Configuration for Basic Load Balancing

The full configuration for basic load balancing appears here for your convenience. It goes in the `http` context. The complete file is available for [download](https://www.nginx.com/resource/conf/nodejs-basic.conf) from the NGINX, Inc. website.

We recommend that you do not copy text directly from this document, but instead use the method described in [Creating and Modifying Configuration Files](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#config-files) to include these directives in your configuration – add an `include` directive to the `http` context of the main **nginx.conf** file to read in the contents of /etc/nginx/conf.d/nodejs-basic.conf.

```text
proxy_cache_path /tmp/NGINX_cache/ keys_zone=backcache:10m;

map $http_upgrade $connection_upgrade {  
    default upgrade;  
    ' '     close;  
}

upstream nodejs {  
    # Use IP Hash for session persistence  
    ip_hash;

    # List of Node.js application servers  
    server 192.168.33.11:8080;  
    server 192.168.33.12:8080;  
}

server {  
    listen 80;  
    server_name example.com;

    # Redirect all HTTP requests to HTTPS  
    location / {  
        return 301 https://$server_name$request_uri;  
    }  
}

server {  
    listen 443 ssl http2;  
    server_name example.com;

    ssl_certificate           /etc/nginx/ssl/certificate-name;  
    ssl_certificate_key       /etc/nginx/ssl/private-key;  
    ssl_session_cache         shared:SSL:1m;  
    ssl_prefer_server_ciphers on;

    # Return a temporary redirect to the /webapp/ directory  
    # when user requests '/'  
    location = / {  
         return 302 /webapp/;  
    }

    # Load balance requests for /webapp/ across Node.js app servers 
    location /webapp/ {  
        proxy_pass http://nodejs;  
        proxy_cache backcache;  
    }

    # WebSocket configuration  
    location /wstunnel/ { 
        proxy_pass https://nodejs;  
        proxy_http_version 1.1;  
        proxy_set_header Upgrade $http_upgrade;  
        proxy_set_header Connection $connection_upgrade;  
    }  
}
```

### Configuring Enhanced Load Balancing with NGINX Plus

This section explains how to configure enhanced load balancing with some of the extended features in NGINX Plus.

**Note:** Before setting up the enhanced features described in this section, you must complete the instructions for basic load balancing in these two sections:

* [Configuring Virtual Servers for HTTP and HTTPS Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#virtual-servers)
* [Configuring Basic Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#load-balancing-basic)

Except as noted, all optional basic features \(described in the other subsections of [Configuring Basic Load Balancing in NGINX Open Source and NGINX Plus](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#basic)\) can be combined with the enhanced features described here.

* [Configuring Advanced Session Persistence](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#session-persistence-advanced)
* [Configuring Application Health Checks](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#health-checks)
* [Enabling Live Activity Monitoring](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#live-activity-monitoring)
* [Enabling Dynamic Reconfiguration of Upstream Groups](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#reconfiguration)

The complete configuration file appears in [Full Configuration for Enhanced Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#full-configuration-enhanced).

#### Configuring Advanced Session Persistence

NGINX Plus has more sophisticated session persistence methods than open source NGINX, implemented in three variants of the `sticky` directive. In the following example, we add the `sticky cookie` directive to the upstream group we created in [Configuring Basic Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#load-balancing-basic).

1. Remove or comment out the `ip_hash` directive, leaving only the `server` directives:

   ```text
   # in the 'http' block
   upstream nodejs {
       #ip_hash;
       server 192.168.33.11:8080;
       server 192.168.33.12:8080;
   }
   ```

   Directive documentation: [`server`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server), [`upstream`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream)

2. Configure session persistence that uses the `sticky cookie` directive.

   ```text
   # in the 'http' block
   upstream nodejs {
       zone nodejs 64k;
       server 192.168.33.11:8080;
       server 192.168.33.12:8080;
       sticky cookie srv_id expires=1h domain=.example.com path=/;
   }
   ```

   Directive documentation: [`sticky cookie`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky), [`zone`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone)

With this method, NGINX Plus adds an HTTP session cookie to the first response to a given client from the upstream group, identifying which server generated the response \(in an encoded fashion\). Subsequent requests from the client include the cookie value and NGINX Plus uses it to route the request to the same upstream server, thereby achieving session persistence.

The `zone` directive creates a shared memory zone for storing information about sessions. The amount of memory allocated – here, 64 KB – determines how many sessions can be stored at a time \(the number varies by platform\). The name assigned to the zone – here, `nodejs` – must be unique for each `sticky` directive.

The first parameter to `sticky cookie` \(in the example, `srv_id`\) sets the name of the cookie to be set or inspected. The `expires` parameter tells the browser how long the cookie is valid, here one hour. The `domain` parameter defines the domain and the `path` parameter defines the URL path for which the cookie is set.

For more information about session persistence, see the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/deployment-guides/admin-guide/load-balancer/http-load-balancer/#sticky).

#### Configuring Application Health Checks

Health checks are out‑of‑band HTTP requests sent to a server at fixed intervals. They are used to determine whether a server is responsive and functioning correctly, without requiring an actual request from a client.

Because the `health_check` directive is placed in the `location` block, we can enable different health checks for each application.

1. In the `location` block that matches HTTPS requests in which the path starts with **/webapp/** \(created in [Configuring Basic Load Balancing](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#load-balancing-basic)\), add the `health_check` directive.

   Here we configure NGINX Plus to send an out‑of‑band request for the top‑level URI **/** \(slash\) to each of the servers in the **nodejs** upstream group every 5 seconds \(the default URI and frequency\). If a server does not respond correctly, it is marked down and NGINX Plus stops sending requests to it until it passes a subsequent health check. We include the `match` parameter so we can define a nondefault set of health‑check tests \(we define them in the next step\).

```text
# in the 'server' block for HTTPS traffic
location /webapp/ {
    proxy_pass http://nodejs;
    proxy_cache backcache;
    health_check match=health_check;
}
```

Directive documentation: [`health_check`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check)

1. In the `http` context, include a `match` directive to define the tests that a server must pass to be considered functional. In this example, it must return status code `200`, the `Content-Type` response header must contain `text/html`, and the response body must match the indicated character string.

```text
# in the 'http' block
match health_check {
     status 200;
     header Content-Type ~ text/html;
     body ~ "Hello world";
}
```

Directive documentation: [`match`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#match)

1. In the **nodejs** upstream group, add the following `zone` directive as necessary \(if you configured [advanced session persistence](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#session-persistence-advanced) you already added it\). It creates a shared memory zone that stores the group’s configuration and run‑time state, which are accessible to all worker processes.

```text
# in the 'http' block
upstream nodejs {
    zone nodejs 64k;
    server 192.168.33.11:8080;
    server 192.168.33.12:8080;
    # ...
}
```

Directive documentation: [`zone`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone)

NGINX Plus also has a slow‑start feature that is a useful auxiliary to health checks. When a failed server recovers, or a new server is added to the upstream group, NGINX Plus slowly ramps up the traffic to it over a defined period of time. This gives the server time to “warm up” without being overwhelmed by more connections than it can handle as it starts up. For more information, see the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/deployment-guides/admin-guide/load-balancer/http-load-balancer/#slow_start).

For example, to set a slow‑start period of 30 seconds for your Node.js application servers, include the `slow_start` parameter to their `server` directives:

```text
# in the 'upstream' block
server 192.168.33.11:8080 slow_start=30s;
server 192.168.33.12:8080 slow_start=30s;
```

Parameter documentation: [`slow_start`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#slow_start)

For information about customizing health checks, see the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/load-balancer/http-health-check.md).

#### Enabling Live Activity Monitoring

NGINX Plus includes a live activity monitoring interface that provides key load and performance metrics in real time, including TCP metrics in NGINX Plus R6 and later. Statistics are reported through a RESTful JSON interface, making it very easy to feed the data to a custom or third‑party monitoring tool. There is also a built‑in dashboard. Follow these instructions to deploy it.

![Dashboard tab in NGINX Plus live activity monitoring dashboard](https://www.nginx.com/wp-content/uploads/2018/02/dashboard-NGINX-Plus-R14.png)

For more information about live activity monitoring, see the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/monitoring/live-activity-monitoring.md).

The quickest way to configure the module and the built‑in NGINX Plus dashboard is to download the sample configuration file from the NGINX, Inc. website and modify it as necessary. For more complete instructions, see [Live Activity Monitoring of NGINX Plus in 3 Simple Steps](https://www.nginx.com/blog/live-activity-monitoring-nginx-plus-3-simple-steps/).

1. Download the **status.conf** file to the NGINX Plus server:

   ```text
   # cd /etc/nginx/conf.d
   # curl https://www.nginx.com/resource/conf/status.conf > status.conf
   ```

2. Customize the file for your deployment as specified by comments in the file. In particular, the default settings in the file allow anyone on any network to access the dashboard. We strongly recommend that you restrict access to the dashboard with one or more of the following methods:

   * **IP address‑based access control lists \(ACLs\)**. In the sample configuration file, uncomment the `allow` and `deny` directives, and substitute the address of your administrative network for 10.0.0.0/8. Only users on the specified network can access the status page.

     ```text
     allow 10.0.0.0/8;
     deny all;
     ```

     Directive documentation: [`allow` and `deny`](https://nginx.org/en/docs/http/ngx_http_access_module.html)

   * **HTTP basic authentication**. In the sample configuration file, uncomment the `auth_basic` and `auth_basic_user_file` directives and add user entries to the **/etc/nginx/users** file \(for example, by using an [`htpasswd` generator](https://httpd.apache.org/docs/2.4/programs/htpasswd.html)\). If you have an Apache installation, another option is to reuse an existing **htpasswd** file.

   ```text
   auth_basic on;
   auth_basic_user_file /etc/nginx/users;
   ```

   Directive documentation: [`auth_basic`](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html#auth_basic), [`auth_basic_user_file`](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html#auth_basic_user_file)

   * **Client certificates**, which are part of a complete configuration of SSL/TLS. For more information, see the [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/security-controls/terminating-ssl-http.md) and the documentation for the [HTTP SSL/TLS module](https://nginx.org/en/docs/http/ngx_http_ssl_module.html).
   * **Firewall**. Configure your firewall to disallow outside access to the port for the dashboard \(8080 in the sample configuration file\).

3. In the **nodejs** upstream group, include the `zone` directive as necessary \(if you configured [advanced session persistence](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#session-persistence-advanced) or [application health checks](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#health-checks), you already added it\). It creates a shared memory zone that stores the group’s configuration and run‑time state, which are accessible to all worker processes.

```text
# in the 'http' block
upstream nodejs {
    zone nodejs 64k;
    server 192.168.33.11:8080;
    server 192.168.33.12:8080;
    # ...
}
```

Directive documentation: [`zone`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone)

1. In the `server` block for HTTPS traffic \(created in [Configuring Virtual Servers for HTTP and HTTPS Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#virtual-servers)\), add the `status_zone` directive:

```text
# in the 'server' block for HTTPS traffic
status_zone nodejs_server;
```

Directive documentation: [`status_zone`](https://nginx.org/en/docs/http/ngx_http_status_module.html#status_zone)

When you reload the NGINX Plus configuration file, for example by running the `nginx -s reload` command, the NGINX Plus dashboard is available immediately at http://_nginx-plus-server-address_:8080.

#### Enabling Dynamic Reconfiguration of Upstream Groups

With NGINX Plus, you can reconfigure load‑balanced server groups \(both HTTP and TCP/UDP\) dynamically using either DNS or the NGINX Plus API introduced in NGINX Plus R13. See the NGINX Plus Admin Guide for a more detailed discussion of the [DNS](https://docs.nginx.com/nginx/deployment-guides/admin-guide/load-balancer/http-load-balancer/#configuring-http-load-balancing-using-dns) and [API](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/load-balancer/dynamic-configuration-api.md) methods.

**Configuring the API Method**

To enable dynamic reconfiguration of your upstream group of Node.js app servers using the NGINX Plus API, you need to grant secured access to it. You can use the API to add or remove servers, dynamically alter their weights, and set their status as `primary`, `backup`, or `down`.

1. Include the `zone` directive in the **nodejs** upstream group to create a shared memory zone for storing the group’s configuration and run‑time state, which makes the information available to all worker processes. \(If you configured [advanced session persistence](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#session-persistence-advanced), [application health checks](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#health-checks), or [live activity monitoring](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#live-activity-monitoring), you already made this change.\)

   ```text
   # in the 'http' block
   upstream nodejs {
       zone nodejs 64k;
       server 192.168.33.11:8080;
       server 192.168.33.12:8080;
       # ...
   }
   ```

   Directive documentation: [`zone`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone)

2. In the `server` block for HTTPS traffic \(created in [Configuring Virtual Servers for HTTP and HTTPS Traffic](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#virtual-servers)\), add a new `location` block for the NGINX Plus API, which enables dynamic reconfiguration among other features. It contains the `api` directive \(**api** is also the conventional name for the location, as used here\).

   \(If you configured [live activity monitoring](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#live-activity-monitoring) by downloading the **status.conf** file, it already includes this block.\)

   We strongly recommend that you restrict access to the location so that only authorized administrators can access the NGINX Plus API. The [`allow` and `deny`](https://nginx.org/en/docs/http/ngx_http_access_module.html) directives in the following example permit access only from the localhost address \(127.0.0.1\).

   ```text
   # In the 'server' block for HTTPS traffic
   location /api {
       api write=on;
       allow 127.0.0.1;
       deny all;
   }
   ```

   Directive documentation: [`allow` and `deny`](https://nginx.org/en/docs/http/ngx_http_access_module.html), [`api`](https://nginx.org/en/docs/http/ngx_http_api_module.html#api)

**Configuring the DNS Method**

In the `http` block, add the `resolver` directive pointing to your DNS server and then add the `resolve` parameter to the `server` directive in the **nodejs** `upstream` block, which instructs NGINX Plus to periodically re‑resolve the domain name \(**example.com** here\) with DNS:

```text
# In the 'http' block
resolver <IP-address-of-DNS-server>;

upstream nodejs {
    zone nodejs 64k;
    server example.com resolve;
}
```

Directive and parameter documentation: [`resolve`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#resolve), [`resolver`](https://nginx.org/en/docs/http/ngx_http_core_module.html#resolver)

[NGINX Plus Release 9](https://www.nginx.com/blog/nginx-plus-r9-released/#dns-srv) and later can also use the additional information in DNS `SRV` records, such as the port number. Include the `service` parameter to the `server` directive, along with the `resolve` parameter:

```text
# In the 'http' block
resolver <IP-address-of-DNS-server>;

upstream nodejs {
    zone nodejs 64k;
    server example.com service=http resolve;
}
```

Parameter documentation: [`service`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#service)

#### Full Configuration for Enhanced Load Balancing

The full configuration for enhanced load balancing appears here for your convenience. It goes in the `http` context. The complete file is available for [download](https://www.nginx.com/resource/conf/nodejs-enhanced.conf) from the NGINX, Inc. website.

We recommend that you do not copy text directly from this document, but instead use the method described in [Creating and Modifying Configuration Files](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#config-files) to include these directives in your configuration – namely, add an `include` directive to the `http` context of the main **nginx.conf** file to read in the contents of /etc/nginx/conf.d/nodejs-enhanced.conf.

**Note:** The `api` block in this configuration summary and the [downloadable](https://www.nginx.com/resource/conf/nodejs-enhanced.conf) nodejs-enhanced.conf file is for the [API method](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#reconfiguration-api) of dynamic reconfiguration. If you want to use the [DNS method](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/#reconfiguration-dns) instead, make the appropriate changes to the block. \(You can also remove or comment out the directives for the NGINX Plus API in that case, but they do not conflict with using the DNS method and enable features other than dynamic reconfiguration.\)

```text
proxy_cache_path /tmp/NGINX_cache/ keys_zone=backcache:10m;

map $http_upgrade $connection_upgrade {  
    default upgrade;  
    ''      close;  
}

match nodejs_check {  
    status 200;  
    header Content-Type ~ "text/html";  
    body ~ "Hello world";  
}

upstream nodejs { 
    # Health-monitored upstream groups must have a zone defined  
    zone nodejs 64k;

    # List of Node.js application servers  
    server 192.168.33.11:8080 slow_start=30s;  
    server 192.168.33.12:8080 slow_start=30s;

    # Session persistence using sticky cookie  
    sticky cookie srv_id expires=1h domain=.example.com path=/;  
}

server {  
    listen 80;  
    server_name example.com;

    # Redirect all HTTP requests to HTTPS  
    location / {  
        return 301 https://$server_name$request_uri;  
    }  
}

server {  
    listen 443 ssl http2;  
    server_name example.com;

    # Required for NGINX Plus to provide extended status information  
    status_zone nodejs;

    ssl_certificate            /etc/nginx/ssl/certificate-name;  
    ssl_certificate_key        /etc/nginx/ssl/private-key;  
    ssl_session_cache          shared:SSL:1m;  
    ssl_prefer_server_ciphers  on;

    # Return a 302 redirect to the /webapp/ directory  
    # when user requests '/'  
    location = / {  
        return 302 /webapp/;  
    }

    # Load balance requests for /webapp/ across Node.js app servers  
    location /webapp/ {  
        proxy_pass http://nodejs;  
        proxy_cache backcache;
        # Set up active health checks  
        health_check match=nodejs_check;  
    }

    # WebSocket configuration  
    location /wstunnel/ {  
        proxy_pass https://nodejs;  
        proxy_http_version 1.1;  
        proxy_set_header Upgrade $http_upgrade;  
        proxy_set_header Connection $connection_upgrade; 
    }

    # Secured access to the NGINX Plus API  
    location /api {  
        api write=on;
        allow 127.0.0.1; # permit access from localhost  
        deny all;        # deny access from everywhere else  
    }  
}
```

### Resources

* [NGINX Plus Overview](https://www.nginx.com/products/nginx)
* [NGINX Plus Admin Guide](https://docs.nginx.com/nginx/deployment-guides/load-balance-third-party/node-js/nginx/deployment-guides/admin-guide/index.rst)
* [NGINX Wiki](https://www.nginx.com/resources/wiki/)

[_NodeSource_](https://nodesource.com/)_, developers of N\|Solid, contributed to this deployment guide._

#### Revision History

* Version 3 \(April 2018\) – Updated information about the NGINX Plus API \(NGINX Plus R13, NGINX Open Source 1.13.4\)
* Version 2 \(May 2017\) – Update about HTTP/2 support \(NGINX Plus R11 and later\)
* Version 1 \(December 2016\) – Initial version \(NGINX Plus R11, NGINX 1.11.5\)

