# Authentication Based on Subrequest Result

### Introduction

NGINX and NGINX Plus can authenticate each request to your website with an external server or service. To perform authentication, NGINX makes an HTTP subrequest to an external server where the subrequest is verified. If the subrequest returns a `2xx` response code, the access is allowed, if it returns `401` or `403`, the access is denied. Such type of authentication allows implementing various authentication schemes, such as multifactor authentication, or allows implementing LDAP or OAuth authentication.

### Prerequisites

* NGINX Plus or NGINX Open Source
* External authentication server or service

### Configuring NGINX and NGINX Plus

1. Make sure your NGINX Open Source is compiled with the `with-http_auth_request_module` configuration option. Run this command and verify that the output includes `--with-http_auth_request_module`:

   ```text
   $ nginx -V 2>&1 | grep -- 'http_auth_request_module'
   ```

   Skip this step for NGINX Plus as it already includes the auth\_request module.

2. In the location that requires request authentication, specify the [`auth_request`](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html#auth_request) directive in which specify an internal location where an authorization subrequest will be forwarded to:

   ```text
   location /private/ {
       auth_request /auth;
       #...
   }
   ```

   Here, for each request to **/private**, a subrequest to the internal **/auth** location will be made.

3. Specify an internal location and the [`proxy_pass`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass) directive inside this location that will proxy authentication subrequests to an authentication server or service:

   ```text
   location = /auth {
       internal;
       proxy_pass http://auth-server;
       #...
   }
   ```

4. As the request body is discarded for authentication subrequests, you will need to set the [`proxy_pass_request_body`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass_request_body) directive to `off` and also set the `Content-Length` header to a null string:

   ```text
   location = /auth {
       internal;
       proxy_pass              http://auth-server;
       proxy_pass_request_body off;
       proxy_set_header        Content-Length "";
       #...
   }
   ```

5. Pass the full original request URI with arguments with the [`proxy_set_header`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_set_header) directive:

   ```text
   location = /auth {
       internal;
       proxy_pass              http://auth-server;
       proxy_pass_request_body off;
       proxy_set_header        Content-Length "";
       proxy_set_header        X-Original-URI $request_uri;
   }
   ```

6. As an option, you can set a variable value basing on the result of the subrequest with the [`auth_request_set`](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html#auth_request_set) directive:

   ```text
   location /private/ {
       auth_request        /auth;
       auth_request_set $auth_status $upstream_status;
   }
   ```

### Complete Example

This example sums up the previous steps into one configuration:

```text
http {
    #...
    server {
    #...
        location /private/ {
            auth_request     /auth;
            auth_request_set $auth_status $upstream_status;
        }

        location = /auth {
            internal;
            proxy_pass              http://auth-server;
            proxy_pass_request_body off;
            proxy_set_header        Content-Length "";
            proxy_set_header        X-Original-URI $request_uri;
        }
    }
}
```

