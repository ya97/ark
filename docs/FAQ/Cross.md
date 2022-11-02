# 跨域问题

## 1.什么是跨域(CORS)

CORS，常被大家称之为跨越问题，准确的叫法是跨域资源共享（CORS，Cross-origin resource sharing）， 是W3C标准，是一种机制，它使用额外的HTTP头来告诉浏览器 让运行在一个 origin (domain)
上的Web应用被 准许访问来自不同源服务器上的指定的资源。当一个资源从与该资源本身所在的服务器不同的域或端口请求一个资源时， 资源会发起一个跨域 HTTP 请求

## 2.出现的原因(同源策略)

同源的定义是：如果两个页面的_协议_，_端口_（如果有指定）和_域名_都相同，则两个页面具有相同的源。 同源策略限制了从同一个源加载的文档或脚本如何与来自另一个源的资源进行交互。这是一个用于隔离潜在恶意文件的重要安全机制。

## 3.场景
| URL | 说明 | 是否允许通信 |
| --- | --- | --- |
| [http://www.domain.com/a.js](http://www.domain.com/a.js)
[http://www.domain.com/b.js](http://www.domain.com/b.js)
[http://www.domain.com/lab/c.js](http://www.domain.com/lab/c.js) | 同一域名，不同文件或路径 | √ |
| [http://www.domain.com/a.js](http://www.domain.com/a.js)
[https://www.domain.com/b.js](https://www.domain.com/b.js) | 同一域名，不同协议 | × |
| [http://www.domain.com/a.js](http://www.domain.com/a.js)
[http://192.168.4.12/b.js](http://192.168.4.12/b.js) | 域名和域名对应相同ip | × |
| [http://www.domain.com/a.js](http://www.domain.com/a.js)
[http://x.domain.com/b.js](http://x.domain.com/b.js)
[http://domain.com/c.js](http://domain.com/c.js) | 主域相同，子域不同 | × |
| [http://www.domain1.com/a.js](http://www.domain1.com/a.js)
[http://www.domain2.com/b.js](http://www.domain2.com/b.js) | 不同域名 | × |


## 4.常用解决方案

1、 nginx代理跨域
2、 跨域资源共享(CORS)
3、 通过jsonp跨域(只支持get请求且比较麻烦)
4、 nodejs中间件代理跨域(Axios)
5、 ..

## 5.Nginx实现

- [Nginx下载](http://nginx.org/en/download.html)

 命令相关： 	
验证配置是否正确: **nginx -t**
查看Nginx的详细信息：**nginx -V**
启动Nginx：**start nginx**
快速停止或关闭Nginx：**nginx -s stop**
正常停止或关闭Nginx：**nginx -s quit**
配置文件修改重装载命令：**nginx -s reload**

1. 修改配置文件(/conf/nginx.conf)  --------->修改后记得reload才能生效喔

-  Http- 
```xml
server {
    listen       80;                    --监听端口
    server_name  somename  alias  another.alias; --监听地址

    location / {                          --请求的url过滤，正则匹配，~为区分大小写，~*为不区分大小写。
        add_header Access-Control-Allow-Origin *;        --设置允许全局跨域
        add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';    --设置允许请求方式
        add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';  --设置允许全局跨域
        proxy_pass http://localhost:3000; --请求转向mysvr 定义的服务器列表
        deny 127.0.0.1;                   --拒绝的ip(可省略)
        allow 172.18.5.54;                --允许的ip(可省略)
    }
}
```
 

-  Https- 
```xml
server {
    listen       443 ssl;                                         --监听端口
    server_name  localhost;                                       --监听地址

    ssl_certificate      D://nginx-1.21.6//ssl//shidian.crt;      --ssl证书地址
    ssl_certificate_key  D://nginx-1.21.6//ssl//shidian.key;      --ssl key地址

    location ^~/page/ {                            --^~/page/表示匹配前缀是page的请求，proxy_pass的结尾有/， 则会把/page/*后面的路径直接拼接到后面，即移除page
        add_header Access-Control-Allow-Origin *;           --设置允许全局跨域
        add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';  --设置允许请求方式
        add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';  --设置允许全局跨域
        root   html;
        index  index.html index.htm;
        proxy_pass   http://localhost:8080/;
    }
    location / {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
        add_header Access-Control-Allow-Headers 'DNT,X-Mx-ReqToken,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Authorization';
        root   html;
        index  index.html index.htm;
        proxy_pass   http://localhost:8080/;
    }
}
```
 
