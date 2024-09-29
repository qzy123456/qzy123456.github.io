---
title: php实现一个简单的安装程序
date: 2024-09-29 16:20:05
tags: [php]
categories: [php]
---
### 只是为了测试，具体可以参考其他开源软件的写法
### 原理都是动态创建数据库，导入基础sql，包含管理员信息。然后生成一个install.lock的文件，下次进来判断有这个文件，证明是安装过了
### html
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>安装程序</title>
</head>
<body>
    <h1>安装程序</h1>
    <form action="install.php" method="post">
        <label for="dbHost">数据库主机:</label>
        <input type="text" id="dbHost" name="dbHost" required><br><br>
        <label for="dbHost">数据库端口:</label>
        <input type="number" id="dbPort" name="dbPort" required><br><br>
        <label for="dbName">数据库名:</label>
        <input type="text" id="dbName" name="dbName" required><br><br>
        <label for="dbUser">数据库用户:</label>
        <input type="text" id="dbUser" name="dbUser" required><br><br>
        <label for="dbPassword">数据库密码:</label>
        <input type="password" id="dbPassword" name="dbPassword"><br><br>

        <input type="submit" value="安装">
    </form>
</body>
</html>
```
### php代码
```
<?php
$hostname = $_POST['dbHost'] ?? 'localhost';
$dbPort  =  $_POST['dbPort'] ?? 3306;
$username = $_POST['dbUser'] ?? '';
$password = $_POST['dbPassword'] ?? '';
$dbName = $dbName ?? '';

// 数据库连接 DSN
$dsn = "mysql:host=$hostname;port=$dbPort";

try {
    // 创建一个新的 PDO 实例
    $db = new PDO($dsn, $username, $password);
    $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    // 动态创建数据库
    $createDbSql = "CREATE DATABASE IF NOT EXISTS `$dbName` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci";
    $db->exec($createDbSql);

    // 选择数据库
    $db->exec("USE `$dbName`");

    // 创建管理员表，根据需要插入数据，或者直接导入sql
    $createTableSql = "CREATE TABLE IF NOT EXISTS users (
        id INT(6) UNSIGNED AUTO_INCREMENT PRIMARY KEY,
        username VARCHAR(30) NOT NULL,
        password VARCHAR(255) NOT NULL,
        email VARCHAR(50),
        reg_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    )";
    $db->exec($createTableSql);

    echo "安装成功，数据库和表已创建。";
} catch (PDOException $e) {
    $err = "安装失败: " . $e->getMessage();
    echo $err;
}
?>
```
