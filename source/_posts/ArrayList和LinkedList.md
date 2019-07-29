---
title: ArrayList和LinkedList #文章标题
date: 2019-06-10 20:00:14 #文章生成时间
tags: #文章标签 可以省略
     - 集合
description: 
categories: 源码学习
---
## 1、ArrayList

1、继承了AbstractList类，实现了List接口、RandomAccess接口。

> RandomAccess接口是一个标识，实现这个接口的数据结构支持随机访问，因为ArrayList底层是数组实现，天然支持随机访问，所以标识。

2、1.8的注释提到：与Vector几乎相同，除了不是同步的，即ArrayList不是线程安全的，线程安全的情况需要用Vector或者CopyOnWriteArrayList。

3、fail\-fast：是Java集合的一种错误检测机制，当利用迭代器遍历集合元素的时候，创建迭代器之后就不能对集合元素修改（删除、增加），否则就抛出 `ConcurrentModificationException` ，但是可以通过调用迭代器本身提供的方法进行增加删除元素。

产生原因：在创建Iterator时就定义了一个变量迭代器在调用 next\(\)、remove\(\) 方法时都是调用 checkForComodification\(\) 方法，该方法主要就是检测 modCount == expectedModCount ? 若不等则抛出 ConcurrentModificationException 异常，从而产生 fail\-fast 机制。ArrayList 中无论 add、remove、clear 方法只要是涉及了改变 ArrayList 元素的个数的方法都会导致 modCount 的改变。

4、默认设置

```java
//默认初始化大小
private static final int DEFAULT_CAPACITY = 10;
//有参构造时调用，即指定初始化大小
private static final Object[] EMPTY_ELEMENTDATA = {};
//空参构造调用，当空数组第一个元素被添加的时候，数组大小才扩大到10
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
//数组最大容量，因为需要8 bytes存储数组大小，所以-8。最大2^31，所以要8 bytes.
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
```
5、扩容具体实现：新建一个大小是原来1.5倍的数组，把原来数组的元素复制过去。

```java
private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        // 移位运算效率更高
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
//比较所需要的数组容量大小minCapacity和数组最大容量的大小关系
private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }
```
 `Arrays.copyOf` 是内部新建一个数组，然后把传入的数组元素拷贝到新数组，返回新数组。

6、remove方法是把元素往前移，然后把末尾的位置指向null，clear方法是把所有位置指向null，然后交给垃圾回收机制处理。

## 2、LinkedList

1、继承了AbstractSequentialList类，是一个实现了List接口、Deque接口的双端链表，支持高效的插入删除，也具有队列的特性。

2、不是线程安全的，线程安全可以调用静态类Collections中的synchronizedList方法。

3、LinkedList节点类的定义：

```java
private static class Node<E> {
        E item;
        //后继节点
        Node<E> next;
        //前驱节点
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```
## 3、两者的比较

1、ArrayList底层实现是数组，访问元素时间复杂度是O\(1\)，LinkedList底层实现是双向链表，不支持快速随机访问；

2、内存空间占用：ArrayList会在末尾预留一定的容量空间，LinkedList每个元素都要存储前后指针和元素数据，消耗的空间更大；

3、插入删除：ArrayList的时间复杂度受元素位置影响，LinkedList时间复杂度近似O\(1\)；

4、两个都不是线程安全的。

ps：其实区别的本质来源就是底层实现不同。
