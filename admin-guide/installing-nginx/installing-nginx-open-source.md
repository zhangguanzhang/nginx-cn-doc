# Installing NGINX Open Source

本文介绍如何安装开源的nginx

* [选择Stable或者Mainline版本](installing-nginx-open-source.md#choosing-between-a-stable-or-a-mainline-version)
* [选择包管理器\(预构建包\)和源码编译](installing-nginx-open-source.md#choosing-between-a-prebuilt-package-and-compiling-from-source)
* [从包管理安装](installing-nginx-open-source.md#installing-a-prebuilt-package)
  * [包管理中包含的模块](installing-nginx-open-source.md#modules-included-in-a-prebuilt-package)
  * [在CentOS和RHEL上使用包管理安装](installing-nginx-open-source.md#installing-prebuilt-centos-and-rhel-packages)
  * [Installing Prebuilt Debian Packages](installing-nginx-open-source.md#prebuilt_debian)
  * [Installing Prebuilt Ubuntu Packages](installing-nginx-open-source.md#prebuilt_ubuntu)
  * [Installing SUSE Packages](installing-nginx-open-source.md#prebuilt_suse)
  * [Installing Alpine Linux Packages](installing-nginx-open-source.md#prebuilt_alpine)
* [从源码编译安装](installing-nginx-open-source.md#compiling-and-installing-from-source)
  * [安装编译nginx的依赖](installing-nginx-open-source.md#dependencies)
  * [下载源码](installing-nginx-open-source.md#configuring-the-build-options)
  * [配置构建选项](installing-nginx-open-source.md#configuring-the-build-options)
    * [配置nginx相关PATH](installing-nginx-open-source.md#configuring-nginx-paths)
    * [配置nginx的gcc选项](installing-nginx-open-source.md#configuring-nginx-gcc-options)
    * [指定NGINX连接处理的方法](installing-nginx-open-source.md#configure_methods)
    * [选择模块来构建NGINX](installing-nginx-open-source.md#modules)
    * [默认构建的模块](installing-nginx-open-source.md#modules_default)
    * [默认情况下不构建的模块](installing-nginx-open-source.md#modules_not_default)
    * [引入第三方模块](installing-nginx-open-source.md#modules_third_party)
  * [从源码完成安装](installing-nginx-open-source.md#install_complete)

### 选择Stable或者Mainline版本 <a id="choosing-between-a-stable-or-a-mainline-version"></a>

开源nginx可用的两种版本:

* **Mainline** – 包括最新功能和错误修复，并始终是最新的。 它是可靠的但可能包括一些实验模块，它也可能引入一些新的bug。
* **Stable** – 不包含所有最新功能，只具有关键错误的修复，始终向后移植到主线版本。 建议生产服务器的稳定版本。

### 选择包管理器\(预构建包\)和源码编译 <a id="choosing-between-a-prebuilt-package-and-compiling-from-source"></a>

无论是**Stable**还是**Mainline**版本无非是以下两种安装方式:

* 预构建包: 简单快速安装开源nginx的方式. 几乎包含了所有主流的官方模块，可用于大多数流行的操作系统. See [Installing a Prebuilt Package](installing-nginx-open-source.md#prebuilt).
* 从源码编译: 这种方式更灵活：您可以添加特定模块，包括第三方模块，或应用最新的安全补丁. See [Compiling and Installing from Source](installing-nginx-open-source.md#sources) for details.

### 从包管理安装 <a id="installing-a-prebuilt-package"></a>

从包管理安装开源NGINX比从源代码构建要容易和快捷得多，但是从源代码构建使您可以在非标准模块中进行编译。 预构建的软件包可用于大多数流行的Linux发行版，包括CentOS，Debian，Red Hat Enterprise Linux（RHEL），SUSE Linux Enterprise Server（SLES）和Ubuntu。 有关当前支持的操作系统的列表，请参见[nginx.org](https://nginx.org/en/linux_packages.html?_ga=2.195891113.440084818.1573281842-1746613347.1571715156)上的Linux软件包。

#### 包管理中包含的模块 <a id="modules-included-in-a-prebuilt-package"></a>

有关每个预构建软件包中包含的模块列表，请见 **nginx.org** 上的[Configure Arguments](https://nginx.org/en/linux_packages.html#arguments)。

#### 在CentOS和RHEL上使用包管理安装 <a id="installing-prebuilt-centos-and-rhel-packages"></a>

NGINX，Inc.提供用于以下CentOS，Oracle Linux和RHEL版本的软件包：

| **版本** | **支持的架构** |
| :--- | :--- |
| 6.x | x86\_64, i386 |
| 7.4+ | x86\_64, ppc64le |
| 8.x | x86\_64 |

可以从下列方式安装:

* 默认的RHEL或CentOS repo库。 这是最快的方法，但是通常提供的软件包已过时。
* 官方仓库位于 **nginx.org**. 你必须初次手动添加 `yum` repo, 在添加后，后续使用yum安装都是最新的.

**在CentOS/RHEL系统自带的repo利用包管理安装** <a id="installing-prebuilt-centos-and-rhel-packages"></a>

1. 安装 EPEL repo:

   ```text
   $ sudo yum install epel-release
   ```

2. 更新 repository:

   ```text
   $ sudo yum update
   ```

3. 安装开源NGINX:

   ```text
   $ sudo yum install nginx
   ```

4. 确认安装:

   ```text
   $ sudo nginx -v
   nginx version: nginx/1.6.3
   ```

**添加NGINX官方repo在CentOS/RHEL系统上安装** <a id="installing-a-prebuilt-centos-rhel-package-from-the-official-nginx-repository"></a>

1. 通过在 **/etc/yum.repos.d** 中创建文件 **nginx.repo** 来为RHEL/CentOS设置`yum`的repo库，例如使用`vi`：:

   ```text
   $ sudo vi /etc/yum.repos.d/nginx.repo
   ```

2. 在文件**nginx.repo**中添加下列内容:

   ```text
   [nginx]
   name=nginx repo
   baseurl=https://nginx.org/packages/mainline/<OS>/<OSRELEASE>/$basearch/
   gpgcheck=0
   enabled=1
   ```

   where:

   * baseurl中的 `/mainline` 元素指向NGINX开源的最新mainline版本，如果你想使用stable版本，删除掉该字段即可
   * `<OS>` 字段可以是 `rhel` 或者 `centos`
   * `<OSRELEASE>` 可以是 \(`6`, `6._x_`, `7`, `7._x_` and so on\)

     例如，要获取用于CentOS 7的最新mainline软件包，:

     ```text
     [nginx]
     name=nginx repo
     baseurl=https://nginx.org/packages/mainline/centos/7/$basearch/
     gpgcheck=0
     enabled=1
     ```

3. 保存更改并退出 `vi` \(press ESC and type `wq` at the `:` prompt\).
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

#### Installing Prebuilt Debian Packages <a id="prebuilt_debian"></a>

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

   * `/mainline` 元素指向NGINX开源的最新mainline版本，如果你想使用stable版本，删除掉该字段即可
   * `<CODENAME>` 是 Debian release的名字

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

#### Installing Prebuilt Ubuntu Packages <a id="prebuilt_ubuntu"></a>

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

#### Installing SUSE Packages <a id="prebuilt_suse"></a>

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

#### Installing Prebuilt Alpine Linux Packages <a id="prebuilt_alpine"></a>

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

### 从源码编译安装 <a id="compiling-and-installing-from-source"></a>

从源代码编译NGINX开源程序比预编译的软件包具有更大的灵活性：您可以添加特定的模块（来自NGINX或第三方），并应用最新的安全补丁。

#### 安装编译nginx的依赖 <a id="installing-nginx-dependencies"></a>

在从源代码编译NGINX开源之前，您需要安装依赖项的库:

* [PCRE](http://pcre.org/) – 支持正则表达式. NGINX [Core](https://nginx.org/en/docs/ngx_core_module.html) and [Rewrite](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html) 模块需要.

  ```text
  $ wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.43.tar.gz
  $ tar -zxf pcre-8.43.tar.gz
  $ cd pcre-8.43
  $ ./configure
  $ make
  $ sudo make install
  ```

* [zlib](https://www.zlib.net/) – 支持http的header压缩. 被NGINX [Gzip](https://nginx.org/en/docs/http/ngx_http_gzip_module.html) 模块需要.

  ```text
  $ wget http://zlib.net/zlib-1.2.11.tar.gz
  $ tar -zxf zlib-1.2.11.tar.gz
  $ cd zlib-1.2.11
  $ ./configure
  $ make
  $ sudo make install
  ```

* [OpenSSL](https://www.openssl.org/) – 支持HTTPS协议. 被NGINX [SSL](https://nginx.org/en/docs/http/ngx_http_ssl_module.html) 模块和一些其他的模块需要.

  ```text
  $ wget http://www.openssl.org/source/openssl-1.1.1c.tar.gz
  $ tar -zxf openssl-1.1.1c.tar.gz
  $ cd openssl-1.1.1c
  $ ./Configure darwin64-x86_64-cc --prefix=/usr
  $ make
  $ sudo make install
  ```

#### 下载源码 <a id="downloading-the-sources"></a>

在页面 [**nginx.org**](https://www.nginx.org/en/download.html)可以下载到 `stable` 和 `mainline` 版本

下载并解压最新 `mainline` 版本的源码:

```text
$ wget https://nginx.org/download/nginx-1.17.3.tar.gz
$ tar zxf nginx-1.17.3.tar.gz
$ cd nginx-1.17.3
```

下载并解压最新 `stable` 版本的源码:

```text
$ wget https://nginx.org/download/nginx-1.16.1.tar.gz
$ tar zxf nginx-1.16.1.tar.gz
$ cd nginx-1.16.1
```

#### 配置构建选项 <a id="configuring-the-build-options"></a>

配置选项由 `./configure` 脚本指定，该脚本设置各种NGINX参数，包括源文件和配置文件的路径，编译器选项，连接处理方法以及模块列表。 该脚本通过创建编译代码并安装NGINX开源程序所需的`Makefile`结束。

下面是一个简单的 `configure` 实例 \(为了方便修改和阅读，一般是利用shell的转义斜线来断成一行行\):

```text
$ ./configure \
  --sbin-path=/usr/local/nginx/nginx \
  --conf-path=/usr/local/nginx/nginx.conf \
  --pid-path=/usr/local/nginx/nginx.pid \
  --with-pcre=../pcre-8.43 \
  --with-zlib=../zlib-1.2.11 \
  --with-http_ssl_module \
  --with-stream \
  --with-mail=dynamic \
  --add-module=/usr/build/nginx-rtmp-module \
  --add-dynamic-module=/usr/build/3party_module \
```

**配置nginx相关PATH** <a id="configuring-nginx-paths"></a>

 `configure` 脚本允许您设置NGINX二进制文件和配置文件以及相关库（例如PCRE或SSL）的路径，以便将它们静态链接到NGINX二进制文件。

| **参数** | **描述** |
| :--- | :--- |
| `--prefix=<PATH>` | nginx安装在哪个目录, 以及其他配置脚本选项\（不包括库路径\）设置的所有相对路径以及 **nginx.conf** 配置文件路径的基本位置。 默认: **/usr/local/nginx**. |
| `--sbin-path=<PATH>` | NGINX可执行文件的名称，仅在安装期间使用。 默认: **&lt;prefix&gt;/sbin/nginx** |
| `--conf-path=<PATH>` | NGINX配置文件的名称。 但是，您可以始终在启动时通过在nginx命令行上使用-c <FILENAME>选项指定其他文件来覆盖此值。 默认: **&lt;prefix&gt;/conf/nginx.conf** |
| `--pid-path=<PATH>` | 文件 **nginx.pid** 的名字, 用于存储master master进程的进程pid. 在安装后可以通过配置文件来[更改pid文件名的路径](https://nginx.org/en/docs/ngx_core_module.html#pid)。默认: **&lt;prefix&gt;/logs/nginx.pid** |
| `--error-log-path=<PATH>` | errors, warnings, diagnostic日志文件路径. 在安装后可以通过配置文件来[更改error\_log的路径](https://nginx.org/en/docs/ngx_core_module.html#error_log)。默认: **&lt;prefix&gt;/logs/error.log** |
| `--http-log-path=<PATH>` | 常规请求日志存放路径. 在安装后可以通过配置文件来[更改access\_log的路径](https://nginx.org/en/docs/http/ngx_http_log_module.html#access_log)。默认: **&lt;prefix&gt;/logs/access.log** |
| `--user=<NAME>` | NGINX工作进程使用其凭据的非特权用户的名称. 在安装后可以通过配置文件来[更改user](https://nginx.org/en/docs/ngx_core_module.html#user)。默认: `nobody` |
| `--group=<NAME>` | NGINX工作进程使用的用户组名称. 在安装后可以通过配置文件来[更改user](https://nginx.org/en/docs/ngx_core_module.html#user)。默认: 由 `--user` 选项设置的值. |
| `--with-pcre=<PATH>` | PCRE源码路径, 正则表达式会在[location](https://nginx.org/en/docs/http/ngx_http_core_module.html#location)和[Rewrite](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html)模块中使用. |
| `--with-pcre-jit` | 带上pcre的时候带上 `just-in-time compilation` 会支持 \(the [pcre\_jit](https://nginx.org/en/docs/ngx_core_module.html#pcre_jit) directive\). |
| `--with-zlib=<PATH>` | zlib源码路径, 被[Gzip](https://nginx.org/en/docs/http/ngx_http_gzip_module.html)模块依赖. |

**配置nginx的gcc选项** <a id="configuring-nginx-gcc-options"></a>

With the `configure` script you can also specify compiler‑related options.

| **参数** | **描述** |
| :--- | :--- |
| `--with-cc-opt="<PARAMETERS>"` | 添加参数到`CFLAGS` 变量里.当在FreeBSD下使用PCRE时必须设置 `--with-cc-opt="-I /usr/local/include"`. 如果需要增加 `select()` 支持的文件数: `--with-cc-opt="-D FD_SETSIZE=2048"`. |
| `--with-ld-opt="<PARAMETERS>"` | ld链接起见使用的其他参数.当在FreeBSD下使用PCRE时必须设置 `--with-ld-opt="-L /usr/local/lib"`. |

**指定NGINX连接处理的方法** <a id="configure_methods"></a>

使用`configure`脚本，你可以重新定义基于事件轮询处理连接， 有关更多信息，请参见NGINX参考文档中的[Connection processing methods](https://nginx.org/en/docs/events.html).

| **Module Name** | **Description** |
| :--- | :--- |
| `--with-select_module`, `--without-select_module` | 启用或者禁用允许NGINX使用 `select()` 模块. 如果平台似乎不支持更合适的方法(如kqueue、epoll或/dev/poll)，则自动构建模块|
| `--with-poll_module`, `--without-poll_module` | 启用或禁用构建允许NGINX使用`poll()`模块。如果平台似乎不支持更合适的方法(如kqueue、epoll或/dev/poll)，则自动构建模块。 |

**选择模块来构建NGINX** <a id="modules"></a>

NGINX由一组特定于函数的 *_modules_* 组成，这些 *_modules_* 由`configure`脚本配置构建选项开启。

有些模块是默认构建的，它们不需要通过 `configure` 脚本指定。但是，可以 `--without-<MODULE-NAME>` 选项显式地从NGINX二进制文件中排除默认模块。

默认情况下不包括的模块，以及第三方模块，必须在 `configure` 脚本中与其他构建选项一起显式指定。这些模块可以链接到NGINX二进制文件 _statically_ (然后在每次NGINX启动\时加载它们)或 _dynamically_ (只有当相关的指令包含在NGINX配置文件中时才加载它们)。

**默认构建的模块** <a id="modules_default"></a>

如果你不需要一个默认开启的模块, 可以通过在 `configure` 后加名命规则 `--without-<MODULE-NAME>` , 例如关闭 [Empty GIF](https://nginx.org/en/docs/http/ngx_http_empty_gif_module.html) 模块:

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
| [http_access_module](https://nginx.org/en/docs/http/ngx_http_access_module.html) | 接受或拒绝来自指定客户端地址的请求. |
| [http_auth_basic_module](https://nginx.org/en/docs/http/ngx_http_auth_basic_module.html) | 通过使用HTTP基本身份验证协议验证用户名和密码来限制对资源的访问. |
| [http_autoindex_module](https://nginx.org/en/docs/http/ngx_http_autoindex_module.html) | 处理以正斜杠字符(/)结尾的请求并生成目录清单. |
| [http_browser_module](https://nginx.org/en/docs/http/ngx_http_browser_module.html) | 创建变量，这些变量的值依赖于 `User-Agent` 请求头的值. |
| [http_charset_module](https://nginx.org/en/docs/http/ngx_http_charset_module.html) | 将指定的字符集添加到 `Content-Type` 响应头。可以将数据从一个字符集转换为另一个字符集. |
| [http_empty_gif_module](https://nginx.org/en/docs/http/ngx_http_empty_gif_module.html) | 发出单像素透明GIF. |
| [http_fastcgi_module](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html) | 转发请求到 FastCGI server. |
| [http_geo_module](https://nginx.org/en/docs/http/ngx_http_geo_module.html) | 创建具有依赖于客户端IP地址的值的变量. |
| [http_gzip_module](https://nginx.org/en/docs/http/ngx_http_gzip_module.html) | 使用gzip压缩响应，将传输的数据量减少一半或更多. |
| [http_limit_conn_module](https://nginx.org/en/docs/http/ngx_http_limit_conn_module.html) | 限制每个已定义键的连接数，特别是来自单个IP地址的连接数. |
| [http_limit_req_module](https://nginx.org/en/docs/http/ngx_http_limit_req_module.html) | 限制每个已定义键的请求处理速率，特别是来自单个IP地址的请求处理速率. |
| [http_map_module](https://nginx.org/en/docs/http/ngx_http_map_module.html) | 创建变量，其值依赖于其他变量的值. |
| [http_memcached_module](https://nginx.org/en/docs/http/ngx_http_memcached_module.html) | 转发请求到 memcached server. |
| [http_proxy_module](https://nginx.org/en/docs/http/ngx_http_proxy_module.html) | 代理 HTTP 请求到另一个 http server. |
| [http_referer_module](https://nginx.org/en/docs/http/ngx_http_referer_module.html) | 阻塞`Referer header`中具有无效值的请求. |
| [http_rewrite_module](https://nginx.org/en/docs/http/ngx_http_rewrite_module.html) | 使用正则表达式更改请求URI并返回重定向;有条件地选择配置. 需要[PCRE](http://pcre.org/)库. |
| [http_scgi_module](https://nginx.org/en/docs/http/ngx_http_scgi_module.html) | 转发请求到 SCGI server. |
| [http_ssi_module](https://nginx.org/en/docs/http/ngx_http_ssi_module.html) | 处理通过SSI(包含服务端)命令的响应. |
| [http_split_clients_module](https://nginx.org/en/docs/http/ngx_http_split_clients_module.html) | 创建适用于 `A/B` 测试（也称为拆分测试）的变量. |
| [http_upstream_hash_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#hash) | 启用通用哈希负载平衡方法. |
| [http_upstream_ip_hash_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#ip_hash) | 启用IP散列负载平衡方法. |
| [http_upstream_keepalive_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#keepalive) | 启用连接保持. |
| [http_upstream_least_conn_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#least_conn) | 启用最少连接负载平衡方法. |
| [http_upstream_zone_module](https://nginx.org/en/docs/http/ngx_http_upstream_module.html#zone) | 启用共享内存区域. |
| [http_userid_module](https://nginx.org/en/docs/http/ngx_http_userid_module.html) | 设置适合于客户端标识的cookie. |
| [http_uwsgi_module](https://nginx.org/en/docs/http/ngx_http_uwsgi_module.html) | 转发请求到 uwsgi server. |

**默认情况下不构建的模块** <a id="modules_not_default"></a>

许多NGINX模块默认不构建的，必须在 `configure` 命令行中开启。

[mail](https://nginx.org/en/docs/mail/ngx_mail_core_module.html), [stream](https://nginx.org/en/docs/stream/ngx_stream_core_module.html), [geoip](https://nginx.org/en/docs/http/ngx_http_geoip_module.html), [image\_filter](https://nginx.org/en/docs/http/ngx_http_image_filter_module.html), [perl](https://nginx.org/en/docs/http/ngx_http_perl_module.html) 和 [xslt](https://nginx.org/en/docs/http/ngx_http_xslt_module.html) 等模块可以动态编译. 可以从[Dynamic Modules](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/)查阅相关信息.

一个包含非默认模块的 `configure` 命令示例 (应该作为单行输入):

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
| `--with-debug` | 开启 [debugging log](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/). |
| `--with-file-aio` | 启用 异步I/O. |
| [--with-google_perftools_module](https://nginx.org/en/docs/ngx_google_perftools_module.html) | 允许使用[Google Performance tools](https://github.com/gperftools/gperftools)库. |
| [--with-http_addition_module](https://nginx.org/en/docs/http/ngx_http_addition_module.html) | 在响应之前和之后添加文本. |
| [--with-http_auth_request_module](https://nginx.org/en/docs/http/ngx_http_auth_request_module.html) | 基于子请求的结果实现客户端授权. |
| [--with-http_dav_module](https://nginx.org/en/docs/http/ngx_http_dav_module.html) | 使用WebDAV协议启用文件管理自动化. |
| `--with-http_degradation_module` | 允许在内存大小超过定义的值时返回错误. |
| [--with-http_flv_module](https://nginx.org/en/docs/http/ngx_http_flv_module.html) | 为Flash视频(FLV)文件提供伪流服务器端支持. |
| [--with-http_geoip_module](https://nginx.org/en/docs/http/ngx_http_geoip_module.html) | 允许创建其值依赖于客户端IP地址的变量。该模块使用[MaxMind](https://www.maxmind.com/) GeoIP 数据库. 要编译为一个单独的 [dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/) 替代的话, 改为 `--with-http_geoip_module=dynamic`. |
| [--with-http_gunzip_module](https://nginx.org/en/docs/http/ngx_http_gunzip_module.html) | 对于不支持 _zip_ 编码的客户端，使用 `Content-Encoding: gzip`解压响应. |
| [--with-http_gzip_static_module](https://nginx.org/en/docs/http/ngx_http_gzip_static_module.html) | 允许发送扩展名为 **.gz** 的预压缩文件，而不是常规文件. |
| [--with-http_image_filter_module](https://nginx.org/en/docs/http/ngx_http_image_filter_module.html) | 转换JPEG、GIF和PNG格式的图像。需要 [LibGD](https://libgd.github.io/) 库编译成[dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/) 替代的话, 选项更改为 `--with-http_image_filter_module=dynamic`. |
| [--with-http_mp4_module](https://nginx.org/en/docs/http/ngx_http_mp4_module.html) | 为MP4文件提供伪流服务器端支持. |
| [--with-http_perl_module](https://nginx.org/en/docs/http/ngx_http_perl_module.html) | 用于在Perl中实现位置和变量处理程序，并将Perl调用插入到SSI中。该模块需要需要 [PERL](https://www.perl.org/get.html) 库。需要编译成[dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/) 替代的话, 选项更改为 `--with-http_perl_module=dynamic`. |
| [--with-http_random_index_module](https://nginx.org/en/docs/http/ngx_http_random_index_module.html) | 处理以斜杠字符(‘/’)结尾的请求，并在目录中选择一个随机文件作为索引文件. |
| [--with-http_realip_module](https://nginx.org/en/docs/http/ngx_http_realip_module.html) | 将客户端地址更改为在指定头字段中发送的地址. |
| [--with-http_secure_link_module](https://nginx.org/en/docs/http/ngx_http_secure_link_module.html) | 用于检查请求链接的真实性，保护资源不受未经授权的访问，并限制链接的生存期. |
| [--with-http_slice_module](https://nginx.org/en/docs/http/ngx_http_slice_module.html) | 允许将请求拆分为子请求，每个子请求返回一定范围的响应。提供更有效的大文件缓存. |
| [--with-http_ssl_module](https://nginx.org/en/docs/http/ngx_http_ssl_module.html) | 开启 HTTPS 支持. 需要一个SSL库，例如 [OpenSSL](https://www.openssl.org/). |
| [--with-http_stub_status_module](https://nginx.org/en/docs/http/ngx_http_stub_status_module.html) | 提供对基本状态信息的访问。注意，NGINX Plus客户不需要这个模块，因为他们已经提供了扩展的状态度量和交互式仪表板. |
| [--with-http_sub_module](https://nginx.org/en/docs/http/ngx_http_sub_module.html) | 通过将一个指定的字符串替换为另一字符串来修改响应. |
| [--with-http_xslt_module](https://nginx.org/en/docs/http/ngx_http_xslt_module.html) | 使用一个或多个XSLT样式表转换XML响应. 该模块需要 [Libxml2](http://xmlsoft.org/) 和 [XSLT](http://xmlsoft.org/XSLT/) 库。需要编译成 [dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/) 替代的话, 更改选项为 `--with-http_xslt_module=dynamic`. |
| [--with-http_v2_module](https://nginx.org/en/docs/http/ngx_http_v2_module.html) | 开启 [HTTP/2](https://tools.ietf.org/html/rfc7540%29)支持. 再NGINX blog上查看 [The HTTP/2 Module in NGINX](https://www.nginx.com/blog/http2-module-nginx/)获取详细信息. |
| [--with-mail](https://nginx.org/en/docs/mail/ngx_mail_core_module.html) | 开启邮件代理功能. 需要编译为单独的 [dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/) 替代的话, 选项更改为 `--with-mail=dynamic`. |
| [--with-mail_ssl_module](https://nginx.org/en/docs/mail/ngx_mail_ssl_module.html) | 提供对邮件代理服务器使用SSL/TLS协议的支持。 需要SSL库，例如 [OpenSSL](https://www.openssl.org/). |
| [--with-stream](https://nginx.org/en/docs/stream/ngx_stream_core_module.html) | 启用TCP和UDP代理功能。 要将其编译为单独的 [dynamic module](https://docs.nginx.com/nginx/admin-guide/dynamic-modules/dynamic-modules/)替代的话, 更改选项为 `--with-stream=dynamic`. |
| [--with-stream_ssl_module](https://nginx.org/en/docs/stream/ngx_stream_ssl_module.html) | 提供对使用SSL/TLS协议的流代理服务器的支持。需要一个SSL库，如 [OpenSSL](https://www.openssl.org/). |
| `--with-threads` | 让NGINX使用线程池. 有关详细信息，请参见NGINX博客上的 [NGINX使用线程池提高性能9倍!](https://www.nginx.com/blog/thread-pools-boost-performance-9x/). |

**引入第三方模块** <a id="including-third-party-modules"></a>

你可以通过自己的模块或第三方模块编译NGINX开源来扩展NGINX的功能。一些第三方模块列在[NGINX Wiki](https://nginx.com/resources/wiki/modules/). 使用第三方模块的风险需要自己承担，因为他们的稳定性不能得到保证.

**静态链接的模块** <a id="statically-linked-modules"></a>

大多数内置于NGINX开放源码中的模块都是静态链接的:它们在编译时内置于NGINX开放源码中，并静态地链接到NGINX二进制文件。这些模块只能通过重新编译NGINX来禁用。

要用静态链接的第三方模块编译开源版NGINX, 请在 `configure` 添加上 `--add-module=<PATH>` , 其中 `<PATH>` 是源码的路径 (下面是一个[RTMP](https://github.com/arut/nginx-rtmp-module) 模块的示例):

```text
$  ./configure ... --add-module=/usr/build/nginx-rtmp-module
```

**动态链接模块** <a id="dynamically-linked-modules"></a>

NGINX模块可以编译为动态链接 (**\*.so** 文件)，然后在运行时动态加载. 这提供了更大的灵活性, 因为可以通过在NGINX中添加或者删除相关的 [`load_module`](https://nginx.org/en/docs/ngx_core_module.html#load_module) 指令来加载和卸载模块，并重新加载配置. 注意模块本身必须支持动态链接.

要用一个动态加载的第三方模块来编译, `configure` 命令加上 `--add-dynamic-module=<PATH>` , 其中 `<PATH>` 是代码路径:

```text
$  ./configure ... --add-dynamic-module=<PATH>
```

生成的 **\*.so** 文件会被写入到 _prefix_**/modules/** 路径里, _prefix_ 是server文件，例如 **/usr/local/nginx/**.

要加载动态模块，请在安装后将[`load_module`](https://nginx.org/en/docs/ngx_core_module.html#load_module)指令添加到nginx配置文件中:

```text
load_module modules/ngx_mail_module.so;
```

更多信息可以查看 [在NGINX and NGINX Plus编译第三方动态模块](https://www.nginx.com/blog/compiling-dynamic-modules-nginx-plus/) on the NGINX blog and [扩展NGINX](https://nginx.com/resources/wiki/extending/) in the Wiki.

#### 从源码完成安装 <a id="install_complete"></a>

* 编译并安装构建:

  ```text
  $ make
  $ sudo make install
  ```

* 完成安装后启动开源版NGINX:

  ```text
  $ sudo nginx
  ```

