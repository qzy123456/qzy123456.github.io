---
title: Zephir构建PHP扩展
date: 2024-09-29 16:26:17
tags: php
categories: php
---
### 工作原理就是把你写好的 php 代码编译成 c，然后你可以将其以扩展.so的形式添加到 'php.ini' 文件中。功能稍微少一点，适合简单场景
### 安装解释器 https://github.com/zephir-lang/php-zephir-parser
```
git clone https://github.com/zephir-lang/php-zephir-parser.git
cd php-zephir-parser
phpize
./configure
make && make install
编辑php.ini
vim /usr/local/php/etc/php.ini
[Zephir Parser]
extension=zephir_parser.so
```
### 或者直接一键安装 pecl install zephir_parser
### 安装zephir.phar
```
wget https://github.com/zephir-lang/zephir/releases/download/0.17.0/zephir.phar

# 移动到 bin
mv zephir.phar /usr/bin/
chmod 755 zephir.phar
ln -s /usr/bin/zephir.phar zephir
```
### 验证是否安装正确：
```
zephir help
```
### 开始编写代码
```
zephir init utils
```
### 执行之后,一个目录称为“utils”创建在当前工作目录:
```
$ cd utils
$ ls
ext/ utils/ config.json
```
### utils/utils/greeting.zep
```
namespace Utils;

class Greeting
{

    public static function say()
    {
        echo "hello world!";
    }

}
```
### 现在,我们需要告诉Zephir编译和生成的扩展,必须在代码根目录，也就是utils/utils目录下:
```
zephir build
```
### 如果一切顺利将看到以下输出:
```
Extension installed!
Add extension=utils.so to your php.ini
Don't forget to restart your web server
```
### 先移动utils.so到扩展目录下，我的在/usr/lib/php/20190902。最后修改php.ini中加入extension=utils.so

### 检查是否正常加载扩展通过执行以下:
```
$ php -m
[PHP Modules]
utils
```
### 测试
```
<?php
echo Utils\Greeting::say(), "\n";
```
