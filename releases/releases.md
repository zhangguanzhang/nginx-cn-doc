# Releases



## NGINX Plus Releases <a id="page_title"></a>

* [NGINX Plus Release 19 \(R19\)](https://docs.nginx.com/nginx/releases/#r19)
* [NGINX Plus Release 18 \(R18\)](https://docs.nginx.com/nginx/releases/#r18)
* [NGINX Plus Release 17 \(R17\)](https://docs.nginx.com/nginx/releases/#r17)
* [NGINX Plus Release 16 \(R16\)](https://docs.nginx.com/nginx/releases/#r16)
* [NGINX Plus Release 15 \(R15\)](https://docs.nginx.com/nginx/releases/#r15)
* [NGINX Plus Release 14 \(R14\)](https://docs.nginx.com/nginx/releases/#r14)
* [NGINX Plus Release 13 \(R13\)](https://docs.nginx.com/nginx/releases/#r13)
* [NGINX Plus Release 12 \(R12\)](https://docs.nginx.com/nginx/releases/#r12)
* [NGINX Plus Release 11 \(R11\)](https://docs.nginx.com/nginx/releases/#r11)
* [NGINX Plus Release 10 \(R10\)](https://docs.nginx.com/nginx/releases/#r10)
* [NGINX Plus Release 9 \(R9\)](https://docs.nginx.com/nginx/releases/#r9)
* [NGINX Plus Release 8 \(R8\)](https://docs.nginx.com/nginx/releases/#r8)
* [NGINX Plus Release 7 \(R7\)](https://docs.nginx.com/nginx/releases/#r7)
* [NGINX Plus Release 6 \(R6\)](https://docs.nginx.com/nginx/releases/#r6)
* [NGINX Plus Release 5 \(R5\)](https://docs.nginx.com/nginx/releases/#r5)
* [NGINX Plus Release 4 \(R4\)](https://docs.nginx.com/nginx/releases/#r4)
* [NGINX Plus Release 3 \(R3\)](https://docs.nginx.com/nginx/releases/#r3)
* [NGINX Plus Release 2 \(R2\)](https://docs.nginx.com/nginx/releases/#r2)
* [NGINX Plus Initial Release \(R1\)](https://docs.nginx.com/nginx/releases/#r1)

### NGINX Plus Release 19 \(R19\)

_13 August 2019_  
_Based on NGINX Open Source 1.17.3_

NGINX Plus R19 is a feature release:

* Metrics for individual [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) blocks \(enabled by [`status_zone`](https://nginx.org/en/docs/http/ngx_http_status_module.html#status_zone) directive\)
* Metrics about DNS resolver functionality \(new `status_zone` parameter to [`resolver`](https://nginx.org/en/docs/http/ngx_http_core_module.html#resolver) directive\)
* Two new tabs on NGINX Plus live activity monitoring dashboard for metrics about DNS and clustering; per‑location metrics are also reported
* Dry‑run mode for testing effects of [request‑rate limits](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req) on production traffic without actually enforcing them \(new [`limit_req_dry_run`](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html#limit_req_dry_run) directive\)
* Support in key‑value store for IP address ranges in CIDR notation as well as individual addresses \(new `type=ip` parameter to [`keyval_zone`](https://nginx.org/en/docs/http/ngx_http_keyval_module.html#keyval_zone) directive\)
* Expiration time can be set for each key‑value entry to override default expiration time, either at creation time for new entry or as a modification to existing entry
* The parameter to the [`limit_rate`](https://nginx.org/en/docs/http/ngx_http_core_module.html#limit_rate), [`limit_rate_after`](https://nginx.org/en/docs/http/ngx_http_core_module.html#limit_rate_after), [`proxy_download_rate`](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_download_rate), and [`proxy_upload_rate`](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_upload_rate) directives can be a variable

NGINX Plus R19 is supported on:

* Alpine Linux 3.8, 3.9, 3.10
* Amazon Linux \(2018.03+\), Amazon Linux 2 LTS
* CentOS 6.5+, 7.4+
* Debian 9, 10
* FreeBSD 11.2+, 12.0
* Oracle Linux 6.5+, 7.4+
* RHEL 6.5+, 7.4+, 8
* SUSE Linux Enterprise Server 12, 15
* Ubuntu 16.04 LTS, 18.04 LTS, 19.04

**Notes:**

* Alpine Linux 3.10 is new in this release
* Debian 8 is no longer supported
* Debian 10 is new in this release
* Ubuntu 14.04 LTS and 18.10 are no longer supported
* Ubuntu 19.04 is new in this release

More information: [Announcing NGINX Plus R19](https://www.nginx.com/blog/nginx-plus-r19-released/)

### NGINX Plus Release 18 \(R18\)

_9 April 2019_  
_Based on NGINX Open Source 1.15.10_

NGINX Plus R18 is a feature release:

* Dynamic SSL certificate loading, either from [file](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate) or from [key-value](https://nginx.org/en/docs/http/ngx_http_keyval_module.html) storage \(for the latter case, prefix the variable with `data:`\)
* New features in [OpenID Connect reference implementation](https://github.com/nginxinc/nginx-openid-connect): opaque session tokens as a browser cookie, refresh tokens to refresh expired ID tokens without user interaction, and a logout URL
* Additional logic for verifying arbitrary variables in active health checks \(new `require` parameter to [`match`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#match) directive\)
* Wildcard support for [`listen`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#listen) directive means same [`zone_sync`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync) configuration can now be used for all instances in a cluster
* Port ranges supported for [`listen`](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive
* For TCP/UDP, existing connections to proxied upstream server can be explicitly closed after server is removed from upstream group due to health check failure, API call, or re-resolve action \(new [`proxy_session_drop`](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_session_drop) directive\)
* New variable, [`$upstream_bytes_sent`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_bytes_sent), contains number of bytes sent to an upstream server
* New or updated dynamic modules:
  * [Brotli](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/brotli/) \(New\): General‑purpose, lossless data compression algorithm
  * [OpenTracing](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/opentracing/) \(New\): Ability to instrument NGINX Plus with OpenTracing‑compliant requests for a range of distributed tracing services, such as Datadog, Jaeger, and Zipkin
  * [Lua](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/lua//) \(Updated\): Scripting language for NGINX Plus, updated to use LuaJIT 2.1
  * [NGINX JavaScript](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/nginscript/) \(Updated\): JavaScript module for NGINX Plus, updated to version [0.3.0](https://nginx.org/en/docs/njs/changes.html#njs0.3.0)

NGINX Plus R18 is supported on:

* Alpine Linux 3.8, 3.9
* Amazon Linux \(2018.03+\), Amazon Linux 2 LTS
* CentOS 6.5+, 7.4+
* Debian 8.0, 9.0
* FreeBSD 11.2+, 12.0
* Oracle Linux 6.5+, 7.4+
* RHEL 6.5+, 7.4+, 8
* SUSE Linux Enterprise Server 12, 15
* Ubuntu 14.04 LTS, 16.04 LTS, 18.04, 18.10

**Notes:**

* Amazon Linux 2017.09 is no longer supported; minimum supported version is now 2018.03
* CentOS/Oracle/Red Hat Enterprise Linux 7.3 is no longer supported; minimum supported version is now 7.4
* Debian 8.0 will be removed at NGINX Plus R19
* Ubuntu 14.04 will be removed at NGINX Plus R19

More information: [Announcing NGINX Plus R18](https://www.nginx.com/blog/nginx-plus-r18-released/)

#### NGINX Plus R18 Update

This is a bug‑fix release for NGINX Plus R18.

NGINX Plus R18 P1  
_6 August 2019_

* Security patch: When using HTTP/2 a client might cause excessive memory consumption and CPU usage \([CVE-2019-9511](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-9511), [CVE-2019-9513](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-9513), [CVE-2019-9516](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-9516)\)

### NGINX Plus Release 17 \(R17\)

_11 December 2018_  
_Based on NGINX Open Source 1.15.7_

NGINX Plus R17 is a feature release:

* Support for TLS 1.3 using `TLSv1.3` parameter to [`ssl_protocols`](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_protocols) directive
* Two‑stage [rate limiting](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html) with the new `delay=` parameter; excessive requests are initially delayed and then ultimately rejected
* Support for the Ed25519 and Ed448 cryptographic algorithms added to the [JSON Web Token \(JWT\) module](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html)
* Ability to fetch JSON Web Keys \(JWK\) directly from identity provider \(IdP\) when using OpenID Connect \(new [`auth_jwt_key_request`](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt_key_request) directive\)
* TCP keepalives between NGINX Plus and the proxied server \(new [`proxy_socket_keepalive`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_socket_keepalive) directive\)
* Control over how long HTTP keepalive connection between NGINX Plus and proxied server can be idle before being closed \(new [`keepalive_timeout`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive_timeout) directive\)
* For UDP, number of packets sent from NGINX Plus to proxied server before new UDP “session” to that server is started can be set explicitly \(new [`proxy_requests`](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_requests) directive\)
* [Zone Synchronization](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html) module can now pass server name using SNI when connecting to cluster nodes for server name verification \(new [`zone_sync_ssl_server_name`](https://nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html#zone_sync_ssl_server_name) directive\)
* The NGINX JavaScript module has been updated:
  * Support for arguments objects
  * Support for non‑integer fractions
  * Support for additional time methods: `console.time()` and `console.timeEnd()`
  * Variables and functions can be redeclared
  * Integration with the NGINX Stream module for TCP/UDP applications has been refactored to use various return functions, including a `send()` method for modifying ingress trafficl egress traffic is now available through a callback

NGINX Plus R17 is supported on:

* Alpine Linux 3.8, 3.9
* Amazon Linux \(2017.09\), Amazon Linux 2 LTS
* CentOS 6.5+, 7.0+
* Debian 8.0, 9.0
* FreeBSD 11.2+, 12.0
* Oracle Linux 6.5+, 7.0+
* RHEL 6.5+, 7.0+
* SUSE Linux Enterprise Server 12, 15
* Ubuntu 14.04 LTS, 16.04 LTS, 18.04, 18.10

**Notes:**

* Alpine Linux 3.8 and 3.9 are new in this release
* CentOS/Oracle Linux/RHEL 7.3 will be removed at NGINX Plus R18
* FreeBSD 11.2 and 12.0 are new in this release; versions 10.4 and 11.1 are no longer supported
* Ubuntu 14.04 will be removed at NGINX Plus R19
* Ubuntu 18.10 is new in this release

More information: [Announcing NGINX Plus R17](https://www.nginx.com/blog/nginx-plus-r17-released/)

### NGINX Plus Release 16 \(R16\)

_5 September 2018_  
_Based on NGINX Open Source 1.15.2_

NGINX Plus R16 is a feature release:

* [Rate limiting](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html) in a cluster using [Zone Synchronization](https://www.nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html) module
* [Key-value store](https://nginx.org/en/docs/http/ngx_http_keyval_module.html) in a cluster using [Zone Synchronization](https://www.nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html) module
* Timeouts in [Key-Value Store](https://nginx.org/en/docs/http/ngx_http_keyval_module.html) module
* New [`random`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#random) load‑balancing algorithm with Random with Two Choices variant, for which [`least_time`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#least_time) or [`least_conn`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#least_conn) can be used to decide between the two choices
* UDP load balancing \([`stream`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html) module\) enhanced with support for multiple UDP packets from the client, enabling use of more complex UDP protocols such as OpenVPN, VoIP, and VDI
* Support for [PROXY Protocol v2 \(PPv2\) header](https://www.haproxy.org/download/1.8/doc/proxy-protocol.txt), and ability to inspect custom TLV values in header
* Support for [AWS PrivateLink](https://aws.amazon.com/privatelink/), Amazon’s technology for creating secure tunnels into a VPC
* opaque session token support in the [OpenID Connect reference implementation](https://github.com/nginxinc/nginx-openid-connect)
* New [`$ssl_preread_protocol`](https://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html#var_ssl_preread_protocol) variable to distinguish between SSL/TLS and other protocols when forwarding traffic using a TCP \([`stream`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html)\) proxy
* New [Encrypted Session](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/encrypted-session/) dynamic module
* The NGINX JavaScript module has been updated:
  * Single object \(`r`\) is used to access both request and response attributes associated with each HTTP request
  * New language support: `bytesFrom()`, `padStart()`, `padEnd()`, `getrandom()`, `getentropy()`, and binary literals

NGINX Plus R16 is supported on:

* Amazon Linux \(2017.09\), Amazon Linux 2 LTS
* CentOS 6.5+, 7.0+
* Debian 8.0, 9.0
* FreeBSD 10.4+, 11.1+
* Oracle Linux 6.5+, 7.0+
* RHEL 6.5+, 7.0+
* SUSE Linux Enterprise Server 12
* Ubuntu 14.04 LTS, 16.04 LTS, 18.04

**Notes:**

* FreeBSD 10.4+ and 11.1+ are new in this release; versions 10.3 and 11.0 are no longer supported
* Amazon Linux 2 \(LTS\) is updated to the GA version.
* Ubuntu 17.10 is no longer supported
* The Upstream Conf and Extended Status modules are superseded by the [NGINX Plus API](https://nginx.org/en/docs/http/ngx_http_api_module.html) module and are no longer distributed in NGINX Plus \(see our [transition guide](https://www.nginx.com/blog/transitioning-to-nginx-plus-api-configuration-monitoring/) for details\)
* The [New Relic plug‑in](https://newrelic.com/plugins/nginx-inc/13) for NGINX has been updated to use the new NGINX Plus API, but is no longer supported by NGINX, Inc.

More information: [Announcing NGINX Plus R16](https://www.nginx.com/blog/nginx-plus-r16-released/)

#### NGINX Plus R16 Update

This is a bug‑fix release for NGINX Plus R16.

NGINX Plus R16 P1  
_30 October 2018_

* Security patch: When using HTTP/2 a client might cause excessive memory consumption \([CVE-2018-16843](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-16843)\) and CPU usage \([CVE-2018-16844](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-16844)\)
* Security patch: Processing of a specially crafted MP4 file with the ngx\_http\_mp4\_module might result in worker process memory disclosure \([CVE-2018-16845](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-16845)\)

### NGINX Plus Release 15 \(R15\)

_10 April 2018_  
_Based on NGINX Open Source 1.13.10_

NGINX Plus R15 is a feature release:

* Proxying, load balancing, and SSL-termination of gRPC traffic
* HTTP/2 server push
* Sticky learn session persistence in a cluster using new [Zone Synchronization](https://www.nginx.org/en/docs/stream/ngx_stream_zone_sync_module.html) module, which synchronizes [shared memory zones](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone) across a cluster of NGINX Plus instances
* [OpenID Connect \(OIDC\) authorization code flow](https://github.com/nginxinc/nginx-openid-connect), enabling integration with CA Single Sign-On \(formerly SiteMinder\), ForgeRock OpenAM, Keycloak, Okta, and other identity providers
* Subrequests from the [NGINX JavaScript](https://www.nginx.com/blog/introduction-nginscript/) module
* Crypto libraries in NGINX JavaScript module with support for common hash functions MD5, SHA-1, and SHA-256
* Inheritance of the `CAP_NET_RAW` Linux capability so that transparent proxying does not require worker processes to have root privileges
* New [`auth_jwt_leeway`](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt_leeway) directive to compensate for clock skew between NGINX Plus and identity provider
* Performance enhancements and bug fixes to [NGINX WAF](https://www.nginx.com/products/nginx-waf/) module
* Updates to [LDAP authentication reference implementation](https://github.com/nginxinc/nginx-ldap-auth)
* New [`$upstream_queue_time`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_queue_time) variable to hold the amount of time a request spends in the [upstream queue](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#queue)
* New [`$ssl_preread_alpn_protocols`](https://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html#var_ssl_preread_alpn_protocols) variable to hold the Application Layer Protocol Negotiation \(ALPN\) protocols presented by client
* New [Cookie-Flag](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/cookie-flag/) dynamic module

NGINX Plus R15 is supported on:

* Amazon Linux \(2017.09\), Amazon Linux 2 LTS
* CentOS 6.5+, 7.0+
* Debian 8.0, 9.0
* FreeBSD 10.3, 11.0
* Oracle Linux 6.5+, 7.0+
* RHEL 6.5+, 7.0+
* SUSE Linux Enterprise Server 12
* Ubuntu 14.04 LTS, 16.04 LTS, 17.10, 18.04

**Notes:**

* Ubuntu 17.04 is no longer supported
* nginScript is now known as the NGINX JavaScript module
* The NGINX Plus API version has been incremented to 3; all previous versions of the NGINX Plus API are still supported
* This is the last release to support the deprecated dynamic \(on-the-fly\) reconfiguration and extended status APIs \(see our [transition guide](https://www.nginx.com/blog/transitioning-to-nginx-plus-api-configuration-monitoring/) for details\)

More information: [Announcing NGINX Plus R15](https://www.nginx.com/blog/nginx-plus-r15-released/)

#### NGINX Plus R15 Updates

These are bug‑fix releases for NGINX Plus R15.

NGINX Plus R15 P2  
_30 October 2018_

* Security patch: When using HTTP/2 a client might cause excessive memory consumption \([CVE-2018-16843](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-16843)\) and CPU usage \([CVE-2018-16844](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-16844)\)
* Security patch: Processing of a specially crafted mp4 file with the ngx\_http\_mp4\_module might result in worker process memory disclosure \([CVE-2018-16845](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-16845)\)

NGINX Plus R15 P1  
_12 April 2018_

* Third‑party modules might not be loaded due to signature incompatibility

### NGINX Plus Release 14 \(R14\)

_12 December 2017NGINX Open Source build 1.13.7_

NGINX Plus R14 is a feature release:

* Nested JSON Web Token \(JWT\) claims, array data, and longer key sizes \(256‑, 384‑, and 512‑bit\) for JWT signing algorithms, providing more flexibility and security when validating JWTs
* Clustering support for the [`sticky_learn`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky_learn) method of session persistence, as a technology preview of distribution of session state data in a cluster
* [Key‑value store](https://nginx.org/en/docs//http/ngx_http_keyval_module.html) and NGINX Plus API in the `stream` context, making the same key‑value store features are available for TCP/UDP applications as for HTTP applications
* New NGINX Plus [dashboard](https://demo.nginx.com/) utilizing the NGINX Plus API which was introduced in [NGINX Plus R13](https://docs.nginx.com/nginx/releases/#r13)
* Improvements to [NGINX JavaScript](https://www.nginx.com/blog/introduction-nginscript/) module, including the ability to manage JSON objects, read content from filesystems, and backtrace to errors and exceptions to further improve troubleshooting
* Ability to encode client certificates in a HTTP header and send them to backend applications with the [`$ssl_client_escaped_cert`](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#variables) variable
* Enhanced DNS resolver that preserves the list of upstream IP addresses across a reload of the NGINX Plus configuration
* Ability to drain upstream servers extended to file‑based configurations with the [`drain`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server) parameter to the upstream `server` directive

NGINX Plus R14 is supported on:

* Amazon Linux \(2016.09\), Amazon Linux 2 \(2017.12\)
* CentOS 6.5+, 7.0+
* Debian 8.0, 9.0
* FreeBSD 10.3, 11.0
* Oracle Linux 6.5+, 7.0+
* RHEL 6.5+, 7.0+
* SUSE Linux Enterprise Server 12
* Ubuntu 14.04 LTS, 16.04 LTS, 17.04, 17.10

**Notes:**

* Debian 7.0 is no longer supported
* Ubuntu 17.10 is new in this release
* The Upstream Conf and Extended Status APIs were deprecated in [NGINX Plus R13](https://docs.nginx.com/nginx/releases/#r13); support will continue only through NGINX Plus R15 \(see our [transition guide](https://www.nginx.com/blog/transitioning-to-nginx-plus-api-configuration-monitoring/) for details\)

More information: [Announcing NGINX Plus R14](https://www.nginx.com/blog/nginx-plus-r14-released/)

#### NGINX Plus R14 Updates

This is a bug‑fix release for NGINX Plus R14.

NGINX Plus R14 P1  
_25 January 2018_

* Live activity monitoring: Reinstated some missing tooltips for the dashboard
* NGINX Plus API: HTTP Basic Authentication support for read‑write mode

### NGINX Plus Release 13 \(R13\)

_29 August 2017_  
_Based on NGINX Open Source 1.13.4_

NGINX Plus R13 is a feature release:

* Ability to send duplicate all incoming traffic to a dedicated server \(the [`mirror`](https://nginx.org/en/docs/http/ngx_http_mirror_module.html#mirror) directive\)
* Improvements to [NGINX JavaScript](https://www.nginx.com/blog/introduction-nginscript/) module, including the new interactive shell to facilitate development of NGINX JavaScript code
* New [NGINX Plus API](https://nginx.org/en/docs/http/ngx_http_api_module.html) that incorporates the functionality of the previous [`upstream_conf`](https://nginx.org/en/docs/http/ngx_http_upstream_conf_module.html) and [\(extended\) `status`](https://nginx.org/en/docs/http/ngx_http_status_module.html) APIs; it includes a [Swagger](https://demo.nginx.com/swagger-ui/) specification and adds support for [key‑value stores](https://nginx.org/en/docs//http/ngx_http_keyval_module.html)
* New build tool \([download here](https://hg.nginx.org/pkg-oss/raw-file/default/build_module.sh)\) that creates installable packages of the many third‑party modules available for NGINX and NGINX Plus
* Ability to gracefully shut down all live client connections when restarting NGINX Plus \(the [`worker_shutdown_timeout`](https://nginx.org/en/docs/ngx_core_module.html#worker_shutdown_timeout) directive\)
* Support for adding HTTP trailers \(the [`add_trailer`](https://docs.nginx.com/nginx/releases/https//nginx.org/en/docs/http/ngx_http_headers_module.html#add_trailer) directive\)
* Improvement to session persistence: quicker establishment of sticky sessions between clients and upstream groups \(the `header` parameter to the [`sticky` `learn`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky) directive\)
* Support for the third‑party [HTTP Substitutions Filter](https://github.com/yaoweibin/ngx_http_substitutions_filter_module) module, distributed in NGINX Plus packages and available on the [Dynamic Modules](https://www.nginx.com/products/modules/) page

NGINX Plus R13 is supported on:

* Amazon Linux 2016.09+
* CentOS 6.5+, 7.0+
* Debian 7.0, 8.0, 9.0
* FreeBSD 10.3, 11.0
* Oracle Linux 6.5+, 7.0+
* RHEL 6.5+, 7.0+
* Ubuntu 14.04 LTS, 16.04 LTS, 17.04

**Notes:**

* CentOS/Oracle Linux/RHEL 5.10+ is no longer supported
* Ubuntu 12.04 LTS and 16.10 are no longer supported
* Ubuntu 17.04 is new in this release
* The `sticky_cookie_insert` directive \(deprecated in [NGINX Plus R2](https://docs.nginx.com/nginx/releases/#r2)\) has been removed
* The [`upstream_conf`](https://nginx.org/en/docs/http/ngx_http_upstream_conf_module.html) and [\(extended\) `status`](https://nginx.org/en/docs/http/ngx_http_status_module.html) APIs are deprecated by the new [NGINX Plus API](https://nginx.org/en/docs/http/ngx_http_api_module.html) and will be removed in a future release

More information: [Announcing NGINX Plus R13](https://www.nginx.com/blog/nginx-plus-r13-released)

### NGINX Plus Release 12 \(R12\)

_14 March 2017_  
_Based on NGINX Open Source 1.11.10_

NGINX Plus R12 is a feature release:

* Synchronization of NGINX Plus configuration across instances in a cluster, from a single master node \(new `config_sync` package\)
* Updates to Extended Status module [data set](https://nginx.org/en/docs/http/ngx_http_status_module.html#data), including NGINX Plus version \(`nginx_build`\), usage statistics for shared memory zones \(under the `slabs/` subtree\), and additional upstream fields \(`name`, `service`\)
* New statistics displayed on [live activity monitoring dashboard](https://demo.nginx.com/): NGINX Plus version, response time metrics, shared memory zones usage, and server names for upstreams
* [Support](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_use_stale) for the `stale-while-revalidate` and `stale-if-error` extensions to the `Cache-Control` header, as defined by [RFC 5861](https://www.ietf.org/rfc/rfc5861.txt)
* Ability to bypass cache for byte range requests after a specified offset \(the [`proxy_cache_max_range_offset`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_max_range_offset) directive\)
* Length of `Vary` and `ETag` cache headers increased to 128 bytes; note that the on‑disk cache format has changed, so cached content is invalidated after the upgrade and must be refreshed from the origin server
* `mandatory` parameter to the `health_check` directive \([HTTP](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check) and [Stream](https://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html#health_check)\) which requires servers newly added to an `upstream` group to pass the associated health check before receiving real traffic
* “Zero config” UDP [health check](https://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html#health_check) which does not require specifying a [`match`](https://nginx.org/en/docs/stream/ngx_stream_upstream_hc_module.html#match) block
* Support in the Stream module for verification of [client SSL certificates](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html#ssl_verify_client) for TCP applications
* [SSL variables](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#variables) representing various details about client certificates and capabilities \(`$ssl_ciphers`, `$ssl_client_v_end`, `$ssl_client_v_start`, `$ssl_client_v_remain`, and `$ssl_curves`\)
* The `$ssl_client_verify` variable includes the reason for failure
* The `$ssl_client_i_dn` and `ssl_client_s_dn` variables comply with [RFC 2253](https://www.ietf.org/rfc/rfc2253.txt); legacy variants are available as `$ssl_client_i_dn_legacy` and `$ssl_client_s_dn_legacy`
* Support for accessing arbitrary JWT fields as [variables](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html)
* Support for JSON escaping in access logs \(the `escape` parameter to the [`log_format`](https://nginx.org/en/docs/http/ngx_http_log_module.html#log_format) directive\)
* WebP support in the [Image-Filter](https://nginx.org/en/docs/http/ngx_http_image_filter_module.html) module.
* Output from the `nginx` `-T` command excludes duplicated sections of configuration
* Improvements to memory usage and performance, including upstream [`queue`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#queue) optimization

NGINX Plus R12 is supported on:

* Amazon Linux 2016.09+
* CentOS 5.10+, 6.5+, 7.0+
* Debian 7.0, 8.0, 9.0
* FreeBSD 10.3, 11.0
* Oracle Linux 5.10+, 6.5+, 7.0+
* RHEL 5.10+, 6.5+, 7.0+
* SLES 12, 12 SP1
* Ubuntu 12.04 LTS, 14.04 LTS, 16.04 LTS, 16.10

**Notes:**

* CentOS/Oracle Linux/RHEL 5.10+ will be removed at NGINX Plus R13
* Debian 9 is new in this release
* FreeBSD 9 is no longer supported
* Ubuntu 12.04 LTS will be removed at NGINX Plus R13

More information: [Announcing NGINX Plus R12](https://www.nginx.com/blog/nginx-plus-r12-released/)

#### NGINX Plus R12 Updates

These are bug‑fix releases for NGINX Plus R12.

NGINX Plus R12 P3  
_29 June 2017_

* Content caching: Cache response might contain additional internal cache header data

NGINX Plus R12 P2  
_30 March 2017_

* Live activity monitoring: Response time metric was miscalculated under certain conditions

NGINX Plus R12 P1  
_14 March 2017_

* Live activity monitoring: Dashboard might hang with certain configurations

### NGINX Plus Release 11 \(R11\)

_25 October 2016_  
_Based on NGINX Open Source 1.11.5_

NGINX Plus R11 is a feature release:

* Dynamic modules binary compatibility between NGINX Plus and the corresponding version of open source NGINX
* Enhancements to the Stream module: custom [logging](https://nginx.org/en/docs/stream/ngx_stream_log_module.html) with a number of additional [variables](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#variables), [PROXY protocol support](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#listen) for incoming connections, support for [obtaining](https://nginx.org/en/docs/stream/ngx_stream_realip_module.html) real IP address and port from PROXY protocol header, and ability to [extract the server name](https://nginx.org/en/docs/stream/ngx_stream_ssl_preread_module.html) from SNI into a variable for purposes such as custom routing
* Updates to the Extended Status module [data set](https://nginx.org/en/docs/http/ngx_http_status_module.html#data), including additional Stream metrics \(`sessions`, `discarded`\)
* Cache manager support for iterative operations mode when deleting old cache files, reducing the disk load \(see the `manager_files`, `manager_threshold`, and `manager_sleep` parameters of the [`proxy_cache_path`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path) directive\)
* Support for variables in the `domain` parameter to the [`sticky`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#sticky) directive
* New variable `$upstream_bytes_received` for both [Stream](https://nginx.org/en/docs/stream/ngx_stream_upstream_module.html#var_upstream_bytes_received) and [HTTP](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_bytes_received)\)

NGINX Plus R11 is supported on:

* Amazon Linux 2016.03+
* CentOS 5.10+, 6.5+, 7.0+
* Debian 7.0, 8.0
* FreeBSD 9.3, 10.1+, 11.0
* Oracle Linux 5.10+, 6.5+, 7.0+
* RHEL 5.10+, 6.5+, 7.0+
* SLES 12, 12 SP1
* Ubuntu 12.04 LTS, 14.04 LTS, 16.04 LTS, 16.10

**Notes:**

* FreeBSD 11.0 is new in this release
* Ubuntu 16.10 is new in this release
* The nginx-plus-extras package is no longer provided; migrate to the nginx-plus package and then install the needed [dynamic modules](https://www.nginx.com/products/dynamic-modules)

More information: [Announcing NGINX Plus R11](https://www.nginx.com/blog/nginx-plus-r11-released/)

### NGINX Plus Release 10 \(R10\)

_23 August 2016_  
_Based on NGINX Open Source 1.11.3_

NGINX Plus R10 is a feature release:

* New dynamic module: [ModSecurity](https://www.nginx.com/waf) \(package name is nginx-plus-module-modsecurity\) built on an early release of ModSecurity 3.0
* New dynamic module: nginScript \(package name is nginx-plus-module-njs\)
* Support for client authentication using [JSON Web Tokens \(JWT\)](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html)
* Enhancements to the [Stream](https://nginx.org/en/docs/stream/ngx_stream_core_module.html) module used for TCP/UDP load balancing \(more [NGINX variables](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#variables), [resolver](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#resolver) support, [map](https://nginx.org/en/docs/stream/ngx_stream_map_module.html) module, [geo](https://nginx.org/en/docs/stream/ngx_stream_geo_module.html) module, [geoip](https://nginx.org/en/docs/stream/ngx_stream_geoip_module.html) module, and [split\_clients](https://nginx.org/en/docs/stream/ngx_stream_split_clients_module.html) A/B testing support\)
* Support for dual‑stack RSA/ECC certificates by defining multiple [ssl\_certificate](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate) and [ssl\_certificate\_key](https://nginx.org/en/docs/http/ngx_http_ssl_module.html#ssl_certificate_key) directives on the same virtual server
* Support for IP Transparency and Direct Server Return \(DSR\) using the `transparent` parameter to the [proxy\_bind](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_bind) directive. DSR only supported for UDP load balancing.
* Support for the `IP_BIND_ADDRESS_NO_PORT` socket option where available, allowing for many more upstream connections \(requires Linux kernel 4.2 or later\)
* HTTP/2 improvements: support for unbuffered upload,and various bug fixes
* New NGINX variables: [$request\_id](https://nginx.org/en/docs/http/ngx_http_core_module.html#var_request_id), [$proxy\_protocol\_port](https://nginx.org/en/docs/http/ngx_http_core_module.html#var_proxy_protocol_port), [$realip\_remote\_port](https://nginx.org/en/docs/http/ngx_http_realip_module.html#var_realip_remote_port)
* Modules updated \(both in nginx-plus-extras and as dynamic modules\):
  * [Headers-More](https://github.com/openresty/headers-more-nginx-module) module updated to version 0.31
  * [Lua](https://github.com/openresty/lua-nginx-module) module updated to version 0.10.6
  * [Phusion Passenger Open Source](https://blog.phusion.nl/tag/passenger-releases/) module updated to version 5.0.30
  * [Set-Misc](https://github.com/openresty/set-misc-nginx-module) module updated to version 0.31

NGINX Plus R10 is supported on:

* Amazon Linux 2016.03+
* CentOS 5.10+, 6.5+, 7.0+
* Debian 7.0, 8.0
* FreeBSD 9.3, 10.1+
* Oracle Linux 5.10+, 6.5+, 7.0+
* RHEL 5.10+, 6.5+, 7.0+
* SLES 12, 12 SP1
* Ubuntu 12.04 LTS, 14.04 LTS, 16.04 LTS

**Notes:**

* Ubuntu 15.10 is no longer supported
* NGINX Plus R10 is the last release to include the nginx-plus-extras package; if using this package, migrate to the nginx-plus package and then install the needed [dynamic modules](https://www.nginx.com/products/dynamic-modules)

More information: [Announcing NGINX Plus R10](https://www.nginx.com/blog/nginx-plus-r10-released/)

### NGINX Plus Release 9 \(R9\)

_12 April 2016_  
_Based on NGINX Open Source 1.9.13_

NGINX Plus R9 is a feature release:

* Dynamic loading of modules \(both NGINX‑authored and third‑party\). The NGINX‑authored modules supported in this release:

  * [nginx-plus-module-geoip](https://nginx.org/en/docs/http/ngx_http_geoip_module.html)
  * [nginx-plus-module-image-filter](https://nginx.org/en/docs/http/ngx_http_image_filter_module.html)
  * [nginx-plus-module-perl](https://nginx.org/en/docs/http/ngx_http_perl_module.html)
  * [nginx-plus-module-xslt](https://docs.nginx.com/nginx/releases/https//nginx.org/en/docs/http/ngx_http_xslt_module.html)

  The third‑party modules supported in this release:

  * [nginx-plus-module-headers-more](https://github.com/openresty/headers-more-nginx-module)
  * [nginx-plus-module-lua](https://github.com/openresty/lua-nginx-module)
  * [nginx-plus-module-passenger](https://www.phusionpassenger.com/)
  * [nginx-plus-module-rtmp](https://github.com/arut/nginx-rtmp-module)
  * [nginx-plus-module-set-misc](https://github.com/openresty/set-misc-nginx-module)

* UDP load balancing support, configured in the [`stream`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html) configuration context
* Support for retrieving upstream servers configuration via DNS `SRV` records, configured with the new `service` parameter to the [`server`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#server) directive
* Automatic retrying of DNS requests over TCP when UDP responses are truncated
* Failed nonidempotent HTTP requests \(`POST`, `LOCK`, `PATCH`\) are no longer retried with the other servers in the `upstream` group, unless the `non_idempotent` parameter is included in the [`proxy_next_upstream`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_next_upstream) directive
* Improved cache metadata accounting
* Automatic binding of worker processes to available CPUs using the the new `auto` parameter of the [`worker_cpu_affinity`](https://nginx.org/en/docs/ngx_core_module.html#worker_cpu_affinity) directive
* Optional offloading of some cache write operations to thread pools, configured with the [`aio_write` `on`](https://nginx.org/en/docs/http/ngx_http_core_module.html#aio_write) directive
* Support for customizing the `Server` response header, as well as the signature in standard error messages
* Updated live activity monitoring dashboard
* In the nginx-plus-extras package:
  * [Headers-More](https://github.com/openresty/headers-more-nginx-module) module updated to version  .29
  * [Lua](https://github.com/openresty/lua-nginx-module) module updated to version 0.10.2
  * [Phusion Passenger Open Source](https://blog.phusion.nl/tag/passenger-releases/) module updated to version 5.0.26

NGINX Plus R9 is supported on:

* Amazon Linux 2016.03+
* CentOS 5.10+, 6.5+, 7.0+
* Debian 7.0, 8.0
* FreeBSD 9.3, 10.1+
* Oracle Linux 5.10+, 6.5+, 7.0+
* RHEL 5.10+, 6.5+, 7.0+
* SLES 12, 12 SP1
* Ubuntu 12.04 LTS, 14.04 LTS, 15.10, 16.04 LTS

**Note:**

* Ubuntu 15.04 is no longer supported.

More information: [Announcing NGINX Plus R9](https://www.nginx.com/blog/nginx-plus-r9-released/)

#### NGINX Plus R9 Updates

This is a bug‑fix release for NGINX Plus R9.

NGINX Plus R9 P1  
_25 May 2016_

* Segmentation fault might occur when writing a client request body to a temporary file
* Specially crafted request might cause NGINX worker process to crash due to a NULL pointer dereference \([CVE-2016-4450](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-4450)\)

### NGINX Plus Release 8 \(R8\)

_19 January 2016_  
_Based on NGINX Open Source 1.9.9_

NGINX Plus R8 is a feature release:

* [OAuth Technology Preview](https://www.nginx.com/blog/oauth-technology-preview/), which performs OAuth 2.0 processing for proxied applications
* Improved [HTTP/2](https://nginx.org/en/docs/http/ngx_http_v2_module.html) implementation now included in the nginx-plus and nginx-plus-extras packages; the nginx-plus-http2 package is deprecated
* Caching improvements, including support for caching [`HEAD`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_convert_head) requests and more effective caching of large files with the [Cache Slice](https://nginx.org/en/docs/http/ngx_http_slice_module.html) module
* Changes to upstream groups made with the [on‑the‑fly reconfiguration API](https://nginx.org/en/docs/http/ngx_http_upstream_conf_module.html) can now be configured to [persist](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#state) across restarts and configuration reloads
* Support for sending health check requests to a specified port \(the `port` parameter to the [`health_check`](https://nginx.org/en/docs/http/ngx_http_upstream_hc_module.html#health_check) directive\)
* Enhancement to the [Real IP](https://nginx.org/en/docs/http/ngx_http_realip_module.html) module: the new `$realip_remote_addr` variable represents the original client IP address
* Enhancement to [`syslog`](https://nginx.org/en/docs/syslog.html) logging: the `nohostname` parameter disables logging of the hostname field, which is unnecessary when logging to a local `syslog` server
* Updated live activity monitoring dashboard
* In the nginx-plus-extras package:
  * [Headers-More](https://github.com/openresty/headers-more-nginx-module) module updated to version 0.28
  * [Lua](https://github.com/openresty/lua-nginx-module) module updated to version 0.9.20
  * [Phusion Passenger Open Source](https://blog.phusion.nl/tag/passenger-releases/) module updated to version 5.0.22
  * [Redis](https://github.com/openresty/lua-resty-redis) module for Lua access updated to version 0.21

NGINX Plus R8 is supported on:

* Amazon Linux
* CentOS 5.10+, 6.5+, 7.0
* Debian 7.0, 8.0
* FreeBSD 9.3, 10.1+
* Oracle Linux 5.10+, 6.5+, 7.0
* RHEL 5.10+, 6.5+, 7.0
* SLES 12, 12 SP1
* Ubuntu 12.04 LTS, 14.04 LTS, 15.04, 15.10

NGINX Plus R8 does not include the nginx-plus-lua package; if you previously used this package, migrate to the nginx-plus-extras package

More information: [Announcing NGINX Plus R8](https://www.nginx.com/blog/nginx-plus-r8-released/)

#### NGINX Plus R8 Updates

These are bug‑fix releases for NGINX Plus R8.

NGINX Plus R8 P3  
_24 February 2016_

* HTTP/2: `client_body_timeout` directive was not handled correctly

NGINX Plus R8 P2  
_11 February 2016_

* Logging: Buffer over‑read might occur while logging invalid request headers
* HTTP/2: Various fixes

NGINX Plus R8 P1  
_26 January 2016_

* Resolver: Limit `CNAME` resolutions to prevent remote attackers from causing a denial of service \([CVE-2016-0747](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-0747)\)

### NGINX Plus Release 7 \(R7\)

_15 September 2015_  
_Based on NGINX Open Source 1.9.4_

NGINX Plus R7 is a feature release:

* Support for HTTP/2 in the new **nginx-plus-http2** package \(the nginx-plus and nginx-plus-extras packages continue to support SPDY\)

  **Note:** Before installing the **nginx-plus-http2** package, you must remove the `spdy` parameter on all `listen` directives in your configuration \(replace it with the `http2` and `ssl` parameters to enable support for HTTP/2\). NGINX Plus fails to start if any `listen` directives have the `spdy` parameter.

* Support for proxying [NTLM](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#ntlm) requests
* Enhancements to [TCP load balancing](https://nginx.org/en/docs/stream/ngx_stream_core_module.html) and proxying:
  * [Access controls](https://nginx.org/en/docs/stream/ngx_stream_access_module.html)
  * [Connection limiting](https://nginx.org/en/docs/stream/ngx_stream_limit_conn_module.html)
  * Bandwidth limiting for [upload](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_upload_rate) and [download](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_download_rate)
  * Client‑side [PROXY protocol](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_protocol) support
  * Ability to [set local IP address](https://nginx.org/en/docs/stream/ngx_stream_proxy_module.html#proxy_bind) of origin for outgoing connections
  * New `backlog` parameter to [`listen`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#listen) directive to limit size of queue of pending connections
  * New [`tcp_nodelay`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html#tcp_nodelay) directive to control use of OS `TCP_NODELAY` option
* More efficient distribution of connections across NGINX Plus worker processes \(new `reuseport` parameter to the [`listen`](https://nginx.org/en/docs/http/ngx_http_core_module.html#listen) directive\)
* [Thread pools](https://nginx.org/en/docs/ngx_core_module.html#thread_pool) for multithreaded reading and sending of files without blocking worker processes
* Live activity monitoring dashboard redesigned to use tabs
* Additional live activity monitoring metrics in the [Status](https://nginx.org/en/docs/http/ngx_http_status_module.html#compatibility) module \(dataset version 6\)
* Additional arguments to playlist and fragment URIs in the [HLS](https://nginx.org/en/docs/http/ngx_http_hls_module.html) module \(`start`, `end`, and `offset`\)
* New `-T` flag on `nginx` command to dump the configuration to standard output in a standardized format
* New [`$upstream_connect_time`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#var_upstream_connect_time) variable to capture the connection time to upstream servers
* [`sub_filter`](https://nginx.org/en/docs/http/ngx_http_sub_module.html#sub_filter) directive now supports variables in both the string being replaced and the replacement string; multiple `sub_filter` directives can appear at a configuration level
* In the nginx-plus-extras package:
  * New [Redis](https://github.com/openresty/lua-resty-redis) module for access to Redis databases through Lua
  * [Headers-More](https://github.com/openresty/headers-more-nginx-module) module updated to version 0.26
  * [Lua](https://github.com/openresty/lua-nginx-module) module updated to version 0.9.16
  * [Phusion Passenger Open Source](https://blog.phusion.nl/tag/passenger-releases/) module updated to version 5.0.15
  * [Set-Misc](https://github.com/openresty/set-misc-nginx-module) module updated to version 0.29

NGINX Plus R7 is supported on:

* CentOS 5.10+, 6.5+, 7.0+
* Debian 7.0, 8.0
* FreeBSD 9.3, 10.1+
* Oracle Linux 5.10+, 6.5+, 7.0+
* RHEL 5.10+, 6.5+, 7.0+
* SLES 12
* Ubuntu 12.04 LTS, 14.04 LTS, 15.04

**Notes:**

* Debian 6.0 is no longer supported
* SLES 11 SP3 is no longer supported
* Ubuntu 10.04 LTS and 14.10 are no longer supported
* The nginx-plus-extras package has additional dependencies
* NGINX Plus R7 is the last release that includes the nginx-plus-lua package; customers using the package will have to migrate to the nginx-plus-extras package in NGINX Plus R8

More information and important upgrade information for users of the Phusion Passenger Open Source module: [Announcing NGINX Plus Release 7](https://www.nginx.com/blog/nginx-plus-r7-released/)

### NGINX Plus Release 6 \(R6\)

_14 April 2015_  
_Based on NGINX Open Source 1.7.11_

NGINX Plus R6 is a feature release:

* TCP proxy enhancements \(health checks, dynamic reconfiguration, SSL support, logging, status counters\)
* New Least-Time load‑balancing algorithm
* Support for unbuffered upload \([`proxy_request_buffering`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_request_buffering) directive\)
* Proxy SSL authentication support for HTTP and uwsgi
* Proxy cache enhancements \(variables in value of [`proxy_cache`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache) directive, new `use_temp_path` parameter to [`proxy_cache_path`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path) directive\)
* Mail proxy supports client SSL certificates
* Enhancement to Autoindex module \(new [`autoindex_format`](https://nginx.org/en/docs/http/ngx_http_autoindex_module.html#autoindex_format) directive\)
* New live activity monitoring dashboard
* In the nginx-plus-extras package:
  * [Lua](https://github.com/openresty/lua-nginx-module) module updated to version 0.9.16rc1
  * [Phusion Passenger Open Source](https://blog.phusion.nl/tag/passenger-releases/) module updated to version 4.0.59
  * [Set-Misc](https://github.com/openresty/set-misc-nginx-module) module updated to version 0.28

NGINX Plus R6 is supported on:

* CentOS 5.10+, 6.5+, 7.0
* Debian 6.0, 7.0, 8.0
* FreeBSD 9.3, 10.1
* Oracle Linux 5.10+, 6.5+, 7.0
* RHEL 5.10+, 6.5+, 7.0
* SLES 11 SP3, 12
* Ubuntu 10.04 LTS, 12.04 LTS, 14.04 LTS, 14.10

The nginx-plus-extras package has additional dependencies.

More information: [Announcing NGINX Plus Release 6 with Enhanced Load Balancing, High Availability, and Monitoring Features](https://www.nginx.com/blog/nginx-plus-r6-released/)

### NGINX Plus Release 5 \(R5\)

_2 December 2014_  
_Based on NGINX Open Source 1.7.7_

NGINX Plus R5 is a feature release:

* Proxying and load balancing of raw TCP traffic \(the [`Stream`](https://nginx.org/en/docs/stream/ngx_stream_core_module.html) module\)
* Sticky session timeout now applies from the most recent request in the session
* Upstream “draining” can be used to remove an upstream server without interrupting any user sessions \(new `drain` parameter to the [`upstream_conf`](https://nginx.org/en/docs/http/ngx_http_upstream_conf_module.html#upstream_conf) directive\)
* Improved control over request retries in the event of failure, based on number of tries and time; also available for FastCGI, memcached, SCGI, and uwsgi modules
* `Vary` field in response header is correctly handled for caching \(multiple variants of the same resource can be cached\); note that the on‑disk cache format has changed, so upgrading to R5 invalidates cached content
* Improved caching support for byte‑range requests
* Control of upstream bandwidth \(new [`proxy_limit_rate`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_limit_rate) directive\)
* In the nginx-plus-extras package:
  * [Lua](https://github.com/openresty/lua-nginx-module) module updated to version 0.9.13
  * [Phusion Passenger Open Source](https://blog.phusion.nl/tag/passenger-releases/) module updated to version 4.0.53
* In the nginx-plus-lua package:
  * [Lua](https://github.com/openresty/lua-nginx-module) module updated to version 0.9.13

NGINX Plus R5 is supported on:

* CentOS 5.9, 6.5, 7.0
* Debian 6.0, 7.0
* FreeBSD 9.3, 10.0
* Oracle Linux 5.10+, 6.5+, 7.0
* RHEL 5.9, 6.5, 7.0
* SLES 11 SP3, 12
* Ubuntu 10.04 LTS, 12.04 LTS, 14.04 LTS, 14.10

The nginx-plus-extras and nginx-plus-lua packages have additional dependencies.

More information: [NGINX Plus R5 Released](https://www.nginx.com/blog/nginx-plus-r5-released/)

### NGINX Plus Release 4 \(R4\)

_24 July 2014_  
_Based on NGINX Open Source 1.7.3_

NGINX Plus R4 is a feature release:

* Ability to verify backend SSL certificates
* Support for SNI while working with SSL backends
* Passphrases for SSL private keys can now be stored in an external file
* New load‑balancing method based on user‑defined keys with optional consistency \([hash](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#hash) directive\)
* New session affinity mechanism \(sticky learn\) based on server‑initiated sessions
* Cache revalidation now uses `If-None-Match` header field when possible
* Conditional logging for requests \(new `if` parameter to the [`access_log`](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log) directive\)
* Ability to retrieve a subset of the live activity monitoring data
* [MP4](https://nginx.org/en/docs/http/ngx_http_mp4_module.html) module now supports the `end` argument in request URIs, which sets the end point of playback
* In the nginx-plus-extras package:
  * [Lua](https://github.com/openresty/lua-nginx-module) module updated to version 0.9.10
  * [Phusion Passenger Open Source](https://blog.phusion.nl/tag/passenger-releases/) module updated to version 4.0.45
* In the nginx-plus-lua package:
  * [Lua](https://github.com/openresty/lua-nginx-module) module updated to version 0.9.10

NGINX Plus R4 is supported on:

* CentOS 5.9, 6.5, 7.0
* Debian 6.0, 7.0
* FreeBSD 9.2, 10.0
* Oracle Linux 5.10+, 6.5+, 7.0
* RHEL 5.9, 6.5, 7.0
* SLES 11 SP3
* Ubuntu 10.04 LTS, 12.04 LTS, 14.04 LTS

The nginx-plus-extras and nginx-plus-lua packages have additional dependencies.

More information: [NGINX Plus R4 Released](https://www.nginx.com/blog/nginx-plus-r4-released/)

### NGINX Plus Release 3 \(R3\)

_2 April 2014_  
_Based on NGINX Open Source 1.5.12‑1_

NGINX Plus R3 is a feature release:

* Automatic re‑resolution of hostnames in upstream groups allows group members to be updated on‑the‑fly using DNS
* New connection limits and an internal connection queue protect servers from connection overload and improve connection scheduling by NGINX Plus’ load balancing
* Support for PROXY protocol
* SPDY support updated to comply with draft 3.1
* Additional controls over SSL have been added to control the use of session tickets and reduce time to first byte
* Support for IPv6 DNS resolution

NGINX Plus R3 is supported on:

* CentOS 5.9, 6.5
* Debian 6.0, 7.0
* FreeBSD 9.2, 10.0
* Oracle Linux 5.10+, 6.5+, 7.0
* RHEL 5.9, 6.5
* SLES 11 SP3
* Ubuntu 10.04 LTS, 12.04 LTS, 12.10, 13.10, 14.04 LTS

The nginx-plus-extras and nginx-plus-lua packages have additional dependencies.

More information: [NGINX Plus R3 Released](https://www.nginx.com/blog/nginx-plus-r3-released/)

### NGINX Plus Release 2 \(R2\)

_12 December 2013_  
_Based on NGINX Open Source 1.5.7‑1_

NGINX Plus R2 is a feature release:

* Enhanced sticky routing support
* Additional status metrics for virtual hosts and cache zones
* Cache purge support \(also available for FastCGI\)
* Support for cache revalidation
* Support for authorization based on the result of a subrequest \(new [`ngx_http_auth_request_module`](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html) module\)

#### NGINX Plus R2 Updates

Security Update to NGINX Plus Release R2_21 March 2014_  
_Based on NGINX Open Source 1.5.7‑4_

* Fixes vulnerability in experimental SPDY implementation in NGINX Open Source 1.5.7‑3 and earlier.

Functional Update to NGINX Plus R2_5 March 2014_  
_Based on NGINX Open Source 1.5.7‑3_

* NGINX Plus now correctly applies the value set with the [`client_max_body_size`](https://nginx.org/en/docs/http/ngx_http_core_module.html#client_max_body_size) directive when processing HTTP requests that contain chunk‑encoded body data.

Functional Update to NGINX Plus R2_13 February 2014_  
_Based on NGINX Open Source 1.5.7‑2_

* Updates to MP4 and HLS streaming functionality
* Fix for premature closing of connections when using SPDY with proxy cache
* Updates to implementation of SPDY/2
* Added **status.html** file for live activity monitoring, missing from some packages

### NGINX Plus Initial Release \(R1\)

_22 August 2013_  
_Based on NGINX Open Source 1.5.3‑1_

NGINX Plus is the fully supported, commercial version of NGINX. It includes most NGINX open source modules and adds further features:

* Application health checks
* Live activity monitoring \(implemented in the Extended Status module\)
* Advanced load balancing
* On‑the‑fly reconfiguration of load‑balanced upstream groups
* Extended logging capabilities
* High availability setup
* Adaptive media streaming

