---
layout:     post
title:       《Algorithms,4th Edition》读书笔记-红黑二叉查找树
subtitle:   "基于偏向红色左链接的红黑二叉查找树实现"
date:       2017-03-30 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - Algorithms
    - 数据结构
    - Tree
tags:
    - 数据结构
    - Algorithms
    - Tree
    - 2017
---


> `红黑二叉查找树`是`2-3查找树`的简单表示方式,它的代码量并不大,并且保证了平衡性.
> 阅读本文前需先了解 [《Algorithms,4th Edition》读书笔记-2-3查找树][1]

## 概述


----------


`红黑树`是一种自平衡的`二叉查找树`,它的基本思想是**用标准的`二叉查找树`(完全由`2-节点`构成)和一些额外的信息(替换`3-节点`)来表示`2-3树`.** 可以说`红黑树`是`2-3树`的一种等同.

`红黑树`中的链接可以分为两种类型: 

 - **红链接** : 它将两个`2-节点`连接起来构成一个`3-节点`(也可以说是将`3-节点`表示为由一条**红色左链接**(两个`2-节点`其中之一是另一个的左子节点)相连的两个`2-节点`).


 - **黑链接** : 表示`2-3树`中的普通链接.

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-encoding.png)

这种表示方式带来的优点如下: 

 1. 无需修改就可以直接使用标准的`二叉查找树`中的查找方法(其他与链接颜色不关联的方法也可以直接使用).


 2. 对于任意的`2-3树`,只要对节点进行转换,我们都可以立即派生出一棵对应的`二叉查找树`.

## 红黑树的性质


----------


`红黑树`是含有红黑链接并满足下列条件的`二叉查找树`(满足这些条件的`红黑树`才是与相应的`2-3树`一一对应的).

 - 红链接均为左链接(这条仅限于偏向左红链接实现的`红黑树`).


 - 每个节点不是红色就是黑色的.


 - 没有任何一个节点同时和两条红链接相连(不可以有两条连续的红链接).


 - 该树是完美黑色平衡的,即**任意空链接到根节点的路径上的黑链接数量相同.**


 - `根节点`是黑色的.


 - 所有`叶子节点`(即null节点)的颜色是黑色的.


## 与2-3树的对应关系


----------


假如我们将一棵`红黑树`中的红链接画平,我们会发现所有的空链接到根节点的距离都将是相同的.如果再把由红链接相连的节点合并,得到的就是一棵`2-3树`.

相对的,如果将一棵`2-3树`中的`3-节点`画作由红色左链接相连的两个`2-节点`,那么不会存在能够和两条红链接相连的节点,且树必然是完美黑色平衡的,因为黑链接就是`2-3树`中的普通链接,根据定义这些链接必然是完美平衡的.

通过这些结论,我们**可以发现`红黑树`即是`二叉查找树`,也是`2-3树`.**

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-1-1.png)

## 节点的实现


----------


我们使用`boolean`类型的变量`color`来表示链接的颜色.如果指向它的链接为红色,则`color`变量为`true`,黑色则为`false`(空链接也为黑色).

并且定义了一个`isRed()`函数用于判断链接的颜色.

这里节点的**颜色指的是指向该节点的链接的颜色.**

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-color.png)

```java
    private static final boolean RED = true;
    private static final boolean BLACK = false;

    private Node root; // root node

    private class Node {
        private K key;
        private V value;
        private Node left, right; // links to left and right subtress
        private boolean color; // color of parent link
        private int size; // subtree count

        public Node(K key, V value, boolean color, int size) {
            this.key = key;
            this.value = value;
            this.color = color;
            this.size = size;
        }
    }
	
	 // node x is red? if x is null return false.
    private boolean isRed(Node x) {
        if (x == null) return false;
        return x.color == RED;
    }
```

## 旋转


----------


当我们在实现某些操作时,可能会产生一些红色右链接或者两条连续的红色左链接.这时就需要在操作完成前进行旋转操作来修复`红黑树`的平衡性(**旋转操作会改变红链接的指向**).

