---
title: Synchronized关键字 #文章标题
date: 2018-10-10 22:00:14 #文章生成时间
tags: #文章标签 可以省略
     - 并发
     - Java
description: 
categories: 读书笔记
---

### 1、作用

保证同一时刻只有一个线程执行该段代码，以达到保证并发安全的效果

### 2、两种用法

* 对象锁
> 方法锁、同步代码块锁

* 类锁

> synchronized修饰静态的方法


指定锁位Class对象

### 3、类锁

不管Java对象有多少个实例对象，但是只有一个class对象，通过给class对象上锁达到同步的效果

### 4、访问同一个对象实例的两个不同的synchronized普通方法，由于默认获取的是`this`锁，所以会出现串行的情况

```java
public class SychronizedObjectMethod implements Runnable{

    static SychronizedObjectMethod instance = new SychronizedObjectMethod();

    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();

        while (t1.isAlive() | t2.isAlive()) {

        }

        System.out.println("finished!");
    }

    @Override
    public void run() {

        if (Thread.currentThread().getName().equals("Thread-0")){
            method1();
        }else {
            method2();
        }

    }

    public synchronized void method1(){
        System.out.println("我是加锁的普通同步方法1，我叫"+Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName()+"运行结束");

    }

    public synchronized void method2(){
        System.out.println("我是加锁的普通同步方法2，我叫"+Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println(Thread.currentThread().getName()+"运行结束");

    }
}
```
当把method2加上static时，这两个方法同步时获取的锁就不是同一个锁了，所以会并行执行这两个方法

### 5、小结

* 一把锁只能同时被一个线程获取
* 每个实例都对应有自己的一把锁，不同的实例之间互不影响。例外：锁对象时`*.class`以及synchronized修饰的static方法时，所有的对象共用同一把锁
* 无论是方法正常执行完毕或者方法抛出异常，都会释放锁

### 6、性质

1、可重入

> 同一线程的外层函数获得锁之后，内层函数可以直接再次获取该锁

可重入原理：加锁次数计数器

2、不可中断（原子性）


