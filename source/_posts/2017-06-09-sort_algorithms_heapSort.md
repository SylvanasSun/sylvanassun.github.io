---
layout:     post
title:         深入浅出排序算法(1)-堆排序
subtitle:   "通过堆排序讲解堆与优先队列"
date:       2017-06-09 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - Algorithms
    - 排序算法
tags:
    - 排序算法
    - Algorithms
    - 2017
---


### 概述


----------



`堆排序`即是利用`堆`这个数据结构来完成排序的.所以,要想理解`堆排序`就要先了解`堆`.

### 堆


----------


`堆(Heap)`是一种数据结构,它可以被看做是一棵树的数组对象.一个`二叉堆`拥有以下性质.

 - 父节点`k`的左子节点在数组中的索引位置为`2 * k + 1`.


 - 父节点`k`的右子节点在数组中的索引位置为`2 * k + 2`.


 - 子节点`i`的父节点在数组中的索引位置为`(i - 1) / 2`.


 - 父节点`k`的任意子节点都必须小于(或大于)`k`.


 - 根节点必须是最大节点(或最小节点).


#### 最大堆代码实现


----------


```java
public class MaxHeap<T extends Comparable> {

    T[] heap;

    private MaxHeap() {
    }

    public MaxHeap(T[] heap) {
        this.heap = heap;
        buildHeap();
    }

    /**
     * 自底向上构建堆
     */
    private void buildHeap() {
        int length = heap.length;
        // 当堆为空或者长度为1时不需要任何操作
        if (length <= 1)
            return;

        int root = (length - 2) >>> 1; // (i - 1) / 2
        while (root >= 0) {
            heapify(heap, length, root);
            root--;
        }
    }

    /**
     * 调整堆的结构
     *
     * @param heap   堆
     * @param length 堆的长度
     * @param root   根节点索引
     */
    public void heapify(T[] heap, int length, int root) {
        if (root >= length)
            return;

        int largest = root; // 表示root,left,right中最大值的变量
        int left = (root << 1) + 1; // 左子节点,root * 2 + 1
        int right = left + 1; // 右子节点,root * 2 + 2

        // 找出最大值
        if (left < length && greater(heap[left], heap[largest]))
            largest = left;
        if (right < length && greater(heap[right], heap[largest]))
            largest = right;

        // 如果largest发生变化,将largest与root交换
        if (largest != root) {
            T t = heap[root];
            heap[root] = heap[largest];
            heap[largest] = t;
            // 继续向下调整堆
            heapify(heap, length, largest);
        }
    }

    private boolean greater(Comparable a, Comparable b) {
        return a.compareTo(b) > 0;
    }

}
```

### 优先队列


----------


普通的队列是基于`先进先出`的,也就是说最先入队的元素永远是在第一位,而`优先队列`中的每一个元素都是拥有`优先级`的,`优先级`最高的元素永远在第一位.

`优先队列`也是`贪心算法`的体现,所谓的`贪心算法`即是在问题求解的每一步中总是选择当前最好的结果.

`堆`就是用于实现`优先队列`的,因为`堆`的性质与`优先队列`十分吻合.

#### 添加


----------


往`优先队列`中添加元素时,我们只需要将元素添加到数组末尾并调整堆(以下例子均是以最大堆为例).

```java
    public boolean add(T t) {
        if (t == null)
            throw new NullPointerException();
        if (size == queue.length)
            resize(queue.length * 2);
        int i = size;
		// 如果当前队列为空,则不需要进行堆调整直接插入元素即可
        if (i == 0)
            queue[0] = t;
        else
            swim(i, t);
        size++;
        return true;
    }
	
	// 上浮调整
    private void swim(int i, T t) {
        Comparable<? super T> key = (Comparable) t;
        while (i > 0) {
            int parent = (i - 1) >>> 1;
            T p = (T) queue[parent];
			// 如果key小于他的父节点(符合最大堆规则)则结束调整
            if (key.compareTo(p) < 0)
                break;
            queue[i] = p;
            i = parent;
        }
        queue[i] = key;
    }	
```

#### 删除


----------


删除操作要稍微麻烦一点,将`优先队列`中末尾的元素放到队头并进行堆调整.

```java
    public T poll() {
        if (isEmpty())
            return null;
        int s = --size;
        Object result = queue[0];
        Object end = queue[s];
        queue[s] = null;
        if (s != 0)
            sink(0, (T) end);
        if (size <= queue.length / 4)
            resize(queue.length / 2);
        return (T) result;
    }
	
	// 下沉调整
    private void sink(int i, T t) {
        Comparable<? super T> key = (Comparable<? super T>) t;
        int half = size >>> 1;
        while (i < half) {
            int child = (i << 1) + 1; // 左子节点
            int right = child + 1; // 右子节点
            T max = (T) queue[child];
            // find maximum element
            if (right < size &&
                    ((Comparable<? super T>) max).compareTo((T) queue[right]) < 0)
                max = (T) queue[child = right];
			// key大于它的最大子节点(符合最大堆规则)则结束调整	
            if (key.compareTo(max) > 0)
                break;
            queue[i] = max;
            i = child;
        }
        queue[i] = key;
    }	
```

[点击查看优先队列完整代码][1]

### 堆排序


----------


实现`堆排序`有两种方法,一种是使用`优先队列`,另一种是直接使用`堆`.

#### 直接使用堆实现堆排序


----------


```java
    // 使用最大堆实现堆排序
    private static void maxHeapSort(Comparable[] a) {
        MaxHeap<Comparable> maxHeap = new MaxHeap<>(a);
        //不断地将最大堆中顶端元素(最大值)与最底部的元素(最小值)交换
        for (int i = a.length - 1; i > 0; i--) {
            Comparable largest = a[0];
            a[0] = a[i];
            a[i] = largest;
            // 堆减少,并调整新的堆
            maxHeap.heapify(a, i, 0);
        }
    }
```

#### 使用优先队列实现堆排序

```java
    // 使用优先队列实现堆排序
    private static void pqSort(Comparable[] a) {
        MinPriorityQueue<Comparable> priorityQueue = new MinPriorityQueue<>();
        for (int i = 0; i < a.length; i++) {
            priorityQueue.add(a[i]);
        }
        for (int i = 0; i < a.length; i++) {
            a[i] = priorityQueue.poll();
        }
    }
```

> 本文作者为[SylvanasSun(sylvanassun_xtz@163.com)][2],转载请务必指明原文链接.

[1]: https://github.com/SylvanasSun/test-demo/blob/master/src/main/java/com/sun/sylvanas/data_struct/heap/MaxPriorityQueue.java
[2]: https://github.com/SylvanasSun