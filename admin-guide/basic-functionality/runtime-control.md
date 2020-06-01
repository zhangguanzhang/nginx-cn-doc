---
description: 本节描述NGINX在运行时启动的过程以及如何控制它们.
---

# Controlling NGINX Processes at Runtime

### Master and Worker Processes

NGINX具有一个主进程和一个或多个工作进程. 如果[caching](https://docs.nginx.com/nginx/admin-guide/content-cache/content-caching/)被开启, 则缓存加载器和缓存管理器进程也会在启动时运行.

主进程的主要目的是读取和评估配置文件，以及维护工作进程.

工作进程执行请求的实际处理。 NGINX依赖于操作系统相关的机制来有效地在工作进程之间分配请求。 辅助进程的数量由 **nginx.conf** 配置文件中的 [`worker_processes`](https://nginx.org/en/docs/ngx_core_module.html#worker_processes)定义， 并且可以设置为固定数量或配置为自动调整为可用的CPU内核数量.

### Controlling NGINX

要重新加载配置，您可以停止或重新启动NGINX，或向主进程发送信号。 可以通过运行带有 `-s` 参数的 `nginx` 命令（调用NGINX可执行文件）来发送信号。

```text
nginx -s <SIGNAL>
```

 `<SIGNAL>` 可为下列值:

* `quit` – 优雅关闭
* `reload` – 重新加载配置文件
* `reopen` – 重新打开日志文件
* `stop` – 立即关闭(快速关闭)

还可以用`kill`直接向主进程发送信号。主进程的进程ID在默认情况下写入**nginx。pid**文件，它位于 **/usr/local/nginx/logs** 或 **/var/run** 目录中。

有关高级信号（例如，执行实时二进制升级）的更多信息，请参阅位于 **nginx.org** 的[Controlling nginx](https://nginx.org/en/docs/control.html).

