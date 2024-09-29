---
title: kafka生产者、消费者-命令行模式模拟
date: 2024-09-29 15:47:05
tags: mq
categories: mq
---
### win环境下，如果是linux，切换目录，用sh脚本就行
### kafka安装在上一篇
### Kraft启动kafka
```
kafka-server-start.bat ..\..\config\kraft\server.properties
```
### 生产者，启动之后，命令行输入要生产的消息
```
kafka-console-producer.bat --topic test-topic --bootstrap-server 192.168.252.1:9092
```
### kafka的消费者是分组的，也就是相同的group认为是一组，同一组下，多个消费者，只能有一个消费者能消费到消息。
### 想要做成发布订阅模式，就做成group名字不一样
### 消费者1
```
kafka-console-consumer.bat --topic test-topic --bootstrap-server 192.168.252.1:9092 --group group1
```
### 消费者2
```
kafka-console-consumer.bat --topic test-topic --bootstrap-server 192.168.252.1:9092 --group group2
```
