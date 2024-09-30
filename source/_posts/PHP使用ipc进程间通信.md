---
title: PHP使用ipc进程间通信
date: 2024-09-29 16:14:18
tags: php
categories: php
---
### que.php
```
<?php
class MsgQueue
{

    public $queue;

    public function __construct($queue)
    {
        $this->queue = $queue;
    }

    public function push($data, $type = 1)
    {
        $result = msg_send($this->queue, $type, $data);
        return $result;
    }

    public function pop($type = 0,$flags = MSG_IPC_NOWAIT)
    {
        msg_receive($this->queue, $type, $message_type, 1024, $message,true,$flags);
//        var_dump($message_type);
        return $message;
    }

    public function close()
    {
        return msg_remove_queue($this->queue);
    }

    public static function getQueue($path_name = __FILE__, $prop = '1', $perms = '0666')
    {
        $data              = array();
        $data['queue＿key'] = ftok($path_name, $prop);
        $data['queue']     = msg_get_queue($data['queue＿key'], $perms);
        return $data;
    }
}

```
### 父子通信
```
<?php
include_once 'que.php';
$message_queue_key= ftok(__FILE__, 'a');
$message_queue= msg_get_queue($message_queue_key, 0666);
$queue_obj = new MsgQueue($message_queue);
$pid = pcntl_fork();
if($pid>0){//主进程入列
	    while(1){
		            $msg = $queue_obj->push((array('a'=>321312,'v'=>'casd')));
			            sleep(2);
			        }
					
}else{//子进程出列
	
            while (1) {
                $message = $queue_obj->pop();
                if ($message !== false) {
                    var_dump($message);
                }
                sleep(1);
            }
}
```
## 跨进程通信A和B。这个时候ipc仅仅相当于一个普通队列
### 生产者
```
<?php
include_once 'que.php';

$message_queue_key = ftok(__FILE__, 'a');
$message_queue = msg_get_queue($message_queue_key, 0666);
$queue_obj = new MsgQueue($message_queue);

while (true) {
    $msg = array('a' => 321312, 'v' => 'casd');
    $queue_obj->push($msg);
    echo "Sent message: " . print_r($msg, true) . "\n";
    sleep(1); // 等待一段时间再发送下一条消息
}
```
### 消费者
```
<?php
include_once 'que.php';

$message_queue_key = ftok(__FILE__, 'a');
$message_queue = msg_get_queue($message_queue_key, 0666);
$queue_obj = new MsgQueue($message_queue);

while (true) {
    $message = $queue_obj->pop();
    if ($message !== false) {
        echo "Received message: " . print_r($message, true) . "\n";
    }
    sleep(2); // 等待一段时间再处理下一条消息
}
```
