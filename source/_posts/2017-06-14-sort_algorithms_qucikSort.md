---
layout:     post
title:         深入浅出排序算法(3)-快速排序
subtitle:   ""
date:       2017-06-14 16:30
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


`快速排序`与`归并排序`一样也是基于分治算法的排序算法.所以它的实现方法也与其他的分治算法一样,需要进行分解子任务,处理子任务,归并子任务这些步骤.

但`快速排序`与`归并排序`不同,它是一种`原地排序`算法(不需要额外的辅助数组),且`快速排序`不使用中间值来分解任务,而是使用`划分函数`.

### 算法过程


----------


![](https://upload.wikimedia.org/wikipedia/commons/6/6a/Sorting_quicksort_anim.gif)

 - 从数组中挑选出一个值,作为`基准值 k`.


 - 重新排序序列,**将所有小于`k`的值放到`k`前面,所有大于`k`的值放到`k`后面**(也可以理解为将数组`a`切分为两个子数组`a[begin...k-1],a[k+1...end]`,其中前一个子数组都小于`k`,后一个子数组都大于`k`).


 - 递归地将两个子数组进行快速排序(递归到最底部时,子数组的大小是零或一,也就是已经排序好了.).



### 划分函数


----------


`划分函数`就是上述步骤中的第二步,它将数组根据`基准值`进行重排序.根据`基准值`选择的位置不同,`划分函数`也有不同的实现方法,不过其根本思想都是将小于`基准值`的值放到前面,大于`基准值`的值放到后面.


#### 使用末尾元素作为基准值


----------


```java
    // 使用末尾元素作为基准值来进行切分
    private static int partitionUseEnd(Comparable[] a, int begin, int end) {
        Comparable pivot = a[end]; // 基准值,切分后的数组应满足左边都小于基准,右边都大于基准
        int i = begin - 1;

        for (int j = begin; j < end; j++) {
            // 如果j小于基准值则与i交换
            if (less(a[j], pivot)) {
                i++;
                swap(a, i, j);
            }
        }

        // 将基准值交换到正确的位置上
        int pivotLocation = i + 1;
        swap(a, pivotLocation, end);
        return pivotLocation;
    }
```

#### 使用首元素作为基准值


----------


```java
    // 使用首元素作为基准值来进行切分
    private static int partitionUseBegin(Comparable[] a, int begin, int end) {
        Comparable pivot = a[begin];
        int i = begin;
        int j = end + 1;

        while (true) {
            // 从左向右扫描,直到找出一个大于等于基准的值
            while (less(a[++i], pivot)) {
                if (i >= end)
                    break;
            }

            // 从右向左扫描,直到找出一个小于等于基准的值
            while (less(pivot, a[--j])) {
                if (j <= begin)
                    break;
            }

            // 如果指针i与j发生碰撞则结束循环
            if (i >= j)
                break;
            // 将左边大于小于基准的值与右边小于等于基准的值进行交换
            swap(a, i, j);
        }
        // 将基准值交换到正确的位置上
        swap(a, begin, j);
        return j;
    }
```


### 代码实现


----------


了解了`划分函数`的实现,剩下就只需要递归地调用`快速排序`不断地分解子任务即可.

注意,`快速排序`与`归并排序`不同,它不需要进行`归并`(划分后就已经是有序的了),并且是先进行`划分函数`,再分解任务.

```java
    public static void sort(Comparable[] a) {
        sort(a, 0, a.length - 1);
    }

    private static void sort(Comparable[] a, int begin, int end) {
        if (begin >= end)
            return;

        int k = partitionUseEnd(a, begin, end);
        sort(a, begin, k - 1);
        sort(a, k + 1, end);
    }
```

> 本文作者为[SylvanasSun(sylvanassun_xtz@163.com)][1],转载请务必指明原文链接.

[1]: https://github.com/SylvanasSun/