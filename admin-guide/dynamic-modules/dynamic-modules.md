# Dynamic Modules

This section describes how to use dynamic modules with NGINX Plus.

### Overview

NGINX Plus uses a modular architecture. New features and functionality can be added with software modules, which can be plugged into a running NGINX Plus instance on demand. Dynamic modules add functionality to NGINX Plus such as geolocating users by IP address, resizing images, and embedding Lua scripts into the NGINX Plus event‑processing model. Modules are created both by NGINX, Inc. and third‑party developers.

![NGINX Plus allows features to be plugged in on demand](https://cdn.wp.nginx.com/wp-content/uploads/2016/05/NGINX-Plus-R9-dynamic-modules.png)

_Dynamic modules plug into NGINX Plus to provide additional functionality_

NGINX, Inc. maintains a repository of dynamic modules for NGINX Plus. All modules in our repository are fully tested and certified for correct interoperation with NGINX Plus.

There are many additional [third‑party modules](https://www.nginx.com/resources/wiki/modules/) that are not included in the repository. You can compile many of them as dynamic modules.

### Getting Started with the Dynamic Modules Repository

You can access and download the modules in the NGINX Plus dynamic modules repository using standard package management tools such as `apt` and `yum`. You then include a [`load_module`](https://nginx.org/en/docs/ngx_core_module.html#load_module) directive in the NGINX Plus configuration for each dynamic module. For module‑specific instructions, see the list of modules in the same section of the NGINX Plus Admin Guide as this article.

#### Prerequisites

To get started using dynamic modules, first install NGINX Plus Release 9 \(R9\) or later, following the instructions at the [NGINX Plus customer portal](https://cs.nginx.com/repo_setup).

#### Displaying the List of Available Modules

To see the list of available modules, run this command \(for Debian and Ubuntu\):

```text
$ apt-cache search nginx-plus-module
nginx-plus-module-geoip - NGINX Plus, provided by Nginx, Inc. (GeoIP dynamic modules)
nginx-plus-module-geoip-dbg - Debugging symbols for the nginx-plus-module-geoip
nginx-plus-module-geoip2 - NGINX Plus, provided by Nginx, Inc. (3rd-party GeoIP2 dynamic modules)
nginx-plus-module-geoip2-dbg - Debugging symbols for the nginx-plus-module-geoip2
nginx-plus-module-headers-more - NGINX Plus, provided by Nginx, Inc. (3rd-party headers-more dynamic module)
nginx-plus-module-headers-more-dbg - Debugging symbols for the nginx-plus-module-headers-more
nginx-plus-module-image-filter - NGINX Plus, provided by Nginx, Inc. (image filter dynamic module)
nginx-plus-module-image-filter-dbg - Debugging symbols for the nginx-plus-module-image-filter
```

**Note:** There is an optional debugging symbols package available for every module. You can load and use the module without installing this package.

#### NGINX Plus Certified Modules

In addition to modules authored by NGINX, Inc. and community third‑party developers, the repository contains NGINX Plus Certified Modules which are available for purchase from commercial third parties. Certified Modules are distributed and supported by their authors. NGINX, Inc. has tested the modules extensively and certifies that they do not interfere with standard NGINX Plus functionality.

NGINX Plus Certified Modules are identified with this checkmark icon on the [Dynamic Modules page](https://www.nginx.com/products/nginx/modules):

![test](https://cdn-1.wp.nginx.com/wp-content/themes/nginx-theme/assets/img/product-page/module-icon.png)

Click the module box on that page and then the **GET MODULE** button.

#### Caveats

Some modules are not available for certain OS versions because of OS limitations. For details, see the [NGINX Plus Technical Specifications](https://docs.nginx.com/nginx/technical-specs/).

### Compiling Your Own Dynamic Modules

To compile your own dynamic modules, please see our [blog](https://www.nginx.com/blog/compiling-dynamic-modules-nginx-plus/).

