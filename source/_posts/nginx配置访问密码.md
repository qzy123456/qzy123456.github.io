---
title: nginx配置访问密码
date: 2024-09-29 15:52:07
tags: nginx
categories: nginx
---
### 安装htpassed工具
```
yum -y install httpd-tools
或者
apt install apache2-utils
```
### 创建用户名和密码
```
htpasswd -c /etc/nginx/.htpasswd username
```
### 修改nginx配置文件
```
server {
   listen ;
   server_name  localhost;
   .......
   #新增下面两行
   auth_basic "Please input password"; #这里是验证时的提示信息
   auth_basic_user_file  /etc/nginx/.htpasswd;
   location /{
   .......
   }
}
```
