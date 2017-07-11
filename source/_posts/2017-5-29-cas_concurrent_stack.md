---
layout:     post
title:         谈谈如何实现一个非阻塞的线程安全的集合
subtitle:   "通过实现一个ConcurrentStack为例讲解了CAS指令"
date:       2017-05-29 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - 多线程
    - CAS
tags:
    - CAS
    - 多线程
    - 2017
---


### 概述


----------


众所周知,想要在`java`中要实现一个线程安全的类有很多方法.最简单直接的即是使用`synchronized`关键字或`ReentrantLock`.

但是,这两种同步方法都是基于锁的,基于锁的同步方法是阻塞的,即未争夺到锁的线程需要阻塞等待(或挂起)直到锁可用.

这种方法具有一些明显的缺点:

 - 被阻塞的线程无法去做任何其他事情,如果这个线程是优先级较高的线程甚至会发生非常不好的结果(优先级倒置).


 - 由于`java`的线程模型是基于内核线程实现的,挂起恢复线程需要来回地切换到内核态,性能开销很大.


 - 当两个(或多个)线程都阻塞着等待另一方释放锁时,将会引发死锁.

那么有非阻塞的方法来实现同步吗?(`volatile`关键字也是非阻塞的,但它只保证了数据的可见性与有序性,并不保证原子性)

有!在`jdk5`中,java增加了大量的原子类来保证无锁下的操作原子性,可以说`java.util.concurrent`包下的所有类都几乎用到了这些原子类.

### CAS


----------


这些原子类都是基于`CAS`实现的,`CAS`即是**Compare And Swap**,它的原理简单来讲就是**在更新新值之前先去比较原值有没有发生变化,如果没发生变化则进行更新**.

`java`中的`CAS`是通过`Unsafe`类中的本地方法实现的,而这些本地方法需要通过现代处理器提供的`CAS`指令实现(在`Intel`处理器中该指令为`cmpxchg`).

所以我们发现,**`CAS`操作的原子性是由处理器来保证的**.

#### 比较的过程

在`CAS`操作中包含了三个数,`V(内存位置)`,`A(预期值)`,`B(新值)`.

 - 首先会将`V`与`A`进行匹配.


 - 如果两个值相等,则使用`B`作为新值进行更新.


 - 如果不相等,则不进行更新操作(一般的补救措施是继续进行请求).

#### 与锁相比的优点

 - `CAS`操作是无锁的实现,所以它不会发生死锁情况.

 - 虽然`CAS`操作失败需要不断的进行请求重试,但相对于不断地挂起或恢复线程来说,性能开销要低得多.

 - `CAS`的粒度更细,操作也更加轻量与灵活.

### ConcurrentStack


----------


我们通过实现一个简单的`ConcurentStack`来看看`CAS`操作是如何保证线程安全的.

[完整代码请从作者的Gist中获取][1]

#### 节点的实现

```java
public class ConcurrentStack<E> implements Iterable<E> {

    private AtomicReference<Node<E>> head = new AtomicReference<>(null);
    private AtomicInteger size = new AtomicInteger(0);

    /**
     * This internal static class represents the nodes in the stack.
     */
    private static class Node<E> {
        private final E value;
        private volatile Node<E> next;

        private Node(E value, Node<E> next) {
            this.value = value;
            this.next = next;
        }

    }
}
```

#### push

push函数主要是通过观察头节点(这里的头节点即是`V`),然后构建一个新的节点(它代表`B`)放于栈顶,如果`V`没有发生变化,则进行更新.如果发生了变化(被其他线程修改),就重新尝试进行`CAS`操作.

```java
    /**
     * Insert a new element to the this stack.
     *
     * @return if {@code true} insert success,{@code false} otherwise
     * @throws IllegalArgumentException if {@code value} is null
     */
    public boolean put(E value) {
        if (value == null)
            throw new IllegalArgumentException();
        return putAndReturnResult(value);
    }

    private boolean putAndReturnResult(E value) {
        Node<E> oldNode;
        Node<E> newNode;
        do {
            oldNode = head.get();
            newNode = new Node<E>(value, oldNode);
        } while (!head.compareAndSet(oldNode, newNode));
        sizePlusOne();
        return true;
    }
```

#### pop

pop函数中的`CAS`操作的思想基本与push函数一致.

```java
    /**
     * Return the element of stack top and remove this element.
     *
     * @throws NullPointerException if this stack is empty
     */
    public E pop() {
        if (isEmpty())
            throw new NullPointerException();
        return removeAndReturnElement();
    }

    private E removeAndReturnElement() {
        Node<E> oldNode;
        Node<E> newNode;
        E result;
        do {
            oldNode = head.get();
            newNode = oldNode.next;
            result = oldNode.value;
        } while (!head.compareAndSet(oldNode, newNode));
        sizeMinusOne();
        return result;
    }

```

### end


----------


非阻塞的算法实现的复杂度要比阻塞算法复杂的多,但它能带来更少的性能开销,在`jdk`中,很多线程安全类都是在尽量地避免使用锁的基础上来实现线程安全.

> 本文作者为[SylvanasSun(sylvanassun_xtz@163.com)][2],转载请务必指明原文链接.

[1]: https://gist.github.com/SylvanasSun/15353e5567e1890b45f516f7fe6a187d
[2]: https://github.com/SylvanasSun