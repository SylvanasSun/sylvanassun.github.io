---
layout:     post
title:         深入浅出排序算法(2)-归并排序
subtitle:   "简单高效的归并排序"
date:       2017-06-12 15:00
author:     "Sylvanas Sun"
header-img: "img/post-bg-unix-linux.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 算法
    - 2017
---


### 概述


----------


`归并排序`是基于分治算法实现的一种排序算法,它将数组分割为两个子数组,然后对子数组进行排序,最终将子数组`归并`为有序的数组.

`归并排序`的时间复杂度为`O(n log n)`,空间复杂度为`O(1)`,并且它是稳定的排序算法(所谓稳定即是不影响值相等元素的相对次序).

### 算法过程


----------


![](https://upload.wikimedia.org/wikipedia/commons/c/cc/Merge-sort-example-300px.gif)

 - 首先,`归并排序`需要将一个大小为`n`个元素的数组分解为各包含`n/2`个元素的子数组(这个分解的过程会不断进行,直到子数组元素个数为`1`).


 - 当子数组的元素个数为`1`时,代表这个子数组已经有序,开始两两归并(将两个个数为`1`的子数组归并为一个个数为`2`的子数组,不断归并,直到所有子数组个数为`2`,然后继续将两个个数为`2`的子数组归并为一个个数为`4`的子数组....以此类推).


 - 不断重复步骤2,直到整个数组有序.


### 归并


----------


通过以上的了解,我们发现`归并排序`中最重要的步骤就是`归并`.

采用类似`洗牌`的方式来理解这个过程.想象辅助数组为一个空牌堆,两个子数组为两堆牌`a`和`b`.我们从`a`堆与`b`堆中**各取出一张牌进行比较,然后将较小的牌放入空牌堆中**,不断重复比较直到任一牌堆为空.最后,再将未空的牌堆全部放入空牌堆中.

```java
    // 将两个子序列进行归并
    private static void merge(Comparable[] a, int lo, int mid, int hi) {
        Comparable[] aux = new Comparable[a.length]; // 辅助数组
        int i = lo, j = mid + 1;
        int count = lo;
        // 对[lo...mid] 与 [mid+1...hi] 两个子序列的首元素进行比较,将较小的元素放入辅助数组
        while (i <= mid && j <= hi) {
            if (less(a[i], a[j]))
                aux[count++] = a[i++];
            else
                aux[count++] = a[j++];
        }

        //将[lo...mid] 与 [mid+1...hi] 两个子序列中剩余的元素放入辅助数组
        while (i <= mid) {
            aux[count++] = a[i++];
        }
        while (j <= hi) {
            aux[count++] = a[j++];
        }

        // 将辅助数组中的元素复制到源数组中
        for (int k = lo; k <= hi; k++) {
            a[k] = aux[k];
        }
    }
```


### 递归实现


----------


只要理解了`归并`的过程,剩下就很容易实现了.`归并排序`的递归实现如下.

```java
    public static void sort(Comparable[] a) {
        sort(a, 0, a.length - 1);
    }
	
    // 递归实现归并排序
    private static void sort(Comparable[] a, int lo, int hi) {
        if (lo >= hi)
            return;

        int mid = (lo + hi) >>> 1; // (lo + hi) / 2
		// 分解数组
        sort(a, lo, mid);
        sort(a, mid + 1, hi);
		// 归并
        merge(a, lo, mid, hi);
    }
```


### 非递归实现


----------


我们已经知道了`归并排序`中最小子数组的元素个数为`1`,非递归实现只需要从`1`开始自底向上地归并即可(递归实现的真实计算过程也是如此,这是由于递归调用是后进先出的).

```java
    // 非递归实现归并排序
    private static void sortUnRecursive(Comparable[] a) {
        int len = 1; // 自底向上实现归并排序,子序列的最小粒度为1
        while (len < a.length) {
            for (int i = 0; i < a.length; i += len << 1) {
                merge(a, i, len);
            }
            len = len << 1; // 子序列规模每次迭代时乘2
        }
    }
	
	// 与递归实现的归并函数不同,需要注意边界检查
    private static void merge(Comparable[] a, int lo, int hi) {
        int length = a.length;
        Comparable[] aux = new Comparable[length];
        int count = lo;
        // 子数组1
        int i = lo;
        int i_bound = lo + hi;
        // 子数组2
        int j = i_bound;
        int j_bound = j + hi;

        // 注意j的边界检查
        while (i < i_bound && j < j_bound && j < length) {
            if (less(a[i], a[j]))
                aux[count++] = a[i++];
            else
                aux[count++] = a[j++];
        }

        // i和j都有可能越界
        while (i < i_bound && i < length) {
            aux[count++] = a[i++];
        }
        while (j < j_bound && j < length) {
            aux[count++] = a[j++];
        }

        int k = lo;
        while (k < j && k < length) {
            a[k] = aux[k];
            k++;
        }
    }	
```

> 本文作者为[SylvanasSun(sylvanassun_xtz@163.com)][1],转载请务必指明原文链接.

[1]: https://github.com/SylvanasSun