---
layout:     post
title:         什么是动态规划?
subtitle:   "了解动态规划的含义与作用"
date:       2017-06-27 18:00
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


动态规划(Dynamic Programming)是一种分阶段求解决策问题的数学思想,它通过把原问题分解为简单的子问题来解决复杂问题.动态规划在很多领域都有着广泛的应用,例如管理学,经济学,数学,生物学.

动态规划适用于解决带有`最优子结构`和`子问题重叠`性质的问题.

 - `最优子结构` : 即是局部最优解能够决定全局最优解(也可以认为是问题可以被分解为子问题来解决),如果问题的最优解所包含的子问题的解也是最优的，我们就称该问题具有`最优子结构`性质.


 - `子问题重叠` : 即是当使用递归进行自顶向下的求解时,**每次产生的子问题不总是新的问题,而是已经被重复计算过的问题**.动态规划利用了这种性质,使用一个集合将已经计算过的结果放入其中,当再次遇见重复的问题时,只需要从集合中取出对应的结果.


### 动态规划与分治算法的区别


----------


相信了解过分治算法的同学会发现,动态规划与分治算法很相似,下面我们例举出一些它们的相同之处与不同之处.

#### 相同点

 - 分治算法与动态规划都是将一个复杂问题分解为简单的子问题.


 - 分治算法与动态规划都只能解决带有`最优子结构`性质的问题.


#### 不同点

 - 分治算法一般都是使用递归自顶向下实现,动态规划使用迭代自底向上实现或带有记忆功能的递归实现.


 - 动态规划解决带有`子问题重叠`性质的问题效率更加高效.


 - 分治算法分解的子问题是相对独立的.


 - 动态规划分解的子问题是互相带有关联且有重叠的.


### 斐波那契数列


----------


斐波那契数列就很适合使用动态规划来求解,它在数学上是使用递归来定义的,公式为`F(n) = F(n-1) + F(n-2) `.

![斐波那契数列求解过程](http://wx3.sinaimg.cn/mw690/63503acbly1fgzut0a5nuj20lp08xq36.jpg)

#### 普通递归实现

一个最简单的实现如下.

```java
	public int fibonacci(int n) {
		if (n < 1)
			return 0;
		if (n == 1)
			return 1;
		if (n == 2)
			return 2;

		return fibonacci(n - 1) + fibonacci(n - 2);
	}
```

但这种算法并不高效,它做了很多重复计算,它的时间复杂度为`O(2^n)`.

#### 动态规划递归实现

使用动态规划来将重复计算的结果具有"记忆性",就可以将时间复杂度降低为`O(n)`.

```java
	public int fibonacci(int n) {
		if (n < 1)
			return 0;
		if (n == 1)
			return 1;
		if (n == 2)
			return 2;

		// 判断当前n的结果是否已经被计算,如果map存在n则代表该结果已经计算过了
		if (map.containsKey(n))
			return map.get(n);

		int value = fibonacci(n - 1) + fibonacci(n - 2);
		map.put(n, value);
		return value;
	}
```

虽然降低了时间复杂度,但需要维护一个集合用于存放计算结果,导致空间复杂度提升了.

#### 动态规划迭代实现

通过观察斐波那契数列的规律,发现n只依赖于前2种状态,所以我们可以自底向上地迭代实现.

```java
	public int fibonacci(int n) {
		if (n < 1)
			return 0;
		if (n == 1)
			return 1;
		if (n == 2)
			return 2;

		// 使用变量a,b来保存上次迭代和上上次迭代的结果
		int a = 1;
		int b = 2;
		int temp = 0;

		for (int i = 3; i <= n; i++) {
			temp = a + b;
			a = b;
			b = temp;
		}

		return temp;
	}
```

这样不仅时间复杂度得到了优化,也不需要额外的空间复杂度.

### 参考资料


----------


 - [Wikipedia][1]


> 本文作者为[SylvanasSun(sylvanassun_xtz@163.com)][2],转载请务必指明原文链接.

[1]: https://zh.wikipedia.org/wiki/%E5%8A%A8%E6%80%81%E8%A7%84%E5%88%92
[2]: https://github.com/SylvanasSun/