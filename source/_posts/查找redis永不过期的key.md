---
title: 查找redis永不过期的key
date: 2024-09-29 15:59:18
tags: Redis
categories: Redis
---
### keys *阻塞进程，消耗比较大，慎用
```
#!/bin/bash

# 设置要遍历的 Redis 数据库数量
db_count=16

# 输出文件名
output_file="never_expire_keys.txt"

# 循环遍历 0 到 15 的数据库
for (( db=0; db<$db_count; db++ )); do
    echo "正在检查数据库 $db 的永不过期键..."
    # 连接到当前数据库
    redis-cli select $db
    
    # 查询当前数据库中的所有键，并检查它们的 TTL
    redis-cli keys "*" | while read -r key; do
        ttl=$(redis-cli ttl "$key")
        
        # 如果 TTL 等于 -1，表示键永不过期
        if [ "$ttl" -eq -1 ]; then
            # 将永不过期的键和它所属的数据库编号写入文件
            echo "$db:$key" >> "$output_file"
        fi
    done
done

echo "永不过期的键已写入到 $output_file 文件中"
```
