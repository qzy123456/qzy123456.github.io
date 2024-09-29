---
title: php生成短链功能
date: 2024-09-29 16:17:26
tags: php
categories: php
---
### 这种是市面上比较常用的，但是需要数据库存储。或者自己写一套加解密的方法，根据code进行解密，效率更高
### 直接上代码
```
<?php
function shortUrl($url)
{
    $charset = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";
    $key = 'this-is-salt'; // 加盐
    $timestamp = time(); // 时间戳
    $random = mt_rand(); // 随机数
    $urlHash = md5($key . $url . $random . $url);
    // 只使用哈希值的一部分来生成短链接
    $shortUrl = '';
    for ($i = 0; $i < 6; $i++) {
        // 取哈希值的某一部分并进行模运算，然后转换为字符
        $index = hexdec(substr($urlHash, $i, 1)) % strlen($charset);
        $shortUrl .= $charset[$index];
    }

    return $shortUrl;
}

$input = 'https://detail.tmall.com/item.htm?spm=a21bo.jianhua/a.201876.d2.5af92a89Ifuxtc&id=749045568815&xxc=ad_ct&priceTId=2147802817238772080956563e4b72&pisk=fsCEs5qKrWFedmXkbhOy7kLPS-RphIEjY_tWrabkRHxnOXilQZIyAzepOO-PjG-h4aUpr3jl436QCS_dJQducQPbGwQLY3l1YUvkIzYDz2fj0wncJQdu49XRseQd0bjKJuJu7CYyz0AkKpVMjhLWqDjkxdmMuEdkqgAu7hYW-DDkE3DMjEhcIwbo_E93unLdbjE6vp-c7-hoaacXKnj2jbqc_ervmwxZZbxLlPQF7wZEMNOdgi8fA5cF0ZCObFj0alRRXTjkzGVxVBBfvspNA-cGSCKMnTva-WbwTHJ25KuSpBWlv_JdL4UASBjOHnp3BlLNOs9yDdogxN_wxKWPAlhkAt7GbK1sfjORXTjkzGmF4ciJSadcw9ooUpY97naaS5VTcqbw5jfoeYpMMF-bReM-epY97naa7YHJIELwcyTC.'; // 长链
$output = shortUrl($input);
var_dump($output);
?>
```
### 存数据库,字段id,short_url,long_url这些字段就够了
### 处理短连的接口逻辑
```
public function longUrl(){
		    $short_url = $_GET['code'];
		    $data = 查数据库;
            //查不到就跳指定url
            if(empty($data)){
               header("location:https://www.taobao.com/");
            }else{
		       $url = $data['long_url'];
		       header("location:$url");
            }
}
```
