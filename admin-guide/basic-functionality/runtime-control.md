---
description: >-
  This section describes the processes that NGINX starts at run time and how to
  control them.
---

# Controlling NGINX Processes at Runtime

### Master and Worker Processes

NGINX has one master process and one or more worker processes. If [caching](https://docs.nginx.com/nginx/admin-guide/content-cache/content-caching/) is enabled, the cache loader and cache manager processes also run at startup.

The main purpose of the master process is to read and evaluate configuration files, as well as maintain the worker processes.

The worker processes do the actual processing of requests. NGINX relies on OS-dependent mechanisms to efficiently distribute requests among worker processes. The number of worker processes is defined by the [`worker_processes`](https://nginx.org/en/docs/ngx_core_module.html#worker_processes) directive in the **nginx.conf** configuration file and can either be set to a fixed number or configured to adjust automatically to the number of available CPU cores.

### Controlling NGINX

To reload your configuration, you can stop or restart NGINX, or send signals to the master process. A signal can be sent by running the `nginx` command \(invoking the NGINX executable\) with the `-s` argument.

```text
nginx -s <SIGNAL>
```

 `<SIGNAL>` 处可为下列值:

* `quit` – Shut down gracefully
* `reload` – Reload the configuration file
* `reopen` – Reopen log files
* `stop` – Shut down immediately \(fast shutdown\)

The `kill` utility can also be used to send a signal directly to the master process. The process ID of the master process is written, by default, to the **nginx.pid** file, which is located in the **/usr/local/nginx/logs** or **/var/run** directory.

For more information about advanced signals \(for performing live binary upgrades, for example\), see [Controlling nginx](https://nginx.org/en/docs/control.html) at **nginx.org**.

