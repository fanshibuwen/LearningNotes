[TOC]

# Nginx学习总结

## 什么是nginx

说白了就是一个服务器

- 代理静态资源

  - 假如说本地有一个图片需要上传到前端页面上，那么就是使用nginx，将本地图片的所在位置和访问的ip和端口配置在nginx.config文件中，并在yml配置文件中中做出相应的配置。例如

  ```yaml
  upload:
    url: http://localhost:83
    path: D:\19769\Pictures
  ```

  ```js
  server {
  
  ​    listen  83;
  
  ​    server_name localhost;
  
  ​    location / {
  
  ​      root D:\19769\Pictures;
  
  ​    }
  
    }
  ```

  例如做出以上配置之后，在前端页面上的图片连接都将是http://localhost:83/xxxx，然后他们都指向D:\19769\Pictures位置。

- 负载均衡

  - 采用轮询或者其他的方式做集群服务器的负载均衡，将负载分别分发到不同的服务器上，减少服务器的压力。一般来说就是将请求转发给服务器集群。

- 反向代理

  - 反向代理是相对于正向代理来说的。

    - 正向代理： 一般的访问流程是客户端直接向目标服务器发送请求并获取内容，使用正向代理后，客户端改为向代理服务器发送请求，并指定目标服务器（原始服务器），然后由代理服务器和原始服务器通信，转交请求并获得的内容，再返回给客户端。正向代理隐藏了真实的客户端，为客户端收发请求，使真实客户端对服务器不可见；==正向代理是站在客户端的角度来说的，客户知道服务器是谁但是服务器不知道客户端是谁，认为是代理服务器==

      举个具体的例子 ??，你的浏览器无法直接访问谷哥，这时候可以通过一个代理服务器来帮助你访问谷哥，那么这个服务器就叫正向代理。

    - 反向代理：与一般访问流程相比，使用反向代理后，直接收到请求的服务器是代理服务器，然后将请求转发给内部网络上真正进行处理的服务器，得到的结果返回给客户端。反向代理隐藏了真实的服务器，为服务器收发请求，使真实服务器对客户端不可见。一般在处理跨域请求的时候比较常用。现在基本上所有的大型网站都设置了反向代理。==反向代理是站在服务器的角度来说的，服务器知道客户端是谁，客户端不知道服务器是谁，认为是代理服务器==

      举个具体的例子 ??，去饭店吃饭，可以点川菜、粤菜、江浙菜，饭店也分别有三个菜系的厨师 ?????，但是你作为顾客不用管哪个厨师给你做的菜，只用点菜即可，小二将你菜单中的菜分配给不同的厨师来具体处理，那么这个小二就是反向代理服务器。

- 动静分离

  - 相当于就是把动态页面和静态资源分开存放，由不同的服务器解析。
  - 一般来说，都需要将动态资源和静态资源分开，由于 Nginx 的高并发和静态资源缓存等特性，经常将静态资源部署在 Nginx 上。如果请求的是静态资源，直接到静态资源目录获取资源，如果是动态资源的请求，则利用反向代理的原理，把请求转发给对应后台应用去处理，从而实现动静分离。

## 安装

### yum安装

先看看执行这个命令之后有什么

```perl
yum list | grep nginx
```

安装

```perl
yum install nginx
```

查看版本

```perl
nginx -v
```

查看配置信息

```perl
cat /etc/nginx/nginx.config
```

看到nginx的默认端口信息，一般是80端口

如果是云，则需要在安全组里面开放相应的端口80，并在防火墙里打开80端口。如果是本地的，则只需要在防火墙里开放相应端口。

```perl
firewall-cmd --permanent --zone=public add-port=端口号/tcp
```

permanent：永久打开

public：都可以访问

此时就可以启动nginx

```perl
systemctl start nginx
```

此时在外部访问自己服务器的ip地址的80端口，会发现是nginx的默认欢迎页面或者是centOS的欢迎页面。

查看启动的默认页面

```perl
cd /usr/share/nginx/html/
```

通常是这个目录下的index.html文件

可以开两个窗口，然后一个打开日志文件，在另一个窗口上操作

```perl
cd /var/log/nginx/   # 日志文件所在位置
 
ls   # 发现有成功日志和错误日志两种 

tail -f access.log/error.log  # -f的意思是持续输出
```

### 相关命令

#### 防火墙相关

```perl
systemctl start firewalld # 开启防火墙

systemctl stop firewalld # 关闭防火墙

systemctl status firewalld # 查看防火墙状态

firewall-cmd reload # 重启防火墙（在开放端口后需要执行的操作）

firewall-cmd --list-all # 查看所有防火墙开放的端口
```

#### nginx相关(除了最后一个都不常用)

````perl
systemctl enable nginx # 设置ngix开机自启

