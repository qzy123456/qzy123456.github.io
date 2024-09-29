---
title: nginx代理mysql
date: 2024-09-29 15:50:24
tags: nginx
categories: nginx
---
### 之前服务器单体架构mysql是直接安装在服务器的，没有买托管，这就造成一个问题，如果想要远程连接mysql就要开启3306防火墙端口，全是恶意ip进行攻击。。。。
### nginx的stream模块可以有效限制远程ip访问
```
stream {
    server {
       listen 13306; # 需要开启云服务器防火墙
       #allow 123.149.112.119; # 允许这个ip访问
       # 允许192.168.110.1到192.168.255.254 虚拟机适用
       #allow 192.168.110.0/16;
       # deny all; # 除了allow的ip都禁止
      # 禁止192.168.110.1访问
       deny 192.168.110.1;
       # 禁止192.168.110.1到192.168.255.254
       deny 192.168.110.0/16; 
       # allow all; 允许所有 
       proxy_connect_timeout 1s;
       proxy_timeout 3s;
       proxy_pass 127.0.0.1:3306;
    }
}
```
