---
title: docker安装es+kibana
date: 2024-09-29 10:41:05
tags: es
categories: es
---
###  es，可以选择自己想要的版本
```
docker run --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -d elasticsearch:7.16.2
```
### kibana
```
docker run --name kibana -e ELASTICSEARCH_HOSTS=http://192.168.252.128:9200 -p 5601:5601 -d kibana:7.16.2
或者
docker run -d --name kibana -p 5601:5601 \
-e ELASTICSEARCH_HOSTS=http://192.168.252.128:9200 \ #Es Url
-e ELASTICSEARCH_USERNAME=root \  #Es 账号，Es不开启认证可不设置
-e ELASTICSEARCH_PASSWORD=123456  \ #Es 密码 ，Es不开启认证可不设置
-e I18N_LOCALE=zh-CN \ #汉化
kibana:7.16.2
```
### 修改kibana的配置
```
docker exec -it kibana  /bin/sh
```
### 修改kibana.yml，然后保存退出：:wq
```
server.name: kibana
server.host: "0"
#注意这里的ip不能是localhost，是本地的ip地址
elasticsearch.hosts: [ "http://192.168.252.128:9200" ]  
xpack.monitoring.ui.container.elasticsearch.enabled: true
#设置kibana中文显示
i18n.locale: zh-CN
```
### 访问 http://192.168.1.1:5601 和 http://192.168.1.1:9200
