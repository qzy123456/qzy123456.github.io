---
title: SplObserver 和 SplSubject 接口实现观察者模式
date: 2024-09-29 16:27:25
tags: php
categories: php
---
```
<?php

class Subject implements SplSubject {
    private $observers = [];
    private $state;

    public function attach(SplObserver $observer) {
        $this->observers[] = $observer;
    }

    public function detach(SplObserver $observer) {
        foreach ($this->observers as $key => $obs) {
            if ($obs === $observer) {
                unset($this->observers[$key]);
            }
        }
    }

    public function notify() {
        foreach ($this->observers as $observer) {
            $observer->update($this);
        }
    }

    public function setState($state) {
        $this->state = $state;
        $this->notify();
    }

    public function getState() {
        return $this->state;
    }
}

class Observer implements SplObserver {
    private $name;

    public function __construct($name) {
        $this->name = $name;
    }

    public function update(SplSubject $subject) {
        echo "观察者==》{$this->name} 开始执行事件: " . $subject->getState() . "\n";
    }
}

// 创建主题
$subject = new Subject();

// 创建观察者
$observer1 = new Observer("观察者 1");
$observer2 = new Observer("观察者 2");

// 将观察者添加到主题
$subject->attach($observer1);
$subject->attach($observer2);

// 改变状态
$subject->setState("事件");
```
### 基础版本
```
<?php
 
/**
* 观察者模式
*/

/**
* 抽象主题角色
*/
interface Subject {
 
    /**
     * 增加一个新的观察者对象
     * @param Observer $observer
     */
    public function attach(Observer $observer);
 
    /**
     * 删除一个已注册过的观察者对象
     * @param Observer $observer
     */
    public function detach(Observer $observer);
 
    /**
     * 通知所有注册过的观察者对象
     */
    public function notifyObservers();
}
 
/**
* 具体主题角色
*/
class ConcreteSubject implements Subject {
 
    private $_observers;
 
    public function __construct() {
        $this->_observers = array();
    }
 
    /**
     * 增加一个新的观察者对象
     * @param Observer $observer
     */
    public function attach(Observer $observer) {
        return array_push($this->_observers, $observer);
    }
 
    /**
     * 删除一个已注册过的观察者对象
     * @param Observer $observer
     */
    public function detach(Observer $observer) {
        $index = array_search($observer, $this->_observers);
        if ($index === FALSE || ! array_key_exists($index, $this->_observers)) {
            return FALSE;
        }
 
        unset($this->_observers[$index]);
        return TRUE;
    }
 
    /**
     * 通知所有注册过的观察者对象
     */
    public function notifyObservers() {
        if (!is_array($this->_observers)) {
            return FALSE;
        }
 
        foreach ($this->_observers as $observer) {
            $observer->update();
        }
 
        return TRUE;
    }
 
}
 
/**
* 抽象观察者角色
*/
interface Observer {
 
    /**
     * 更新方法
     */
    public function update();
}
 
class ConcreteObserver implements Observer {
 
    /**
     * 观察者的名称
     * @var <type>
     */
    private $_name;
 
    public function __construct($name) {
        $this->_name = $name;
    }
 
    /**
     * 更新方法
     */
    public function update() {
        echo 'Observer', $this->_name, ' has notified.<br />';
    }
 
}
//实例化类：
$subject = new ConcreteSubject();
 
/* 添加第一个观察者 */
$observer1 = new ConcreteObserver('aaa');
$subject->attach($observer1);
$subject->notifyObservers();
 
/* 添加第二个观察者 */
$observer2 = new ConcreteObserver('bbb');
$subject->attach($observer2);
$subject->notifyObservers();
 
/* 删除第一个观察者 */
$subject->detach($observer1);
$subject->notifyObservers();
 
```
