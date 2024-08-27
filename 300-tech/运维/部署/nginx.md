---
title: nginx
date: 2024-08-22T01:47:30+08:00
updated: 2024-08-22T01:48:37+08:00
dg-publish: false
---

> Nginx (engine x) 是一个轻量级高性能的 HTTP 和反向代理服务器，采用事件驱动的异步非阻塞处理方式,具有较好的 IO 性能,时常用于服务端的反向代理和负载失衡

```
nginx -t             检查配置文件是否有语法错误
nginx -s reload       热加载，重新加载配置文件
nginx -s stop         快速关闭
nginx -s quit         等待工作进程处理完成后关闭

```

## 1\. 快速查看 nginx 配置文件的方法

> nginx 的配置放在 nginx.conf 文件中，一般我们可以使用以下命令查看服务器中存在的 nginx.conf 文件。

```
locate nginx.conf
/usr/local/etc/nginx/nginx.conf
/usr/local/etc/nginx/nginx.conf.default
...
```

如果服务器中存在多个 nginx.conf 文件，我们并不知道实际上调用的是哪个配置文件，因此我们必须找到实际调用的配置文件才能进行修改。

## 2\. 查看 nginx 实际调用的配置文件

> 查看 nginx 路径

```
ps aux|grep nginx
root              352   0.0  0.0  2468624    924   ??  S    10:43上午   0:00.08 nginx: worker process  
root              232   0.0  0.0  2459408    532   ??  S    10:43上午   0:00.02 nginx: master process /usr/local/opt/nginx/bin/nginx -g daemon off;  
root             2345   0.0  0.0  2432772    640 s000  S+    1:01下午   0:00.00 grep nginx
```

nginx 的路径为：/usr/local/opt/nginx/bin/nginx

> 查看 nginx 配置文件路径

```
/usr/local/opt/nginx/bin/nginx -t
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful

```

测试可知，nginx 的配置文件路径为：/usr/local/etc/nginx/nginx.conf 且调用有效。

## 3\. 首次安装

```
eg:  sudo apt-get install nginx (ubantu)
```

> 默认位置：
> /usr/sbin/nginx：主程序
> /etc/nginx：存放配置文件
> 默认使用/etc/nginx/conf.d/\*.conf 的配置，以后写 nginx 代理都放在 conf.d 目录下面。
> /usr/share/nginx：存放静态文件
> /var/log/nginx：存放日志

## 4\. 首次配置 nginx

> Nginx 安装目录下, 我们复制一份 `nginx.conf` 成 `nginx.conf.default` 作为配置文件备份，然后修改 `nginx.conf`

```
# 工作进程的数量
worker_processes  1;
events {
    worker_connections  1024; # 每个工作进程连接数
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    # 日志格式
    log_format  access  '$remote_addr - $remote_user [$time_local] $host "$request" '
                  '$status $body_bytes_sent "$http_referer" '
                  '"$http_user_agent" "$http_x_forwarded_for" "$clientip"';
    access_log  /srv/log/nginx/access.log  access; # 日志输出目录
    gzip  on;
    sendfile  on;

    # 链接超时时间，自动断开
    keepalive_timeout  60;

    # 虚拟主机
    server {
        listen       8080;
        server_name  localhost; # 浏览器访问域名

        charset utf-8;
        access_log  logs/localhost.access.log  access;

        # 路由
        location / {
            root   www; # 访问根目录
            index  index.html index.htm; # 入口文件
        }
    }

    # 引入其他的配置文件
    include servers/*;
}

```

> 在其他配置文件 `servers` 目录下，添加新建站点配置文件 xx.conf。

```
# 虚拟主机
server {
    listen       8080;
    server_name  xx_domian; # 浏览器访问域名

    charset utf-8;
    access_log  logs/xx_domian.access.log  access;

    # 路由
    location / {
        root   www; # 访问根目录
        index  index.html index.htm; # 入口文件
    }
}

```

> 根据文件类型设置过期时间

```
location ~.*\.css$ {
    expires 1d;
    break;
}
location ~.*\.js$ {
    expires 1d;
    break;
}

location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$ {
    access_log off;
    expires 15d;    #保存15天
    break;
}

# curl -x127.0.0.1:80 http://www.test.com/static/image/common/logo.png -I #测试图片的max-age

```

> 开发环境经常改动代码，由于浏览器缓存需要强制刷新才能看到效果。这是我们可以禁止浏览器缓存提高效率

```
location ~* \.(js|css|png|jpg|gif)$ {
    add_header Cache-Control no-store;
}
```

> 防盗链 (可以防止文件被其他网站调用)

```
location ~* \.(gif|jpg|png)$ {
    # 只允许 192.168.0.1 请求资源
    valid_referers none blocked 192.168.0.1;
    if ($invalid_referer) {
       rewrite ^/ http://$host/logo.png;
    }
}
```

> 静态文件压缩

```
server {
   gzip on;  # 开启gzip压缩 默认仅支持html文件
   gzip_http_version 1.1; # 压缩版本
   gzip_min_length 1k;  # 启用gzip压缩的最小文件，小于设置值的文件将不会压缩
   gzip_comp_level 2; # 压缩级别，1-9，数字越大压缩的越好，也越占用CPU时间
   gzip_types text/plain application/javascript application/x-javascript text/css application/xml 
 text/javascript application/x-httpd-php image/jpeg image/gif image/png;  # 进行压缩的文件类型
   gzip_disable "MSIE [1-6]\."; # 禁用IE 6 gzip
}

```

