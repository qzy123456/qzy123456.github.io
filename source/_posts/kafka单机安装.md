---
title: kafka单机安装
date: 2024-09-29 15:48:27
tags: mq
categories: mq
---
### 新版本支持kraft，并且在后面会彻底抛弃zookeeper
### 二进制包地址 https://downloads.apache.org/kafka/
![](1.png)
### 解压之后，编辑config/kraft/server.properties文件，改成自己的ip
### 在Kafka安装目录的bin文件夹下执行以下命令生成一个新的集群ID，如果只有1个机器也没关系,windows的命令在windows文件夹
```
kafka-storage.sh random-uuid
或者
kafka-storage.bat random-uuid
```
### 用上一步生成的UUID格式化Kafka的存储目录：
```
kafka-storage.bat format -t <uuid> -c ..\..\config\kraft\server.properties
或者
kafka-storage.sh  format -t <uuid> -c ../../config/kraft/server.properties
```
### 启动Kafka服务器,这里用的是Kraft的目录下的配置
```
kafka-server-start.bat ..\..\config\kraft\server.properties
或者
kafka-server-start.sh ../../config/kraft/server.properties
```
### docker安装单机版的kafka
### 文件夹下有.env和docker-compose.yml两个文件
#### .env文件内容如下：
```
# 把下面的 192.168.252.1 改为你的ip地址
ACCESS_ADDR=192.168.252.1:9092
```
#### docker-compose.yml内容如下
```
version: '3.8'

services:
  broker:
    image: apache/kafka:3.7.0
    container_name: broker
    ports:
      - '9092:9092'
    environment:
      kafka_NODE_ID: 1
      kafka_LISTENER_SECURITY_PROTOCOL_MAP: 'CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT'
      kafka_ADVERTISED_LISTENERS: 'PLAINTEXT_HOST://${ACCESS_ADDR},PLAINTEXT://broker:19092'
      kafka_PROCESS_ROLES: 'broker,controller'
      kafka_CONTROLLER_QUORUM_VOTERS: '1@broker:29093'
      kafka_LISTENERS: 'CONTROLLER://:29093,PLAINTEXT_HOST://:9092,PLAINTEXT://:19092'
      kafka_INTER_BROKER_LISTENER_NAME: 'PLAINTEXT'
      kafka_CONTROLLER_LISTENER_NAMES: 'CONTROLLER'
      CLUSTER_ID: '4L6g3nShT-eMCtK--X86sw'
      kafka_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      kafka_GROUP_INITIAL_REBALANCE_DELAY_MS: 0
      kafka_TRANSACTION_STATE_LOG_MIN_ISR: 1
      kafka_TRANSACTION_STATE_LOG_REPLICATION_FACTOR: 1
      kafka_LOG_DIRS: '/var/lib/kafka/data'
    volumes:
      - $PWD/data/:/var/lib/kafka/data

  kafka-ui:
    image: provectuslabs/kafka-ui:v0.7.2
    container_name: kafka-ui
    ports:
      - "18080:8080"
    environment:
      kafka_CLUSTERS_0_NAME: 'Local kafka Cluster'
      kafka_CLUSTERS_0_BOOTSTRAPSERVERS: 'broker:19092'
      DYNAMIC_CONFIG_ENABLED: "true"
    depends_on:
      - broker
```
### 安装 kafka 集群