旋转操作保证了`红黑树`的两个重要性质 : **有序性**和**完美平衡性**. 

#### 左旋转

假设当前有一条红色右链接需要被修正旋转为左链接.这个操作叫做`左旋转`.

`左旋转`函数接受一条指向`红黑树`中的某个节点的链接作为参数.然后**会对树进行必要的调整并返回一个指向包含同一组键的子树且其左链接为红色的根节点的链接.**

也可以认为是**将用两个键中的较小者作为根节点变为将较大者作为根节点**(右旋转中逻辑相反).

旋转操作返回的链接可能是左链接也可能是右链接,这个链接可能是红色也可能是黑色的(在实现中我们使用`x.color = h.color`保留了它原本的颜色).这**可能会产生两条连续的红链接,但算法会在后续操作中继续使用旋转操作修正这种情况.**

**旋转操作只影响了根节点**(返回的节点的子树中的所有键和旋转前都相同,只有根节点发生了变化).

具体的实现如下图: 

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-left-rotate.png)

#### 右旋转

实现`右旋转`的逻辑基本与`左旋转`相同,只需要将`left`和`right`互换即可.

![](http://algs4.cs.princeton.edu/33balanced/images/redblack-right-rotate.png)

## 颜色转换


----------


颜色转换操作也是用于保证`红黑树`的性质的.**它将`父节点`的颜色由黑变红,将`子节点`的颜色由红变黑.**

这项操作与旋转操作一样是局部变换,**不会影响整棵树的黑色平衡性.**

![](http://algs4.cs.princeton.edu/33balanced/images/color-flip.png)

#### 根节点总是为黑

颜色转换可能会使`根节点`变为红色,但红色的`根节点`说明`根节点`是一个`3-节点`的一部分,实际情况并不是这样的.所以我们需要将`根节点`设为黑色.

**每当`根节点`由红变黑时,树的黑链接高度就会加1.**

## 插入


----------


在`红黑树`中实现插入操作是比较复杂的,因为需要保持`红黑树`的平衡性.但只要利用好`左旋转`、`右旋转`、`颜色转换`这三个辅助操作,就能够保证插入操作后树的平衡性.

#### 向单个2-节点中插入新键

当一棵只含有一个键的`红黑树`只含有一个`2-节点`时,插入另一个键后需要马上进行`旋转`操作修正树的平衡性.

 - 如果新键小于老键,只需要新增一个红色的节点即可(这时,新的`红黑树`等价于一个`3-节点`).


 - 如果新键大于老键,那么新增的红色节点将会产生一条红色的右链接,这时就需要使用`左旋转`修正根节点的链接.


 - 以上两种情况最终的结果均为一棵等价于单个`3-节点`的`红黑树`,它含有两个键,一条红链接,树的黑链接高度为1.

#### 向树底部的2-节点插入新键

和`二叉查找树`一样,向`红黑树`中插入一个新键会在树的底部新增一个节点,但**在`红黑树`中总是用红链接将新节点和它的父节点相连.**

如果它的父节点是一个`2-节点`,那么上一节讨论的方法依然适用.

 - 如果指向新节点的是父节点的左链接,那么父节点就直接成为一个`3-节点`.


 - 如果指向新节点的是父节点的右链接,那么就需要一次`左旋转`进行修正.


#### 向一棵双键树(一个3-节点)中插入新键

当向一个`3-节点`中插入新键时,会发生以下三种情况且每种情况都会产生一个同时连接到两条红链接的节点,我们需要修正这一点.

 - 如果`新键大于原树中的两个键` : 这是最容易处理的一种情况,这个**键会被连接到`3-节点`的右链接**.此时树是平衡的,**根节点为中间大小的键**,它有**两条红链接分别和较小和较大的节点相连**.只需要**把这两条链接的颜色都由红变黑,那么就可以得到一棵由三个节点组成、高度为2的平衡树**(其他两种情况最终也会转化为这样的树).


 - 如果`新键小于原树中的两个键` : 这个**键会被连接到最左边的空链接,这样就产生了两条连续的红链接.**此时**只需要将上层的红链接`右旋转`即可得到第一种情况**(中值键为根节点并和其他两个节点用红链接相连).


 - 如果`新键介于原树中的两个键之间` : 这种情况依然**会产生两条连续的红链接:一条红色左链接接一条红色右链接.**此时**只需要将下层的红链接`左旋转`即可得到第二种情况**(两条连续的红色左链接).

通过以上这三种情况可以总结出 : 我们只需要通过0次、1次、2次旋转以及颜色转换就可以完成对`红黑树`的修正.

#### 将红链接向上传递

当每次旋转操作之后都会进行`颜色转换`,它会使得中间节点变为红色.**从父节点的角度来看,处理这样一个红色节点的方式和处理一个新插入的红色节点完全相同**(继续将红链接转移到中间节点).

这个操作对应于`2-3树`中向`3-节点`进行插入的操作 : 即在一个`3-节点`下插入新键,需要创建一个临时的`4-节点`,将其分解并将中间键插入父节点(在`红黑树`中,是将红链接由中间键传递给它的父节点).重复这个过程,直至遇到一个`2-节点`或者根节点.

当根节点变为红色时需要将根节点的颜色转换为黑色(对应`2-3树`中的根节点分解).


#### 实现

插入操作的实现除了每次递归调用之后的对平衡性修正的操作,其他与`二叉查找树`中的插入操作没什么不同.

```java
    public void put(K key, V val) {
        if (key == null) throw new IllegalArgumentException("first argument to put() is null");
        if (val == null) {
            delete(key);
            return;
        }

        root = put(root, key, val);
        root.color = BLACK;
        assert check();
    }

    // insert the key-value pair in the subtree rooted at h
    private Node put(Node h, K key, V val) {
        if (h == null) return new Node(key, val, RED, 1);

        int cmp = key.compareTo(h.key);
        if (cmp < 0) h.left = put(h.left, key, val);
        else if (cmp > 0) h.right = put(h.right, key, val);
        else h.value = val;

        // fix-up any right-leaning links
        if (isRed(h.right) && !isRed(h.left)) h = rotateLeft(h);
        if (isRed(h.left) && isRed(h.left.left)) h = rotateRight(h);
        if (isRed(h.left) && isRed(h.right)) flipColors(h);
        h.size = size(h.left) + size(h.right) + 1;

        return h;
    }
```

#### 总结

只要在沿着插入点到根节点的路径向上移动时**在所经过的每个节点中顺序完成以下操作**,就能够实现`红黑树`的插入操作.

 - 如果`右子节点`是红色的而`左子节点`是黑色的,那么进行`左旋转`.


 - 如果`左子节点`是红色的而且它的`左子节点`也是红色的,那么进行`右旋转`.


 - 如果`左右子节点`都是红色的,那么进行`颜色转换`.

## 删除


----------


删除操作也需要定义一系列`局部变换`来在**删除一个节点的同时保持树的完美平衡性**.然而,这个过程要比插入操作还要复杂,它**不仅要在(为了删除一个节点而)构造临时`4-节点`时沿着查找路径向下进行变换,还要在分解遗留的`4-节点`时沿着查找路径向上进行变换(同插入操作)**.

#### 自顶向下的2-3-4树

`2-3-4树`是一种允许存在`4-节点`的树.它的插入算法就是一种沿着查找路径既能向上也能向下进行变换的算法.

 - 沿查找路径向下进行变换(向下变换与`2-3树`中分解`4-节点`所进行的变换完全相同)是为了保证当前节点不是`4-节点`(这样树的底部才有足够的空间插入新的键).


 - 沿查找路径向上进行变换是为了将之前创建的`4-节点`配平.


 - 如果`根节点`是一个`4-节点`,就将它分解成三个`2-节点`,树的高度加1.


 - 如果在向下查找的过程中,遇到了一个`父节点`为`2-节点`的`4-节点`,就将`4-节点`分解为两个`2-节点`并将`中间键`传递给它的`父节点`(这时`父节点`变为了一个`3-节点`).


 - 如果遇到了一个`父节点`为`3-节点`的`4-节点`,将`4-节点`分解为两个`2-节点`并将`中间键`传递给它的`父节点`(这时`父节点`变为了一个`4-节点`).


 - 不必担心遇见`父节点`为`4-节点`的`4-节点`,算法本身保证了不会出现这种情况,到达树的底部之后,只会遇到`2-节点`或者`3-节点`.

如果要使用`红黑树`来实现这个算法,需要以下步骤 : 

 - 将`4-节点`表示为由三个`2-节点`组成的一棵平衡的子树,`根节点`和两个子节点都用红链接相连.


 - 在向下的过程中分解所有`4-节点`并进行`颜色转换`.


 - 在向上的过程中使用`旋转`将`4-节点`配平.

只需要将插入一节中的`put()`实现方法里的`flipColors`语句(及其if语句)移动到递归调用之前(null判断和比较操作之间)就能实现`2-3-4树`的插入操作.

#### 删除最小键

从`2-节点`中删除一个键会留下一个空节点,一般会将它替换为一个空链接,但这样会破坏树的完美平衡性.所以在删除操作中,**为了避免删除一个`2-节点`,我们沿着`左链接`向下进行变换时,需要确保当前节点不是`2-节点`**.

`根节点`可能有以下两种情况:

 1. 如果`根节点`是一个`2-节点`且它的两个子节点都是`2-节点`,可以直接将这三个节点变成一个`4-节点`.


 2. 否则,需要保证`根节点`的左子节点不是`2-节点`,必要时可以从它右侧的兄弟节点借走一个键.

在沿着`左链接`向下的过程中,保证以下情况之一成立: 

 - 如果当前节点的左子节点不是`2-节点`.


 - 如果当前节点的左子节点是`2-节点`而它的兄弟节点不是`2-节点`,将左子节点的兄弟节点中的一个键移动到左子节点中


 - 如果当前节点的左子节点和它的兄弟节点都是`2-节点`,将左子节点、父节点中的最小键和左子节点最近的兄弟节点合并为一个`4-节点`,使父节点由`3-节点`变为`2-节点`(或是从`4-节点`变为`3-节点`).

只要保证了以上的条件,我们最终能够得到一个含有最小键的`3-节点`或`4-节点`(然后进行删除即可),之后再不断向上分解所有临时的`4-节点`.

##### 代码实现

在删除操作中,`颜色转换`的操作与插入操作中的实现略微有些不同(需要将父节点设为黑,而将两个子节点设为红).

```java
	private void flipColors(Node h) {
        h.color = !h.color;
        h.left.color = !h.left.color;
        h.right.color = !h.right.color;
    }

    // restore red-black tree invariant
    private Node balance(Node h) {
        if (isRed(h.right)) h = rotateLeft(h);
        if (isRed(h.left) && isRed(h.left.left)) h = rotateRight(h);
        if (isRed(h.left) && isRed(h.right)) flipColors(h);

        h.size = size(h.left) + size(h.right) + 1;
        return h;
    }
	
    // Assuming that h is red and both h.left and h.left.left
    // are black, make h.left or one of its children red.
    private Node moveRedLeft(Node h) {
        flipColors(h);
        if (isRed(h.right.left)) {
            h.right = rotateRight(h.right);
            h = rotateLeft(h);
            flipColors(h);
        }
        return h;
    }
	
    public void deleteMin() {
        if (isEmpty()) throw new NoSuchElementException("RedBlackBST underflow.");

        // if both children of root are black, set root to red
        if (!isRed(root.left) && !isRed(root.right))
            root.color = RED;

        root = deleteMin(root);
        if (!isEmpty())
            root.color = BLACK;
    }

    // delete the key-value pair with the minimum key rooted at h
    private Node deleteMin(Node h) {
        if (h.left == null)
            return null;

        if (!isRed(h.left) && !isRed(h.left.left))
            h = moveRedLeft(h);

        h.left = deleteMin(h.left);
        return balance(h);
    }	
```

#### 删除最大键

```java
    // Assuming that h is red and both h.right and h.right.left
    // are black, make h.right or one of its children red.
    private Node moveRedRight(Node h) {
        flipColors(h);
        if (isRed(h.left.left)) {
            h = rotateRight(h);
            flipColors(h);
        }
        return h;
    }
	
    public void deleteMax() {
        if (isEmpty()) throw new NoSuchElementException("RedBlackBST underflow.");

        // if both children of root are black, set root to red
        if (!isRed(root.left) && !isRed(root.right))
            root.color = RED;

        root = deleteMax(root);
        if (!isEmpty())
            root.color = BLACK;
    }

    // delete the key-value pair with the maximum key rooted at h
    private Node deleteMax(Node h) {
        if (isRed(h.left))
            h = rotateRight(h);

        if (h.right == null)
            return null;

        if (!isRed(h.right) && !isRed(h.right.left))
            h = moveRedRight(h);

        h.right = deleteMax(h.right);

        return balance(h);
    }	
```

#### 删除操作

同样也需要像删除最小键那样在查找路径上进行变换来保证查找过程中任意当前节点均不是`2-节点`.如果目标键在树的底部,可以直接删除它;如果不在,则需要将它和它的后继节点交换.

在删除操作之后需要向上变换分解余下的`4-节点`.

```java
    public void delete(K key) {
        if (key == null) throw new IllegalArgumentException("called delete() with key is null.");
        if (!contains(key)) return;

        // if both children of root are black, set root to red
        if (!isRed(root.left) && !isRed(root.right))
            root.color = RED;

        root = delete(root, key);
        if (!isEmpty()) root.color = BLACK;
    }

    // delete the key-value pair with the given key rooted at h
    private Node delete(Node h, K key) {
        if (key.compareTo(h.key) < 0) {
            if (!isRed(h.left) && !isRed(h.left.left))
                h = moveRedLeft(h);
            h.left = delete(h.left, key);
        } else {
            if (isRed(h.left))
                h = rotateRight(h);
            if (key.compareTo(h.key) == 0 && (h.right == null))
                return null;
            if (!isRed(h.right) && !isRed(h.right.left))
                h = moveRedRight(h);
            if (key.compareTo(h.key) == 0) {
                Node x = min(h.right);
                h.key = x.key;
                h.value = x.value;
                h.right = deleteMin(h.right);
            } else h.right = delete(h.right, key);
        }
        return balance(h);
    }
```

## 总结


----------


无论键的插入顺序如何,`红黑树`都几乎是完美平衡的,基于它实现的有序符号表操作的运行时间均为对数级别(除了范围查询).

在`红黑树`的实现中复杂的代码仅限于`put()`和`delete()`方法,像`get()`这些不会涉及检查颜色的方法与`二叉查找树`中的实现一致(因为这些操作与平衡性无关).

## end


----------


 - Author : [SylvanasSun][2]


 - Email : sylvanassun_xtz@163.com


 - 文中的完整实现代码见我的[GitHub][3] & [Gist][4]


 - 本文参考资料引用自[《Algorithms,4th Editio》][5]

[1]: http://sylvanassun.github.io/2017/03/28/2_3tree/
[2]: https://github.com/SylvanasSun
[3]: https://github.com/SylvanasSun/algs4-study
[4]: https://gist.github.com/SylvanasSun/731a1438c61492628cfaa1e9e618ecfb
[5]: http://algs4.cs.princeton.edu/33balanced/