> 指定定错误页面

```
# 根据状态码，返回对于的错误页面
error_page 500 502 503 504 /50x.html;
location = /50x.html {
    root /source/error_page;
}
```

> 适配 pc 环境与移动环境

用于实现根据用户的浏览环境自动切换站点。Nginx 可以通过内置变量 $http\_user\_agent 识别出用户是 pc 端还是移动端，进而控制重定向到 H5 站还是 PC 站。 配置如下

```
location / {
    if ($http_user_agent ~* '(Android|webOS|iPhone|iPod)') {
        set $mobile_request '1';
    }
    if ($mobile_request = '1') {
        rewrite ^.+ http://mysite-base-H5.com; # 重定向
    }
}

```

> 代理和跨域

在 server1.com 向 server2.com 发起请求时,可以配置代理,或者通过设置请求头解决跨域.配置如下:

```
server {
    listen 80; # 监听端口
    server_name  http://server1.com; ## 当前服务器名称
    location / {
        proxy_pass http://server2.com; # 进行服务器代理,也可依次实现跨域
        add_header Access-Control-Allow-Origin *; # 设置请求头实现跨域
    }
}
```

### 负载均衡

> nginx 负载均衡策略:

- 轮询 (默认): 每个请求按照时间顺序逐一分配到不同的后端服务器,如果服务器 down 掉,可以自动剔除

```
upstream balanceServer {
    server  10.1.22.33:12345;
    server  10.1.22.34:12345;
}
```

- ip\_hash: 给每个访问 ip 指定 hash,根据 hash 固定访问某个服务器,由此可以解决 session 不能跨域的问题

```
upstream balanceServer {
    ip_hash;
    server 10.1.22.33:12345;
    server 10.1.22.34:12345;
}
```

- weight: 加权轮询,weight 和访问比率成正比,用于后端服务器性能不均

```
upstream balanceServer {
    server  10.1.22.33:12345 weight=2;
    server  10.1.22.34:12345 weight=1;
}
```

- least\_conn: 最小连接数 哪个连接少就分配给谁

```
upstream balanceServer {
    least_conn;
    server 10.1.22.33:12345;
    server 10.1.22.34:12345;
}
```

> 跨域的定义
> 同源策略限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的重要安全机制。通常不允许不同源间的读操作。

> 同源的定义
> 如果两个页面的协议，端口（如果有指定）和域名都相同，则两个页面具有相同的

nginx 解决跨域的原理

例如：

前端 server 域名为：http://xx\_domain

后端 server 域名为：[https://github.com](https://github.com/)

现在 http://xx\_domain 对 [https://github.com发起请求一定会出现跨域。](https://github.xn--com-s18dk4cg9hezdmrlrviio7b5crys0evceovb./)

不过只需要启动一个 nginx 服务器，将 server\_name 设置为 xx\_domain,然后设置相应的 location 以拦截前端需要跨域的请求，最后将请求代理回 github.com。如下面的配置：

```
## 配置反向代理的参数
server {
    listen    8080;
    server_name xx_domain

    ## 1. 用户访问 http://xx_domain，则反向代理到 https://github.com
    location / {
        proxy_pass  https://github.com;
        proxy_redirect     off;
        proxy_set_header   Host             $host;        # 传递域名
        proxy_set_header   X-Real-IP        $remote_addr; # 传递ip
        proxy_set_header   X-Scheme         $scheme;      # 传递协议
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
    }
}

```

```
cd /etc/nginx/conf.d/
vi docs.conf

server {
    listen 80;
    server_name docs.icodin.cn;
    rewrite ^(.*) https://$server_name$1 permanent; // 自动从http跳转到https
}
server{
    listen      443 ssl;  // 端口出来443还可以是其他端口，访问时加上端口号即可，同时开启ssl
    server_name docs.icodin.cn; // 可以使用localhost也可以使用自己的域名，记得将dns记录指向你的服务器ip
    charset     utf-8;
    client_max_body_size 75M;
    ssl_certificate      /etc/nginx/conf.d/candy.crt; // 等会会生成的证书
    ssl_certificate_key  /etc/nginx/conf.d/candy.key;
    ssl_ciphers ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;
    location / {
      root /home/ubuntu/web/docs/;  // 将你的静态文件放在这个目录，默认渲染index.html或者index.htm
    }
}
```

## 5\. 生成证书

简单粗暴，在/etc/nginx/conf.d 目录下面执行,反正遇到 permission deny 啥的直接加 sudo 就好了

```
openssl genrsa -des3 -out candy.key 1024 // 生成私钥
openssl req -new -key candy.key -out candy.csr // 生成证书签名请求
openssl rsa -in candy.key-out candy.key // 移除私钥的密码
openssl x509 -req -days 365 -in candy.csr -signkey candy.key -out candy.crt // 生成证书
ls
candy.crt  candy.csr  candy.key

```

## 6\. vue 项目更改为 histroy 模式

```
1. vue.config.js 更改

publicPath: '/admin',

2. nginx 配置
   location ^~ /admin {
      root /root/html/dist;
      index index.html;
      try_files $uri $uri/ /admin/index.html;
   }
   location ~ \.css {
      add_header Content-Type text/css;
   }
   location ~ \.js {
      add_header Content-Type application/x-javascript;
   }
```
