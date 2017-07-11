---
layout:     post
title:         红黑树那点事儿
subtitle:   "深入浅出红黑树的构造原理"
date:       2017-06-16 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - Algorithms
    - 数据结构
    - Tree
tags:
    - Tree
    - 数据结构
    - Algorithms
    - 2017
---


### 概述


----------


`红黑树`是一种`自平衡二叉查找树`,它相对于`二叉查找树`性能会更加高效(查找、删除、添加等操作需要`O(log n)`,其中`n`为树中元素的个数),但实现较为复杂(需要保持自身的平衡).


### 性质


----------


![](https://upload.wikimedia.org/wikipedia/commons/6/66/Red-black_tree_example.svg)

`红黑树`与`二叉查找树`不同,它的节点多了一个颜色属性,每个节点非黑即红,这也是它名字的由来.

`红黑树`的节点定义如以下代码: 

```java
    private static final boolean RED = true;
    private static final boolean BLACK = false;
    private Node root;

    private class Node {
        private int size = 0;
        private boolean color = RED; //颜色
        private Node parent, left, right;
        private int orderStatus = 0;
        private K key;
        private V value;

        public Node(K key, V value) {
            this.key = key;
            this.value = value;
            this.size = 1;
        }
    }
```

完整的代码我已经放在了我的`Gist`中,[点击查看完整代码][1].

`红黑树`需要保证以下性质: 

 1. 每个节点的颜色非黑即红.


 2. **根节点的颜色为黑色.**


 3. 所有叶子节点都为黑色(即NIL节点).


 4. **每个红色节点的两个子节点都必须为黑色(不能有两个连续的红节点).**

 5. **从任一节点到其叶子的所有简单路径包含相同数量的黑色节点.**


### 插入


----------


`红黑树`的查找操作与`二叉查找树`一致(因为查找不会影响树的结构),而插入与删除操作需要在最后对树进行调整.

我们将新的节点的颜色设为红色(如果设为黑色会使根节点到叶子的一条路径上多了一个黑节点,违反了性质5,这个是很难调整的).

现在我们假设新节点为`N`,它的父节点为`P`(且`P`为`G`的左节点,如果为右节点则与其操作互为镜像),祖父节点为`G`,叔叔节点为`U`.插入一个节点会有以下种情况.

#### 情况1

**`N`位于根,它没有父节点与子节点,这时候只需要把它重新设置为黑色即可**,无需其他调整.

#### 情况2

**`P`的颜色为黑色**,这种情况下保持了性质4(`N`只有两个叶子节点,它们都为黑色)与性质5(`N`是一个红色节点,不会对其造成影响)的有效,所以**无需调整**.

#### 情况3

如果`P`与`U`都为红色,我们可以将它们两个重新绘制为黑色,然后将`G`绘制为红色(保持性质5),最后再从`G`开始继续向上进行调整.

![](https://upload.wikimedia.org/wikipedia/commons/c/c8/Red-black_tree_insert_case_3.png)

#### 情况4

**`P`为红色,`U`为黑色,且`N`为`P`的左子节点,这种情况下,我们需要在`G`处进行一次`右旋转`**,结果满足了性质4与性质5,因为通过这三个节点中任何一个的所有路径以前都通过祖父节点`G`，现在它们都通过以前的父节点`P`.

关于旋转操作,可以查看这篇文章[《Algorithms,4th Edition》读书笔记-红黑二叉查找树][2].

![](https://upload.wikimedia.org/wikipedia/commons/6/66/Red-black_tree_insert_case_5.png)

#### 情况5

`P`为红色,`U`为黑色,且`N`为`P`的右子节点,我们需要先在`P`处进行一次`左旋转`,这样就又回到了情况4.

![](https://upload.wikimedia.org/wikipedia/commons/5/56/Red-black_tree_insert_case_4.png)

#### 代码

```java
    private void fixAfterInsertion(Node x) {
        while (x != null && x != root && colorOf(parentOf(x)) == RED) {
            if (parentOf(x) == grandpaOf(x).left) {
                x = parentIsLeftNode(x);
            } else {
                x = parentIsRightNode(x);
            }
            fixSize(x);
        }
        setColor(root, BLACK);
    }
	
    private Node parentIsLeftNode(Node x) {
        Node xUncle = grandpaOf(x).right;
		// 情况3
        if (colorOf(xUncle) == RED) {
            x = uncleColorIsRed(x, xUncle);
        } else {
			// 情况5
            if (x == parentOf(x).right) {
                x = parentOf(x);
                rotateLeft(x);
            }
			// 情况4
            rotateRight(grandpaOf(x));
        }
        return x;
    }

    private Node parentIsRightNode(Node x) {
        Node xUncle = grandpaOf(x).left;

        if (colorOf(xUncle) == RED) {
            x = uncleColorIsRed(x, xUncle);
        } else {
            if (x == parentOf(x).left) {
                x = parentOf(x);
                rotateRight(x);
            }
            rotateLeft(grandpaOf(x));
        }
        return x;
    }

    private Node uncleColorIsRed(Node x, Node xUncle) {
        setColor(parentOf(x), BLACK);
        setColor(xUncle, BLACK);
        setColor(grandpaOf(x), RED);
        x = grandpaOf(x);
        return x;
    }	
```

### 删除

我们只考虑删除节点只有一个子节点的情况,且只有后继节点与删除节点都为黑色(如果删除节点为红色,从根节点到叶子节点的每条路径上少了一个红色节点并不会违反`红黑树`的性质,而如果后继节点为红色,只需要将它重新绘制为黑色即可).

先将删除节点替换为后继节点,且后继节点定义为`N`,它的兄弟节点为`S`.

#### 情况1

`N`为新的根节点,在这种情况下只需要把根节点保持为黑色即可.

#### 情况2

**`S`为红色,只需要在`P`进行一次`左旋转`**,接下来则**继续按以下情况进行处理**(尽管路径上的黑色节点数量没有改变,但`N`有了一个黑色的兄弟节点与红色的父节点).

![](https://upload.wikimedia.org/wikipedia/commons/3/39/Red-black_tree_delete_case_2.png)


#### 情况3

`S`和它的子节点都是黑色的,而`P`为红色.这种情况下只需要将`S`与`P`的颜色进行交换


![](https://upload.wikimedia.org/wikipedia/commons/d/d7/Red-black_tree_delete_case_4.png)

#### 情况4

**`S`和它的子节点都是黑色的,这种情况下需要把`S`重新绘制为红色**.这时不通过`N`的路径都将少一个黑色节点(通过`N`的路径因为删除节点是黑色的也都少了一个黑色节点),这让它们平衡了起来.

但现在通过`P`的路径比不通过`P`的路径都少了一个黑色节点,所以还需要在`P`上继续进行调整.

![](https://upload.wikimedia.org/wikipedia/commons/c/c7/Red-black_tree_delete_case_3.png)

#### 情况5

**`S`为黑色,它的左子节点为红色,右子节点为黑色.这种情况下,我们在`S`上做`右旋转`**,这样`S`的左儿子成为`S`的父亲和N的新兄弟。我们接着交换`S`和它的新父亲的颜色。所有路径仍有同样数目的黑色节点，但是现在`N`有了一个右儿子是红色的黑色兄弟，所以我们进入了情况6。`N`和`P`都不受这个变换的影响。

![](https://upload.wikimedia.org/wikipedia/commons/3/30/Red-black_tree_delete_case_5.png)

#### 情况6

**`S`是黑色，它的右子节点是红色,我们在`N`的父亲`P`上做`左旋转`**.这样`S`成为`N`的父亲和`S`的右儿子的父亲。我们接着交换`N`的父亲和`S`的颜色，**并使`S`的右儿子为黑色**。子树在它的根上的仍是同样的颜色,但是,`N`现在增加了一个黑色祖先.所以,通过`N`的路径都增加了一个黑色节点.此时,如果一个路径不通过`N`,则有两种可能性:

 - 它通过`N`的新兄弟.那么它以前和现在都必定通过`S`和`N`的父亲,而它们只是交换了颜色.所以路径保持了同样数目的黑色节点.


 - 它通过`N`的新叔父,`S`的右儿子.那么它以前通过`S`、`S`的父亲和`S`的右儿子,但是现在只通过`S`,它被假定为它以前的父亲的颜色,和`S`的右儿子,它被从红色改变为黑色.合成效果是这个路径通过了同样数目的黑色节点.

在任何情况下,在这些路径上的黑色节点数目都没有改变.所以我们恢复了性质4.在示意图中的白色节点可以是红色或黑色,但是在变换前后都必须指定相同的颜色.


![](https://upload.wikimedia.org/wikipedia/commons/3/31/Red-black_tree_delete_case_6.png)

#### 代码

```java
    private void fixAfterDeletion(Node x) {
        while (x != null && x != root && colorOf(x) == BLACK) {
            if (x == parentOf(x).left) {
                x = successorIsLeftNode(x);
            } else {
                x = successorIsRightNode(x);
            }
        }
        setColor(x, BLACK);
    }

    private Node successorIsLeftNode(Node x) {
        Node brother = parentOf(x).right;
		// 情况2
        if (colorOf(brother) == RED) {
            rotateLeft(parentOf(x));
            brother = parentOf(x).right;
        }
		// 情况3,4
        if (colorOf(brother.left) == BLACK && colorOf(brother.right) == BLACK) {
            x = brotherChildrenColorIsBlack(x, brother);
        } else {
			// 情况5
            if (colorOf(brother.right) == BLACK) {
                rotateRight(brother);
                brother = parentOf(x).right;
            }
			// 情况6
            setColor(brother.right, BLACK);
            rotateLeft(parentOf(x));
            x = root;
        }
        return x;
    }

    private Node brotherChildrenColorIsBlack(Node x, Node brother) {
        setColor(brother, RED);
        x = parentOf(x);
        return x;
    }
```

### 参考资料

 - [Wikipedia][4]



> 本文作者为[SylvanasSun(sylvanassun_xtz@163.com)][3],转载请务必指明原文链接.

[1]: https://gist.github.com/SylvanasSun/147672912cc5bc6da27e15528542877f
[2]: http://sylvanassun.github.io/2017/03/30/red_black_binary_search_tree/
[3]: https://github.com/SylvanasSun/
[4]: https://zh.wikipedia.org/wiki/%E7%BA%A2%E9%BB%91%E6%A0%91