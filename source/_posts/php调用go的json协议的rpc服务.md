---
title: php调用go的json协议的rpc服务
date: 2024-09-29 16:30:48
tags: [Go,php]
categories: [php,Go]
---
### main.go
```
package main

import (
	"net/rpc"
	"net"
	"log"
	"net/rpc/jsonrpc"
)

//自己的数据类
type MyMath struct{
	
}

//加法--只能两个参数
func (mm *MyMath) Add(num map[string]float64,reply *float64) error {
    *reply = num["num1"] + num["num2"]
    return nil
}

//减法--只能两个参数
func (mm *MyMath) Sub(num map[string]string,reply *string) error {
    *reply = num["num1"] + num["num2"]
    return nil
}

func main() {
	//注册MyMath类，以代客户端调用
    rpc.Register(new(MyMath))
    listener, err := net.Listen("tcp", ":1234")
    if err != nil {
        log.Fatal("listen error:", err)
    }
    for {
        conn, err := listener.Accept()
        if err != nil {
            continue
        }
		//新协程来处理--json
        go jsonrpc.ServeConn(conn)
    }
}
```
### index.php
```
<?php
class JsonRPC
{
    private $conn;

    function __construct($host, $port) {
        $this->conn = fsockopen($host, $port, $errno, $errstr, 3);
        if (!$this->conn) {
            return false;
        }
    }

    public function Call($method, $params) {
        if ( !$this->conn ) {
            return false;
        }
        $err = fwrite($this->conn, json_encode(array(
                'method' => $method,
                'params' => array($params),
                'id'     => 0,
            ))."\n");
        if ($err === false)
            return false;
        stream_set_timeout($this->conn, 0, 3000);
        $line = fgets($this->conn);
        if ($line === false) {
            return NULL;
        }
        return json_decode($line,true);
    }
}

$client = new JsonRPC("127.0.0.1", 1234);
$r = $client->Call("MyMath.Add",array('num1'=>1,'num2'=>2));
var_export($r);
echo "<br/>";
$r = $client->Call("MyMath.Sub",array('num1'=>'1','num2'=>'2'));
var_dump($r);
```