nginx -s reload # 向主进程发送信号，重新加载配置文件，热重启

nginx -s reopen # 重启nginx

nginx -s stop # 关闭nginx

nginx -t # 检查nginx当前配置文件是存在语法错误（常用）
````

#### systemctl相关

systemctl是Linux系统应用管理systemd的命令，用于管理系统，我们也可以用来管理nginx

```perl
systemctl start nginx # 启动nginx

systemctl stop nginx # 关闭nginx

systemctl restart nginx # 重启nginx

systemctl reload nginx # 重新加载nginx，常用于修改nginx配置文件之后

systemctl enable nginx # 设置nginx开机自启

systemctl disable nginx # 关闭nginx开机自启

systemctl status nginx # 查看nginx当前状态 是否激活
```

## 配置

### nginx配置文件相关配置

配置文件的位置主要就是/etc/nginx/nginx.conf。在改配置文件之前记得先备份，防止自己该废了回不去。

```perl
events {     # 配置影响Nginx服务器或与用户的网络连接
    worker_connections 1024; # 每个进程允许最大并发数
}

http { # 配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置

    server { # 配置虚拟主机的相关参数，一个 http 块中可以有多个 server 块

	listen 8888;  # 监听8888端口
	server_name 101.42.251.80;   # 访问ip
	location / {
		root /usr/share/nginx/zhaol/eduaddpage/;  # 代理到这个目录下
		index index.html; # 首页面定位index.html
		allow all; # 允许所有人访问
		}
	}

    server {
        listen       80;
       # listen       [::]:80;
        server_name  localhost;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;
        error_page 404 /404.html;
        location = /404.html {
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
```

### nginx配置解决CORS

#### 反向代理解决跨域问题

- 所谓跨域，就是由于访问的服务地址与请求地址 协议或者域名或者端口不一致，导致的浏览器产生的CORS报错。这是由于浏览器的同源策略导致的，使用postman等调试工具就不会出现跨域问题。

在前端服务地址为 `fe.sherlocked93.club` 的页面请求 `be.sherlocked93.club` 的后端服务导致的跨域，可以这样配置：

```vbscript
server {


  listen 9001;


  server_name fe.sherlocked93.club;


  location / {


    proxy_pass be.sherlocked93.club;

  }

}
```

这样就将对前一个域名 `fe.sherlocked93.club` 的请求全都代理到了 `be.sherlocked93.club`，前端的请求都被我们用服务器代理到了后端地址下，绕过了跨域。

#### 配置header解决跨域（常用）

在配置文件中新建conf文件，对应原来要跨域的那个站点的名称

比如前端站点是`fe.sherlocked93.club`，在这个地址下的前端页面请求 `be.sherlocked93.club` 下的资源

这时候需要在刚刚新建的配置文件中填写以下配置

```perl
# /etc/nginx/conf.d/be.sherlocked93.club.conf
server {

  listen       80;

  server_name  be.sherlocked93.club;

	add_header 'Access-Control-Allow-Origin' $http_origin;   # 全局变量获得当前请求origin，带cookie的请求不支持*

	add_header 'Access-Control-Allow-Credentials' 'true';    # 为 true 可带上 cookie

	add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';  # 允许请求方法

	add_header 'Access-Control-Allow-Headers' $http_access_control_request_headers;  # 允许请求的 header，可以为 *
	
	add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';

  if ($request_method = 'OPTIONS') {

		add_header 'Access-Control-Max-Age' 1728000;   # OPTIONS 请求的有效期，在有效期内不用发出另一条预检请求

		add_header 'Content-Type' 'text/plain; charset=utf-8';

		add_header 'Content-Length' 0;

		return 204;                  # 200 也可以

	}

	location / {

		root  /usr/share/nginx/html/be;

		index index.html;

	}

}
```

配置完成之后

```perl
nginx -t # 检查nginx配置

nginx -s reload # 重新加载配置文件
```

这时候再次在原来站点跨域请求资源，可以接收到相应的资源

#### nginx配置实现网页压缩

- 就是开启gzip压缩，网页经过压缩之后可以变成原来的一半以下。目的就是节约带宽，增加传输速度。

使用 gzip 不仅需要 Nginx 配置，浏览器端也需要配合，需要在请求消息头中包含 `Accept-Encoding: gzip`（IE5 之后所有的浏览器都支持了，是现代浏览器的默认设置）。一般在请求 html 和 css 等静态资源的时候，支持的浏览器在 request 请求静态资源的时候，会加上 `Accept-Encoding: gzip` 这个 header，表示自己支持 gzip 的压缩方式，Nginx 在拿到这个请求的时候，如果有相应配置，就会返回经过 gzip 压缩过的文件给浏览器，并在 response 相应的时候加上 `content-encoding: gzip` 来告诉浏览器自己采用的压缩方式（因为浏览器在传给服务器的时候一般还告诉服务器自己支持好几种压缩方式），浏览器拿到压缩的文件后，根据自己的解压方式进行解析。

