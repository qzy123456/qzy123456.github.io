---
title: Redis事务
date: 2024-09-29 16:01:35
tags: Redis
categories: Redis
---
# 其实redis的事务是个假事务，没有实现原子性，

### 若要php支持事务，必须一起执行,其中incr会报错

    $status =  $redis->multi()->lPush($key1, '1123')->lPush($key2, '2123')->incr("age","age")->exec();

<!---->

    try {
        $redis = new Redis();
        $redis->connect('192.168.75.132', 6379);
        //开启事务
        $redis->multi();
        $redis->setex('keyTest', 60, 1);
        $redis->get('keyTest');
        $redis->incr('keyTest');
        $redis->get('keyTest');
        //执行事务
        $ret = $redis->exec();
        print_r($ret);
    } catch (Exception $e){
        echo $e->getMessage();
    }
    //输出
    Array
    (
        [0] => 1
        [1] => 1
        [2] => 2
        [3] => 2
    )

# 取消事务

    try {
        $redis = new Redis();
        $redis->connect('192.168.75.132', 6379);
        //先设置缓存keyTest为1
        $redis->setex('keyTest', 60, 1);
        //开启事务
        $redis->multi();
        $redis->setex('keyTest', 60, 10);
        $redis->get('keyTest');
        $redis->incr('keyTest');
        $redis->get('keyTest');
        //取消事务
        $redis->discard();
        $ret = $redis->get('keyTest');
        var_dump($ret);
        //查看keyTest
    } catch (Exception $e){
        echo $e->getMessage();
    }
    //输出
    string(1) "1"

# 监视键，并执行事务

    try {
        $redis = new Redis();
        $redis->connect('192.168.75.132', 6379);
        //先设置缓存keyTest为1
        $redis->setex('keyTest', 60, 1);
        //监视keyTest
        $redis->watch(array('keyTest'));
        //假设在开始监视之后，执行事务之前，keyTest被并发操作redis的其他用户修改了
        $redis->setex('keyTest', 60, 10);
        //开启事务
        $redis->multi();
        $redis->incr('keyTest');
        //执行事务
        $ret = $redis->exec();
        var_dump($ret);
        $ret = $redis->get('keyTest');
        var_dump($ret);
        //查看keyTest
    } catch (Exception $e){
        echo $e->getMessage();
    }
    //输出 
    bool(false)
    string(2) "10"

# redis抢购

```
<?php
header("content-type:text/html;charset=utf-8");
$redis = new Redis();
$redis->connect('127.0.0.1', 6379);
$redis->watch("mywatchlist");
$len = $redis->hlen("mywatchlist");
$rob_total = 100; //抢购数量
if ($len < $rob_total) {
    $redis->multi();
    $redis->hSet("mywatchlist", "user_id_" . mt_rand(1, 999999), time());
    $rob_result = $redis->exec();
    //file_put_contents("log.txt", $len . PHP_EOL, FILE_APPEND);
    if ($rob_result) {
        $mywatchlist = $redis->hGetAll("mywatchlist");
        echo '抢购成功' . PHP_EOL;
        echo '剩余数量：' . ($rob_total - $len - 1) . PHP_EOL;
        echo '用户列表：' . PHP_EOL;
        print_r($mywatchlist);
        exit;
    } else {
        exit('手气不好，再抢购！');
    }
} else {
    exit('已卖光');
}

```

