---
title: array_reduce实现路由中间件
date: 2024-09-29 16:11:15
tags: php
categories: php
---
```
<?php
interface Middleware
{
    public static function handle(Closure $next);
}

class Middleware1 implements Middleware
{
    public static function handle(Closure $next)
    {
        echo "Middleware1 before\n";
        $next();
        echo "Middleware1 after\n";
    }
}

class Middleware2 implements Middleware
{
    public static function handle(Closure $next)
    {
        echo "Middleware2 before\n";
        $next();
        echo "Middleware2 after\n";
    }
}

class Middleware3 implements Middleware
{
    public static function handle(Closure $next)
    {
        echo "Middleware3 before\n";
        $next();
        echo "Middleware3 after\n";
    }
}

function getSlice() // 返回一个函数，与上文的f一致
{
    return function ($stack, $pipe) {
        return function () use ($stack, $pipe) {
            return $pipe::handle($stack);
        };
    };
}

function then()
{
    $pipes = [
        "Middleware1",
        "Middleware2",
        "Middleware3",
    ];

    $firstSlice = function () { // 上文的目标函数 target
        echo "请求向路由器传递，返回响应.\n";
    };
    //嵌套闭包的解包方式是先从外到内，然后从内回归到外面,所以要根据注册顺序进行逆序
    $pipes = array_reverse($pipes);
    $closure = array_reduce($pipes, getSlice(), $firstSlice);
    var_dump($closure);//打印下该闭包
    // 因为最终返回了一个函数，所以需要call_user_func
    call_user_func(array_reduce($pipes, getSlice(), $firstSlice));
}
then();
echo "\n";
//array_reduce($pipes, getSlice(), $firstSlice)，执行完成后相当于生成了f嵌套闭包
//function f()
//{
//    return Middleware1::handle(
//        function () {
//            return Middleware2::handle(
//                function () {
//                    return Middleware3::handle(function () {
//                        echo "请求向路由器传递，返回响应.\n";
//                    });
//                }
//            );
//        }
//    );
//}
//call_user_func('f');
```
![](3302358-20240923164950137-837763612.png)

