# Debugging NGINX



### Introduction

Debugging helps to identify a bug in the program code if something goes wrong. It is generally used in developing or testing third-party or experimental modules.

NGINX debugging features include the debugging log and creation of a core dump file with its further backtrace.

#### Configuring NGINX Binary For Debugging

First, you will need to enable debugging in NGINX binary. NGINX Plus already provides you with _nginx-debug_ binary while NGINX Open Source requires recompilation.

#### Configuring NGINX Plus Binary

Starting from [Release 8](https://docs.nginx.com/nginx/releases/), NGINX Plus ships the _nginx-debug_ binary together with the standard binary. To enable debugging in NGINX Plus, you will need to switch from _nginx_ to _nginx-debug_ binary. Open terminal and run the command:

```text
$ service nginx stop && service nginx-debug start
```

When finished, [enable](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/#error_log) the debugging log in the configuration file.

#### Compiling NGINX Open Source Binary

To enable debugging in NGINX Open Source, you will need to recompile it with the `--with-debug` flag specified in the configure script.

To compile NGINX Open Source with the debug support:

1. Download and unpack NGINX source files, go to the directory with the source files. See [Downloading the Sources](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/).
2. Get the list of NGINX configure arguments. Run the command:

   ```text
   $ nginx -V 2>&1 | grep arguments
   ```

3. Add the `--with-debug` option to the list of configure commands and run the configure script:

   ```text
   $ ./configure --with-debug <other configure arguments>
   ```

4. Compile and install NGINX:

   ```text
   $ sudo make
   $ sudo make install
   ```

5. Restart NGINX.

#### NGINX and Debugging Symbols

Debug symbols helps obtain additional information for debugging, such as functions, variables, data structures, source file and line number information.

NGINX by default is compiled with the “_-g_” flag that includes debug symbols.

However, if you get the “No symbol table info available” error when you run a [backtrace](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/#backtrace), then debugging symbols are missing and you will need to recompile NGINX with support of debugging symbols.

The exact set of compiler flags depends on the compiler. For example, for the GCC compiler system:

* include debugging symbols with the “_-g_” flag
* make the debugger output easier to understand by disabling compiler optimization with the “_-O0_” flag:

  ```text
  $ ./configure --with-debug --with-cc-opt='-O0 -g' ...
  ```

### Enabling Debug Logging in NGINX Configuration

The debugging log records errors and any debugging-related information and is disabled by default. To enable it, make sure NGINX is compiled to support debugging \(see [Configuring NGINX Binary For Debugging](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/#enable)\) and then enable it in NGINX configuration file with the `debug` parameter of the [`error_log`](https://nginx.org/en/docs/ngx_core_module.html#error_log) directive. The debugging log may be written to a file, an allocated _buffer_ in memory, _stderr_ output, or to _syslog_.

It is recommended enabling the debugging log on the ”_main_“ level of NGINX configuration to get the full picture of what’s going on.

#### Writing the Debugging Log to a File

Writing the debugging log to a file may slow down performance under high load. Also note that the file can grow very large and quickly eat up disk space. To reduce the negative impact, you can configure the debugging log to be written into a memory buffer, or set the debugging log for particular IP addresses. See [Writing the Debugging Log to Memory](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/#error_log_memory) and [Debug Log for Selected IPs](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/#error_log_ip) for details.

To enable writing the debugging log to a file:

1. Make sure your NGINX is configured with the `--with-debug` configuration option. Run the command and check if the output contains the `--with-debug` line:

   ```text
   $ nginx -V 2>&1 | grep -- '--with-debug'
   ```

2. Open NGINX configuration file:

   ```text
   $ sudo vi /etc/nginx/nginx.conf
   ```

3. Find the [`error_log`](https://nginx.org/en/docs/ngx_core_module.html#error_log) directive which is by default located in the `main` context, and change the logging level to `debug`. If necessary, change the path to the log file:

   ```text
   error_log  /var/log/nginx/error.log debug;
   ```

4. Save the configuration and exit the configuration file.

#### Writing the Debugging Log to Memory

The debugging log can be written to a memory using a cyclic buffer. The advantage is that logging on the debug level will not have significant impact on performance under high load.

To enable writing the debug log to memory:

1. Make sure your NGINX is configured with the `--with-debug` configuration option. Run the command and check if the output contains the `--with-debug` line:

   ```text
   $ nginx -V 2>&1 | grep -- '--with-debug'
   ```

2. In NGINX configuration file, enable a memory buffer for debug logging with the [`error_log`](https://nginx.org/en/docs/ngx_core_module.html#error_log) directive specified in the `main` context:

   ```text
   error_log memory:32m debug;
   ...
   http {
       ...
   }
   ```

**Extracting Debug Log From Memory**

The log can be extracted from the memory buffer using a script executed in the GDB debugger.

To extract the debugging log from memory:

1. Obtain the PID of NGINX worker process:

   ```text
   $ ps axu |grep nginx
   ```

2. Launch the GDB debugger:

   ```text
   $ sudo gdb -p <nginx PID obtained at the previous step>
   ```

3. Copy the script, paste it to GDB and press “Enter”. The script will save the log in the _debug\_log.txt_ file located in the current directory:

   ```text
   set $log = ngx_cycle->log
   while $log->writer != ngx_log_memory_writer
       set $log = $log->next
   end
   set $buf = (ngx_log_memory_buf_t *) $log->wdata
   dump binary memory debug_log.txt $buf->start $buf->end
   ```

4. Quit GDB by pressing CTRL+D.
5. Open the file “_debug\_log.txt_” located in the current directory:

   ```text
   $ sudo less debug_log.txt
   ```

#### Debug Log for Selected IPs

It is possible to enable the debugging log for a particular IP address or a range of IP addresses. Logging particular IPs may useful in a production environment as it will not negatively affect performance. The IP address is specified in the [`debug_connection`](https://nginx.org/en/docs/ngx_core_module.html#debug_connection) directive within the [`events`](https://nginx.org/en/docs/ngx_core_module.html#events) block; the directive can be defined more than once:

```text
error_log /path/to/log;
...
events {
    debug_connection 192.168.1.1;
    debug_connection 192.168.10.0/24;
}
```

#### Debug Log for Each Virtual Host

Generally, the [`error_log`](https://nginx.org/en/docs/ngx_core_module.html#error_log) directive is specified in the `main` context and thus is applied to all other contexts including [`server`](https://nginx.org/en/docs/http/ngx_http_core_module.html#server) and [`location`](https://nginx.org/en/docs/http/ngx_http_core_module.html#location). But if there is another `error_log` directive specified inside a particular `server` or a `location` block, the global settings will be overridden and such `error_log` directive will set its own path to the log file and the level of logging.

To set up the debugging log for a particular virtual host, add the `error_log` directive inside a particular `server` block, in which set a new path to the log file and the `debug` logging level:

```text
error_log /path1/to/log debug;
...
http {
    ...
    server {
    error_log /path2/to/log debug;
    ...
    }
}
```

To disable the debugging log per a particular virtual host, specify the `error_log` directive inside a particular `server` block, and specify a path to the log file only:

```text
error_log /path/to/log debug;
...
http {
    ...
    server {
    error_log /path/to/log;
    ...
    }
}
```

### Enabling Core Dumps

A core dump file helps identify and fix a problem that lead to NGINX crash. Note that a core dump file may contain sensitive information such as passwords and private keys, so ensure that they are treated in a secure manner.

Core dumps can be enabled in two different ways:

* in the operating system
* in the NGINX configuration file

#### Enabling Core Dumps in the Operating System

Perform the following steps in your operating system:

1. Specify a working directory in which a core dump file will be saved, for example, “_/tmp/cores_”:

   ```text
   $ mkdir /tmp/cores
   ```

2. Make sure the directory is writable by NGINX worker process:

   ```text
   $ sudo chown root:root /tmp/cores
   $ sudo chmod 1777 /tmp/cores
   ```

3. Disable the limit for the maximum size of a core dump file:

   ```text
   $ ulimit -c unlimited
   ```

   If the operation ends up with “Cannot modify limit: operation not permitted”, run the command:

   ```text
   $ sudo sh -c "ulimit -c unlimited && exec su $LOGNAME"
   ```

4. Enable core dumps for the _setuid_ and _setgid_ processes.

   For CentOS 7.0, Debian 8.2, Ubuntu 14.04, run the commands:

   ```text
   $ echo "/tmp/cores/core.%e.%p" | sudo tee /proc/sys/kernel/core_pattern
   $ sudo sysctl -w fs.suid_dumpable=2
   $ sysctl -p
   ```

   For FreeBSD, run the commands:

   ```text
   $ sudo sysctl kern.sugid_coredump=1
   $ sudo sysctl kern.corefile=/tmp/cores/%N.core.%P
   ```

#### Enabling Core Dumps in NGINX Configuration

Skip these steps if you have already configured creation of a core dump file in your operating system.

To enable core dumps in NGINX configuration file:

1. Open the NGINX configuration file:

   ```text
   $ sudo vi  /usr/local/etc/nginx/nginx.conf
   ```

2. Define a directory that will keep core dump files with the [`working_directory`](https://nginx.org/en/docs/ngx_core_module.html#working_directory) directive. The directive is specified on the _main_ configuration level:

   ```text
   working_directory /tmp/cores/;
   ```

3. Make sure the directory exists and is writable by NGINX worker process. Open terminal and run the commands:

   ```text
   $ sudo chown root:root /tmp/cores
   $ sudo chmod 1777 /tmp/cores
   ```

4. Specify the maximum possible size of the core dump file with the [`worker_rlimit_core`](https://nginx.org/en/docs/ngx_core_module.html#worker_rlimit_core) directive. The directive is also specified on the `main` configuration level. If the core dump file size exceeds the value, the core dump file will not be created.

   ```text
   worker_rlimit_core 500M;
   ```

Example:

```text
worker_processes   auto;
error_log          /var/log/nginx/error.log debug;
working_directory  /tmp/cores/;
worker_rlimit_core 500M;

events {
    ...
}

http {
    ...
}
```

With these settings, a core dump file will be created in the “_/tmp/cores/_” directory, and only if its size does not exceed 500 megabytes.

### Obtaining Backtrace From a Core Dump File

Backtraces provide information from a core dump file about what was wrong when a program crashed.

To get a backtrace from a core dump file:

1. Open a core dump file with the GDB debugger using the pattern:

   ```text
   $ sudo gdb <nginx_executable_path> <coredump_file_path>
   ```

2. Type-in the “_backtrace_ command to get a stack trace from the time of the crash:

   ```text
   (gdb) backtrace
   ```

If the “_backtrace_” command resulted with the “No symbol table info available” message, you will need to recompile NGINX binary to include debugging symbols. See [NGINX and Debugging Symbols](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/#compile_symbols).

### Dumping NGINX Configuration From a Running Process

You can extract the current NGINX configuration from the master process in memory. This can be useful when you need to:

* verify which configuration has been loaded
* restore a previous configuration if the version on disk has been accidentally removed or overwritten

The configuration dump can be obtained with a GDB script provided that your NGINX has the debug support.

1. Make sure your NGINX is built with the debug support \(the `--with-debug` configure option in the list of the configure arguments\). Run the command and check if the output contains the `--with-debug` line:

   ```text
   $  nginx -V 2>&1 | grep -- '--with-debug'
   ```

2. Obtain the PID of NGINX worker process:

   ```text
   $ ps axu | grep nginx
   ```

3. Launch the GDB debugger:

   ```text
   $ sudo gdb -p <nginx PID obtained at the previous step>
   ```

4. Copy and paste the script to GDB and press “Enter”. The script will save the configuration in the _nginx\_conf.txt_ file in the current directory:

   ```text
   set $cd = ngx_cycle->config_dump
   set $nelts = $cd.nelts
   set $elts = (ngx_conf_dump_t*)($cd.elts)
   while ($nelts-- > 0) 
   set $name = $elts[$nelts]->name.data
   printf "Dumping %s to nginx_conf.txt\n", $name
   append memory nginx_conf.txt \
         $elts[$nelts]->buffer.start $elts[$nelts]->buffer.end
   end
   ```

5. Quit GDB by pressing _CTRL+D_.
6. Open the file _nginx\_conf.txt_ located in the current directory:

   ```text
   $ sudo vi nginx.conf.txt
   ```

### Asking for help

When asking for help with debugging, please provide the following information:

1. NGINX version, compiler version, and configure parameters. Run the command:

   ```text
   $ nginx -V
   ```

2. Current full NGINX configuration. See [Dumping NGINX Configuration From a Running Process](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/#configdump)
3. The debugging log. See [Enabling Debug Logging in NGINX Configuration](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/#error_log)
4. The obtained backtrace. See [Enabling Core Dumps](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/#coredump), [Obtaining Backtrace](https://docs.nginx.com/nginx/admin-guide/monitoring/debugging/#backtrace)

