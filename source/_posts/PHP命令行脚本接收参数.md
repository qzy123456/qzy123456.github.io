---
title: PHP命令行脚本接收参数
date: 2024-09-29 16:12:53
tags: [php]
categories: [php]
---
### 使用$argv or $argc参数接收
```
<?php
echo "接收到{$argc}个参数";
print_r($argv);
```
```
php test.php

接收到1个参数Array
(

    [0] => test.php

)
php test.php a b c d
接收到5个参数Array
(

    [0] => test.php

    [1] => a

    [2] => b

    [3] => c

    [4] => d

)
```
### getopt
```
<?php
$param_arr = getopt('a:b:');
print_r($param_arr);
```
```
php test.php -a 345

Array

(

    [a] => 345

)

php test.php -a 345 -b 12q3

Array

(

    [a] => 345

    [b] => 12q3

)

php test.php -a 345 -b 12q3 -e 3322ff
Array

(

    [a] => 345

    [b] => 12q3

)
```
### fwrite
```
<?php
fwrite(STDOUT,'请输入您的信息：');

echo '您输入的信息是：'.fgets(STDIN);
```
