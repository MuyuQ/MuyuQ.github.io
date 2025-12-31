# Nginx 基础&常用模块
## Nginx 的中间件架构

简介  
```shell
nginx -v 
#查看 nginx 编译参数
nginx -V
#检查 nginx 配置文件语法是否正确
nginx -t -c /etc/nginx/nginx.conf  
#重新载入配置文件
nginx -s reload
```

Nginx 是一个开源且高性能可靠的 HTTP 中间件,代理服务.

##为什么选择 Nginx?

原因1,IO 多路复用 epoll.

> IO 复用就是将多个 IO 流使用一个 socket 来传递.

原因2,轻量级

原因3,CPU 亲和(affinity)

`CPU 亲和是一种把 CPU 核心和 Nginx 工作进程绑定方式,把每个 worker 进程固定在一个 CPU 上执行,减少切换 CPU 的 cache miss, 获得更好的性能.`

原因4,sendfile

## Nginx 目录讲解

| 路径                                        | 类型          | 作用                                           |
| ------------------------------------------- | ------------- | ---------------------------------------------- |
| /etc/logrotate.d/nginx                      | 配置文件      | Nginx 日志轮转,用于 logrotate 服务的日志切割   |
|                                             |               |                                                |
| / etc/nginx                                 | 目录,配置文件 | Nginx 主配置文件                               |
| / etc/nginx/nginx.conf                      |               | 主要配置文件                                   |
| /etc/nginx/nginx/conf.d                     |               |                                                |
| /etc/nginx/conf.d/default.conf              |               | 默认配置文件                                   |
|                                             |               |                                                |
| /etc/nginx/fastcgi_params                   | 配置文件      | fastcgi 配置                                   |
| /etc/nginx/uwsgi_params                     |               | param(参数)                                    |
| /etc/nginx/scgi_params                      |               |                                                |
|                                             |               |                                                |
| /etc/nginx/koi-utf                          | 配置文件      | 编码转换映射转化文件                           |
| /etc/nginx/koi-win                          |               |                                                |
| /etc/nginx/win-utf                          |               |                                                |
|                                             |               |                                                |
| /etc/nginx/mime.types                       | 配置文件      | 设置 http 协议的 Content-Type 与扩展名对应关系 |
|                                             |               |                                                |
| /usr/lib/systemd/system/nginx-debug.service | 配置文件      | centos7中,用于配置出系统守护进程管理器管理方式 |
| /usr/lib/systemd/system/nginx.service       |               |                                                |
| /etc/sysconfig/nginx                        |               |                                                |
| /etc/sysconfig/nginx-debug                  |               |                                                |

看/ etc/nginx/conf.d

看/ etc/nginx/nginx.conf

## HTTP请求

request-包括请求行,请求头部,请求数据

response-包括状态行,消息报头,响应正文

Curl -CommandLine Uniform Resource Locator

curl是利用URL语法在命令行方式下工作的开源文件传输工具.



通过编辑配置文件,可以更改 error.log和 access.log 的记录,  具体哪些参数可以添加,可以参考 Nginx.org 上面的文档.

###Nginx 变量

1.http请求变量

2.Nginx 内置变量

3.自定义变量



## Nginx 模块讲解 

```shell
nginx -t -c /etc/nginx/nginx.conf
```

#### __`_stub_status_moule`__模块,

显示 Nginx 当前处理链接的状态,用于监控 Nginx 当前连接的信息

需要在`/etc/nginx/conf.d/default.conf`添加

```shell
location /mystatus {
    stub_status;
}
```



#### __`--with-http_random_index_module`__

#### __`gx_http_sub_module`__

`sub_filter`,`sub_filter_last_modified`,`sub_filter_once`,`sub_filter_types`替换 HTTP 页面 

#### __`limit_conn_module`__连接频率限制

#### __`limit_req_module`__请求频率限制

HTTP协议的连接与请求

| HTTP 协议版本 | 连接关系          |
| ------------- | ----------------- |
| HTTP1.0       | TCP 不能复用      |
| HTTP1.1       | 顺序性 TCP 复用   |
| HTTP2.0       | 多路复用 TCP 复用 |

HTTP 请求建立在一次 TCP 连接基础上

一次 TCP 连接至少产生一次 HTTP 请求

```
ab -n 50 -c 20 http://www.qq.com

-n 表示请求数为50 , -c 20是同时并发请求数
ab 是 Apache 自带的压力测试工具,是一个很实用的测试工具.
```

###Nginx 访问控制

__`http_access_module`__基于 ip 的访问控制.

如果要写

``` shell
location / {
    allow all;
    deny 10.211.55.2;
}
##需要注意 allow all的位置, 如果 allow all 在前面,则后面所有的 deny 都会无效.
```

`access_module`的局限性:因为存在7lay LSB和 CDN 等,所以 remote_addr 不一定是需要进行限制的 ip 地址. 无法保证 remote_addr 的准确性

__进阶方法__

方案1.采用别的 HTTP 头信息控制访问, __`http_x_forwarded_for`__

![](http://ojho2g8px.bkt.clouddn.com/WX20180312-111918@2x.png)

```shell
http_x_forwarded_for = Client IP, Proxy(1)IP,Proxy(2)IP...
```

但是 x_forwarded_for 只是一个协议,并没有强制要求对方遵守,故是可以被篡改的,安全性并不高.

方案2.结合 geo 模块

方案3.通过 HTTP 自定义变量传递

#### __`http_auth_basic_module`__基于用户的信任登录

```
location / {
    auth_basic 'this is a test';
    auth_basic_user_file /path;
}
```

可以使用 htpasswd来进行加密.  该工具集成在 httpd-tools 里面.

局限性

1.用户信息依赖文件方式

2.操作机械,效率低下

解决方案:

1.Nginx 结合 LUA 实现高效验证

2.Nginx 和 LDAP 大同,利用 Nginx-auth-ldap 模块.

