# Installing NGINX Plus



本段介绍如何在 Amazon Linux, CentOS, Debian, FreeBSD, Oracle Linux, Red Hat Enterprise Linux \(RHEL\), SUSE Linux Enterprise Server \(SLES\), and Ubuntu 等系统上安装 `NGINX Plus`

* [Prerequisites](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#prereq)
* [Installing NGINX Plus on Amazon Linux, CentOS, Oracle Linux, and RHEL](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#install_rhel_centos)
* [Installing NGINX Plus on Debian and Ubuntu](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#install_debian_ubuntu)
* [Installing NGINX Plus on FreeBSD](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#install_freebsd)
* [Installing NGINX Plus on SUSE Linux Enterprise Server](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#install_suse)
* [Installing NGINX Plus on Alpine Linux](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#install_alpine)
* [Installing Dynamically Loadable Modules](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#install_modules)
  * [Installing Dynamic Modules from the NGINX, Inc. Repository](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#install_modules_plus)
  * [Installing NGINX Community Modules](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#install_modules_oss)
  * [Enabling Dynamic Modules](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#enable_dynamic)
* [Upgrading NGINX Plus](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#upgrade)
* [Upgrading NGINX Plus Modules](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#upgrade_modules)

### Prerequisites

* An NGINX Plus subscription \(purchased or trial\)
* A [supported operating system](https://docs.nginx.com/nginx/technical-specs/)
* `root` privilege
* Your credentials to the [NGINX Plus Customer Portal](https://cs.nginx.com/), provided by email from NGINX, Inc.
* Your NGINX Plus certificate and public key \(**nginx-repo.crt** and **nginx-repo.key** files\), provided by email from NGINX, Inc.

### Installing NGINX Plus on Amazon Linux, CentOS, Oracle Linux, and RHEL

NGINX Plus can be installed on the following versions of Amazon Linux CentOS/Oracle Linux/RHEL:

* Amazon Linux 2018.03+ \(x86\_64\)
* Amazon Linux 2 LTS \(x86\_64\)
* CentOS/Oracle Linux/ Red Hat Enterprise Linux 6.5+, 7.4+, 8.0+

To install NGINX Plus on Amazon Linux, CentOS, Oracle Linux, and RHEL:

1. Create the **/etc/ssl/nginx** directory:

   ```text
   $ sudo mkdir /etc/ssl/nginx
   $ cd /etc/ssl/nginx
   ```

2. Log in to [NGINX Plus Customer Portal](https://cs.nginx.com/) and download your **nginx-repo.crt** and **nginx-repo.key** files.
3. Copy the files to the **/etc/ssl/nginx/** directory:

   ```text
   $ sudo cp nginx-repo.crt /etc/ssl/nginx/
   $ sudo cp nginx-repo.key /etc/ssl/nginx/
   ```

4. Install the required **ca-certificates** dependency:

   ```text
   $ sudo yum install ca-certificates
   ```

5. Download the **nginx-plus-repo** file and copy it to the **/etc/yum.repos.d/** directory. Each version of Amazon Linux, CentOS, Oracle Linux, or RHEL has its own repo file.
   * For Amazon Linux:

     ```text
     $ sudo wget -P /etc/yum.repos.d https://cs.nginx.com/static/files/nginx-plus-amazon.repo
     ```

   * For Amazon Linux 2:

     ```text
     $ sudo wget -P /etc/yum.repos.d https://cs.nginx.com/static/files/nginx-plus-amazon2.repo
     ```

   * For version 6 of CentOS, Oracle Linux, or RHEL:

     ```text
     $ sudo wget -P /etc/yum.repos.d https://cs.nginx.com/static/files/nginx-plus-6.repo
     ```

   * For version 7.4+ of CentOS, Oracle Linux, or RHEL:

     ```text
     $ sudo wget -P /etc/yum.repos.d https://cs.nginx.com/static/files/nginx-plus-7.4.repo
     ```

   * For version 8.0+ of CentOS, Oracle Linux, or RHEL:

     ```text
     $ sudo wget -P /etc/yum.repos.d https://cs.nginx.com/static/files/nginx-plus-8.repo
     ```
6. Install the **nginx-plus** package. Any older NGINX Plus package is automatically replaced.

   ```text
   $ sudo yum install nginx-plus
   ```

7. To enable the nginx service start at boot, run the command:

   ```text
   $ sudo systemctl enable nginx.service
   ```

### Installing NGINX Plus on Debian and Ubuntu

NGINX Plus can be installed on the following versions of Debian or Ubuntu:

* Debian 9 \(“Stretch”\)
* Debian 10 \(“Buster”\)
* Ubuntu 16.04 \(“Xenial”\) \(i386, x86\_64, ppc64le, aarch64\)
* Ubuntu 18.04 \(“Bionic”\)
* Ubuntu 19.04 \(“Disco”\)

To install NGINX Plus on Debian or Ubuntu:

1. Create the **/etc/ssl/nginx** directory:

   ```text
   $ sudo mkdir /etc/ssl/nginx
   $ cd /etc/ssl/nginx
   ```

2. Log in to [NGINX Plus Customer Portal](https://cs.nginx.com/) and download your **nginx-repo.crt** and **nginx-repo.key** files.
3. Copy the files to the **/etc/ssl/nginx/** directory:

   ```text
   $ sudo cp nginx-repo.crt /etc/ssl/nginx/
   $ sudo cp nginx-repo.key /etc/ssl/nginx/
   ```

4. Download the NGINX signing key from [nginx.org](https://nginx.org/keys/nginx_signing.key) and add it:

   ```text
   $ sudo wget https://nginx.org/keys/nginx_signing.key
   $ sudo apt-key add nginx_signing.key
   ```

5. Install the **apt-utils** package and the NGINX Plus repository.
   * For Debian:

     ```text
     $ sudo apt-get install apt-transport-https lsb-release ca-certificates
     $ printf "deb https://plus-pkgs.nginx.com/debian `lsb_release -cs` nginx-plus\n" | sudo tee /etc/apt/sources.list.d/nginx-plus.list
     ```

   * For Ubuntu:

     ```text
     $ sudo apt-get install apt-transport-https lsb-release ca-certificates
     $ printf "deb https://plus-pkgs.nginx.com/ubuntu `lsb_release -cs` nginx-plus\n" | sudo tee /etc/apt/sources.list.d/nginx-plus.list
     ```
6. Download the **90nginx** file to **/etc/apt/apt.conf.d**:

   ```text
   $ sudo wget -q -O /etc/apt/apt.conf.d/90nginx https://cs.nginx.com/static/files/90nginx
   ```

7. Update the repository information:

   ```text
   $ sudo apt-get update
   ```

8. Install the **nginx-plus** package. Any older NGINX Plus package is automatically replaced.

   ```text
   $ sudo apt-get install -y nginx-plus
   ```

### Installing NGINX Plus on FreeBSD

NGINX Plus can be installed on the following versions of FreeBSD:

* FreeBSD 11.2+ \(x86\_64\)
* FreeBSD 12.0+

To install NGINX Plus on FreeBSD:

1. Create the **/etc/ssl/nginx** directory:

   ```text
   $ sudo mkdir -p /etc/ssl/nginx
   $ cd /etc/ssl/nginx
   ```

2. Log in to [NGINX Plus Customer Portal](https://cs.nginx.com/) and download your **nginx-repo.crt** and **nginx-repo.key** files.
3. Copy the files to the **/etc/ssl/nginx/** directory:

   ```text
   $ sudo cp nginx-repo.crt /etc/ssl/nginx/
   $ sudo cp nginx-repo.key /etc/ssl/nginx/
   ```

4. Install the prerequisite **ca\_root\_nss** package:

   ```text
   $ sudo pkg install ca_root_nss
   ```

5. Copy the [nginx-plus.conf](https://cs.nginx.com/static/files/nginx-plus.conf) file to the **/etc/pkg/** directory:

   ```text
   $ sudo fetch -O /etc/pkg/nginx-plus.conf http://cs.nginx.com/static/files/nginx-plus.conf
   ```

6. Add the following lines to the **/usr/local/etc/pkg.conf** file:

   ```text
   PKG_ENV: { SSL_NO_VERIFY_PEER: "1",
   SSL_CLIENT_CERT_FILE: "/etc/ssl/nginx/nginx-repo.crt",
   SSL_CLIENT_KEY_FILE: "/etc/ssl/nginx/nginx-repo.key" }
   ```

7. Install the **nginx-plus** package. Any older NGINX Plus package is automatically replaced. If you have older NGINX Plus package installed, it is recommended backing up the configuration and log files \(see “[Upgrading NGINX Plus](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#upgrade)” for details\).

   ```text
   $ sudo pkg install nginx-plus
   ```

### Installing NGINX Plus on SUSE Linux Enterprise Server

NGINX Plus can be installed on the following versions of SUSE Linux Enterprise Server:

* SUSE Linux Enterprise Server 12, 15 \(x86\_64\)

To install NGINX Plus on SLES:

1. Create the **/etc/ssl/nginx** directory:

   ```text
   $ sudo mkdir /etc/ssl/nginx
   $ cd /etc/ssl/nginx
   ```

2. Log in to [NGINX Plus Customer Portal](https://cs.nginx.com/) and download your **nginx-repo.crt** and **nginx-repo.key** files.
3. Create a file bundle of the certificate and key:

   ```text
   $ cat /etc/ssl/nginx/nginx-repo.crt /etc/ssl/nginx/nginx-repo.key > /etc/ssl/nginx/nginx-repo-bundle.crt
   ```

4. Install the required **ca-certificates** dependency:

   ```text
   $ zypper install ca-certificates
   ```

5. Add the **nginx-plus** repo.

   For SLES 12:

   ```text
   $ zypper addrepo -G -t yum -c 'https://plus-pkgs.nginx.com/sles/12?ssl_clientcert=/etc/ssl/nginx/nginx-repo-bundle.crt&ssl_verify=peer' nginx-plus
   ```

   For SLES 15:

   ```text
   $ zypper addrepo -G -t yum -c 'https://plus-pkgs.nginx.com/sles/15?ssl_clientcert=/etc/ssl/nginx/nginx-repo-bundle.crt&ssl_verify=peer' nginx-plus
   ```

6. Install the **nginx-plus** package. Any older NGINX Plus package is automatically replaced.

   ```text
   $ zypper install nginx-plus
   ```

### Installing NGINX Plus on Alpine Linux

NGINX Plus can be installed on the following versions of Alpine Linux:

* Alpine Linux 3.8, 3.9, 3.10 \(x86\_64\)

To install NGINX Plus on Alpine Linux:

1. Create the **/etc/ssl/nginx** directory:

   ```text
   $ sudo mkdir /etc/ssl/nginx
   $ cd /etc/ssl/nginx
   ```

2. Log in to [NGINX Plus Customer Portal](https://cs.nginx.com/) and download your **nginx-repo.crt** and **nginx-repo.key** files.
3. Upload **nginx-repo.key** to **/etc/apk/cert.key** and **nginx-repo.crt** to **/etc/apk/cert.pem**. Please make sure that files does not contain other certificates and keys: Alpine Linux does not support mixing client certificates for different repositories.
4. Put NGINX signing public key to directory **/etc/apk/keys**:

   ```text
   $ sudo wget -O /etc/apk/keys/nginx_signing.rsa.pub https://nginx.org/keys/nginx_signing.rsa.pub
   ```

5. Add NGINX repository to **/etc/apk/**repositories file:

   ```text
   $ printf "https://plus-pkgs.nginx.com/alpine/v`egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release`/main\n" | tee -a /etc/apk/repositories
   ```

6. It is recommended to remove all community-supported NGINX packages. Please note all NGINX modules will be removed too.

   ```text
   $ sudo apk del -r nginx
   ```

7. Install the **nginx-plus** package.

   ```text
   $ sudo apk add nginx-plus
   ```

8. Check the `nginx` binary version to ensure that you have NGINX Plus installed correctly:

   ```text
   $ nginx -v
   ```

### Installing Dynamically Loadable Modules

NGINX Plus functionality can be extended with dynamically loadable modules that are not included in the prebuilt packages:

* NGINX-authored dynamic modules – Modules written and maintained by NGINX, Inc. Install these modules directly from the NGINX, Inc. modules repository:
  * [nginx-plus-module-geoip](https://nginx.org/en/docs/http/ngx_http_geoip_module.html)
  * [nginx-plus-module-image-filter](https://nginx.org/en/docs/http/ngx_http_image_filter_module.html)
  * [nginx-plus-module-njs](https://nginx.org/en/docs/njs_about.html)
  * [nginx-plus-module-perl](https://nginx.org/en/docs/http/ngx_http_perl_module.html)
  * [nginx-plus-module-xslt](https://nginx.org/en/docs/http/ngx_http_xslt_module.html)
* NGINX-certified community dynamic modules – Popular third‑party modules that NGINX tests and distributes, and for which NGINX provides installation and basic configuration support. Install these modules directly from the NGINX, Inc. modules repository:
  * [nginx-plus-module-headers-more](https://github.com/openresty/headers-more-nginx-module)
  * [nginx-plus-module-lua](https://github.com/openresty/lua-nginx-module)
  * [nginx-plus-module-passenger](https://www.phusionpassenger.com/)
  * [nginx-plus-module-rtmp](https://github.com/arut/nginx-rtmp-module)
  * [nginx-plus-module-set-misc](https://github.com/openresty/set-misc-nginx-module)
* Community dynamic modules – Modules written and distributed by third‑party members of the NGINX community. Download the source code from the author’s repository and [compile it against the NGINX Open Source version](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#install_modules_oss) corresponding to your NGINX Plus version. For a list, see the [NGINX Wiki](https://www.nginx.com/resources/wiki/modules/index.html).

#### Installing Dynamic Modules from the NGINX, Inc. Repository

NGINX‑authored and NGINX‑certified dynamic modules can be installed directly from the modules repository. To install the modules:

* For Amazon Linux, CentOS, Oracle Linux, and RHEL:

  ```text
  $ yum install <MODULE-NAME>
  ```

* For Debian and Ubuntu:

  ```text
  $ apt-get install <MODULE-NAME>
  ```

* For FreeBSD:

  ```text
  $ pkg install <MODULE-NAME>
  ```

* For SLES:

  ```text
  $ zypper install <MODULE-NAME>
  ```

Note that some modules are not available for certain OS versions because of OS limitations. For details as well as descriptions of the modules, see the [NGINX Plus Technical Specifications](https://docs.nginx.com/nginx/technical-specs/).

After installing the module, you need to enable it in the NGINX Plus configuration file. See [Enabling Dynamic Modules](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#enable_dynamic) for details.

#### Installing NGINX Community Modules

For a community dynamic module to work with NGINX Plus, it must be compiled alongside the corresponding open source NGINX version.

1. Prepare the build environment.

   We strongly recommend that you compile dynamic modules on a separate system we will refer to as the “build environment”. Doing so minimizes the risk and complexity of the system you plan to upgrade NGINX Plus on \(we will refer to this as the “production environment”\). The build environment must meet the following requirements:

   * The same operating system as the production environment
   * The same NGINX version as the production environment
   * Compiler and make utilities
   * [PCRE](http://pcre.org/) library \(development files\)
   * [Zlib](https://www.zlib.net/) compression libraries \(development files\)

   To ensure your build environment has these prerequisites installed, run the appropriate command.

   * For Debian and Ubuntu:

     ```text
     $ sudo apt-get install gcc make libpcre3-dev zlib1g-dev
     ```

   * For CentOS, Oracle Linux, and RHEL:

     ```text
     $ sudo yum install gcc make pcre-devel zlib-devel
     ```

2. Obtain open source NGINX.
   * Identify the open source NGINX version that corresponds to your version of NGINX Plus. See [NGINX Plus Releases](https://docs.nginx.com/nginx/releases/).
   * Download the sources for the appropriate open source NGINX version, in this case 1.15.10:

     ```text
     $ wget -qO - https://nginx.org/download/nginx-1.15.10.tar.gz | tar zxfv -
     ```
3. Obtain the source for the dynamic module.

   The source code for the dynamic module can be placed in any directory in the build environment. As an example, here we’re copying the [NGINX “Hello World” module](https://github.com/perusio/nginx-hello-world-module.git/) from GitHub:

   ```text
   $ git clone https://github.com/perusio/nginx-hello-world-module.git
   ```

4. Compile the dynamic module.

   First establish binary compatibility by running the `configure` script with the `‑‑with‑compat` option. Then compile the module with `make modules`.

   ```text
   $ cd nginx-1.17.3/
   $ ./configure --with-compat --add-dynamic-module=../<MODULE-SOURCES>
   $ make modules
   ```

   The **.so** file generated by the build process is placed in the **objs** subdirectory

   ```text
   $ ls objs/*.so
   objs/ngx_http_hello_world.so
   ```

5. Make a copy of the module file and include the open source NGINX version in the filename. This makes it simpler to manage multiple versions of a dynamic module in the production environment.

   ```text
   $ cp objs/ngx_http_hello_world.so ./ngx_http_hello_world_1.17.3.so
   ```

#### Enabling Dynamic Modules

Dynamic modules are located in the **/etc/nginx/modules** directory, which is created automatically at NGINX Plus installation.

To enable a dynamic module:

1. In the `main` \(top-level\) context in **/etc/nginx/nginx.conf**, add a [`load_module`](https://nginx.org/en/docs/ngx_core_module.html#load_module) directive for each dynamically loadable module you installed.

   ```text
   load_module modules/<MODULE-NAME>.so;
   ```

2. Check the new configuration for syntactic validity and reload NGINX Plus.

   ```text
   $ nginx -t && nginx -s reload
   ```

### Upgrading NGINX Plus

To upgrade your NGINX Plus installation to the newest version:

1. If your system has previous NGINX or NGINX Plus packages on it, back up the configuration and log files.
   * For Linux distributions:

     ```text
     $ sudo cp -a /etc/nginx /etc/nginx-plus-backup
     $ sudo cp -a /var/log/nginx /var/log/nginx-plus-backup
     ```

   * For FreeBSD:

     ```text
     $ sudo cp -a /usr/local/etc/nginx /usr/local/etc/nginx-plus-backup
     $ sudo cp -a /var/log/nginx /var/log/nginx-plus-backup
     ```
2. Upgrade to the new NGINX Plus package.
   * For Amazon Linux, CentOS, Oracle Linux, and RHEL:

     ```text
     $ sudo yum upgrade nginx-plus
     ```

     **Note:** Before upgrading to version 6.6 of CentOS, Oracle Linux, or RHEL, see [this advisory](https://www.nginx.com/blog/nginx-se-linux-changes-upgrading-rhel-6-6/).

   * For Debian and Ubuntu:

     ```text
     $ sudo apt-get update
     $ sudo apt-get install nginx-plus
     ```

   * For FreeBSD:

     ```text
     $ sudo pkg upgrade nginx-plus
     ```

To verify that the new NGINX Plus version is running, run:

```text
$ nginx -v
nginx version: nginx/1.17.3 (nginx-plus-r19)
```

### Upgrading NGINX Plus Modules

The upgrade procedure depends on how the module was supplied and installed.

* NGINX‑authored and NGINX‑certified community dynamic modules are updated automatically together with NGINX Plus.

  **Note**: For FreeBSD, each NGINX‑authored and NGINX‑certified module must be updated separately using FreeBSD package management tool.

* Community dynamic modules must be recompiled against the corresponding open source NGINX version. See [Installing NGINX Community Modules](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-plus/#install_modules_oss).

