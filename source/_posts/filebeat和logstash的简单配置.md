---
title: filebeat和logstash的简单配置
date: 2024-09-29 10:49:18
tags: es
categories: es
---

### &#x20;filebeat基本配置

```javascript
# 输入
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - Z:\webman\runtime\logs\ad-*.log
  #json.keys_under_root: true
  #json.overwrite_keys: true
  #json.add_error_key: true  
  fields_under_root: true
  #这种配置既可以解析json，又支持原日志
  processors:
    - decode_json_fields:
        fields: ['message']
        target: "" 
        overwrite_keys: false 
        process_array: false 
        max_depth: 1
    #过滤系统自带的字段,有几个内置的8.9删不掉，低版本貌似可以.或者用logstash删除
    - drop_fields:
        fields: ["log","host","input","agent","ecs","host"]
    #把日志的时间当成es的时间
    - date:
        field: log.time # 日志里的时间字段，
        formats:
          - "yyyy-MM-dd HH:mm:ss" #时间字段的格式
        timezone: Asia/Shanghai
        target_field: "@timestamp" #要替换的字段，或者可以新增

# 指定索引的分区数
setup.template.settings:
  index.number_of_shards: 3
# 输出到指定ES的配置
output.elasticsearch:
  hosts: ["localhost:9200"]
  index: "emps-ad-log-%{+yyyy.MM.dd}"
setup.template.name: "emps-ad-log"
setup.template.pattern: "emps-ad-log-*"  
```

### logstash的简单配置

```
input {
 file {
    path => [ "/home/ubuntu/php/webman/runtime/logs/ad-*.log" ]
   #从头开始读，sincedb_path 文件存在时，会从记录的开始读
   start_position => "beginning"
   #不记录读取文件的位置，测试时候用
   sincedb_path => "/dev/null"
  }
}

filter {
  json {
    source => "message"
  }
 
  mutate {
    remove_field => ["event",'file','log', "path", "host", "@version"]
  }
  #把日志时间作为es的时间
  date {
     match =>["time","yyyy-MM-dd HH:mm:ss"] #把日志的time解析成对应的格式
	 timezone => "Asia/Shanghai"
   }
}


output {
  #stdout {
  #  codec => rubydebug
  #}
  elasticsearch {
    hosts => ["localhost:9200"]  # Elasticsearch 服务器的地址和端口
    index => "your_custom_index"  # 自定义索引名称，替换为你想要的索引名
 	template_name => "your_custom_template"
    #template_overwrite => true
    template_pattern => "your_pattern*"  # 指定匹配索引名称的模式
  }
}

```
![](3302358-20231108173702655-1012356458.png)
