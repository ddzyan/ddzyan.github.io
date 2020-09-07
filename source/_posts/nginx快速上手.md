---
title: nginx快速上手
date: 2018-09-03 01:31:19
categories: 
- nginx
tags:
- linux
- nginx
---
### 简介
Nginx是一款使用C语言开发的高性能的http 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，官方测试nginx能够支支撑5万并发链接，并且cpu、内存等资源消耗却非常低，运行非常稳定。

### 应用场景
1. 网页静态服务器。
2. 反向代理，负载均衡。

<!--more-->

#### 安装
```bash
//安装Nginx
yum install -y nginx

//启动
service nginx start

//设置开机自启
service enable nginx.service

//重新加载
service nginx reload

//查看状态
service nginx status

//重启
service nginx restart

//卸载
yum remove nginx
```
默认安装最新版本，如果安装其他版本可能造成配置文件失效。

##### Nginx全局配置
/etc/nginx/nginx.conf

##### 自定义Nginx站点配置文件存放目录
/etc/nginx/conf.d/

### nginx 设置静态目录
step1 修改 nginx.conf 文件
```json
server {
        listen       80;
        #listen       [::]:80;
        server_name  zmscode.cn;
        root   /data/blog/public/;
        index  index.html index.htm;
       # location /{ 
           #root   /home/vsftpd/blog/public/;
           #index  index.html index.htm;
       # }
        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
```
- listen：监听端口
- server_name：配置域名
- root：文件夹路径
- index默认文件

修改配置文件后，需要重启 nginx 服务

---
### 问题汇总

Q：问题
```
nginx 403 forbidden
```
A:解决办法
1. 网站禁止特定的用户访问所有内容，例：网站屏蔽某个ip访问。
2. 访问禁止目录浏览的目录，例：设置autoindex off后访问目录。
3. 用户访问只能被内网访问的文件。
4. 不存在的index文件导致
