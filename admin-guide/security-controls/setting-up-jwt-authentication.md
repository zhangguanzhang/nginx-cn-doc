# Setting up JWT Authentication



This article explains how to control authentication of your web resources using JWT authentication.

* [Introduction](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-jwt-authentication/#intro)
* [Prerequisites](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-jwt-authentication/#prereq)
* [Configuring NGINX to Authenticate API](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-jwt-authentication/#auth_jwt)
* [How NGINX Plus Validates a JWT](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-jwt-authentication/#jwt_validate)
* [Creating a JSON Web Key File](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-jwt-authentication/#jwk_create)
* [Issuing JWT to Clients](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-jwt-authentication/#jwt_create)
* [Getting JWKs from Subrequest](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-jwt-authentication/#auth_jwt_key_request)
* [See Also](https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-jwt-authentication/#see_also)

### Introduction

With NGINX Plus it is possible to control access to your resources using JWT authentication. JWT is data format for user information in the OpenID Connect standard, which is the standard identity layer on top of the OAuth 2.0 protocol. Deployers of APIs and microservices are also turning to the JWT standard for its simplicity and flexibility. With JWT authentication, a client provides a JSON Web Token, and the token will be validated against a local key file or a remote service.

### Prerequisites

* NGINX Plus [Release 10](https://docs.nginx.com/nginx/releases/#r10) \(R10\) for native [JWT support](https://www.nginx.com/blog/nginx-plus-r10-released/#r10-jwt)
* NGINX Plus [Release 14](https://docs.nginx.com/nginx/releases/#r14) \(R14\) for access to [nested JWT claims and longer signing keys](https://www.nginx.com/blog/nginx-plus-r14-released/#jwt)
* NGINX Plus [Release 17](https://docs.nginx.com/nginx/releases/#r17) \(R17\) for [getting JSON Web keys from a remote location](https://www.nginx.com/blog/nginx-plus-r17-released/#r17-openid)
* An identity provider \(IdP\) or service that creates JWT. For manual JWT generation, see “Issuing a JWT to API Clients” section of the [Authenticating API Clients with JWT and NGINX Plus](https://www.nginx.com/blog/authenticating-api-clients-jwt-nginx-plus/) blog post.

### Configuring NGINX to Authenticate API

Let’s assume that NGINX Plus serves as a gateway \(`proxy_pass http://api_server`\) to a number of API servers \(the [`upstream {}`](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#upstream) block\):

```text
upstream api_server {
    server 10.0.0.1;
    server 10.0.0.2;
}

server {
    listen 80;

    location /products/ {
        proxy_pass http://api_server;
        #...
    }
}
```

Requests passed to the API servers should be authenticated. To implement JWT for authentication, specify the [`auth_jwt`](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt) directive that enables JWT authentication and also defines the authentication area \(or “realm”, “API” in the example\):

```text
server {
    listen 80;

    location /products/ {
        proxy_pass http://api_server;
        auth_jwt   "API";
        #...
    }
}
```

Specify the path to the JWT key file against which the JWT signature will be validated. This can be done with the [`auth_jwt_key_file`](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt_key_file) directive. It indicates the JSON Web Key that will be used to validate the token signature \(functions like the public key in SSL/TLS encryption\):

```text
server {
    listen 80;

    location /products/ {
        proxy_pass        http://api_server;
        auth_jwt          "API";
        auth_jwt_key_file conf/key.jwk;
    }
}
```

### How NGINX Plus Validates a JWT

A JWT is considered to be valid when the following conditions are met:

* The signature can be validated with the key found in the [`auth_jwt_key_file`](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt_key_file) \(matching on the `kid` header field if present\).
* The JWT is presented inside the validity period, when defined by one or both of the `nbf` \(“not before”\) and `exp` \(“expires”\) claims.

### Creating a JSON Web Key File

In order to validate the signature with a key, a JSON Web Key \(`key.jwk`\) should be created.The file format is defined by [JSON Web Key specification](https://tools.ietf.org/html/rfc7517):

```text
{"keys":
    [{
        "k":"ZmFudGFzdGljand0",
        "kty":"oct",
        "kid":"0001"
    }]
}
```

where:

* the `k` field is the generated symmetric key \(base64url-encoded\) basing on a `secret` \(`fantasticjwt` in the example\). The secret can be generated with the following command:

```text
$ echo -n fantasticjwt | base64 | tr '+/' '-_' | tr -d '='
ZmFudGFzdGljand0
```

* the `kty` field defines the key type as a symmetric key \(octet sequence\)
* the `kid` \(Key ID\) field defines a serial number for this JSON Web Key

When the key is created, it is possible to issue the JWT to clients.

### Issuing JWT to Clients

First, it is necessary to create a JWT for a client and configure NGINX Plus to accept JWT. You can use your an identity provider \(IdP\) or your own service to create JWTs. For testing purposes, you can create your own JWT, see [Authenticating API Clients with JWT and NGINX Plus](https://www.nginx.com/blog/authenticating-api-clients-jwt-nginx-plus/) blog post for details.

#### Configuring NGINX Plus to Accept JWT from Query String

NGINX Plus can obtain the JWT from a query string parameter. To configure this, include the `token=` parameter to the [`auth_jwt`](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt) directive:

```text
server {
    listen 80;

    location /products/ {
        proxy_pass        http://api_server;
        auth_jwt          "API" token=$arg_apijwt;
        auth_jwt_key_file conf/key.jwk;
    }
}
#...
```

With this configuration the JWT can be validated using the following command:

```text
$ curl http://192.168.1.1/products/widget1?apijwt=`cat token.jwt`
```

### Getting JWKs from Subrequest

NGINX Plus can be configured to fetch JSON Web Keys from the remote location - usually an identity provider, especially when using OpenID Connect. The IdP URI where the subrequest will be sent to is configured with the [`auth_jwt_key_request`](https://nginx.org/en/docs/http/ngx_http_auth_jwt_module.html#auth_jwt_key_request) directive:

```text
http {
    #...

    server {
        listen 80;
            #...

    location / {
        auth_jwt "closed site";
        auth_jwt_key_request /_jwks_uri; # Keys will be fetched by subrequest

        proxy_pass http://my_backend;
    }
```

The URI may refer to an internal location \(`_jwks_uri`\) so that the JSON Web Key Set can be cached \([`proxy_cache`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache) and [`proxy_cache_path`](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path) directives\) to avoid validation overhead:

```text
http {
    proxy_cache_path /var/cache/nginx/jwk levels=1 keys_zone=jwk:1m max_size=10m;
    #...

    server {
        listen 80;
            #...

            location = /_jwks_uri {
                internal;
                proxy_cache jwk; # Cache responses
                proxy_pass https://idp.example.com/oauth2/keys; # Obtain keys from here
        }
    }
}
```

The full example of getting JWKs from a subrequest:

```text
#
proxy_cache_path /var/cache/nginx/jwk levels=1 keys_zone=jwk:1m max_size=10m;

server {
    listen 80; # Use SSL/TLS in production

    location / {
        auth_jwt "closed site";
        auth_jwt_key_request /_jwks_uri; # Keys will be fetched by subrequest

        proxy_pass http://my_backend;
    }

    location = /_jwks_uri {
        internal;
        proxy_cache jwk; # Cache responses
        proxy_pass https://idp.example.com/oauth2/keys; # Obtain keys from here
    }
}
```

### See Also

* [Authenticating API Clients with JWT and NGINX Plus](https://www.nginx.com/blog/authenticating-api-clients-jwt-nginx-plus/)

