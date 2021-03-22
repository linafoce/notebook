# nginx应用

## 基本概念

### 正向代理和反向代理

正向代理即是客户端代理, 代理客户端, 服务端不知道实际发起请求的客户端

反向代理即是服务端代理, 代理服务端, 客户端不知道实际提供服务的服务端

正向代理中，proxy和client同属一个LAN，对server透明；反向代理中，proxy和server同属一个LAN，对client透明。实际上proxy在两种代理中做的事都是代为收发请求和响应，不过从结构上来看正好左右互换了下，所以把后出现的那种代理方式叫成了反向代理

正向代理: 买票的黄牛

反向代理: 租房的代理

### 负载均衡

用来在多个计算机（计算机集群）、网络连接、CPU、磁盘驱动器或其他资源中分配负载，以达到最优化资源使用、最大化吞吐率、最小化响应时间、同时避免过载的目的。

## 安装部署

1、/etc/yum.repos.d 文件下新建nginx.repo

文件内容：

```
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
[nginx-mainline]
name=nginx mainline repo
baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
gpgcheck=1
enabled=0
gpgkey=https://nginx.org/keys/nginx_signing.key
```



也可以使用命令：rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm/

2、执行命令：yum install -y nginx

### 基本命令

防火墙相关：

开放端口列表：firewall-cmd --list-all

添加开放端口：firewall-cmd --add-port=8080/tcp --permanent

重新加载防火墙配置：firewall-cmd --reload

关闭防火墙：systemctl stop firewalld.service

监听列表：netstat -tlnp

安装ab测试工具：yum -y install httpd-tools

apache  ab测试（压力测试）：ab -kc 1000 -n 1000 http://127.0.0.1/

## 配置文件

| 域名称   | 域类型 | 域说明                                                       |
| -------- | ------ | ------------------------------------------------------------ |
| main     | 全局域 | Nginx 的根级别指令区域。该区域的配置指令是全局有效的，该指令名为隐性显示，nginx.conf 的整个文件内容都写在该指令域中 |
| events   | 指令域 | Nginx 事件驱动相关的配置指令域                               |
| http     | 指令域 | Nginx HTTP 核心配置指令域，包含客户端完整 HTTP 请求过程中每个过程的处理方法的配置指令 |
| upstream | 指令域 | 用于定义被代理服务器组的指令区域，也称“上游服务器”           |
| server   | 指令域 | Nginx 用来定义服务 IP、绑定端口及服务相关的指令区域          |
| location | 指令域 | 对用户 URI 进行访问路由处理的指令区域                        |
| stream   | 指令域 | Nginx 对 TCP 协议实现代理的配置指令域                        |
| types    | 指令域 | 定义被请求文件扩展名与 MIME 类型映射表的指令区域             |
| if       | 指令域 | 按照选择条件判断为真时使用的配置指令域                       |

配置文件名：安装目录下 nginx.conf

格式

```
user  nginx;
worker_processes  1;#工作进程个数，建议匹配CPU核数
error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;#进程号定义文件
events {
    worker_connections  1024;#定义工作进程支持最大连接数
}
#web功能核心配置
http {
    include       /etc/nginx/mime.types; #支持的文件类型
    default_type  application/octet-stream;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;#访问日志文件
    sendfile        on;
    #tcp_nopush     on;
    keepalive_timeout  65;
    #gzip  on;
   
    #负载均衡配置
    upstream springboot {
        server 127.0.0.1:8080 weight=80;
        server 127.0.0.1:8081 weight=20;
    }
    #虚拟主机配置    一个server对应一台虚拟主机
    server{
        listen 80;#虚拟主机的访问端口  也可以配  ip:port
        server_name localhost;#虚拟主机的访问域名   如果没有可以填localhost或则下划线 _
        #server_name只有在需要区分listen匹配的多个结果时才会被使用
       
        #nginx路径匹配
        location / {
            root html;#定义网页根目录，相对路径
            index index.html;#默认首页(可以写多个)
            proxy_pass http://springboot;
        }
    }
    include /etc/nginx/conf.d/*.conf;
}
```

## 应用配置

nginx的配置指令可以分为两大类：指令块与单个指令。指令块就是像events，http，server等，单独指令就是像root html;这样的。

指令块可以嵌套，http块中可以嵌套server指令，server块中可以嵌套location指令，指令可以同时出现在不同的指令块，如root指令可以同时出现在http、server和location指令块，在location中定义的指令会覆盖server，http的指令。作用范围受大括号限制

### 接入日志

