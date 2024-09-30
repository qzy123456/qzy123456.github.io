---
title: Swoole之Hyperf框架的迁移和填充
date: 2024-09-29 10:28:03
tags: php
categories: php
---
### hyperf框架的orm其实就是基于laravel改造的。会laravel就会hyperf，只不过目前为止hyperf的文档都没有填充相关的。
### 生成迁移文件,这点文档有，具体可以参考文档 https://hyperf.wiki/3.1/#/zh-cn/db/migration
```
php bin/hyperf.php gen:migration create_users_table
```
### 修改migrations文件夹下的对应文件(驼峰命名修改了create_users_table),为了测试，数据库只有2个字段
```
public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name');
        });
    }
```
### 开始迁移
```
php bin/hyperf.php migrate
```
### 迁移+填充，得有填充文件
```
php bin/hyperf.php migrate --seed
```
### 迁移单个
```
php bin/hyperf.php migrate --path=migrations/xx.php
```
### 生成填充文件。PS：这点详细的可以参考laravel的文档。
```
php bin/hyperf.php gen:seeder UserSeeder
```
### 修改seeders文件夹下的对应文件，如果有表关系映射，跟laravel的写法一样
```
<?php

declare(strict_types=1);

use Hyperf\Database\Seeders\Seeder;
use Hyperf\DbConnection\Db;

class UserSeeder extends Seeder
{
    public function run()
    {
        DB::table('users')->insert([
            ['name' => "hello1"],
            ['name' => "hello2"]
        ]);
    }
}
```
### 执行填充，单个填充跟laravel有区别，要指定路径+文件名
```
 //全部填充
 php bin/hyperf.php db:seed 
//根据绝对路径进行单个填充
php bin/hyperf.php db:seed --path=seeder/user_seeder.php
```
