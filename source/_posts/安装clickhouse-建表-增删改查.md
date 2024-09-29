---
title: 安装clickhouse+建表+增删改查
date: 2024-09-29 10:36:36
tags: es
categories: es
---

### 安装
#### docker
```
docker run --name clickhouse-server --ulimit nofile=262144:262144 --volume=$HOME/clickhouse:/var/lib/clickhouse -p 8123:8123 -p 9000:9000 -p 9004:9004 clickhouse/clickhouse-server
```
### apt
```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754
```
#### 向 /etc/apt/sources.list.d/ 目录添加 ClickHouse 存储库：
```
echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee /etc/apt/sources.list.d/clickhouse.list
```
#### 更新 apt 包索引：
```
sudo apt-get update
```
#### 安装 ClickHouse 服务器和客户端：
```
sudo apt-get install -y clickhouse-server clickhouse-client
```
#### 启动 ClickHouse 服务：
```
sudo service clickhouse-server start
```
#### 连接到 ClickHouse 服务器：
```
clickhouse-client
```
### apt安装的中间会让你输入密码。下面是如何修改密码
```
echo -n 密码 | openssl dgst -sha256
```
### 修改对应的xml文件，apt的在/etc/clickhouse-server/users.d/。docker的在刚才的$HOME/clickhouse
```
<clickhouse>
    <users>
        <default>
            <password remove='1' />
            #### 修改这个
            <password_sha256_hex>86250f09bcf44ec7efb5864e12201a8e63f04e9d377f6a66470e2604325523c7</password_sha256_hex>
        </default>
    </users>
</clickhouse>
```
### 使用默认账号，登陆。PS：docker需要进入到容器内 docker exec -it 
```
clickhouse-client 
#输入密码
```
### 剩下的基本跟mysql一样了
### 建库
```
CREATE DATABASE IF NOT EXISTS my_database;
```
### 显示所有库 show database；
![](3302358-20240919095852116-1495472751.png)
### 使用某个库 use my_database；
### 建表略有不同，引擎常用的是MergeTree,ORDER BY是定义主键
### 注意，不支持主键自增，需要自己维护主键，或者使用uuid。例如 `id` UUID DEFAULT generateUUIDv4(),
#### partition by (id) 分区，可选
#### PRIMARY KEY (id) 主键，可选
#### ORDER BY 必选
```
CREATE TABLE my_database.my_table
(
    `id` UInt32,
    `name` String,
    `timestamp` DateTime,
    `value` Float32
)
ENGINE = MergeTree()
partition by (id)
PRIMARY KEY (id)
ORDER BY id;
```
### 插入数据，插入多行数据
```
INSERT INTO my_database.my_table (id, name, timestamp, value) VALUES (1, 'John Doe', '2021-01-01 00:00:00', 1.0);
INSERT INTO my_database.my_table (id, name, timestamp, value) VALUES (2, 'Jane Doe', '2021-01-02 00:00:00', 2.0), (3, 'Mike Smith', '2021-01-03 00:00:00', 3.0);
```
### 查询数据,查询特定列,条件查询,范围查询,排序查询结果,限制查询结果数量
```
SELECT * FROM my_database.my_table;
SELECT name, value FROM my_database.my_table;
SELECT * FROM my_database.my_table WHERE id = 1;
SELECT * FROM my_database.my_table WHERE timestamp >= '2021-01-01 00:00:00' AND timestamp <= '2021-01-31 23:59:59';
SELECT * FROM my_database.my_table ORDER BY timestamp DESC;
SELECT * FROM my_database.my_table LIMIT 10;
```
### 更新数据,ClickHouse 支持 ALTER TABLE 来修改表结构，例如添加或删除列，但不支持传统意义上的 UPDATE 语句来修改现有数据。需要使用 ALTER TABLE。
```
ALTER TABLE my_database.my_table UPDATE name = 'new_value' WHERE id = 2;
```
### 删除数据
```
DELETE FROM my_database.my_table WHERE id = 1
或
ALTER TABLE my_database.my_table DELETE WHERE id = 1;
```
### 使用python操作
```
from clickhouse_driver import Client
 
# 连接到ClickHouse服务器
client = Client(host='127.0.0.1', user='default', password='123456')
 
# 创建数据库
#client.execute('CREATE DATABASE IF NOT EXISTS test_db')
 
# 使用特定数据库
client.execute('USE my_database')
 
# 创建表
#client.execute("CREATE TABLE my_table(`id` UInt32,`name` String,`timestamp` DateTime,`value` Float32)ENGINE = MergeTree() ORDER BY id;")
 
# 插入数据
#client.execute("INSERT INTO my_database.my_table (id, name, timestamp, value) VALUES (2, 'Jane Doe', '2021-01-02 00:00:00', 2.0), (3, 'Mike Smith', '2021-01-03 00:00:00', 3.0);")
 
# 查询数据
result = client.execute('SELECT * FROM my_table')
for row in result:
    print(row)
 
# 关闭客户端连接
client.disconnect()
```