还是在配置文件所在目录新建gip.conf

```perl
# /etc/nginx/conf.d/gzip.conf

gzip on; # 默认off，是否开启gzip

gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

# 上面两个开启基本就能跑起了，下面的愿意折腾就了解一下

gzip_static on;

gzip_proxied any;

gzip_vary on;

gzip_comp_level 6;

gzip_buffers 16 8k;

# gzip_min_length 1k;

gzip_http_version 1.1;
```

稍微解释一下：

1. **gzip_types**：要采用 gzip 压缩的 MIME 文件类型，其中 text/html 被系统强制启用；
2. **gzip_static**：默认 off，该模块启用后，Nginx 首先检查是否存在请求静态文件的 gz 结尾的文件，如果有则直接返回该 `.gz` 文件内容；
3. **gzip_proxied**：默认 off，nginx做为反向代理时启用，用于设置启用或禁用从代理服务器上收到相应内容 gzip 压缩；
4. **gzip_vary**：用于在响应消息头中添加 `Vary：Accept-Encoding`，使代理服务器根据请求头中的 `Accept-Encoding` 识别是否启用 gzip 压缩；
5. **gzip_comp_level**：gzip 压缩比，压缩级别是 1-9，1 压缩级别最低，9 最高，级别越高压缩率越大，压缩时间越长，建议 4-6；
6. **gzip_buffers**：获取多少内存用于缓存压缩结果，16 8k 表示以 8k*16 为单位获得；
7. **gzip_min_length**：允许压缩的页面最小字节数，页面字节数从header头中的 `Content-Length` 中进行获取。默认值是 0，不管页面多大都压缩。建议设置成大于 1k 的字节数，小于 1k 可能会越压越大；
8. **gzip_http_version**：默认 1.1，启用 gzip 所需的 HTTP 最低版本；

#### nginx配置负载均衡

```perl
http {

  upstream myserver {  #这是配置均衡到哪些服务器上

  	# ip_hash;  # ip_hash 方式

    # fair;   # fair 方式

    server 127.0.0.1:8081;  # 负载均衡目的服务地址

    server 127.0.0.1:8080;

    server 127.0.0.1:8082 weight=10;  # weight 方式，不写默认为 1

  }

  server {

    location / {

    	proxy_pass http://myserver; # 反向代理到这些服务器集群上，对应上面的 upstream myserver

      proxy_connect_timeout 10; # 代理连接超时时间

    }

  }

}
```

nginx实现负载均衡方式：轮询（默认）、加权轮询、ip_hash、fair（第三方）

ip_hash：根据请求用户的IP地址进行hash运算，之后分配到不同的服务器上，然后下次这个用户再次访问的时候，还是会把请求分配到指定的服务器上。这样的话，就使不同权重失效。好处是，这样直接解决了动态网页sesssion共享的问题。在普通的负载均衡中，负载均衡每次会将请求重新定位到服务器集群中的一个服务器，那么已经登录服务器的用户再重新定位到另一个服务器，其登录信息将会丢失，这样显然是不妥的。

## 剽窃老师的一些经验

### 图片防盗链

```pel
server {

  listen       80;

  server_name  *.sherlocked93.club;

  # 图片防盗链

  location ~* \.(gif|jpg|jpeg|png|bmp|swf)$ {

    valid_referers none blocked 192.168.0.2;  # 只允许本机 IP 外链引用

    if ($invalid_referer){

      return 403;

    }

  }

}
```

### 请求过滤

```perl
# 非指定请求全返回 403

if ( $request_method !~ ^(GET|POST|HEAD)$ ) {

  return 403;

}

location / {

  # IP访问限制（只允许IP是 192.168.0.2 机器访问）

  allow 192.168.0.2;

  deny all;

  root   html;

  index  index.html index.htm;

}
```

### 请求转发

就是比如说是http转发到https上

```perl
server {

    listen      80;

    server_name www.sherlocked93.club;

    # 单域名重定向   只是将指定的url重定向到https上去

    if ($host = 'www.sherlocked93.club' && $scheme != 'https'){

        return 301 https://www.sherlocked93.club$request_uri;
        
    }

    # 全局非 https 协议时重定向   所有的url请求只要协议不是https，就都转发到https上，并返回状态码301

    if ($scheme != 'https') {

        return 301 https://$server_name$request_uri;

    }

    # 或者全部重定向 简单点，不管什么请求，都重定向一次

    return 301 https://$server_name$request_uri;

    # 以上配置选择自己需要的即可，不用全部加

}
```