#### .env文件内容如下：
```
# 把下面的 192.168.251.1 改为你的ip地址
kafka_1_ACCESS_ADDR=192.168.251.1:33001
kafka_2_ACCESS_ADDR=192.168.251.1:33002
kafka_3_ACCESS_ADDR=192.168.251.1:33003
```
#### docker-compose.yml内容如下：
```
version: "3.8"

services:
  kafka-1:
    image: docker.io/bitnami/kafka:3.7
    container_name: kafka-1
    ports:
      - "33001:9092"
    environment:
      # KRaft settings
      - kafka_CFG_NODE_ID=0
      - kafka_CFG_PROCESS_ROLES=controller,broker
      - kafka_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka-1:9093,1@kafka-2:9093,2@kafka-3:9093
      - kafka_KRAFT_CLUSTER_ID=abcdefghijklmnopqrstuv
      # Listeners
      - kafka_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093
      #- kafka_CFG_ADVERTISED_LISTENERS=PLAINTEXT://:9092
      - kafka_CFG_ADVERTISED_LISTENERS=PLAINTEXT://${kafka_1_ACCESS_ADDR}
      - kafka_CFG_LISTENER_SECURITY_PROTOCOL_MAP=PLAINTEXT:PLAINTEXT,CONTROLLER:PLAINTEXT
      - kafka_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - kafka_CFG_INTER_BROKER_LISTENER_NAME=PLAINTEXT
      # Clustering
      - kafka_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR=3
      - kafka_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=3
      - kafka_CFG_TRANSACTION_STATE_LOG_MIN_ISR=2
    volumes:
      - $PWD/data/kafka-1:/bitnami/kafka
    networks:
      - kafka-net

  kafka-2:
    image: docker.io/bitnami/kafka:3.7
    container_name: kafka-2
    ports:
      - "33002:9092"
    environment:
      # KRaft settings
      - kafka_CFG_NODE_ID=1
      - kafka_CFG_PROCESS_ROLES=controller,broker
      - kafka_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka-1:9093,1@kafka-2:9093,2@kafka-3:9093
      - kafka_KRAFT_CLUSTER_ID=abcdefghijklmnopqrstuv
      # Listeners
      - kafka_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093
      #- kafka_CFG_ADVERTISED_LISTENERS=PLAINTEXT://:9092
      - kafka_CFG_ADVERTISED_LISTENERS=PLAINTEXT://${kafka_2_ACCESS_ADDR}
      - kafka_CFG_LISTENER_SECURITY_PROTOCOL_MAP=PLAINTEXT:PLAINTEXT,CONTROLLER:PLAINTEXT
      - kafka_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - kafka_CFG_INTER_BROKER_LISTENER_NAME=PLAINTEXT
      # Clustering
      - kafka_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR=3
      - kafka_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=3
      - kafka_CFG_TRANSACTION_STATE_LOG_MIN_ISR=2
    volumes:
      - $PWD/data/kafka-2:/bitnami/kafka
    networks:
      - kafka-net

  kafka-3:
    image: docker.io/bitnami/kafka:3.7
    container_name: kafka-3
    ports:
      - "33003:9092"
    environment:
      # KRaft settings
      - kafka_CFG_NODE_ID=2
      - kafka_CFG_PROCESS_ROLES=controller,broker
      - kafka_CFG_CONTROLLER_QUORUM_VOTERS=0@kafka-1:9093,1@kafka-2:9093,2@kafka-3:9093
      - kafka_KRAFT_CLUSTER_ID=abcdefghijklmnopqrstuv
      # Listeners
      - kafka_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093
      #- kafka_CFG_ADVERTISED_LISTENERS=PLAINTEXT://:9092
      - kafka_CFG_ADVERTISED_LISTENERS=PLAINTEXT://${kafka_3_ACCESS_ADDR}
      - kafka_CFG_LISTENER_SECURITY_PROTOCOL_MAP=PLAINTEXT:PLAINTEXT,CONTROLLER:PLAINTEXT
      - kafka_CFG_CONTROLLER_LISTENER_NAMES=CONTROLLER
      - kafka_CFG_INTER_BROKER_LISTENER_NAME=PLAINTEXT
      # Clustering
      - kafka_CFG_OFFSETS_TOPIC_REPLICATION_FACTOR=3
      - kafka_CFG_TRANSACTION_STATE_LOG_REPLICATION_FACTOR=3
      - kafka_CFG_TRANSACTION_STATE_LOG_MIN_ISR=2
    volumes:
      - $PWD/data/kafka-3:/bitnami/kafka
    networks:
      - kafka-net

  kafka-ui:
    image: provectuslabs/kafka-ui:v0.7.2
    restart: always
    container_name: kafka-ui
    ports:
      - "18080:8080"
    environment:
      - kafka_CLUSTERS_0_NAME=Local-Kraft-Cluster
      - kafka_CLUSTERS_0_BOOTSTRAPSERVERS=kafka-1:9092,kafka-2:9092,kafka-3:9092
      - DYNAMIC_CONFIG_ENABLED=true
      - kafka_CLUSTERS_0_AUDIT_TOPICAUDITENABLED=true
      - kafka_CLUSTERS_0_AUDIT_CONSOLEAUDITENABLED=true
    depends_on:
      - kafka-1
      - kafka-2
      - kafka-3
    networks:
      - kafka-net

networks:
  kafka-net:
```
#### 第一次使用时，创建一个data文件夹作为数据持久化，并且修改目录data权限：
```
mkdir data/kafka-1 data/kafka-2 data/kafka-3
chmod -R 0777 data
```
#### docker-compose up -d ,启动服务成功后，可以在浏览器打开 http://localhost:18080 查看kafka信息。
