# Nginx介绍

## **什么是Nginx？**

> Nginx是一个高性能的HTTP和反向代理web服务器，同时也提供了IMAP/POP3/SMTP服务。
>
> Nginx是由伊戈尔·赛索耶夫为俄罗斯访问量第二的Rambler.ru站点开发的，第一个公开版本0.1.0发布于2004年10月4日。2011年6月1日，nginx1.0.4发布。
>
> 其特点是占有内存少,并发能力强，事实上Nginx的并发能力在同类型的网页服务器中表现较好，中国大陆使用Nginx网站用户有:百度、京东、新浪、网易、腾讯、淘宝等。在全球活跃的网站中有12.18%的使用比率,大约为2220万个网站。
>
> Nginx是一个安装非常的简单、配置文件非常简洁(还能够支持Perl语法) 、Bug非常少的服务。Nginx 启动特别容易，并且几乎可以做到7*24不间断运行，即使运行数个月也不需要重新启动。你还能够不间断服务的情况下进行软件版本的升级。
>
> Nginx代码完全用C语言从头写成。官方数据测试表明能够支持高达50,000个并发连接数的响应。

## **Nginx作用**

### **1、代理**

> Http代理，反向代理：作为web服务器最常用的功能之一，尤其是反向代理。

正向代理![image-20211129231030882](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/image-20211129231030882.png)

反向代理![image-20211129231213541](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/image-20211129231213541.png)

### **2、负载均衡**

> Nginx提供的负载均衡策略有2种：内置策略和扩展策略。
>
> 内置策略为轮询，加权轮询，Ip hash
>
> 扩展策略，就天马行空，只有你想不到的没有他做不到的。

轮询![image-20211129231447493](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/image-20211129231447493.png)

加权轮询![image-20211129231523746](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/image-20211129231523746.png)

> IP hash对客户端请求的IP进行hash操作，然后根据hash结果将同一-个客户端ip的请求分发给同一台服务器进行处理，可以解决session不共享的问题。

![image-20211129231750756](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/image-20211129231750756.png)

### **3、动静分离**

> 动静分离，在我们的软件开发中，有些请求是需要后台处理的，有些请求是不需要经过后台处理的(如: CSS、HTML、jpg、js等等文件)，这些不需要经过后台处理的文件称为静态文件。让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作。提高资源响应的速度。

![image-20211129232547078](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/image-20211129232547078.png)

# Nginx安装

## 1、下载

