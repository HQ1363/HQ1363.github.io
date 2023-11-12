---
title: nginx配置实战   
date: 2023-06-24 14:36:50   
tags: nginx   
categories: Nginx   
excerpt: 日常工作中，经常需要同nginx打交道；没有经验的情况下，配置起来很是鸡肋，无法达到想要的效果，这里列举一些常见的nginx应用场景的配置.
---

## 前端路由结合Nginx
> 如果你遇到 https://cdn.com/users/123 刷新后 404 的问题，你需要按照这个章节进行处理。

Ant Design Pro 使用的 Umi 支持两种路由方式：browserHistory 和 hashHistory。

可以在 config/config.js 中进行配置选择用哪个方式：

```shell
export default {
  history: 'hash', // 默认是 browser
};
```
hashHistory 使用如 https://cdn.com/#/users/123 这样的 URL，取井号后面的字符作为路径。browserHistory 则直接使用 https://cdn.com/users/123 这样的 URL。使用 hashHistory 时浏览器访问到的始终都是根目录下 index.html。使用 browserHistory 则需要服务器做好处理 URL 的准备，处理应用启动最初的 / 这样的请求应该没问题，但当用户来回跳转并在 /users/123 刷新时，服务器就会收到来自 /users/123 的请求，这时你需要配置服务器能处理这个 URL 返回正确的 index.html。强烈推荐使用默认的 browserHistory。
强烈推荐使用默认的 browserHistory.

nginx 作为最流行的 web 容器之一，配置和使用相当简单，只要简单的配置就能拥有高性能和高可用。推荐使用 nginx 托管。示例配置如下：
```shell
server {
    listen 80;
    # gzip config
    gzip on;
    gzip_min_length 1k;
    gzip_comp_level 9;
    gzip_types text/plain application/javascript application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
    gzip_vary on;
    gzip_disable "MSIE [1-6]\.";

    root /usr/share/nginx/html;

    location / {
        # 用于配合 browserHistory使用
        try_files $uri $uri/ /index.html;

        # 如果有资源，建议使用 https + http2，配合按需加载可以获得更好的体验
        # rewrite ^/(.*)$ https://preview.pro.ant.design/$1 permanent;

    }
    location /api {
        proxy_pass https://preview.pro.ant.design;
        proxy_set_header   X-Forwarded-Proto $scheme;
        proxy_set_header   Host              $http_host;
        proxy_set_header   X-Real-IP         $remote_addr;
    }
}
server {
  # 如果有资源，建议使用 https + http2，配合按需加载可以获得更好的体验
  listen 443 ssl http2 default_server;

  # 证书的公私钥
  ssl_certificate /path/to/public.crt;
  ssl_certificate_key /path/to/private.key;

  location / {
        # 用于配合 browserHistory使用
        try_files $uri $uri/ /index.html;

  }
  location /api {
      proxy_pass https://preview.pro.ant.design;
      proxy_set_header   X-Forwarded-Proto $scheme;
      proxy_set_header   Host              $http_host;
      proxy_set_header   X-Real-IP         $remote_addr;
  }
}
```

## 反向代理
```shell
upstream backend {
    server backend1.example.com:8080 weight=3;
    server backend2.example.com:8080;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 跨域配置
在标准的跨域请求中，Access-Control-Allow-Origin 头部只能包含单个域名或*。它指示浏览器允许哪些源（域名）进行跨域请求。
所以根据规范，Access-Control-Allow-Origin 不支持返回多个域名。如果需要允许多个域名进行跨域请求，您需要在后端服务器根据请求的 Origin 值进行逻辑判断，然后动态设置正确的 Access-Control-Allow-Origin 头部。
例如，在后端服务器代码中，可以根据请求头中的 Origin 判断是哪个域名发起的请求，然后设置相应的 Access-Control-Allow-Origin 头部值。
请注意，当在响应头中设置 Access-Control-Allow-Origin 为 * 时，表示允许任何源进行跨域请求。这是一种宽松的设置，适用于公开的 API，但对于一些需要更严格访问控制的场景，建议根据具体需求设置具体的域名。
总结来说，Access-Control-Allow-Origin 头部是单值的，不支持返回多个域名，但您可以根据你的后端逻辑设置合适的值。
> 跨域配置有很多种写法，而且有其生效的作用域，具体视情况而定，这里列一些case：
```shell
# case 1
if ($request_method = OPTIONS) {
    return 200 '';
}

if ($http_origin ~* (xx\.cn|yy\.zh)$) {
    set $cors "true";
}

if ($request_method = 'OPTIONS') {
    set $cors "${cors}options";
}

if ($request_method = 'GET') {
    set $cors "${cors}get";
}

if ($request_method = 'POST') {
    set $cors "${cors}post";
}

if ($cors = 'true') {
    add_header 'Access-Control-Allow-Origin' "$http_origin";
}

