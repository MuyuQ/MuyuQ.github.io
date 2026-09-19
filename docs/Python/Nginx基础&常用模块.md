# Nginx 基础与常用模块

## Nginx 的中间件架构

Nginx 是一个开源且高性能的 HTTP 中间件和代理服务。

```shell
nginx -v
# 查看 Nginx 编译参数
nginx -V
# 检查 Nginx 配置文件语法是否正确
nginx -t -c /etc/nginx/nginx.conf
# 重新载入配置文件
nginx -s reload
```

## 为什么选择 Nginx

**原因1：IO 多路复用 epoll**

> IO 多路复用是指通过一个线程同时监听多个文件描述符（socket），当某个描述符就绪时立即处理，从而以较少的线程资源高效管理大量并发连接。epoll 是 Linux 下高效的 IO 多路复用实现。

**原因2：轻量级**

Nginx 采用模块化设计，核心功能占用资源极少。相比 Apache 等传统 Web 服务器，Nginx 在高并发场景下内存和 CPU 消耗更低，启动速度更快。

**原因3：CPU 亲和（affinity）**

CPU 亲和是一种将 CPU 核心与 Nginx 工作进程绑定的机制，使每个 worker 进程固定在一个 CPU 核心上执行，从而减少 CPU 切换带来的缓存未命中（cache miss），获得更好的性能。

**原因4：sendfile**

sendfile 是一种零拷贝传输机制，允许数据直接从内核空间的文件描述符传输到 socket，无需在用户空间和内核空间之间反复拷贝，显著提升了静态文件的传输效率。

## Nginx 目录讲解

| 路径 | 类型 | 作用 |
| --- | --- | --- |
| /etc/logrotate.d/nginx | 配置文件 | Nginx 日志轮转，用于 logrotate 服务的日志切割 |
| /etc/nginx | 目录/配置文件 | Nginx 主配置文件目录 |
| /etc/nginx/nginx.conf | 配置文件 | 主要配置文件 |
| /etc/nginx/conf.d | 目录 | 子配置目录 |
| /etc/nginx/conf.d/default.conf | 配置文件 | 默认配置文件 |
| /etc/nginx/fastcgi_params | 配置文件 | FastCGI 配置参数 |
| /etc/nginx/uwsgi_params | 配置文件 | uWSGI 配置参数 |
| /etc/nginx/scgi_params | 配置文件 | SCGI 配置参数 |
| /etc/nginx/koi-utf | 配置文件 | 编码转换映射文件 |
| /etc/nginx/koi-win | 配置文件 | 编码转换映射文件 |
| /etc/nginx/win-utf | 配置文件 | 编码转换映射文件 |
| /etc/nginx/mime.types | 配置文件 | 设置 HTTP 协议的 Content-Type 与扩展名对应关系 |
| /usr/lib/systemd/system/nginx-debug.service | 配置文件 | CentOS 7 中，用于配置系统守护进程管理器 |
| /usr/lib/systemd/system/nginx.service | 配置文件 | Nginx 系统服务配置 |
| /etc/sysconfig/nginx | 配置文件 | Nginx 系统配置 |
| /etc/sysconfig/nginx-debug | 配置文件 | Nginx 调试模式配置 |

主配置文件路径：`/etc/nginx/nginx.conf`

子配置目录：`/etc/nginx/conf.d`

## HTTP 请求

- **请求报文（Request）**：包括请求行、请求头部、请求体
- **响应报文（Response）**：包括状态行、响应头部、响应体

**curl** —— CommandLine Uniform Resource Locator

curl 是一款利用 URL 语法在命令行环境下工作的开源文件传输工具，支持多种协议（如 HTTP、HTTPS、FTP 等）。

通过修改 Nginx 配置文件中的 `error_log` 和 `access_log` 指令，可以自定义错误日志和访问日志的记录路径与格式。更多可配置参数请参阅 [Nginx 官方文档](http://nginx.org/en/docs/)。

### Nginx 变量

1. HTTP 请求变量
2. Nginx 内置变量
3. 自定义变量

## Nginx 模块讲解

```shell
nginx -t -c /etc/nginx/nginx.conf
```

### stub_status_module 模块

用于展示 Nginx 当前的连接处理状态，包括活跃连接数、已接受连接数、已处理请求数等信息，适用于运维监控场景。

需要在 `/etc/nginx/conf.d/default.conf` 中添加：

```nginx
location /mystatus {
    stub_status;
}
```

### http_random_index_module 模块

随机首页模块，用于从目录中随机选择一个文件作为首页返回。

### ngx_http_sub_module 模块

通过 `sub_filter`、`sub_filter_last_modified`、`sub_filter_once`、`sub_filter_types` 等指令替换 HTTP 页面内容。

### limit_conn_module 连接频率限制

用于限制同一客户端在同一时间的并发连接数，防止单个客户端占用过多服务器资源。

### limit_req_module 请求频率限制

用于限制单位时间内的请求速率，有效防御 CC 攻击和恶意刷请求等行为。

**HTTP 协议的连接与请求：**

| HTTP 协议版本 | 连接关系 |
| --- | --- |
| HTTP 1.0 | TCP 不能复用 |
| HTTP 1.1 | 有序 TCP 复用（管线化） |
| HTTP 2.0 | 多路复用 TCP 复用（流式传输） |

- HTTP 请求建立在一次 TCP 连接基础上
- 一次 TCP 连接至少产生一次 HTTP 请求

```shell
ab -n 50 -c 20 http://www.qq.com

# -n 表示总请求数为 50，-c 20 表示同时并发请求数为 20
# ab（Apache Bench）是 Apache 自带的压力测试工具
```

## Nginx 访问控制

### http_access_module 基于 IP 的访问控制

```nginx
location / {
    allow all;
    deny 10.211.55.2;
}
# 注意：Nginx 按顺序匹配规则，首个匹配生效后即停止。
# 若 allow all 置于 deny 之前，则后续所有 deny 规则均不会生效
```

`http_access_module` 的局限性：在存在七层负载均衡或 CDN 的场景下，Nginx 获取到的 `remote_addr` 往往是中间节点的 IP 而非客户端真实 IP，因此无法准确基于 `remote_addr` 进行访问控制。

**进阶方法：**

**方案1：** 采用其他 HTTP 头信息控制访问 —— `http_x_forwarded_for`

![](http://ojho2g8px.bkt.clouddn.com/WX20180312-111918@2x.png)

```shell
http_x_forwarded_for = Client IP, Proxy(1) IP, Proxy(2) IP...
```

然而 `X-Forwarded-For` 仅为事实标准而非正式协议，代理服务器可以随意修改甚至伪造该字段，因此安全性较低，不宜作为唯一的访问控制依据。

**方案2：** 结合 geo 模块

**方案3：** 通过 HTTP 自定义变量传递

### http_auth_basic_module 基于用户的身份认证

通过 HTTP Basic Authentication 实现简单的用户名密码验证。

```nginx
location / {
    auth_basic 'this is a test';
    auth_basic_user_file /path;
}
```

可使用 `htpasswd` 工具生成加密密码文件，该工具包含在 `httpd-tools` 软件包中。

**局限性：**

1. 用户信息依赖本地文件存储，难以规模化管理
2. 每次修改用户信息均需手动操作，效率低下且不易自动化

**解决方案：**

1. Nginx 结合 Lua 实现高效验证
2. Nginx 和 LDAP 打通，利用 nginx-auth-ldap 模块
