---
title: Redis 的混合持久化
date: 2024-09-29 15:56:33
tags: Redis 
categories: Redis
---
```
aof-use-rdb-preamble 选项设置为 yes，并且要同时启用 RDB 和 AOF 两种持久化
```
### 启用 AOF 模式
```
将 appendonly 设置为 yes。默认是 no。
always: 每次写操作后都同步。
everysec: 每秒同步一次。
no: 由操作系统决定何时同步。
默认设置是 everysec。
当 Redis 进行 AOF 重写或快照保存时，避免主进程 fsync 的延迟？
设置 no-appendfsync-on-rewrite 为 yes。默认是 no
```
### 启用RDB
```
save 3600 1        # 3600秒内如果超过1个key被修改则生成 RDB
save 300 100       # 300秒内如果超过100个key被修改则生成 RDB
save 60 10000      # 60秒内如果超过10000个key被修改则生成 RDB
```
### 混合持久化的优缺点
#### 优点：
```
更快的启动速度：混合持久化结合了RDB的速度优势，所以Redis可以更快地重新启动，不用等待很久。
数据安全：利用AOF的方式，即使服务器突然断电，也只会丢失极短的时间内的数据。
文件更小巧：因为混合持久化结合了 RDB 和 AOF 的优势，所以文件大小和冗余度都可以得到控制。
两全其美：简单说，它就是RDB和AOF的结合体，带来了两者的好处。
```
#### 缺点：
```
稍微复杂：因为它结合了两种技术，所以处理起来比单一的 RDB 或 AOF 要复杂一点。
可能占更多空间：在某些情况下，保存数据的文件可能会比只使用 RDB 或AOF 的文件要大一些。
写入速度：可能会稍慢一些，特别是当数据需要经常被保存到硬盘时（比如当 appendfsync 配置为“always”时）
```
