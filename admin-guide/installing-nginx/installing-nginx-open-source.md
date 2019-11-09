# Installing NGINX Open Source

本文介绍如何安装开源的nginx

* \*\*\*\*[**选择Stable或者Mainline版本**](installing-nginx-open-source.md#choosing-between-a-stable-or-a-mainline-version)\*\*\*\*
* [**选择包管理器\(预构建包\)和源码编译**](installing-nginx-open-source.md#choosing-between-a-prebuilt-package-and-compiling-from-source)
* [Installing a Prebuilt Package](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#prebuilt)
  * [Modules Included in a Prebuilt Package](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#prebuilt_modules)
  * [Installing Prebuilt CentOS and RHEL Packages](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#prebuilt_redhat)
  * [Installing Prebuilt Debian Packages](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#prebuilt_debian)
  * [Installing Prebuilt Ubuntu Packages](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#prebuilt_ubuntu)
  * [Installing SUSE Packages](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#prebuilt_suse)
  * [Installing Alpine Linux Packages](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#prebuilt_alpine)
* [Compiling and Installing from Source](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#sources)
  * [Installing NGINX Dependencies](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#dependencies)
  * [Downloading the Sources](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#sources_download)
  * [Configuring the Build Options](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#configure)
    * [Configuring NGINX Paths](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#configure_paths)
    * [Configuring NGINX GCC Options](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#configure_gcc)
    * [Specifying NGINX Connection Processing Methods](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#configure_methods)
    * [Selecting the NGINX Modules to Build](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#modules)
    * [Modules Built by Default](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#modules_default)
    * [Including Modules Not Built by Default](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#modules_not_default)
    * [Including Third-Party Modules](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#modules_third_party)
  * [Completing the Installation from Source](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#install_complete)

### 选择Stable或者Mainline版本 <a id="choosing-between-a-stable-or-a-mainline-version"></a>

开源nginx可用的两种版本:

* **Mainline** – 包括最新功能和错误修复，并始终是最新的。 它是可靠的但可能包括一些实验模块，它也可能引入一些新的bug。
* **Stable** – 不包含所有最新功能，只具有关键错误的修复，始终向后移植到主线版本。 建议生产服务器的稳定版本。

### 选择包管理器\(预构建包\)和源码编译 <a id="choosing-between-a-prebuilt-package-and-compiling-from-source"></a>

无论是**Stable**还是**Mainline**版本无非是以下两种安装方式:

* 预构建包: 简单快速安装开源nginx的方式. 几乎包含了所有主流的官方模块，可用于大多数流行的操作系统. See [Installing a Prebuilt Package](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#prebuilt).
* 从源码编译: 这种方式更灵活：您可以添加特定模块，包括第三方模块，或应用最新的安全补丁. See [Compiling and Installing from Source](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#sources) for details.

### Installing a Prebuilt Package

Installing NGINX open source from a package is much easier and faster than building from source, but building from source enables you to compile in non-standard modules. Prebuilt packages are available for most popular Linux distributions, including CentOS, Debian, Red Hat Enterprise Linux \(RHEL\), SUSE Linux Enterprise Server \(SLES\), and Ubuntu. See [Linux packages](https://nginx.org/en/linux_packages.html) at **nginx.org** for the list of currently supported operating systems.

#### Modules Included in a Prebuilt Package

See [Configure Arguments](https://nginx.org/en/linux_packages.html#arguments) at **nginx.org** for the list of modules included in each prebuilt package.

#### Installing Prebuilt CentOS and RHEL Packages

NGINX, Inc. provides packages for the following CentOS, Oracle Linux, and RHEL versions:

| **版本** | **支持的架构** |
| :--- | :--- |
| 6.x | x86\_64, i386 |
| 7.4+ | x86\_64, ppc64le |
| 8.x | x86\_64 |

The package can be installed from:

* A default RHEL or CentOS repository. This is the quickest way, but generally the provided package is outdated.
* The official repo at **nginx.org**. You have to set up the `yum` repository the first time, but after that the provided package is always up to date.

**Installing a Prebuilt CentOS/RHEL Package from an OS Repository**

1. Install the EPEL repository:

   ```text
   $ sudo yum install epel-release
   ```

2. Update the repository:

   ```text
   $ sudo yum update
   ```

3. Install NGINX open source:

   ```text
   $ sudo yum install nginx
   ```

4. Verify the installation:

   ```text
   $ sudo nginx -v
   nginx version: nginx/1.6.3
   ```

**Installing a Prebuilt CentOS/RHEL Package from the Official NGINX Repository**

1. Set up the `yum` repository for RHEL or CentOS by creating the file **nginx.repo** in **/etc/yum.repos.d**, for example using `vi`:

   ```text
   $ sudo vi /etc/yum.repos.d/nginx.repo
   ```

2. Add the following lines to **nginx.repo**:

   ```text
   [nginx]
   name=nginx repo
   baseurl=https://nginx.org/packages/mainline/<OS>/<OSRELEASE>/$basearch/
   gpgcheck=0
   enabled=1
   ```

   where:

   * The `/mainline` element in the pathname points to the latest mainline version of NGINX open source; delete it to get the latest stable version
   * `<OS>` is either `rhel` or `centos`
   * `<OSRELEASE>` is the release number \(`6`, `6._x_`, `7`, `7._x_` and so on\)

     For example, to get the latest mainline package for CentOS 7, insert:

     ```text
     [nginx]
     name=nginx repo
     baseurl=https://nginx.org/packages/mainline/centos/7/$basearch/
     gpgcheck=0
     enabled=1
     ```

3. Save the changes and quit `vi` \(press ESC and type `wq` at the `:` prompt\).
4. Update the repository:

   ```text
   $ sudo yum update
   ```

5. Install the NGINX open source package:

   ```text
   $ sudo yum install nginx
   ```

6. Start NGINX open source:

   ```text
   $ sudo nginx
   ```

7. Verify that NGINX open source is up and running:

   ```text
   $ curl -I 127.0.0.1
   HTTP/1.1 200 OK
   Server: nginx/1.13.8
   ```

#### Installing Prebuilt Debian Packages

NGINX provides packages for the following Debian operating systems:

| **Version** | **Codename** | **Supported Platforms** |
| :--- | :--- | :--- |
| 9.x | stretch | x86\_64, i386 |
| 10.x | buster | x86\_64, i386 |

The package can be installed from:

* A default Debian repository. This is the quickest way, but generally the provided package is outdated.
* The official repo at **nginx.org**. You have to set up the `apt-get` repository the first time, but after that the provided package is always up to date.

**Installing a Prebuilt Debian Package from an OS Repository**

1. Update the Debian repository information:

   ```text
   $ sudo apt-get update
   ```

2. Install the NGINX open source package:

   ```text
   $ sudo apt-get install nginx
   ```

3. Verify the installation:

   ```text
   $ sudo nginx -v
   nginx version: nginx/1.6.2
   ```

**Installing a Prebuilt Debian Package from the Official NGINX Repository**

1. Download the key used to sign NGINX packages and the repository, and add it to the `apt` program’s key ring:

   ```text
   $ sudo wget https://nginx.org/keys/nginx_signing.key
   $ sudo apt-key add nginx_signing.key
   ```

2. Edit the **/etc/apt/sources.list** file, for example with `vi`:

   ```text
   $ sudo vi /etc/apt/sources.list
   ```

3. Add these lines to **sources.list** to name the repositories from which the NGINX open source can be obtained:

   ```text
   deb https://nginx.org/packages/mainline/debian/ <CODENAME> nginx
   deb-src https://nginx.org/packages/mainline/debian/ <CODENAME> nginx
   ```

   where:

   * The `/mainline` element in the pathname points to the latest mainline version of NGINX open source; delete it to get the latest stable version
   * `<CODENAME>` is the codename of a Debian release

     For example, to get the latest mainline package for Debian 8.2 \(“jessie”\), insert:

     ```text
     deb https://nginx.org/packages/mainline/debian/ jessie nginx
     deb-src https://nginx.org/packages/mainline/debian/ jessie nginx
     ```

4. Save the changes and quit `vi` \(press ESC and type `wq` at the `:` prompt\).
5. Install the NGINX package:

   ```text
   $ sudo apt-get remove nginx-common
   $ sudo apt-get update
   $ sudo apt-get install nginx
   ```

6. Start NGINX open source:

   ```text
   $ sudo nginx
   ```

7. Verify that NGINX open source is up and running:

   ```text
   $ curl -I 127.0.0.1
   HTTP/1.1 200 OK
   Server: nginx/1.13.8
   ```

#### Installing Prebuilt Ubuntu Packages

NGINX provides packages for the following Ubuntu operating systems:

| **Version** | **Codename** | **Supported Platforms** |
| :--- | :--- | :--- |
| 16.04 | xenial | x86\_64, i386, ppc64el, aarch64/arm64 |
| 18.04 | bionic | x86\_64, aarch64/arm64 |
| 18.10 | cosmic | x86\_64 |
| 19.04 | disco | x86\_64 |

The package can be installed from:

* A default Ubuntu repository. This is the quickest way, but generally the provided package is outdated.
* The official repo at **nginx.org**. You have to set up the `apt-get` repository the first time, but after that the provided package is always up to date.

**Installing a Prebuilt Ubuntu Package from an Ubuntu Repository**

1. Update the Ubuntu repository information:

   ```text
   $ sudo apt-get update
   ```

2. Install the package:

   ```text
   $ sudo apt-get install nginx
   ```

3. Verify the installation:

   ```text
   $ sudo nginx -v
   nginx version: nginx/1.4.6 (Ubuntu)
   ```

**Installing a Prebuilt Ubuntu Package from the Official NGINX Repository**

1. Download the key used to sign NGINX packages and the repository, and add it to the `apt` program’s key ring:

   ```text
   $ sudo wget https://nginx.org/keys/nginx_signing.key
   $ sudo apt-key add nginx_signing.key
   ```

2. Edit the **/etc/apt/sources.list** file, for example with `vi`:

   ```text
   $ sudo vi /etc/apt/sources.list
   ```

3. Add these lines **sources.list** to name the repositories from which the NGINX open source source can be obtained:

   ```text
   deb https://nginx.org/packages/mainline/ubuntu/ <CODENAME> nginx
   deb-src https://nginx.org/packages/mainline/ubuntu/ <CODENAME> nginx
   ```

   where:

   * The `/mainline` element in the pathname points to the latest mainline version of NGINX open source; delete it to get the latest stable version
   * `<CODENAME>` is the codename of an Ubuntu release

   For example, to get the latest mainline package for Ubuntu 14.04 \(“trusty”\), add:

   ```text
   deb https://nginx.org/packages/mainline/ubuntu/ trusty nginx
   deb-src https://nginx.org/packages/mainline/ubuntu/ trusty nginx
   ```

4. Save the changes and quit `vi` \(press ESC and type `wq` at the `:` prompt\).
5. Install NGINX open source:

   ```text
   $ sudo apt-get remove nginx-common
   $ sudo apt-get update
   $ sudo apt-get install nginx
   ```

6. Start NGINX open source:

   ```text
   $ sudo nginx
   ```

7. Verify that NGINX open source is up and running:

   ```text
   $ curl -I 127.0.0.1
   HTTP/1.1 200 OK
   Server: nginx/1.13.8
   ```

#### Installing SUSE Packages

NGINX provides packages for SUSE Linux Enterprise Server:

| **Version** | **Supported Platforms** |
| :--- | :--- |
| SLES 12 | x86\_64 |
| SLES 15 | x86\_64 |

To download and install the latest stable version for SLES 12, run:

```text
$ zypper addrepo -G -t yum -c 'https://nginx.org/packages/sles/12' nginx
$ zypper install nginx
```

To download and install the latest mainline version for SLES 12, run:

```text
$ zypper addrepo -G -t yum -c 'https://nginx.org/packages/mainline/sles/12' nginx
$ zypper install nginx
```

#### Installing Prebuilt Alpine Linux Packages

NGINX provides packages for the following Alpine Linux operating systems:

| **Version** | **Supported Platforms** |
| :--- | :--- |
| 3.8 | x86\_64 |
| 3.9 | x86\_64 |
| 3.10 | x86\_64 |

The package can be installed from the official repo at **nginx.org**. You have to set up the `apt-get` repository the first time, but after that the provided package is always up to date.

**Installing a Prebuilt Alpine Linux Package from the Official NGINX Repository**

1. Install the prerequisites:

   ```text
   $ sudo apk add openssl curl ca-certificates
   ```

2. To set up the apk repository for stable nginx packages, run the command:

   ```text
   $ printf "%s%s%s\n" \
   "http://nginx.org/packages/alpine/v" \
   `egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release` \
   "/main" \
   | sudo tee -a /etc/apk/repositories
   ```

   For mainline nginx packages, run the following command instead:

   ```text
   $ printf "%s%s%s\n" \
   "http://nginx.org/packages/mainline/alpine/v" \
   `egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release` \
   "/main" \
   | sudo tee -a /etc/apk/repositories
   ```

3. Import an official nginx signing key so apk could verify the packages authenticity. Fetch the key:

   ```text
   $ curl -o /tmp/nginx_signing.rsa.pub https://nginx.org/keys/nginx_signing.rsa.pub
   ```

4. Install NGINX open source:

   ```text
   $ sudo apt-get remove nginx-common
   $ sudo apt-get update
   $ sudo apt-get install nginx
   ```

5. Verify that the downloaded file contains the proper key:

   ```text
   $ openssl rsa -pubin -in /tmp/nginx_signing.rsa.pub -text -noout
   ```

   The output should contain the following modulus:

   ```text
   Public-Key: (2048 bit)
   Modulus:
       00:fe:14:f6:0a:1a:b8:86:19:fe:cd:ab:02:9f:58:
       2f:37:70:15:74:d6:06:9b:81:55:90:99:96:cc:70:
       5c:de:5b:e8:4c:b2:0c:47:5b:a8:a2:98:3d:11:b1:
       f6:7d:a0:46:df:24:23:c6:d0:24:52:67:ba:69:ab:
       9a:4a:6a:66:2c:db:e1:09:f1:0d:b2:b0:e1:47:1f:
       0a:46:ac:0d:82:f3:3c:8d:02:ce:08:43:19:d9:64:
       86:c4:4e:07:12:c0:5b:43:ba:7d:17:8a:a3:f0:3d:
       98:32:b9:75:66:f4:f0:1b:2d:94:5b:7c:1c:e6:f3:
       04:7f:dd:25:b2:82:a6:41:04:b7:50:93:94:c4:7c:
       34:7e:12:7c:bf:33:54:55:47:8c:42:94:40:8e:34:
       5f:54:04:1d:9e:8c:57:48:d4:b0:f8:e4:03:db:3f:
       68:6c:37:fa:62:14:1c:94:d6:de:f2:2b:68:29:17:
       24:6d:f7:b5:b3:18:79:fd:31:5e:7f:4c:be:c0:99:
       13:cc:e2:97:2b:dc:96:9c:9a:d0:a7:c5:77:82:67:
       c9:cb:a9:e7:68:4a:e1:c5:ba:1c:32:0e:79:40:6e:
       ef:08:d7:a3:b9:5d:1a:df:ce:1a:c7:44:91:4c:d4:
       99:c8:88:69:b3:66:2e:b3:06:f1:f4:22:d7:f2:5f:
       ab:6d
   Exponent: 65537 (0x10001)
   ```

6. Move the key to apk trusted keys storage:

   ```text
   $ sudo mv /tmp/nginx_signing.rsa.pub /etc/apk/keys/
   ```

7. To install nginx, run the command:

   ```text
   sudo apk add nginx
   ```

### Compiling and Installing from Source

Compiling NGINX open source from source affords more flexibility than prebuilt packages: you can add particular modules \(from NGINX or third parties\), and apply latest security patches.

#### Installing NGINX Dependencies

Prior to compiling NGINX open source from source, you need to install libraries for its dependencies:

* [PCRE](http://pcre.org/) – Supports regular expressions. Required by the NGINX [Core](https://nginx.org/en/docs/ngx_core_module.html) and [Rewrite](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html) modules.

  ```text
  $ wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.43.tar.gz
  $ tar -zxf pcre-8.43.tar.gz
  $ cd pcre-8.43
  $ ./configure
  $ make
  $ sudo make install
  ```

* [zlib](https://www.zlib.net/) – Supports header compression. Required by the NGINX [Gzip](https://nginx.org/en/docs/http/ngx_http_gzip_module.html) module.

  ```text
  $ wget http://zlib.net/zlib-1.2.11.tar.gz
  $ tar -zxf zlib-1.2.11.tar.gz
  $ cd zlib-1.2.11
  $ ./configure
  $ make
  $ sudo make install
  ```

* [OpenSSL](https://www.openssl.org/) – Supports the HTTPS protocol. Required by the NGINX [SSL](https://nginx.org/en/docs/http/ngx_http_ssl_module.html) module and others.

  ```text
  $ wget http://www.openssl.org/source/openssl-1.1.1c.tar.gz
  $ tar -zxf openssl-1.1.1c.tar.gz
  $ cd openssl-1.1.1c
  $ ./Configure darwin64-x86_64-cc --prefix=/usr
  $ make
  $ sudo make install
  ```

#### Downloading the Sources

Download the source files for both the stable and mainline versions from [**nginx.org**](https://www.nginx.org/en/download.html).

To download and unpack the source for the latest _mainline_ version, run:

```text
$ wget https://nginx.org/download/nginx-1.17.3.tar.gz
$ tar zxf nginx-1.17.3.tar.gz
$ cd nginx-1.17.3
```

To download and unpack source files for the latest _stable_ version, run:

```text
$ wget https://nginx.org/download/nginx-1.16.1.tar.gz
$ tar zxf nginx-1.16.1.tar.gz
$ cd nginx-1.16.1
```

#### Configuring the Build Options

Configure options are specified with the `./configure` script that sets up various NGINX parameters, including paths to source and configuration files, compiler options, connection processing methods, and the list of modules. The script finishes by creating the `Makefile` required to compile the code and install NGINX open source.

An example of options to the `configure` script \(should be typed as a single line\):

```text
$ ./configure
--sbin-path=/usr/local/nginx/nginx
--conf-path=/usr/local/nginx/nginx.conf
--pid-path=/usr/local/nginx/nginx.pid
--with-pcre=../pcre-8.43
--with-zlib=../zlib-1.2.11
--with-http_ssl_module
--with-stream
--with-mail=dynamic
--add-module=/usr/build/nginx-rtmp-module
--add-dynamic-module=/usr/build/3party_module
```

**Configuring NGINX Paths**

The `configure` script allows you to set paths to NGINX binary and configuration files, and to dependent libraries such as PCRE or SSL, in order to link them statically to the NGINX binary.

| **Parameter** | **Description** |
| :--- | :--- |
| `--prefix=<PATH>` | Directory for NGINX files, and the base location for all relative paths set by the other configure script options \(excluding paths to libraries\) and for the path to the **nginx.conf** configuration file. Default: **/usr/local/nginx**. |
| `--sbin-path=<PATH>` | Name of the NGINX executable file, which is used only during installation. Default: **&lt;prefix&gt;/sbin/nginx** |
| `--conf-path=<PATH>` | Name of the NGINX configuration file. You can, however, always override this value at startup by specifying a different file with the `-c <FILENAME>` option on the nginx command line. Default: **&lt;prefix&gt;conf/nginx.conf** |
| `--pid-path=<PATH>` | Name of the **nginx.pid** file, which stores the process ID of the nginx master process. After installation, the path to the filename can be changed with the [pid](https://nginx.org/en/docs/ngx_core_module.html#pid) directive in the NGINX configuration file. Default: **&lt;prefix&gt;/logs/nginx.pid** |
| `--error-log-path=<PATH>` | Name of the primary log file for errors, warnings, and diagnostic data. After installation, the filename can be changed with the the [error\_log](https://nginx.org/en/docs/ngx_core_module.html#error_log) directive in the NGINX configuration file. Default: **&lt;prefix&gt;/logs/error.log** |
| `--http-log-path=<PATH>` | Name of the primary log file for requests to the HTTP server. After installation, the filename can always be changed with the [access\_log](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log) directive in the NGINX configuration file. Default: **&lt;prefix&gt;/logs/access.log** |
| `--user=<NAME>` | Name of the unprivileged user whose credentials are used by the NGINX worker processes. After installation, the name can be changed with the [user](https://nginx.org/en/docs/ngx_core_module.html#user) directive in the NGINX configuration file. Default: `nobody` |
| `--group=<NAME>` | Name of the group whose credentials are used by the NGINX worker processes. After installation, the name can be changed with the [user](https://nginx.org/en/docs/ngx_core_module.html#user) directive in the NGINX configuration file. Default: the value set by the `--user` option. |
| `--with-pcre=<PATH>` | Path to the source for the PCRE library, which is required for regular expressions support in the [location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location) directive and the [Rewrite](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html) module. |
| `--with-pcre-jit` | Builds the PCRE library with “just-in-time compilation” support \(the [pcre\_jit](https://nginx.org/en/docs/ngx_core_module.html#pcre_jit) directive\). |
| `--with-zlib=<PATH>` | Path to the source for the zlib library, which is required by the [Gzip](https://nginx.org/en/docs/http/ngx_http_gzip_module.html) module. |

**Configuring NGINX GCC Options**

With the `configure` script you can also specify compiler‑related options.

| **Parameter** | **Description** |
| :--- | :--- |
| `--with-cc-opt="<PARAMETERS>"` | Additional parameters that are added to the `CFLAGS` variable. When using the system PCRE library under FreeBSD, the mandatory value is `--with-cc-opt="-I /usr/local/include"`. If the number of files supported by `select()` needs to be increased, it can also specified here as in this example: `--with-cc-opt="-D FD_SETSIZE=2048"`. |
| `--with-ld-opt="<PARAMETERS>"` | Additional parameters that are used during linking. When using the system PCRE library under FreeBSD, the mandatory value is `--with-ld-opt="-L /usr/local/lib"`. |

**Specifying NGINX Connection Processing Methods**

With the `configure` script you can redefine the method for event‑based polling. For more information, see [Connection processing methods](https://nginx.org/en/docs/events.html) in the NGINX reference documentation.

| **Module Name** | **Description** |
| :--- | :--- |
| `--with-select_module`, `--without-select_module` | Enables or disables building a module that enable NGINX to work with the `select()` method. The modules is built automatically if the platform does not appear to support more appropriate methods such as kqueue, epoll, or /dev/poll. |
| `--with-poll_module`, `--without-poll_module` | Enables or disables building a module that enables NGINX to work with the `poll()` method. The module is built automatically if the platform does not appear to support more appropriate methods such as kqueue, epoll, or /dev/poll. |

**Selecting the NGINX Modules to Build**

NGINX consists of a set of function‑specific _modules_, which are specified with `configure` script along with other build options.

Some modules are built by default – they do not have to be specified with the `configure` script. Default modules can however be explicitly excluded from the NGINX binary with the `--without-<MODULE-NAME>` option on the `configure` script.

Modules not included by default, as well as third‑party modules, must be explicitly specified in the `configure` script together with other build options. Such modules can be linked to NGINX binary either _statically_ \(they are then loaded each time NGINX starts\) or _dynamically_ \(they are loaded only if associated directives are included in the NGINX configuration file.

**Modules Built by Default**

If you do not need a module that is built by default, you can disable it by naming it with the `--without-<MODULE-NAME>` option on the `configure` script, as in this example which disables the [Empty GIF](https://nginx.org/en/docs/http/ngx_http_empty_gif_module.html) module \(should be typed as a single line\):

```text
$ ./configure
--sbin-path=/usr/local/nginx/nginx
--conf-path=/usr/local/nginx/nginx.conf
--pid-path=/usr/local/nginx/nginx.pid
--with-http_ssl_module
--with-stream
--with-pcre=../pcre-8.43
--with-zlib=../zlib-1.2.11
--without-http_empty_gif_module
```

| **Module Name** | **Description** |
| :--- | :--- |
| [http\_access\_module](https://nginx.org/en/docs/http/ngx_http_access_module.html) | Accepts or denies requests from specified client addresses. |
| [http\_auth\_basic\_module](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html) | Limits access to resources by validating the user name and password using the HTTP Basic Authentication protocol. |
| [http\_autoindex\_module](https://nginx.org/en/docs/http/ngx_http_autoindex_module.html) | Processes requests ending with the forward-slash character \(_/_\) and produces a directory listing. |
| [http\_browser\_module](https://nginx.org/en/docs/http/ngx_http_browser_module.html) | Creates variables whose values depend on the value of the `User-Agent` request header. |
| [http\_charset\_module](https://nginx.org/en/docs/http/ngx_http_charset_module.html) | Adds the specified character set to the `Content-Type` response header. Can convert data from one character set to another. |
| [http\_empty\_gif\_module](https://nginx.org/en/docs/http/ngx_http_empty_gif_module.html) | Emits a single-pixel transparent GIF. |
| [http\_fastcgi\_module](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html) | Passes requests to a FastCGI server. |
| [http\_geo\_module](https://nginx.org/en/docs/http/ngx_http_geo_module.html) | Creates variables with values that depend on the client IP address. |
| [http\_gzip\_module](https://nginx.org/en/docs/http/ngx_http_gzip_module.html) | Compresses responses using gzip, reducing the amount of transmitted data by half or more. |
| [http\_limit\_conn\_module](https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html) | Limits the number of connections per a defined key, in particular, the number of connections from a single IP address. |
| [http\_limit\_req\_module](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html) | Limits the request processing rate per a defined key, in particular, the processing rate of requests coming from a single IP address. |
| [http\_map\_module](https://nginx.org/en/docs/http/ngx_http_map_module.html) | Creates variables whose values depend on the values of other variables. |
| [http\_memcached\_module](https://nginx.org/en/docs/http/ngx_http_memcached_module.html) | Passes requests to a memcached server. |
| [http\_proxy\_module](https://nginx.org/en/docs/http/ngx_http_proxy_module.html) | Passes HTTP requests to another server. |
| [http\_referer\_module](https://nginx.org/en/docs/http/ngx_http_referer_module.html) | Blocks requests with invalid values in the Referer header. |
| [http\_rewrite\_module](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html) | Changes the request URI using regular expressions and return redirects; conditionally selects configurations. Requires the [PCRE](http://pcre.org/) library. |
| [http\_scgi\_module](https://nginx.org/en/docs/http/ngx_http_scgi_module.html) | Passes requests to an SCGI server. |
| [http\_ssi\_module](https://nginx.org/en/docs/http/ngx_http_ssi_module.html) | Processes SSI \(Server Side Includes\) commands in responses passing through it. |
| [http\_split\_clients\_module](https://nginx.org/en/docs/http/ngx_http_split_clients_module.html) | Creates variables suitable for A/B testing, also known as split testing. |
| [http\_upstream\_hash\_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#hash) | Enables the generic Hash load-balancing method. |
| [http\_upstream\_ip\_hash\_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#ip_hash) | Enables the IP Hash load-balancing method. |
| [http\_upstream\_keepalive\_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) | Enables keepalive connections. |
| [http\_upstream\_least\_conn\_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#least_conn) | Enables the Least Connections load-balancing method. |
| [http\_upstream\_zone\_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone) | Enables shared memory zones. |
| [http\_userid\_module](https://nginx.org/en/docs/http/ngx_http_userid_module.html) | Sets cookies suitable for client identification. |
| [http\_uwsgi\_module](https://nginx.org/en/docs/http/ngx_http_uwsgi_module.html) | Passes requests to a uwsgi server. |

**Including Modules Not Built by Default**

Many NGINX modules are not built by default, and must be listed on the `configure` command line to be built.

The [mail](https://nginx.org/en/docs/mail/ngx_mail_core_module.html), [stream](https://nginx.org/en/docs/stream/ngx_stream_core_module.html), [geoip](https://nginx.org/en/docs/http/ngx_http_geoip_module.html), [image\_filter](https://nginx.org/en/docs/http/ngx_http_image_filter_module.html), [perl](https://nginx.org/en/docs/http/ngx_http_perl_module.html) and [xslt](https://nginx.org/en/docs/http/ngx_http_xslt_module.html) modules can be compiled as dynamic. See [Dynamic Modules](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/) for details.

An example of the `configure` command that includes nondefault modules \(should be typed as a single line\):

```text
$ ./configure
--sbin-path=/usr/local/nginx/nginx
--conf-path=/usr/local/nginx/nginx.conf
--pid-path=/usr/local/nginx/nginx.pid
--with-pcre=../pcre-8.43
--with-zlib=../zlib-1.2.11
--with-http_ssl_module
--with-stream
--with-mail
```

| **Module Name** | **Description** |
| :--- | :--- |
| `--with-cpp_test_module` | Tests the C++ compatibility of header files. |
| `--with-debug` | Enables the [debugging log](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/). |
| `--with-file-aio` | Enables asynchronous I/O. |
| [\`\`](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#id2)–[\`\`](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#id4)[with-google\_perftools\_module](https://nginx.org/en/docs/ngx_google_perftools_module.html) | Allows using [Google Performance tools](https://github.com/gperftools/gperftools) library. |
| `--` [with-http\_addition\_module](https://nginx.org/en/docs/http/ngx_http_addition_module.html) | Adds text before and after a response. |
| `--` [with-http\_auth\_request\_module](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html) | Implements client authorization based on the result of a subrequest. |
| `--` [with-http\_dav\_module](https://nginx.org/en/docs/http/ngx_http_dav_module.html) | Enables file management automation using the WebDAV protocol. |
| `--with-http_degradation_module` | Allows returning an error when a memory size exceeds the defined value. |
| `--` [with-http\_flv\_module](https://nginx.org/en/docs/http/ngx_http_flv_module.html) | Provides pseudo-streaming server-side support for Flash Video \(FLV\) files. |
| `--` [with-http\_geoip\_module](https://nginx.org/en/docs/http/ngx_http_geoip_module.html) | Enables creating variables whose values depend on the client IP address. The module uses [MaxMind](https://www.maxmind.com/) GeoIP databases. To compile as a separate [dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/) instead, change the option to –with-http\_geoip\_module=dynamic. |
| `--` [with-http\_gunzip\_module](https://nginx.org/en/docs/http/ngx_http_gunzip_module.html) | Decompresses responses with Content-Encoding: gzip for clients that do not support the \_zip\_ encoding method. |
| `--` [with-http\_gzip\_static\_module](https://nginx.org/en/docs/http/ngx_http_gzip_static_module.html) | Allows sending precompressed files with the **.gz** filename extension instead of regular files. |
| `--` [with-http\_image\_filter\_module](https://nginx.org/en/docs/http/ngx_http_image_filter_module.html) | Transforms images in JPEG, GIF, and PNG formats. The module requires the [LibGD](https://libgd.github.io/) library. To compile as a separate [dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/) instead, change the option to `--with-http_image_filter_module=dynamic`. |
| `--` [with-http\_mp4\_module](https://nginx.org/en/docs/http/ngx_http_mp4_module.html) | Provides pseudo-streaming server-side support for MP4 files. |
| `--` [with-http\_perl\_module](https://nginx.org/en/docs/http/ngx_http_perl_module.html) | Used to implement location and variable handlers in Perl and insert Perl calls into SSI. Requires the [PERL](https://www.perl.org/get.html) library. To compile as a separate [dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/) instead, change the option to `--with-http_perl_module=dynamic`. |
| `--` [with-http\_random\_index\_module](https://nginx.org/en/docs/http/ngx_http_random_index_module.html) | Processes requests ending with the slash character \(‘/’\) and picks a random file in a directory to serve as an index file. |
| `--` [with-http\_realip\_module](https://nginx.org/en/docs/http/ngx_http_realip_module.html) | Changes the client address to the one sent in the specified header field. |
| `--` [with-http\_secure\_link\_module](https://nginx.org/en/docs/http/ngx_http_secure_link_module.html) | Used to check authenticity of requested links, protect resources from unauthorized access, and limit link lifetime. |
| `--` [with-http\_slice\_module](https://nginx.org/en/docs/http/ngx_http_slice_module.html) | Allows splitting a request into subrequests, each subrequest returns a certain range of response. Provides more effective caching of large files. |
| `--` [with-http\_ssl\_module](https://nginx.org/en/docs/http/ngx_http_ssl_module.html) | Enables HTTPS support. Requires an SSL library such as [OpenSSL](https://www.openssl.org/). |
| `--` [with-http\_stub\_status\_module](https://nginx.org/en/docs/http/ngx_http_stub_status_module.html) | Provides access to basic status information. Note that NGINX Plus customers do not require this module as they are already provided with extended status metrics and interactive dashboard. |
| `--` [with-http\_sub\_module](https://nginx.org/en/docs/http/ngx_http_sub_module.html) | Modifies a response by replacing one specified string by another. |
| `--` [with-http\_xslt\_module](https://nginx.org/en/docs/http/ngx_http_xslt_module.html) | Transforms XML responses using one or more XSLT stylesheets. The module requires the [Libxml2](http://xmlsoft.org/) and [XSLT](http://xmlsoft.org/XSLT/) libraries. To compile as a separate [dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/) instead, change the option to `--with-http_xslt_module=dynamic`. |
| `--` [with-http\_v2\_module](https://nginx.org/en/docs/http/ngx_http_v2_module.html) | Enable support for [HTTP/2](https://tools.ietf.org/html/rfc7540%29). See [The HTTP/2 Module in NGINX](https://www.nginx.com/blog/http2-module-nginx/) on the NGINX blog for details. |
| `--` [with-mail](https://nginx.org/en/docs/mail/ngx_mail_core_module.html) | Enables mail proxy functionality. To compile as a separate [dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/) instead, change the option to `--with-mail=dynamic`. |
| `--` [with-mail\_ssl\_module](https://nginx.org/en/docs/mail/ngx_mail_ssl_module.html) | Provides support for a mail proxy server to work with the SSL/TLS protocol. Requires an SSL library such as [OpenSSL](https://www.openssl.org/). |
| `--` [with-stream](https://nginx.org/en/docs/stream/ngx_stream_core_module.html) | Enables the TCP and UDP proxy functionality. To compile as a separate [dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/) instead, change the option to `--with-stream=dynamic`. |
| `--` [with-stream\_ssl\_module](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html) | Provides support for a stream proxy server to work with the SSL/TLS protocol. Requires an SSL library such as [OpenSSL](https://www.openssl.org/). |
| `--with-threads` | Enables NGINX to use thread pools. For details, see [Thread Pools in NGINX Boost Performance 9x!](https://www.nginx.com/blog/thread-pools-boost-performance-9x/) on the NGINX blog. |

**Including Third-Party Modules**

You can extend NGINX functionality by compiling NGINX open source with your own module or a third‑party module. Some third‑party modules are listed in the [NGINX Wiki](https://nginx.com/resources/wiki/modules/). Use third‑party modules at your own risk as their stability is not guaranteed.

**Statically Linked Modules**

Most modules built into NGINX open source are _statically linked_: they are built into NGINX open source at compile time and are linked to the NGINX binary statically. These modules can be disabled only by recompiling NGINX.

To compile NGINX open source with a statically linked third‑party module, include the `--add-module=<PATH>` option on the `configure` command, where `<PATH>` is the path to the source code \(this example is for the [RTMP](https://github.com/arut/nginx-rtmp-module) module\):

```text
$  ./configure ... --add-module=/usr/build/nginx-rtmp-module
```

**Dynamically Linked Modules**

NGINX modules can also be compiled as a shared object \(**\*.so** file\) and then dynamically loaded into NGINX open source at runtime. This provides more flexibility, as the module can be loaded or unloaded at any time by adding or removing the associated [`load_module`](https://nginx.org/en/docs/ngx_core_module.html#load_module) directive in the NGINX configuration file and reloading the configuration. Note that the module itself must support dynamic linking.

To compile NGINX open source with a dynamically loaded third‑party module, include the `--add-dynamic-module=<PATH>` option on the `configure` command, where `<PATH>` is the path to the source code:

```text
$  ./configure ... --add-dynamic-module=<PATH>
```

The resulting **\*.so** files are written to the the _prefix_**/modules/** directory, where the _prefix_ is a directory for server files such as **/usr/local/nginx/**.

To load a dynamic module, add the [`load_module`](https://nginx.org/en/docs/ngx_core_module.html#load_module) directive to the NGINX configuration after installation:

```text
load_module modules/ngx_mail_module.so;
```

For more information, see [Compiling Third‑Party Dynamic Modules for NGINX and NGINX Plus](https://www.nginx.com/blog/compiling-dynamic-modules-nginx-plus/) on the NGINX blog and [Extending NGINX](https://nginx.com/resources/wiki/extending/) in the Wiki.

#### Completing the Installation from Source

* Compile and install the build:

  ```text
  $ make
  $ sudo make install
  ```

* After the installation is finished, start NGINX open source:

  ```text
  $ sudo nginx
  ```