if ($cors = "trueget") {
    add_header 'Access-Control-Allow-Origin' "$http_origin";
    add_header 'Access-Control-Allow-Credentials' 'true';
    add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, PATCH, OPTIONS';
    add_header 'Access-Control-Allow-Headers' 'Content-Type, X-System-USERID, X-System-RequestID';
}
# case 2
location / {
    # 允许跨域的请求，可以自定义变量$http_origin，*表示所有  
    add_header 'Access-Control-Allow-Origin' *;  
    # 允许携带cookie请求  
    add_header 'Access-Control-Allow-Credentials' 'true';  
    # 允许跨域请求的方法：GET,POST,OPTIONS,PUT  
    add_header 'Access-Control-Allow-Methods' 'GET,POST,OPTIONS,PUT';  
    # 允许请求时携带的头部信息，*表示所有  
    add_header 'Access-Control-Allow-Headers' *;  
    # 允许发送按段获取资源的请求  
    add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';  
    # 一定要有！！！否则Post请求无法进行跨域！
    # 在发送Post跨域请求前，会以Options方式发送预检请求，服务器接受时才会正式请求  
    if ($request_method = 'OPTIONS') {  
        add_header 'Access-Control-Max-Age' 1728000;  
        add_header 'Content-Type' 'text/plain; charset=utf-8';  
        add_header 'Content-Length' 0;  
        # 对于Options方式的请求返回204，表示接受跨域请求  
        return 204;  
    }  
}
```
如果因API返回了多层跨域配置，导致API无法调用通，需要将多个跨域配置，改为单个
当 API 的响应头中出现了多个 `Access-Control-Allow-Origin` 头部时，浏览器会视为无效的响应头，并拒绝处理跨域请求。为了解决这个问题，您可以采取下面的步骤：
确保后端 API 只返回一个 `Access-Control-Allow-Origin` 头部。在后端服务器的响应中，只设置一个允许跨域请求的源，而不是返回多个 `Access-Control-Allow-Origin` 头部。例如，将以下代码添加到后端服务器的响应中：
response.headers['Access-Control-Allow-Origin'] = 'https://example.com'
如果后端服务器无法修复此问题，您可以使用 `Nginx` 来移除多余的 `Access-Control-Allow-Origin` 头部。示例如下：
```
location /api {
    proxy_pass http://backend;
    proxy_hide_header Access-Control-Allow-Origin;
    add_header Access-Control-Allow-Origin $http_origin always;
}
```
在该配置中使用了 `proxy_hide_header` 指令来隐藏 `Access-Control-Allow-Origin` 头部，然后再使用 `add_header` 指令添加正确的 `Access-Control-Allow-Origin` 头部。
这样配置后，`Nginx` 会先隐藏原始的 `Access-Control-Allow-Origin` 头部，然后添加一个根据请求头中的 `Origin` 值动态生成的正确的 `Access-Control-Allow-Origin` 头部。
请注意，使用 `proxy_hide_header` 只是将该头部从响应头中隐藏，并不代表它不存在。因此，您仍然需要使用 `add_header` 指令添加正确的 `Access-Control-Allow-Origin` 头部。
用以上方法可以解决返回多个 `Access-Control-Allow-Origin` 头部的问题，并确保正确的跨域配置。

## 负载均衡
```shell
upstream backend {
    server backend1.example.com:8080;
    server backend2.example.com:8080;
    server backend3.example.com:8080;
}

server {
    listen 80;
    server_name example.com;
    location / {
    proxy_pass http://backend;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
	}
}
```

## 部署静态网站
```shell
server {
    listen       80;
    server_name  example.com;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    location /images/ {
        alias /var/www/images/;
    }

    location /downloads/ {
        alias /var/www/downloads/;
    }
}
```

## 定制403/404页面
```shell
# nginx http模块加入如下配置
fastcgi_intercept_errors on;
# 在server模块加入如下配置
location / {
    root   /data;
    index  index.html index.htm yunmai.html;
    error_page   403  /403.html;
    error_page   404  /404.html;
    location = /403.html {
        root   /usr/share/nginx/html;
    }
    location = /404.html {
        root   /usr/share/nginx/html;
    }
}
```

## WebSocket服务器
在使用Nginx作为WebSocket服务器时，Nginx会将客户端请求转发到后端的WebSocket服务器上，并实现WebSocket协议的连接管理。这种场景通常用于实时通信、游戏等应用程序。下面是一个示例Nginx配置：
```shell
map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```
上述配置中，Nginx会将WebSocket请求转发到http://backend上，并设置HTTP头信息中的Upgrade、Connection、Host和X-Real-IP字段，从而实现WebSocket协议的连接管理。

## 动静分离
动静分离是让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作，这就是网站静态化处理的核心思路。
```shell
upstream test{  
       server localhost:;  
       server localhost:;  
    }   

    server {  
        listen       ;  
        server_name  localhost;  

        location / {  
            root   e:\wwwroot;  
            index  index.html;  
        }  

        # 所有静态请求都由nginx处理，存放目录为html  
        location ~ \.(gif|jpg|jpeg|png|bmp|swf|css|js)$ {  
            root    e:\wwwroot;  
        }  

        # 所有动态请求都转发给tomcat处理  
        location ~ \.(jsp|do)$ {  
            proxy_pass  http://test;  
        }  

        error_page        /50x.html;  
        location = /50x.html {  
            root   e:\wwwroot;  
        }  
    }
