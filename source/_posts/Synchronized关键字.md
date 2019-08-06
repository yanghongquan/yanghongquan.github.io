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

### 7、Synchronized和volatile区别

* volatile本质是在告诉jvm当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取； synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住。
* volatile仅能使用在变量级别；synchronized则可以使用在变量、方法、和类级别的。
* volatile仅能实现变量的修改可见性，不能保证原子性；而synchronized则可以保证变量的修改可见性和原子性。
* volatile不会造成线程的阻塞；synchronized可能会造成线程的阻塞。
* volatile标记的变量不会被编译器优化；synchronized标记的变量可以被编译器优化。
### 8、volatile

1、实现原理（可见性、有序性）
volatile关键字修饰的变量在被修改的时候会有一个lock前缀指令，这个指令会引发：
* 将当前处理器缓存行的数据会写回到系统内存。
* 这个写回内存的操作会引起在其他CPU里缓存了该内存地址的数据无效。
如果对声明了Volatile变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题，所以在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器要对这个数据进行修改操作的时候，会强制重新从系统内存里把数据读到处理器缓存里。

2、lock指令相当于一个内存屏障，防止指令重排序。

3、volatile经常用于：状态标记量、单例的double check
### 9、synchronized和Lock的区别

公平锁：尽量以请求锁的顺序来获取锁，多线程下等待时间最长的线程获得锁。
非公平锁：无法保证等待的线程获取锁的顺序。
区别：
* synchronized是Java的关键字，委托给JVM执行；Lock是一个类，是Java写的控制锁的代码。
* synchronized在发生异常时自动释放线程占有的锁，不会导致死锁；Lock发生异常时如果没有显式的 unlock() 释放锁很可能造成死锁；
* Lock可以让等待锁的线程响应中断，synchronized不能，等待的线程只能一直等待；
* 通过Lock可以知道线程是否成功获取锁，synchronized不行。
* Lock可以提高多线程进行读操作的效率，可以通过readwritelock实现读写分离。

