---
title: Redis内存淘汰策略
date: 2024-09-29 16:02:51
tags: Redis
categories: Redis
---
```
内存淘汰策略分类
早期版本的 Redis 有以下 6 种淘汰策略：
noeviction：不淘汰任何数据，当内存不足时，新增操作会报错，Redis 默认内存淘汰策略；
allkeys-lru：淘汰整个键值中最久未使用的键值；
allkeys-random：随机淘汰任意键值;
volatile-lru：淘汰所有设置了过期时间的键值中最久未使用的键值；
volatile-random：随机淘汰设置了过期时间的任意键值；
volatile-ttl：优先淘汰更早过期的键值。
在 Redis 4.0 版本中又新增了 2 种淘汰策略：
volatile-lfu：淘汰所有设置了过期时间的键值中，最少使用的键值；
allkeys-lfu：淘汰整个键值中最少使用的键值。
```
