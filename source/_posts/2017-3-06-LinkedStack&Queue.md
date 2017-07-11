---
layout:     post
title:      "使用链表做为Stack、Queue中的数据表示结构的基本思路"
subtitle:   "LinkedStack&LinkedQueue"
date:       2017-03-06 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - Algorithms
    - 数据结构
    - LinkedTable
tags:
    - 数据结构
    - LinkedTable
    - 2017
---


### 什么是链表?


----------


链表是一种常见的基础数据结构(数组也是基础数据结构),它是一种递归的数据结构,由一系列节点(Node)组成,节点含有一个存储数据的数据域和一个指向下一个节点地址位置的引用.

链表是线性表的一种,但是它的物理存储结构是非连续、 非顺序的,元素的逻辑顺序是通过节点之间的链接确定的.

### 数据结构与数据类型的区别


----------


 - <font color="red">数据类型</font>是一组数据和一组对这些值进行操作的集合.

 - <font color="red">数据结构</font>强调的是数据的存储和组织方式.

常见的数据结构有:数组、栈、队列、链表、树、图、堆、散列表.

### 链表是否可以替代数组?


----------


由于创建数组需要预先知道数组的大小,所以想要动态的扩容需要不断地创建新数组,而链表则可以充分利用内存空间,实现较为灵活的动态扩展.

使用链表替代数组有优点也有缺点:

**优点**

 1. 在链表中进行插入操作或是删除操作都更加方便快速.


 2. 链表所需的空间总是和集合的大小成正比.


 3. 链表操作所需的时间总是和集合的大小无关.

**缺点**

 1. 无法像数组一样可以通过索引来进行随机访问.


 2. 由于每一个元素节点都是一个对象,所以需要的空间开销比较大.


### Stack


----------

![](/img/note-img/2017-3-06-LinkedStack&Queue/LinkedStack.png)


栈是一种基于后进先出(LIFO)的数据结构,其中的元素除了头尾之外,每个元素都有一个前驱和一个后继.

使用链表来表示栈内部的数据时,栈顶就是链表的头部,当push元素时将元素添加在表头,当pop元素时将元素从表头删除.


#### 代码实现

```java
/**
 * 使用链表实现的可迭代的下压栈(后进先出)
 * <p>
 * Created by SylvanasSun on 2017/3/6.
 */
public class Stack<T> implements Iterable<T> {

    private Node first; //栈顶(链表头部)
    private int N; //元素个数

    /**
     * 用于表示链表中的节点
     */
    private class Node {
        T item;
        Node next;
    }

    /**
     * 判断Stack是否为空
     *
     * @return true代表Stack为空, false为未空
     */
    public boolean isEmpty() {
        return first == null; //也可以用N==0来判断
    }

    /**
     * 返回Stack中的元素数量
     *
     * @return 元素数量
     */
    public int size() {
        return N;
    }

    /**
     * 将元素t添加到栈顶
     *
     * @param t 添加的元素
     */
    public void push(T t) {
        Node oldFirst = first;
        first = new Node();
        first.item = t;
        first.next = oldFirst;
        N++;
    }

    /**
     * 将栈顶的元素弹出
     *
     * @return 栈顶节点的item
     */
    public T pop() {
        T item = first.item;
        first = first.next;
        N--;
        return item;
    }

    public Iterator<T> iterator() {
        return new ListIterator();
    }

    /**
     * 迭代器,维护了一个实例变量current来记录链表的当前结点.
     * 这段代码可以在Stack和Queue之间复用,因为它们内部数据的数据结构是相同的,
     * 只是访问顺序分别为后进先出和先进先出而已.
     */
    private class ListIterator implements Iterator<T> {
        private Node current = first;

        public boolean hasNext() {
            return current != null;
        }

        public T next() {
            T item = current.item;
            current = current.next;
            return item;
        }

        public void remove() {

        }
    }
}
```

### Queue


----------


![](/img/note-img/2017-3-06-LinkedStack&Queue/LinkedQueue.png)

队列是一种基于先进先出(FIFO)的数据结构,元素的处理顺序就是它们被添加到队列中的顺序.

可以使用实例变量first指向队列的队头,实例变量last指向队列的队尾,当将一个元素入列时,就将这个元素添加到队尾,当要将一个元素出列时,就删除队头的节点.

#### 代码实现

```java
/**
 * 使用链表实现的Queue,它与Stack的区别在于链表的访问顺序.
 * Queue的访问顺序是先进先出的.
 * <p>
 * Created by SylvanasSun on 2017/3/6.
 */
public class Queue<T> implements Iterable<T> {

    private Node first; //链表头部,即队头
    private Node last; //链表尾部,即队尾
    private int N; //size

    /**
     * 用于表示链表中的节点
     */
    private class Node {
        T item;
        Node next;
    }

    /**
     * 判断Queue是否为空
     *
     * @return true代表Stack为空, false为未空
     */
    public boolean isEmpty() {
        return first == null;
    }

    /**
     * 返回Queue中的元素数量
     *
     * @return 元素数量
     */
    public int size() {
        return N;
    }

    /**
     * 入队,向队尾添加新的元素
     *
     * @param item 添加的元素
     */
    public void enqueue(T item) {
        Node oldLast = last;
        last = new Node();
        last.item = item;
        last.next = null;

        /**
         * 如果队列为空,队头指向队尾(队列中只有一个元素),
         * 否则将旧的队尾的next指向新的队尾
         */
        if (isEmpty()) first = last;
        else oldLast.next = last;
        N++;
    }

    /**
     * 出队,将队头节点弹出队列
     *
     * @return 队头节点的item
     */
    public T dequeue() {
        T item = first.item;
        first = first.next;
        N--;
        //如果队列为空,队尾则为null
        if (isEmpty()) last = null;
        return item;
    }

    public Iterator<T> iterator() {
        return new ListIterator();
    }

    /**
     * 迭代器,与Stack中的实现一致
     */
    private class ListIterator implements Iterator<T> {

        private Node current = first;

        public boolean hasNext() {
            return current != null;
        }

        public T next() {
            T item = current.item;
            current = current.next;
            return item;
        }

        public void remove() {

        }
    }
}
```

### end


----------


 - Author: SylvanasSun
 - GitHub: https://github.com/SylvanasSun
 - Email: sylvanassun_xtz@163.com
 - Reference: 《Algorithms 4th edition》& [wiki][1]


  [1]: https://zh.wikipedia.org/wiki/%E9%93%BE%E8%A1%A8