```
这样我们就可以吧HTML以及图片和css以及js放到wwwroot目录下，而tomcat只负责处理jsp和请求，例如当我们后缀为gif的时候，Nginx默认会从wwwroot获取到当前请求的动态图文件返回，当然这里的静态文件跟Nginx是同一台服务器，我们也可以在另外一台服务器，然后通过反向代理和负载均衡配置过去就好了，只要搞清楚了最基本的流程，很多配置就很简单了，另外localtion后面其实是一个正则表达式，所以非常灵活

## 防盗链
> 盗链：比如我们线上的图片等静态资源，经常会被其他网站盗用，他们发大财的同时，成本确实我们在买单，这就很可恶。一些名不见经传的小网站来盗取一些有实力的大网站的地址（比如一些音乐、图片、软件的下载地址）然后放置在自己的网站中，通过这种方法盗取大网站的空间和流量。直接在自己的网站上向最终用户提供其它服务提供商的服务内容，骗取最终用户的浏览和点击率。受益者不提供资源或提供很少的资源，而真正的服务提供商却得不到任何的收益。
```shell
# 在项目中,经常会有不想让本站点的静态资源被他人盗取访问的需求,比如网站中的图片,前端加载的一些js文件等,此时就可以配置nginx的防盗链来实现网站资源的防盗。
server {
    location ~ .*\.(txt|xml)$ {
        # 配置防盗链规则
        valid_referers none blocked 192.168.1.110 *.example.com example.* ~\.google\.;
        # 如果不符合防盗链规则，则返回403
        if ($invalid_referer) {
            return 403;
        }
        root /vagrant/doc;
    }
}
```

## https强跳
```shell
set $rewrite_status 0;
if ($https_status = off) { set $rewrite_status "${rewrite_status}1"; }
if ($scheme = http) { set $rewrite_status "${rewrite_status}2"; }
if ($uri !~ '^/test') { set $rewrite_status "${rewrite_status}3"; }
if ($https = on) { set $https_status $https; }
if ($rewrite_status = 0123) {
   rewrite / https://$host$uri permanent;
   break;
}
```

## rewrite案例
```shell
# case 1
location ~ ^/backend/(.*) {
  proxy_set_header X-Forwarded-For $x_real_ip;
  proxy_set_header Host xxxx.test.com;
  proxy_http_version 1.1;
  proxy_set_header Connection "";
  rewrite /backend/(.*) /$1 break;
  proxy_pass http://web.loop;
}
# case 2
location /goods {  #判断商品的路径
  #{1,5} 表示1-5位的数字
  #商品为goods-121.html
  rewrite "goods-(\d{1,5})\.html" /goods-ctrl.html?id=$(1);
  #路径
  root yellowcong.com;
  #页面
  index index.html;
}
# case 3
location ~ ^/api/([0-9]+)(\.[0-9]+)*/client/ {
    rewrite /(.*)$ /$1 break;
    proxy_pass http://bbb.example.com;
    proxy_set_header Host $proxy_host;
}
```
将`/info/22/yellowcong/717350389@11.com` 转化为 `/info?age=12&name=yellowcon&email=717350389`
`[0-9]`表示 `0-9` 范围`i`数字 也可以使用`\d+`
`+` 表示1个或多个
`w+` 表示是字符串
`$` 表示结尾
`rewrite ^/info/([0-9]+)\/(\w+)\/(\w+)$ /info?age=$1&name=$2&email=$3 break;`

## rewrite+基于url参数的location
```shell
# case 1
location ~ ^/api/v4/projects/\d+/merge_requests/\d+/changes {
    proxy_hide_header X-Frame-Options;
    proxy_hide_header Content-Security-Policy;
    proxy_cookie_path / "/; httponly; SameSite=None; secure";
    proxy_cache off;
    set $is_matched 0;
    set $real_host $host;
    set $real_proxy "gitlab-workhorse";
    if ( $query_string ~* ^(.*)real_addr=true(.*)$ ){
      set $is_matched 1;
      set $real_proxy "xx-yy-dd.test.com";
      set $real_host "xx-yy-dd.test.com";
    }
    proxy_set_header Host $proxy_host;
    if ( $is_matched = 0 ) {
      rewrite ^/api/v4/projects/(.*)/changes$ /api/v2/projects/$1/changesV2 break;
      proxy_pass http://xx-yy-dd.test.com;
    }
    proxy_pass  http://gitlab-workhorse;
}
# case 2
upstream dynamic_cn {
  server localhost:8088   weight=5;
  server localhost:8087   weight=10;
}
upstream dynamic_jp {
  server localhost:8086   weight=10;
  server localhost:8085   weight=10;
}
server {
  listen 8089;
  location / {
    proxy_pass http://dynamic_cn;
  }
  location /hello {
    if ( $query_string ~* "world=cn" ) {
      proxy_pass http://dynamic_cn;
    }
    if ( $query_string ~* "world=jp" ) {
      proxy_pass http://dynamic_jp;
    }
    proxy_pass http://dynamic_cn;
  }
}
```
