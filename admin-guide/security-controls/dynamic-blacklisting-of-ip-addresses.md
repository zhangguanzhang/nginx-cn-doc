# Dynamic Blacklisting of IP Addresses



This section describes how to create a blacklist or whitelist of specific client IP addresses, which denies or allows them access to your site, and how to dynamically maintain the list of addresses.

* [Overview](https://docs.nginx.com/nginx/admin-guide/security-controls/blacklisting-ip-addresses/#overview)
* [Prerequisites](https://docs.nginx.com/nginx/admin-guide/security-controls/blacklisting-ip-addresses/#prereq)
* [Setup](https://docs.nginx.com/nginx/admin-guide/security-controls/blacklisting-ip-addresses/#configure)
* [Managing the Key-Value Database](https://docs.nginx.com/nginx/admin-guide/security-controls/blacklisting-ip-addresses/#manage)
* [Full Example](https://docs.nginx.com/nginx/admin-guide/security-controls/blacklisting-ip-addresses/#example)
* [See Also](https://docs.nginx.com/nginx/admin-guide/security-controls/blacklisting-ip-addresses/#see-also)

### Overview

In NGINX Plus [Release 13](https://docs.nginx.com/nginx/releases/#r13) \(R13\) and later, you can blacklist some IP addresses as well as create and maintain a database of blacklisted IP addresses. You can also explicitly whitelist other IP addresses. The IP addresses database is managed with the NGINX Plus [API](https://nginx.org/en/docs/http/ngx_http_api_module.html) and [keyval](https://nginx.org/en/docs/http/ngx_http_keyval_module.html) modules.

In NGINX Plus [Release 19](https://docs.nginx.com/nginx/releases/#r13) \(R19\) extends this capability by matching an IP address to any address within the subnet or network range.

### Prerequisites

NGINX Plus [Release 13](https://docs.nginx.com/nginx/releases/#r13) or later, NGINX Plus [Release 19](https://docs.nginx.com/nginx/releases/#r19) for network ranges support.

### Setup

First, enable the database for storing the list of blacklisted and whitelisted IP addresses.

1. In NGINX Plus configuration file, include the [`keyval_zone`](https://nginx.org/en/docs/http/ngx_http_keyval_module.html#keyval_zone) directive in the [`http`](https://nginx.org/en/docs/http/ngx_http_core_module.html#http) context to create a memory zone for storing keys and values. This sample directive creates a 1‑MB zone called **one**.

   ```text
   http {
       # ...
       keyval_zone zone=one:1m;
    }
   ```

   To perform matching of an IP address against subnets \(for example, `192.168.13.0/24`\), specify the the `type=ip` parameter of the [`keyval_zone`](https://nginx.org/en/docs/http/ngx_http_keyval_module.html#keyval_zone) directive:

   ```text
   http {
       # ...
       keyval_zone zone=one:1m type=ip;
    }
   ```

   Note that the size of [`keyval_zone`](https://nginx.org/en/docs/http/ngx_http_keyval_module.html#keyval_zone) should also be increased as the `type=ip` parameter also enables an extra index stored in the zone.

   You can optionally include the `state` parameter to create a file where the key‑value database is stored and so persists across NGINX Plus reloads and restarts; in this example, **one.keyval**:

   ```text
   keyval_zone zone=one:1m state=one.keyval;
   ```

2. Enable the NGINX Plus API in read‑write mode with the [`api`](https://nginx.org/en/docs/http/ngx_http_api_module.html#api) directive:

   ```text
   # ...
    server {
        listen 80;
        server_name www.example.com;

        location /api {
            api write=on;
        }
    }
   ```

   We strongly recommend [restricting access](https://docs.nginx.com/nginx/admin-guide/security-controls/controlling-access-proxied-http/#restrict) to this location, for example by allowing access only from `localhost` \(`127.0.0.1`\), and by restricting access to `PATCH`, `POST`, and `DELETE` methods to some users with HTTP basic authentication:

   ```text
   # ...
    server {
        listen 80;
        server_name www.example.com;

        location /api {
            limit_except GET {
                auth_basic "NGINX Plus API";
                auth_basic_user_file /path/to/passwd/file;
            }

            api   write=on;

            allow 127.0.0.1;
            deny  all;
        }
    }
   ```

3. Populate the key‑value database with the API’s [`POST`](https://nginx.org/en/docs/http/ngx_http_api_module.html#postHttpKeyvalZoneData) method, supplying the data in JSON format. You can use the `curl` command as in the following example. If the zone is empty, you can enter several key‑value pairs at once; otherwise, pairs must be added one at a time.

   ```text
   $ curl -X POST -d '{
      "10.0.0.1": "1",
      "10.0.0.2": "1",
      "10.0.0.3": "0",
      "10.0.0.4": "0"
    }' -s http://www.example.com/api/5/http/keyvals/one
   ```

   In case you have specified matching an IP address against network ranges \(the `type=ip` parameter of the [`keyval_zone`](https://nginx.org/en/docs/http/ngx_http_keyval_module.html#keyval_zone) directive, send the POST command with the network range. The range must be specified in CIDR notation:

   ```text
   $ curl -X POST -d '{
      "192.168.13.0/24": "1"
    }' -s http://www.example.com/api/5/http/keyvals/one
   ```

4. Define how client IP addresses are evaluated against the key‑value database, by including the [`keyval`](https://nginx.org/en/docs/http/ngx_http_keyval_module.html#keyval) directive in the [`http`](https://nginx.org/en/docs/http/ngx_http_core_module.html#http) context.

   The directive takes advantage of the standard NGINX and NGINX Plus variable [`$remote_addr`](https://nginx.org/en/docs/http/ngx_http_core_module.html#var_remote_addr), which is set to the client IP address automatically for every request.

   As it processes each request, NGINX Plus:

   * Looks up the first parameter \(here, `$remote_addr`, preset to the client’s IP address\) in the key‑value database specified by the `zone=` parameter \(here, **one**\).
   * If a key in the database exactly matches `$remote_addr`, sets the second parameter \(here, `$target`\) to the value corresponding to the key. In our example, the value is `1` for blacklisted addresses or `0` for whitelisted addresses.

   ```text
   http {
        # ...
        keyval_zone zone=one:1m type=ip state=one.keyval;
        keyval $remote_addr $target zone=one; # Client address is the key, 
                                              # $target is the value;
    }
   ```

5. Create a rule with the [`if`](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html#if) directive that either allows or denies access depending on the client IP address. With this rule, access is allowed when `$target` is `0` and denied when it is `1`:

   ```text
   if ($target) {
       return 403;
   }
   ```

### Managing the Key-Value Database

You can use API methods to update a key‑value database on‑the‑fly, without requiring a reload of NGINX Plus.

All of the following examples operate on the **one** zone, which is accessible at **http://www.example.com/api/5/http/keyvals/one**.

* To get the list of all database entries for a zone:

  ```text
  $ curl -X GET 'http://www.example.com/api/5/http/keyvals/one'
  ```

* To update the value for an existing entry \(in this example to change the access status for IP address `10.0.0.4` from whitelisted to blacklisted\):

  ```text
  $ curl -X PATCH -d '{"10.0.0.4": "1"}' -s 'http://www.example.com/api/5/http/keyvals/one'
  ```

* To add an entry to a populated zone:

  ```text
  $ curl -X POST -d '{"10.0.0.5": "1"}' -s 'http://www.example.com/api/5/http/keyvals/one'
  ```

* To delete an entry:

  ```text
  $ curl -X PATCH -d '{"10.0.0.4":null}' -s 'http://www.example.com/api/5/http/keyvals/one'
  ```

### Full Example

The full NGINX Plus configuration:

```text
http {
    # ...
    keyval_zone zone=one:1m type=ip state=one.keyval;
    keyval $remote_addr $target zone=one;

    server {
        listen 80;
        server_name www.example.com;

        location /api {
            limit_except GET {
                auth_basic "NGINX Plus API";
                auth_basic_user_file /path/to/passwd/file;
            }

            api   write=on;

            allow 127.0.0.1;
            deny  all;
        }

        if ($target) {
            return 403;
        }
    }
}
```

This configuration:

* Creates a 1 MB keyval zone **one** that accepts network ranges and also creates the file **one.keyval** to make the database of key‑value pairs persists across reloads and restarts of NGINX Plus.
* Enables the NGINX Plus API in write mode so that the zone can populated with IP addresses
* Enables lookup of the IP address `$remote_addr` in the key-value database as the key, and puts the value of the found key into the `$target` variable.
* Enables a simple rule to check for the resulting value: if the value of `$target` is `1` \(address is blacklisted\), return `403 (Forbidden)` to the client.

The following `curl` command populates the empty keyval zone **one** with IP addresses that are blacklisted \(value is `1`\) or whitelisted \(value is `0`\):

```text
$ curl -X POST -d '{
     "10.0.0.1": "1",
     "192.168.13.0/24": "1",
     "10.0.0.3": "0",
     "10.0.0.4": "0"
}' -s 'http://www.example.com/api/5/http/keyvals/one'
```

### See Also

* [Dynamic IP Blacklisting with NGINX Plus and fail2ban](https://www.nginx.com/blog/dynamic-ip-blacklisting-with-nginx-plus-and-fail2ban/)

