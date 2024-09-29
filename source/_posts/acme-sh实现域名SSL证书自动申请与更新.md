---
title: acme.sh实现域名SSL证书自动申请与更新
date: 2024-09-29 15:54:58
tags: nginx
categories: nginx
---
### 域名注册与解析位于阿里云

### 安装acme.sh
```
curl https://get.acme.sh | sh
或者
wget -O -  https://get.acme.sh | sh
```
### 这个自动安装过程完成了以下几个步骤（上面那一步默认已经执行了这个操作，如果成功了，这步省略）
```
拷贝sh脚本到~/.acme.sh/
创建alias别名acme.sh=~/.acme.sh/acme.sh
启动定时器crontab
```
### 配置阿里云解析
#### 运行如下命令，配置阿里云api接口的key和secret，其中的值需要到阿里云控制台中去寻找。
```
export Ali_Key="xxxxxxxxxxxx"
export Ali_Secret="yyyyyyyyyyyyyy"
```

### 这两个配置将永久保存在文件~/.acme.sh/account.conf中

#### 为域名申请证书
#### 运行如下命令，一键申请证书。
#### 这一步需要在dns后台加一个泛域名的解析，也就是*
```
acme.sh --issue --dns dns_ali -d *.example.com
```
### 证书申请成功后，保存在~/.acme.sh/*.example.com目录下

### 将证书部署到nginx
### 运行如下命令，自动将证书部署到nginx。
### 比如我们要部署www域名，这个www也要在dns后台先解析，acmh只是申请一个通用证书
```
acme.sh --install-cert -d www.example.com --key-file /etc/nginx/cert/www.example.com.key --fullchain-file /etc/nginx/cert/www.example.com.pem 
```
### 该命令中的参数将自动保存在~/.acme.sh/www.example.com目录下的www.example.com.conf文件里，定时器更新证书的时候实现自动部署。
### 配置nginx
```
server {
        listen 80;
        listen 443;
        server_name www.example.com;

        ssl on;
        ssl_certificate  /etc/nginx/cert/www.example.com.pem;
        ssl_certificate_key /etc/nginx/cert/www.example.com.key;
        ssl_session_timeout 5m;
        ssl_session_cache shared:SSL:20m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers  HIGH:!aNULL:!MD5;
        ssl_prefer_server_ciphers   on;
        location / {
            proxy_pass http://localhost:3000;
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        }
}
```
