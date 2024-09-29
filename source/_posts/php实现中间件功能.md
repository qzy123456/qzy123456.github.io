---
title: php实现中间件功能
date: 2024-09-29 16:28:50
tags: php
categories: php
---
```
<?php
// 框架核心应用层
$application = function($name) {
    echo "this is a {$name} application\n";
};
 
// 前置校验中间件
$auth = function($handler) {
    return function($name) use ($handler) {
        echo "{$name} need a auth middleware\n";
        return $handler($name);
    };
};
 
// 前置过滤中间件
$filter = function($handler) {
    return function($name) use ($handler) {
        echo "{$name} need a filter middleware\n";
        return $handler($name);
    };
};
 
// 后置日志中间件
$log = function($handler) {
    return function($name) use ($handler) {
        $return = $handler($name);
        echo "{$name} need a log middleware\n";
        return $return;
    };
};
 
// 中间件栈
$stack = [];
 
// 打包
function pack_middleware($handler, $stack)
{
    foreach (array_reverse($stack) as $key => $middleware) 
    {
        $handler = $middleware($handler);
    }
    return $handler;
}
 
// 注册中间件
// 这里用的都是全局中间件，实际应用时还可以为指定路由注册局部中间件
$stack['log'] = $log;
$stack['filter'] = $filter;
$stack['auth'] = $auth;
 
$run = pack_middleware($application, $stack);
$run('do');
```
### 输出
```
Laravle need a filter middleware
Laravle need a auth middleware
this is a Laravle application
Laravle need a log middleware
```
### 中间件的执行顺序是由打包函数(pack_middleware)决定，这里返回的闭包实际上相当于:
```
$run = $log($filter($auth($application)));
$run('do');
```
```
Laravel 框架中间件使用 array_reverse 的原因是为了实现中间件的逆序执行。‌

在 Laravel 框架中，‌中间件是通过管道流（‌Pipeline）‌的方式执行的，‌其中 Illuminate\Pipeline\Pipeline 
类负责协调中间件的执行顺序。‌中间件的执行顺序对于应用程序的处理流程至关重要，‌而 Laravel 通过 array_reverse 函数来实现中间件的逆序执行，‌
确保了中间件的处理逻辑能够按照预期的方式工作。‌
具体来说，‌array_reverse 函数用于将数组反转，‌使得数组的元素按照相反的顺序排列。‌在 Laravel 的管道流实现中，‌中间件数组通过 array_reverse 
函数进行反转后，‌再通过 array_reduce 函数进行处理。‌这样做的好处是，‌当闭包函数（‌即管道流的下一步）‌被调用时，‌它能够按照从内到外的顺序执行中间件，‌
形成了一个类似洋葱的结构，‌其中最内层的中间件首先被执行，‌最外层的中间件最后被执行。‌这种执行顺序确保了中间件的处理逻辑能够按照预期的方式工作，‌
从而实现了 Laravel 框架中间件的功能和性能优化。‌
```
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