access_log 文件路径

log_format 日志内容

### 错误日志

error_log 文件路径

### 访问控制

限制http块访问ip

```
http { #也可以配置在server和location块中
    #限制192.168.40.150这个ip访问
    deny 192.168.40.150;#黑名单  报403  也可以配置网段和all
    allow 192.168.40.151;#白名单 
}
```



### 多虚拟主机配置

基于ip、端口、域名

![多虚拟主机配置.png](https://cdn.nlark.com/yuque/0/2021/png/1618119/1610112104367-cc997628-029e-4609-ab56-a201a64e155d.png?x-oss-process=image%2Fwatermark%2Ctype_d3F5LW1pY3JvaGVp%2Csize_10%2Ctext_6bKB54-t5a2m6Zmi5Ye65ZOB%2Ccolor_FFFFFF%2Cshadow_50%2Ct_80%2Cg_se%2Cx_10%2Cy_10)

访问：http://192.168.50.161/test/    http://192.168.50.151/test/

### 静态资源压缩

```
server、location
gzip on;
gzip_http_version 1.1;
gzip_comp_level 4;#压缩等级，数字越小压缩率越高，cpu消耗越大
gzip_types text/plain application/javascript text/css image/png;#支持压缩的文件格式
```



### 状态页

```
server {
    listen 85;
    location / {
        stub_status on;#开启状态页功能
        access_log off;#关闭访客日志功能
    }
}
访问 localhost:85页面显示：
Active connections: 2    #活动链接数
server accepts handled requests
 555 555 1024 
Reading: 0 Writing: 1 Waiting: 1
```



server: nginx启动后一共处理的请求数

accepts handled ：nginx启动后创建的握手数

requests：nginx一共处理的请求数

reading：nginx读取到客户端的headers数量

writing：nginx响应给客户端的headers数量

Waiting:nginx处理完毕请求之后，等待下一次请求驻留的连接数（active-（Reading+Writing））

### 目录浏览

```
location / {
        root html;#根目录下的index.html不能存在  否则浏览器会默认打开该页面
        #index index.html; 关闭虚拟主机的默认首页
        autoindex on; #展示root所定义的目录内容
}
```



### 身份认证

```
yum install -y httpd-tools
生成秘钥文件：htpasswd -bc /data/nginx/htpasswd.user 用户名 密码
server{
        listen 8091;
        server_name 192.168.50.151;
        auth_basic "Restricted Access";
        auth_basic_user_file /data/nginx/htpasswd.user;
        location / {
            root html1/static;
            index index.html;
        }
}
```



### location匹配

location表达式类型

如果直接写一个路径，则匹配该路径下的

~ 表示执行一个正则匹配，区分大小写

~* 表示执行一个正则匹配，不区分大小写

^~ 表示普通字符匹配。使用前缀匹配。如果匹配成功，则不再匹配其他location。

= 进行普通字符精确匹配。也就是完全匹配。

优先级

等号类型（=）的优先级最高。一旦匹配成功，则不再查找其他匹配项。

^~类型表达式。一旦匹配成功，则不再查找其他匹配项。

正则表达式类型（~ ~*）的优先级次之。如果有多个location的正则能匹配的话，则使用正则表达式最长的那个。

常规字符串匹配类型。按前缀匹配。

root与alias主要区别在于nginx如何解释location后面的uri，这会使两者分别以不同的方式将请求映射到服务器文件上。

root的处理结果是：root路径＋location路径

alias的处理结果是：使用alias路径替换location路径

alias是一个目录别名的定义，root则是最上层目录的定义。

还有一个重要的区别是alias后面必须要用“/”结束，否则会找不到文件的，而root则可有可无

### url重写

rewrite功能就是，使用nginx提供的全局变量或自己设置的变量，结合正则表达式和标志位实现url重写以及重定向。rewrite只能放在server{},location{},if{}中，并且只能对域名后边的除去传递的参数外的字符串起作用，例如`http://luban.com/a/we/index.jsp?id=1&u=str` 只对/a/we/index.php重写。语法`rewrite regex replacement [flag];`

如果相对域名或参数字符串起作用，可以使用全局变量匹配，也可以使用proxy_pass反向代理。

表明看rewrite和location功能有点像，都能实现跳转，主要区别在于rewrite是在同一域名内更改获取资源的路径，而location是对一类路径做控制访问或反向代理，可以proxy_pass到其他机器。很多情况下rewrite也会写在location里，它们的执行顺序是：

1. 执行server块的rewrite指令
2. 执行location匹配
3. 执行选定的location中的rewrite指令

如果其中某步URI被重写，则重新循环执行1-3，直到找到真实存在的文件；循环超过10次，则返回500 Internal Server Error错误

rewrite 规则 定向路径 重写类型;

用在server和location模块，if模块

```
server {
    # 访问 /last.html 的时候，页面内容重写到 /index.html 中
    rewrite /last.html /index.html last;
    # 访问 /break.html 的时候，页面内容重写到 /index.html 中，并停止后续的匹配
    rewrite /break.html /index.html break;
    # 访问 /redirect.html 的时候，页面直接302定向到 /index.html中
    rewrite /redirect.html /index.html redirect;
    # 访问 /permanent.html 的时候，页面直接301定向到 /index.html中
    rewrite /permanent.html /index.html permanent;
    # 把 /html/*.html => /post/*.html ，301定向
    rewrite ^/html/(.+?).html$ /post/$1.html permanent;
    # 把 /search/key => /search.html?keyword=key
    rewrite ^/search\/([^\/]+?)(\/|$) /search.html?keyword=$1 permanent;
}
```



- 规则：可以是字符串或者正则来表示想匹配的目标url
- 定向路径：表示匹配到规则后要定向的路径，如果规则里有正则，则可以使用`$index`来表示正则里的捕获分组
- 重写类型：

- - last ：相当于Apache里德(L)标记，表示完成rewrite，浏览器地址栏URL地址不变
  - break；本条规则匹配完成后，终止匹配，不再匹配后面的规则，浏览器地址栏URL地址不变
  - redirect：返回302临时重定向，浏览器地址会显示跳转后的URL地址
  - permanent：返回301永久重定向，浏览器地址栏会显示跳转后的URL地址

last和break的区别

因为301和302不能简单的只返回状态码，还必须有重定向的URL，这就是return指令无法返回301,302的原因了。这里 last 和 break 区别有点难以理解：

- last一般写在server和if中，而break一般使用在location中
- last不终止重写后的url匹配，即新的url会再从server走一遍匹配流程，而break终止重写后的匹配
- break和last都能阻止继续执行后面的rewrite指令

在location里一旦返回break则直接生效并停止后续的匹配location

```
server {
    location / {
        rewrite /last/ /q.html last;
        rewrite /break/ /q.html break;
    }
    location = /q.html {
        return 400;
    }
}
```



访问/last/时重写到/q.html，然后使用新的uri再匹配，正好匹配到locatoin = /q.html然后返回了400

访问/break时重写到/q.html，由于返回了break，则直接停止了

```
server {
    # 用 xxoo_admin 来掩饰 admin
    location / {
        # 使用break拿一旦匹配成功则忽略后续location
        rewrite /xxoo_admin /admin break;
    }
    # 访问真实地址直接报没权限
    location /admin {
        return 403;
    }
}
```



### 限流

一是控制速率，二是控制并发连接数。

#### 控制速率

##### 正常限流

漏桶算法(leaky bucket)

在 nginx.conf http 中添加限流配置：

格式：limit_req_zone key zone rate

   

```
http {
    limit_req_zone $binary_remote_addr zone=myRateLimit:10m rate=10r/s;
}
server {
    location / {
        limit_req zone=myRateLimit;
        proxy_pass http://my_upstream;
    }
}
```



- **key** ：定义限流对象，**binary_remote_addr** 是一种key，表示基于 **remote_addr(客户端IP)** 来做限流，**binary_** 的目的是压缩内存占用量。
- **zone**：定义共享内存区来存储访问信息， **myRateLimit:10m** 表示一个大小为10M，名字为myRateLimit的内存区域。1M能存储16000 IP地址的访问信息，10M可以存储16W IP地址访问信息。
- **rate** 用于设置最大访问速率，**rate=10r/s** 表示每秒最多处理10个请求。Nginx 实际上以毫秒为粒度来跟踪请求信息，因此 10r/s 实际上是限制：**每100毫秒处理一个请求**。这意味着，自上一个请求处理完后，若后续100毫秒内又有请求到达，将拒绝处理该请求。

##### 突发限流

```
server {
    location / {
        limit_req zone=myRateLimit burst=20 nodelay;
        proxy_pass http://my_upstream;
    }
}
```



**burst**表示在超过设定的处理速率后能额外处理的请求数。当 **rate=10r/s** 时，将1s拆成10份，即每100ms可处理1个请求。

此处，**burst=20** ，若同时有21个请求到达，Nginx 会处理第一个请求，剩余20个请求将放入队列，然后每隔100ms从队列中获取一个请求进行处理。若请求数大于21，将拒绝处理多余的请求，直接返回503.

不过，单独使用 **burst** 参数并不实用。假设 **burst=50** ，rate依然为10r/s，排队中的50个请求虽然每100ms会处理一个，但第50个请求却需要等待 50 * 100ms即 5s，这么长的处理时间自然难以接受。

因此，**burst** 往往结合 **nodelay** 一起使用。

**nodelay** 针对的是 burst 参数，**burst=20 nodelay** 表示这20个请求立马处理，不能延迟，相当于特事特办。不过，即使这20个突发请求立马处理结束，后续来了请求也不会立马处理。**burst=20** 相当于缓存队列中占了20个坑，即使请求被处理了，这20个位置这只能按 100ms一个来释放。

这就达到了速率稳定，但突然流量也能正常处理的效果。

#### 控制并发连接数

```
http{
    limit_conn_zone $binary_remote_addr zone=perip:10m;
    limit_conn_zone $server_name zone=perserver:10m;
}
server {
    ...
    limit_conn perip 10;
    limit_conn perserver 100;
}
```



**limit_conn perip 10** 作用的key 是 **$binary_remote_addr**，表示限制单个IP同时最多能持有10个连接。

**limit_conn perserver 100** 作用的key是 **$server_name**，表示虚拟主机(server) 同时能处理并发连接的总数。

需要注意的是：只有当 **request header** 被后端server处理后，这个连接才进行计数。

#### 设置白名单

```
http配置
geo $limit {
    default 1;
    10.0.0.0/8 0;
    192.168.0.0/24 0;
    172.20.0.35 0;
}
map $limit $limit_key {
    0 "";
    1 $binary_remote_addr;
}
limit_req_zone $limit_key zone=myRateLimit:10m rate=10r/s;
```



**geo** 对于白名单(子网或IP都可以) 将返回0，其他IP将返回1。

**map** 将 limit转换为limit_key**，如果是** $limit 是0(白名单)，则返回空字符串；如果是1，则返回客户端实际IP。

**limit_req_zone** 限流的key不再使用 binary，remote，addr ， 而是limit_key 来动态获取值。如果是白名单，limit_req_zone 的限流key则为空字符串，**将不会限流**；若不是白名单，将会对客户端真实IP进行限流。

### 动静分离

### 反向代理，负载均衡 

```
upstream springboot {
    server 127.0.0.1:8080 weight=80;
    server 127.0.0.1:8081 weight=20;
}
server{
    listen 90;
    #   server_name localhost;
    location / {
        proxy_pass http://springboot;
    }
}
```



### 长链接配置

client端长链接：默认开启了keep alive支持

```
http {
    keepalive_timeout  120s 120s;
    keepalive_requests 10000;
}
```

keepalive_timeout：

第一个参数：设置keep-alive客户端连接在服务器端保持开启的超时值（默认75s）；值为0会禁用keep-alive客户端连接；

第二个参数：可选、在响应的header域中设置一个值“Keep-Alive: timeout=time”；通常可以不用设置；

keepalive_requests：

设置一个keep-alive连接上可以服务的请求的最大数量，当最大请求数量达到时，连接被关闭。默认是100。

keep alive建立之后，nginx就会为这个连接设置一个计数器，记录这个keep alive的长连接上已经接收并处理的客户端请求的数量。如果达到这个参数设置的最大值时，则nginx会强行关闭这个长连接，客户端重新建立新的长连接。

服务端长链接：默认短链接

```
http {
    upstream backend {
      server 192.168.0.1:8080 weight=1 max_fails=2 fail_timeout=30s;
      server 192.168.0.2:8080 weight=1 max_fails=2 fail_timeout=30s;
      keepalive 300; #连接池最大空闲连接数（相当于核心线程数）
    }   
    server {
        listen 8080 default_server;
        server_name "";
        location / {
            proxy_pass http://backend;
            proxy_http_version 1.1;# 设置http版本为1.1
            proxy_set_header Connection "";# 清理Connection，这样即便是 Client 和 Nginx 之间是短连接，Nginx 和 upstream 之间也是可以开启长连接,也可以通过传递“Connection: Keep-Alive”头
        }
    }
}
```



### 连接池

```
worker_processes  8;#定义工作进程数量
 
events {
        use epoll;
        worker_connections  2048;#定义工作进程的连接池大小
}

同一个进程，上游下游共用一个连接池
```