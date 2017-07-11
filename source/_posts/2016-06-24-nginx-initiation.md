---
layout:     post
title:      "如何安装与搭建一个Nginx服务器"
subtitle:   "Nginx initiation"
date:       2016-06-24 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - 负载均衡
    - Nginx
tags:
    - Nginx
---



![](http://ww2.sinaimg.cn/mw690/63503acbjw1f67nyh9gq7j20cp075wel.jpg)

### Nginx介绍

`Nginx`是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，并在一个BSD-like 协议下发行。由俄罗斯的程序设计师Igor Sysoev所开发，供俄国大型的入口网站及搜索引擎Rambler使用。

### Nginx应用场景

 1. 作为http服务器使用并独立提供http服务。
 2. 虚拟主机。
 3. 用于反向代理与负载均衡,当并发量非常大时,可以使用`Nginx`对服务器集群进行反向代理与负载均衡,提高吞吐量。

### 反向代理

![](http://ww1.sinaimg.cn/mw690/63503acbjw1f67nz4ekjmj20tq0eqgmx.jpg)

反向代理是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

### Nginx安装流程

 1. Nginx依赖于以下4个库

    - gcc
        安装nginx需要先将官网下载的源码进行编译，编译依赖gcc环境
          **yum install gcc-c++**
        
    - pcre
        PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 perl 兼容的正则表达式库。nginx的http模块使用pcre来解析正则表达式，所以需要在linux上安装pcre库。
         **yum install -y pcre pcre-devel**
         
    - zlib
        zlib库提供了很多种压缩和解压缩的方式，nginx使用zlib对http包的内容进行gzip，所以需要在linux上安装zlib库。
         **yum install -y zlib zlib-devel**
         
    - openssl
        OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及SSL协议，并提供丰富的应用程序供测试或其它目的使用。
	nginx不仅支持http协议，还支持https（即在ssl协议上传输http），所以需要在linux安装openssl库。
        **yum install -y openssl openssl-devel**
        
    - 一键安装依赖包 **yum -y install zlib zlib-devel openssl openssl--devel pcre pcre-devel**     
    
 2. 解压Nginx源码包  

 3. 使用configure创建makefile

```
./configure \
--prefix=/usr/local/nginx \
--pid-path=/var/run/nginx/nginx.pid \
--lock-path=/var/lock/nginx.lock \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--with-http_gzip_static_module \
--http-client-body-temp-path=/var/temp/nginx/client \
--http-proxy-temp-path=/var/temp/nginx/proxy \
--http-fastcgi-temp-path=/var/temp/nginx/fastcgi \
--http-uwsgi-temp-path=/var/temp/nginx/uwsgi \
--http-scgi-temp-path=/var/temp/nginx/scgi
```

 4. make & make install

 5. cd /usr/local/nginx/sbin 目录 
    
    - **./nginx** 开启`Nginx`

    - **./nginx -s stop** 关闭`Nginx`
    
    - **./nginx -s reload** 重新加载`Nginx`配置文件
    
### Nginx配置文件

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f67o1jecrjj20ic0jwdj6.jpg)

nginx.conf是`Nginx`的主要配置文件,它的主要结构如下

 - worker_process表示工作进程的数量，一般设置为cpu的核数

 - worker_connections表示每个工作进程的最大连接数

 - server{} 定义了一个虚拟机,如果要添加一个虚拟机,则添加一个server{}即可。

    - listen监听的端口
    
    - server_name监听的域名
    
    - location{}配置匹配的URI
    
        - root 指定URI查找的资源路径,为相对路径。
        
        - index 指定首页的名称,可以配置多个。
        
        - proxy_pass 指定反向代理转发的路径。
        
### Nginx配置负载均衡        

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f67o1j87tcj20cf08pq44.jpg)

对upstream test{}中的2个web服务器配置了负载均衡,weight设置权值,权值越高的服务器承载的压力就越大。


