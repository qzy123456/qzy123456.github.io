---
title: nginx文件服务器根据文件类型判断预览还是下载
date: 2024-09-29 15:53:32
tags: nginx
categories: nginx
---
```
 location /file {
    charset utf-8;
    alias /usr/share/nginx/html/files; #文件根目录
    autoindex off;
    autoindex_exact_size off;
    autoindex_localtime on;
    # 按理说只用配置这一个，但是下面不生效，只能复制几份
    add_header 'Access-Control-Allow-Origin' '*' always;
    add_header 'Access-Control-Allow-Methods' 'GET, OPTIONS' always;
    add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept' always;
    # 配置如果是json文件就为下载模式
    if ($request_filename ~* ^.*?\.(json)$) {
        add_header Content-Disposition attachment;  # 添加响应头，配置文件作为附件下载
        add_header Content-Type application/octet-stream;
         add_header 'Access-Control-Allow-Origin' '*' always;
        add_header 'Access-Control-Allow-Methods' 'GET, OPTIONS' always;
        add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept' always;
    }
    #默认为预览，这个都可以不配
    location ~* \.(jpg|png)$ {
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'GET, OPTIONS' always;
        add_header 'Access-Control-Allow-Headers' 'Origin, X-Requested-With, Content-Type, Accept' always;
    }
}

```
