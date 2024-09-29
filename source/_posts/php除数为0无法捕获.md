---
title: php除数为0无法捕获
date: 2024-09-29 16:18:24
tags: [php]
categories: [php]
---
### 先看一个代码
```
$num = 0;

try {
    echo 1 / $num;
} catch (Exception $e) {
    echo $e->getMessage(); 
}
```
### 这时候得catch是无法捕获除数为0得错误
### 修复
```
<?php
function errorHandler($errno, $errstr, $errfile, $errline) {
    // 检查错误类型是否为除以零
    if ($errno == E_WARNING && strpos($errstr, 'Division by zero') !== false) {
        throw new Exception('Division by zero error');
    }
    // 可以在这里处理其他类型的错误
}

// 设置自定义错误处理函数
set_error_handler('errorHandler');

$num = 0;

try {
    echo 1 / $num;
} catch (Exception $e) {
    echo $e->getMessage(); // 这将输出 "Division by zero error"
}
```
