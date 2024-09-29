---
title: 分表分库之laravel表名重写
date: 2024-09-29 16:33:08
tags: php
categories: php
---
### 分表分库，不使用第三方中间件的话，自己根据分库分表的逻辑进行重写表名、库名
```
use Illuminate\Support\Str;
class Item extends Model {
   
    public $uid;
    //设置用户id，根据用户id进行取模(测试而已,正常用户信息可以放到token里，这样全局公用)
    public function setUid($uid)
    {
        $this->uid = $uid;
    }    
    //重写表名
    public function getTable()
    {
       $tableName = str_replace('\\', '', Str::snake(Str::pluralStudly(class_basename($this)));
       //假设当前道具表分成了10个
       return $tableName.'_'.($this->uid % 10);
       //return 'item_'.($this->uid % 10);
    }
    //重写数据库连接，提前配置好的，具体看自己的业务，不写就是默认连接
   public function getConnection()
    {
        return "default";
    }
}
```