[Nginx下载地址](http://nginx.org/en/download.html)

| 下载版本           | 解释   |
| ------------------ | ------ |
| Mainline version   | 最新版 |
| Stable version     | 稳定版 |
| Legacy versions    | 过去版 |
| Source Code        | 源码   |
| Pre-Built Packages | 依赖包 |

**PGP**是加密密钥![image-20220217220854832](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/202202172208870.png)

更新说明![image-20220217220803488](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/202202172208534.png)

Windows下载![image-20220217220822306](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/202202172208340.png)

Linux下载![image-20220217220838285](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/202202172208318.png)

## 2、安装

**Windows解压下载的包

![image-20211130001016803](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/image-20211130001016803.png)

出现以上目录即可，nginx.exe就是可执行文件，conf为配置文件目录，运行nginx.exe即可运行服务

**Linux解压下载的包**

> 提前解决依赖包
>
> **gcc**、**pcre**、**pcre-devel**、**zlib**、**make**、**openssl**、**openssl-devel**

执行**configure**文件会获得**Makefile**文件，执行make，在执行make install即可

> 带SSL模块
> `./configure --prefix=/usr/local/nginx --with-http_ssl_module`

> 执行**configure**时报错
>
>
>
> ./configure: error: the HTTP rewrite module requires the PCRE library.
> 安装pcre-devel解决问题
> yum -y install pcre-devel
>
>
>
> 错误提示：./configure: error: the HTTP cache module requires md5 functions
> from OpenSSL library.   You can either disable the module by using
> --without-http-cache option, or install the OpenSSL library into the system,
> or build the OpenSSL library statically from the source with nginx by using
> --with-http_ssl_module --with-openssl=<path> options.
> 解决办法：
> yum -y install openssl openssl-devel
>
>
>
> 总结：
>
> yum -y install pcre-devel openssl openssl-devel
>
> ./configure --prefix=/usr/local/nginx
>
> make
>
> make install
>
> 一切搞定

可以使用`whereis nginx`命令查找Nginx的执行目录所在

![image-20211130000723562](https://cdn.jsdelivr.net/gh/zerobiubiu/Figure-bed/image-20211130000723562.png)

Nginx的执行文件放在了sbin/目录下，执行nginx文件，成功无返回值

## 3、关闭

**Windows：**

如果使用命令窗口启动nginx， 关闭窗口是不能结束Nginx进程的，可使用两种方法关闭Nginx

1. 输入Nginx命令 `nginx -s stop`(快速停止Nginx) 或 `nginx -s quit`(完整有序的停止Nginx)
2. 使用taskkill `taskkill /f /t /im nginx.exe`

> `taskkill是用来终止进程的，`
>
> `/f是强制终止 .`
>
> `/t终止指定的进程和任何由此启动的子进程。`
>
> `/im示指定的进程名称 .`

**Linux：**

1. `cd /usr/local/nginx/sbin/`
2. `./nginx  启动`
3. `./nginx -s stop  停止`
4. `./nginx -s quit  安全退出`
5. `./nginx -s reload  重新加载配置文件`
6. `ps aux|grep nginx  查看nginx进程`

## 4、相关问题

连接失败、修改防火墙

1. 开启
2. `service firewalld start`
3. 重启
4. `service firewalld restart`
5. 关闭
6. `service firewalld stop`
7. 查看防火墙规则
8. `firewall-cmd --list-all`
9. 查询端口是否开放
10. `firewall-cmd --query-port=8080/tcp`
11. 开放80端口
12. `firewall-cmd --permanent --add-port=80/tcp`
13. 移除端口
14. `firewall-cmd --permanent --remove-port=8080/tcp`
15. 重启防火墙(修改配置后要重启防火墙)
16. `firewall-cmd --reload`
17. 参数解释
18. `firwall-cmd：是Linux提供的操作firewall的一个工具；`
19. `--permanent：表示设置为持久；`
20. `--add-port：标识添加的端口；`

## 5、编译安装配置模块

使用configure命令配置构建。它定义了系统的各个方面，包括允许nginx用于连接处理的方法。最后，它会创建一个Makefile。
该configure命令支持以下参数：
**--help**

```
打印帮助信息。
```

**--prefix=path**

```
定义将保留服务器文件的目录。此相同目录还将用于设置的所有相对路径 configure（库源路径除外）和nginx.conf配置文件中。/usr/local/nginx默认情况下设置为目录。
```

**--sbin-path=path**

```
设置nginx可执行文件的名称。此名称仅在安装期间使用。默认情况下，文件名为 prefix/sbin/nginx。
```

**--modules-path=path**

```
定义将在其中安装nginx动态模块的目录。默认情况下使用prefix/modules目录。
```

**--conf-path=path**

```
设置nginx.conf配置文件的名称。如果需要，可以通过在命令行参数中指定nginx来始终使用其他配置文件来启动它 。默认情况下，文件名为 。 -c fileprefix/conf/nginx.conf
```

**--error-log-path=path**

```
设置主要错误，警告和诊断文件的名称。安装后，可以始终nginx.conf使用error_log伪指令在配置文件中 更改文件名 。默认情况下，文件名为 prefix/logs/error.log。
```

**--pid-path=path**

```
设置nginx.pid将存储主进程的进程ID 的文件名。安装后，可以始终nginx.conf使用pid伪指令在配置文件中 更改文件名 。默认情况下，文件名为 prefix/logs/nginx.pid。
```

**--lock-path=path**

```
为锁定文件的名称设置前缀。安装后，可以始终nginx.conf使用lock_file伪指令在配置文件中 更改该值 。默认情况下，值为 prefix/logs/nginx.lock。
```

**--user=name**

```
设置一个非特权用户的名称，其凭据将由工作进程使用。安装后，可以始终nginx.conf使用用户指令在配置文件中 更改名称 。默认用户名是nobody。
```

**--group=name**

```
设置其凭据将由工作进程使用的组的名称。安装后，可以始终nginx.conf使用用户指令在配置文件中 更改名称 。默认情况下，组名称设置为非特权用户的名称。
```

**--build=name**

```
设置一个可选的nginx构建名称。
```

**--builddir=path**

```
设置构建目录。
```

**--with-select_module 和 --without-select_module**

```
启用或禁用构建允许服务器使用该select()方法的模块。如果平台似乎不支持kqueue，epoll或/ dev / poll等更合适的方法，则会自动构建此模块。
```

**--with-poll_module 和 --without-poll_module**

```
启用或禁用构建允许服务器使用该poll()方法的模块。如果平台似乎不支持kqueue，epoll或/ dev / poll等更合适的方法，则会自动构建此模块。
```

**--with-threads** 

```
启用线程池的使用 。
```

**--with-file-aio**

```
支持 在FreeBSD和Linux上使用 异步文件I / O（AIO）。
```

**--with-http_ssl_module** 

```
启用构建将HTTPS协议支持添加 到HTTP服务器的模块的功能。默认情况下未构建此模块。需要OpenSSL库来构建和运行此模块。
```

**--with-http_v2_module**

```
支持构建一个模块，该模块提供对HTTP / 2的支持 。默认情况下未构建此模块。
```

**--with-http_realip_module**

```
支持构建ngx_http_realip_module 模块，该 模块将客户端地址更改为在指定的标头字段中发送的地址。默认情况下未构建此模块。
```

**--with-http_addition_module**

```
允许构建ngx_http_addition_module 模块，该 模块在响应之前和之后添加文本。默认情况下未构建此模块。
```

**--with-http_xslt_module 和 --with-http_xslt_module=dynamic**

```
支持构建ngx_http_xslt_module 模块，该 模块使用一个或多个XSLT样式表转换XML响应。默认情况下未构建此模块。该libxml2的和 的libxslt库需要构建和运行此模块。
```

**--with-http_image_filter_module 和 --with-http_image_filter_module=dynamic**

```
支持构建ngx_http_image_filter_module 模块，该 模块可以转换JPEG，GIF，PNG和WebP格式的图像。默认情况下未构建此模块。
```

**--with-http_geoip_module 和 --with-http_geoip_module=dynamic** 

```
支持构建ngx_http_geoip_module 模块，该 模块根据客户端IP地址和预编译的MaxMind数据库创建变量 。默认情况下未构建此模块。
```

**--with-http_sub_module**

```
支持构建ngx_http_sub_module 模块，该 模块通过将一个指定的字符串替换为另一个指定的字符串来修改响应。默认情况下未构建此模块。
```

**--with-http_dav_module**

```
支持构建ngx_http_dav_module 模块，该 模块通过WebDAV协议提供文件管理自动化。默认情况下未构建此模块。
```

**--with-http_flv_module**

```
支持构建ngx_http_flv_module 模块，该 模块为Flash Video（FLV）文件提供伪流服务器端支持。默认情况下未构建此模块。
```

**--with-http_mp4_module**

```
支持构建ngx_http_mp4_module 模块，该 模块为MP4文件提供伪流服务器端支持。默认情况下未构建此模块。
```

**--with-http_gunzip_module**

```
支持为不支持“ gzip”编码方法的客户端构建ngx_http_gunzip_module 模块，该 模块使用“ Content-Encoding: gzip” 解压缩响应。默认情况下未构建此模块。
```

**--with-http_gzip_static_module**

```
支持构建ngx_http_gzip_static_module 模块，该 模块支持发送.gz扩展名为“ ”的预压缩文件，而不是常规文件。默认情况下未构建此模块。
```

**--with-http_auth_request_module**

```
允许构建ngx_http_auth_request_module 模块，该 模块基于子请求的结果实现客户端授权。默认情况下未构建此模块。
```

**--with-http_random_index_module**

```
支持构建ngx_http_random_index_module 模块，该 模块处理以斜杠（' /'）结尾的请求，并从目录中选择一个随机文件作为索引文件。默认情况下未构建此模块。
```

**--with-http_secure_link_module**

```
启用构建 ngx_http_secure_link_module 模块。默认情况下未构建此模块。
```

**--with-http_degradation_module**

```
启用构建 ngx_http_degradation_module模块。默认情况下未构建此模块。
```

**--with-http_slice_module**

```
支持构建ngx_http_slice_module 模块，该 模块将请求拆分为子请求，每个子请求返回一定范围的响应。该模块提供了更有效的大响应缓存。默认情况下未构建此模块。
```

**--with-http_stub_status_module**

```
支持构建ngx_http_stub_status_module 模块，该 模块提供对基本状态信息的访问。默认情况下未构建此模块。
```

**--without-http_charset_module**

```
禁用构建ngx_http_charset_module 模块，该 模块将指定的字符集添加到“ Content-Type”响应头字段中，并且可以将数据从一个字符集转换为另一个字符集。
```

**--without-http_gzip_module**

```
禁用构建可压缩 HTTP服务器响应的模块。zlib库是构建和运行此模块所必需的。
```

**--without-http_ssi_module**

```
禁用构建 处理通过SSI（服务器端包含）命令的 ngx_http_ssi_module模块的响应。
```

**--without-http_userid_module**

```
禁用构建ngx_http_userid_module 模块，该 模块设置适用于客户端标识的cookie。
```

**--without-http_access_module**

```
禁用构建ngx_http_access_module 模块，该 模块允许限制对某些客户端地址的访问。
```

**--without-http_auth_basic_module**

```
禁用构建ngx_http_auth_basic_module 模块，该 模块允许通过使用“ HTTP基本身份验证”协议验证用户名和密码来限制对资源的访问。
```

**--without-http_mirror_module**

```
禁用构建ngx_http_mirror_module 模块，该 模块通过创建后台镜像子请求来实现原始请求的镜像。
```

**--without-http_autoindex_module**

```
禁用构建 ngx_http_autoindex_module 模块，以处理以斜杠（' /'）结尾的请求，并在ngx_http_index_module模块找不到索引文件的情况下生成目录列表 。
```

**--without-http_geo_module**

```
禁用构建ngx_http_geo_module 模块，该 模块创建的变量的值取决于客户端IP地址。
```

**--without-http_map_module**

```
禁用构建ngx_http_map_module 模块，该 模块创建的变量的值取决于其他变量的值。
```

**--without-http_split_clients_module**

```
禁用构建ngx_http_split_clients_module 模块，该 模块创建用于A / B测试的变量。
```

**--without-http_referer_module**

```
禁用构建ngx_http_referer_module 模块，该 模块可以阻止对“ Referer”标头字段中具有无效值的请求的站点访问。
```

**--without-http_rewrite_module**

```
禁用构建允许HTTP服务器 重定向请求并更改请求URI的模块。构建和运行此模块需要PCRE库。
```

**--without-http_proxy_module**

```
禁用构建HTTP服务器 代理模块。
```

**--without-http_fastcgi_module**

```
禁用构建 将请求传递到FastCGI服务器的 ngx_http_fastcgi_module模块。
```

**--without-http_uwsgi_module**

```
禁用构建 将请求传递到uwsgi服务器的 ngx_http_uwsgi_module模块。
```

**--without-http_scgi_module**

```
禁用构建 将请求传递到SCGI服务器的 ngx_http_scgi_module模块。
```

**--without-http_grpc_module**

```
禁用构建 将请求传递到gRPC服务器的 ngx_http_grpc_module模块。
```

**--without-http_memcached_module**

```
禁用构建ngx_http_memcached_module 模块，该 模块从memcached服务器获取响应。
```

**--without-http_limit_conn_module**

```
禁用构建ngx_http_limit_conn_module 模块，该 模块限制每个键的连接数，例如，单个IP地址的连接数。
```

**--without-http_limit_req_module**

```
禁用构建ngx_http_limit_req_module 模块，该 模块限制每个密钥的请求处理速率，例如，来自单个IP地址的请求的处理速率。
```

**--without-http_empty_gif_module**

```
禁用构建发出单像素透明GIF的模块 。
```

**--without-http_browser_module**

```
禁用构建ngx_http_browser_module 模块，该 模块创建的变量的值取决于“ User-Agent”请求标头字段的值。
```

**--without-http_upstream_hash_module**

```
禁用构建实现哈希 负载平衡方法的模块 。
```

**--without-http_upstream_ip_hash_module**

```
禁用构建实现ip_hash 负载平衡方法的模块 。
```

**--without-http_upstream_least_conn_module**

```
禁用构建实现了minimum_conn 负载平衡方法的模块 。
```

**--without-http_upstream_keepalive_module**

```
禁用构建一个模块来提供 对上游服务器连接的缓存。
```

**--without-http_upstream_zone_module**

```
禁用构建模块，该模块可以将上游组的运行时状态存储在共享内存 区域中。
```

**--with-http_perl_module 和 --with-http_perl_module=dynamic**

```
支持构建 嵌入式Perl模块。默认情况下未构建此模块。
```

**--with-perl_modules_path=path**

```
定义一个目录，该目录将保留Perl模块。
```

**--with-perl=path**

```
设置Perl二进制文件的名称。
```

**--http-log-path=path**

```
设置HTTP服务器的主请求日志文件的名称。安装后，可以始终nginx.conf使用access_log伪指令在配置文件中 更改文件名 。默认情况下，文件名为 prefix/logs/access.log。
```

**--http-client-body-temp-path=path**

```
定义用于存储包含客户端请求正文的临时文件的目录。安装后，可以始终nginx.conf使用client_body_temp_path 指令在配置文件中 更改目录 。默认情况下，目录名为 prefix/client_body_temp。
```

**--http-proxy-temp-path=path**

```
定义一个目录，用于存储包含从代理服务器接收到的数据的临时文件。安装后，可以始终nginx.conf使用proxy_temp_path 指令在配置文件中 更改目录 。默认情况下，目录名为 prefix/proxy_temp。
```

**--http-fastcgi-temp-path=path**

```
定义一个目录，用于存储包含从FastCGI服务器接收到的数据的临时文件。安装后，可以始终nginx.conf使用fastcgi_temp_path 指令在配置文件中 更改目录 。默认情况下，目录名为 prefix/fastcgi_temp。
```

**--http-uwsgi-temp-path=path**

```
定义一个目录，用于存储带有从uwsgi服务器接收到的数据的临时文件。安装后，可以始终nginx.conf使用uwsgi_temp_path 指令在配置文件中 更改目录 。默认情况下，目录名为 prefix/uwsgi_temp。
```

**--http-scgi-temp-path=path**

```
定义一个目录，用于存储带有从SCGI服务器接收到的数据的临时文件。安装后，可以始终nginx.conf使用scgi_temp_path 指令在配置文件中 更改目录 。默认情况下，目录名为 prefix/scgi_temp。
```

**--without-http**

```
禁用HTTP服务器。
```

**--without-http-cache**

```
禁用HTTP缓存。
```

**--with-mail 和 --with-mail=dynamic**

```
启用POP3 / IMAP4 / SMTP 邮件代理服务器。
```

**--with-mail_ssl_module**

```
支持构建一个模块，该模块 向邮件代理服务器添加 SSL / TLS协议支持。默认情况下未构建此模块。需要OpenSSL库来构建和运行此模块。
```

**--without-mail_pop3_module**

```
在邮件代理服务器中 禁用POP3协议。
```

**--without-mail_imap_module**

```
在邮件代理服务器中 禁用IMAP协议。
```

**--without-mail_smtp_module**

```
在邮件代理服务器中 禁用SMTP协议。
```

**--with-stream 和 --with-stream=dynamic**

```
支持构建 用于通用TCP / UDP代理和负载平衡的 流模块。默认情况下未构建此模块。
```

**--with-stream_ssl_module**

```
支持构建一个模块，该模块 向流模块添加 SSL / TLS协议支持。默认情况下未构建此模块。需要OpenSSL库来构建和运行此模块。
```

**--with-stream_realip_module**

```
启用构建ngx_stream_realip_module 模块的功能，该 模块将客户端地址更改为PROXY协议标头中发送的地址。默认情况下未构建此模块。
```

**--with-stream_geoip_module 和 --with-stream_geoip_module=dynamic**

```
支持构建ngx_stream_geoip_module 模块，该 模块根据客户端IP地址和预编译的MaxMind数据库创建变量 。默认情况下未构建此模块。
```

**--with-stream_ssl_preread_module**

```
支持构建ngx_stream_ssl_preread_module 模块，该 模块允许从ClientHello 消息中提取信息， 而无需终止SSL / TLS。默认情况下未构建此模块。
```

**--without-stream_limit_conn_module**

```
禁用构建ngx_stream_limit_conn_module 模块，该 模块限制每个键的连接数，例如，单个IP地址的连接数。
```

**--without-stream_access_module**

```
禁用构建ngx_stream_access_module 模块，该 模块允许限制对某些客户端地址的访问。
```

**--without-stream_geo_module**

```
禁用构建ngx_stream_geo_module 模块，该 模块创建的变量值取决于客户端IP地址。
```

**--without-stream_map_module**

```
禁用构建ngx_stream_map_module 模块，该 模块创建的变量值取决于其他变量的值。
```

**--without-stream_split_clients_module**

```
禁用构建ngx_stream_split_clients_module 模块，该 模块创建用于A / B测试的变量。
```

**--without-stream_return_module**

```
禁用构建ngx_stream_return_module 模块，该 模块向客户端发送一些指定的值，然后关闭连接。
```

**--without-stream_upstream_hash_module**

```
禁用构建实现哈希 负载平衡方法的模块 。
```

**--without-stream_upstream_least_conn_module**

```
禁用构建实现了minimum_conn 负载平衡方法的模块 。
```

**--without-stream_upstream_zone_module**

```
禁用构建模块，该模块可以将上游组的运行时状态存储在共享内存 区域中。
```

**--with-google_perftools_module**

```
允许构建ngx_google_perftools_module 模块，该 模块可以使用Google Performance Tools对nginx工作进程进行 性能分析。该模块适用于Nginx开发人员，默认情况下未构建。
```

**--with-cpp_test_module**

```
启用构建 ngx_cpp_test_module模块。
```

**--add-module=path**

```
启用外部模块。
```

**--add-dynamic-module=path**

```
启用外部动态模块。
```

**--with-compat**

```
启用动态模块兼容性。
```

**--with-cc=path**

```
设置C编译器的名称。
```

**--with-cpp=path**

```
设置C预处理器的名称。
```

**--with-cc-opt=parameters**

```
设置将添加到CFLAGS变量的其他参数。在FreeBSD下使用系统PCRE库时， --with-cc-opt="-I /usr/local/include" 应指定。如果select()需要增加支持的文件数量，也可以在此处指定，例如： --with-cc-opt="-D FD_SETSIZE=2048"。
```

**--with-ld-opt=parameters**

```
设置将在链接期间使用的其他参数。在FreeBSD下使用系统PCRE库时， --with-ld-opt="-L /usr/local/lib" 应指定。
```

**--with-cpu-opt=cpu**

```
每个指定的CPU能够使建筑： pentium，pentiumpro， pentium3，pentium4， athlon，opteron， sparc32，sparc64， ppc64。
```

**--without-pcre**

```
禁用PCRE库的使用。
```

**--with-pcre**

```
强制使用PCRE库。
```

**--with-pcre=path** 

```
设置PCRE库源的路径。需要从PCRE站点下载并分发库分发（版本4.4 — 8.43） 。其余的由nginx的./configure和完成 make。该库对于location指令中的正则表达式支持和 ngx_http_rewrite_module 模块是必需的 。
```

**--with-pcre-opt=parameters**

```
为PCRE设置其他构建选项。
```

**--with-pcre-jit**

```
使用“及时编译”支持（1.1.12，pcre_jit指令）构建PCRE库 。
```

**--with-zlib=path** 

```
设置zlib库源的路径。需要从zlib站点下载库发行版（版本1.1.3-1.2.11） 并解压缩。其余的由nginx的./configure和完成 make。ngx_http_gzip_module模块需要该库 。
```

**--with-zlib-opt=parameters**

```
为zlib设置其他构建选项。
```

**--with-zlib-asm=cpu**

```
使得能够使用指定的CPU中的一个优化的zlib汇编源程序： pentium，pentiumpro。
```

**--with-libatomic**

```
强制使用libatomic_ops库。
```

**--with-libatomic=path**

```
设置libatomic_ops库源的路径。
```

**--with-openssl=path**

```
设置OpenSSL库源的路径。
```

**--with-openssl-opt=parameters**

```
为OpenSSL设置其他构建选项。
```

**--with-debug**

```
启用调试日志。
```
