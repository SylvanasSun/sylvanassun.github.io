---
layout:     post
title:      《Algorithms,4th Edition》读书笔记-二叉查找树
subtitle:   "使用二叉查找树实现符号表的一些操作实现"
date:       2017-03-26 18:00
author:     "Sylvanas Sun"
header-img: "img/post-bg-universe.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 数据结构
    - 符号表
    - 2017
---



### 概述


----------


二叉查找树是一颗**有序的二叉树**,它有以下性质:

 1. 若任意节点的左子树不为空,则**左子树上所有节点的值均小于它的根节点的值**.


 2. 若任意节点的右子树不为空,则**右子树上所有节点的值均大于它的根节点的值**.


 3. 可以将**每个链接看做指向另一颗二叉查找树**,而这棵树的根节点就是被指向的节点.


 4. 没有键值相等的节点.

![](http://algs4.cs.princeton.edu/32bst/images/binary-tree-anatomy.png) 

![](http://algs4.cs.princeton.edu/32bst/images/bst-anatomy.png)

使用二叉查找树实现的符号表结合了**链表插入的灵活性和有序数组查找的高效性**.通常采取二叉链表作为存储结构.

每个节点都含有一个键、一个值、一条左链接、一条右链接和一个节点计数器(用于统计其所有子节点数).**左链接指向一棵由小于该节点的所有键组成的二叉查找树,右链接指向一棵由大于该节点的所有键组成的二叉查找树.**

如果将一棵二叉查找树的所有键投影到一条直线上,保证一个节点的**左子树中的键出现在它的左边**,**右子树种的键出现在它的右边**,那么我们一定**可以得到一条有序的键列**.

可以说二叉查找树和快速排序很相似.树的根节点就是快速排序中的第一个`基准数(切分元素)`,左侧的键都比它小,右侧的键都比它大.

### 基本实现


----------


```java
public class BinarySearchTree<K extends Comparable<K>, V> {

    private Node root; // root node

    private class Node {
        private K key;
        private V value;
        private Node left, right; // left and right subtree
        private int size; // number of nodes in subtree

        public Node(K key, V value, int size) {
            this.key = key;
            this.value = value;
            this.size = size;
        }
    }
	
	/**
     * Returns true if this symbol table is empty.
     *
     * @return {@code true} is this symbol table is empty, {@code false} otherwise.
     */
    public boolean isEmpty() {
        return size() == 0;
    }

    /**
     * Returns the number of key-value pairs in this symbol table.
     *
     * @return the number of key-value pairs in this symbol table.
     */
    public int size() {
        return size(root);
    }
	
	// return number of key-value pairs in binary search tree rooted at x
    private int size(Node x) {
        if (x == null) return 0;
        else return x.size;
    }
}	
```

在以上代码中,使用私有嵌套类`Node`来表示一个二叉链表,每个`Node`对象都是一棵含有N个节点的子树的根节点.变量`root`指向二叉查找树的根节点(这棵树包含了符号表中的所有键值对).

### 查找与插入


----------


#### 查找

```java
 /**
     * Returns the value associated with the given key.
     *
     * @param key the key
     * @return the value associated with the given key if the key is in the symbol table
     * and {@code null} if the key is not in the symbol table.
     * @throws IllegalArgumentException if {@code key} is {@code null}
     */
    public V get(K key) {
        if (key == null)
            throw new IllegalArgumentException("called get() with a null key.");
        return get(root, key);
    }


    private V get(Node x, K key) {
        if (x == null) return null;
        int cmp = key.compareTo(x.key);
        if (cmp < 0) {
            // if key < x.key , search left subtree
            return get(x.left, key);
        } else if (cmp > 0) {
            // if key > x.key , search right subtree
            return get(x.right, key);
        } else {
            // hit target
            return x.value;
        }
    }
```

在二叉查找树中实现查找操作是很简单而简洁的,这也是二叉查找树的特性之一.-**如果树为空,就返回null,如果被查找的键小于根节点的键,我们就继续在左子树中查找,否则在右子树中查找.**

#### 插入

插入操作的逻辑与查找差不多,只不过需要在**判定树为空时,返回一个含有该键值对的新节点,还需要重置搜索路径上每个父节点指向子节点的链接,并增加路径上每个节点中的子节点计数器的值**.

![](http://algs4.cs.princeton.edu/32bst/images/bst-insert.png)

```java
/**
     * Inserts the specified key-value pair into  the symbol table.
     * overwriting the old value with the new value if the symbol table already contains
     * the specified key.
     * Deletes the specified key (and its associated value) from this symbol table
     * if the specified value is {@code null}.
     *
     * @param key   the key
     * @param value the value
     * @throws IllegalArgumentException if {@code key} is {@code null}.
     */
    public void put(K key, V value) {
        if (key == null)
            throw new IllegalArgumentException("called put() with a null key.");
        if (value == null) {
            delete(key);
            return;
        }
        root = put(root, key, value);
        assert check();
    }

    private Node put(Node x, K key, V value) {
        // if tree is empty, return a new node.
        if (x == null) return new Node(key, value, 1);

        int cmp = key.compareTo(x.key);
        if (cmp < 0) {
            x.left = put(x.left, key, value);
        } else if (cmp > 0) {
            x.right = put(x.right, key, value);
        } else {
            // hit target,overwriting old value with the new value
            x.value = value;
        }
        x.size = 1 + size(x.left) + size(x.right); // compute subtree node size
        return x;
    }
```

### 最大键和最小键


----------


 - 如果根节点的**左链接为空**,那么一棵二叉查找树中**最小的键就是根节点**.


 - 如果**左链接非空**,那么树中的**最小键就是左子树中的最小键**.找出最大键的逻辑也是类似的,只是变为查找右子树而已.

```java
   /**
     * Returns the smallest key in the symbol table.
     *
     * @return the smallest key in the symbol table.
     * @throws NoSuchElementException if the symbol table is empty
     */
    public K min() {
        if (isEmpty())
            throw new NoSuchElementException("called min() with empty symbol table.");
        return min(root).key;
    }

    private Node min(Node x) {
        if (x.left == null) return x;
        else return min(x.left);
    }

    /**
     * Returns the largest key in the symbol table.
     *
     * @return the largest key in the symbol table.
     * @throws NoSuchElementException if the symbol table is empty
     */
    public K max() {
        if (isEmpty())
            throw new NoSuchElementException("called max() with empty symbol table.");
        return max(root).key;
    }

    private Node max(Node x) {
        if (x.right == null) return x;
        else return max(x.right);
    }
```

### 向上取整和向下取整


----------


![向下取整的路径轨迹](http://algs4.cs.princeton.edu/32bst/images/bst-floor.png)


 - 如果给定的键**key小于二叉查找树的根节点的键**,那么小于等于key的最大键`floor(key)`**一定在根节点的左子树中**.


 - 如果给定的键**key大于二叉查找树的根节点**,那么**只有当根节点右子树中存在小于等于key的节点时,小于等于key的最大键才会出现在右子树中,否则根节点就是小于等于key的最大键**.(将左变为右,小于变为大于就是向上取整的实现逻辑)

```java
	public K floor(K key) {
        if (key == null)
            throw new IllegalArgumentException("called floor() with a null key.");
        if (isEmpty())
            throw new NoSuchElementException("called floor() with empty symbol table.");
        Node x = floor(root, key);
        if (x == null) return null;
        else return x.key;
    }

    private Node floor(Node x, K key) {
        if (x == null) return null;

        int cmp = key.compareTo(x.key);
        if (cmp == 0) return x;
        if (cmp < 0) return floor(x.left, key);
        Node t = floor(x.right, key);
        if (t != null) return t;
        else return x;
    }
	
	public K ceiling(K key) {
        if (key == null)
            throw new IllegalArgumentException("called ceiling() with a null key.");
        if (isEmpty())
            throw new NoSuchElementException("called ceiling() with empty symbol table.");
        Node x = ceiling(root, key);
        if (x == null) return null;
        else return x.key;
    }

    private Node ceiling(Node x, K key) {
        if (x == null) return null;

        int cmp = key.compareTo(x.key);
        if (cmp == 0) return x;
        if (cmp < 0) {
            Node t = ceiling(x.left, key);
            if (t != null) return t;
            else return x;
        }
        return ceiling(x.right, key);
    }
```

### 选择和排名


----------


#### select

假设我们想找到排名为`k`的键(即树中正好有k个小于它的键).

 - 如果**左子树中的节点数`t`大于`k`**,那么我们就继续递归地**在左子树中查找排名为`k`的键**.


 - **如果`t`等于`k`**,我们就**返回根节点中的键**.


 - **如果`t`小于`k`**,我们就递归地**在右子树中查找排名为`(k-t-1)`的键**.

![选择操作路径轨迹](http://algs4.cs.princeton.edu/32bst/images/bst-select.png)

```java
    public K select(int k) {
        if (k < 0 || k >= size())
            throw new IllegalArgumentException("called select() with invalid argument: " + k);
        return select(root, k).key;
    }

    private Node select(Node x, int k) {
        if (x == null) return null;
        int t = size(x.left);
        // if left subtree node size greater than k,in left subtree search
        if (t > k) return select(x.left, k);
            // otherwise,in right subtree search
        else if (t < k) return select(x.right, k - t - 1);
        else return x;
    }
```

#### rank

`rank()`函数是`select()`函数的逆函数,它会返回指定键的排名.

 - 如果给定的**键和根节点的键相等**,就**返回左子树中的节点总数t**.


 - 如果给定的**键小于根节点**,就**返回该键在左子树中的排名**(递归计算).


 - 如果给定的**键大于根节点**,就**返回`t+1(根节点)`加上它在右子树中的排名**(递归计算).

```java
    public int rank(K key) {
        if (key == null)
            throw new IllegalArgumentException("called rank() with a null key.");
        return rank(root, key);
    }


    private int rank(Node x, K key) {
        if (x == null) return 0;
        int cmp = key.compareTo(x.key);
        if (cmp < 0)
            return rank(x.left, key);
        else if (cmp > 0)
            return 1 + size(x.left) + rank(x.right, key);
        else
            return size(x.left);
    }
```

### 删除最大键和删除最小键


----------


对于删除最小键,需要不断深入根节点的左子树**直至遇见一个空链接**,然后将指向该节点的链接指向该节点的右子树(只需要在递归调用中返回它的右链接即可).此时已经没有任何链接指向要被删除的节点,因此它会被垃圾回收器gc掉.

删除最大键与其逻辑相似,只是方向相反.

```java
    /**
     * Removes the smallest key and associated value from the symbol table.
     *
     * @throws NoSuchElementException if the symbol table is empty.
     */
    public void deleteMin() {
        if (isEmpty())
            throw new NoSuchElementException("Symbol table underflow.");
        root = deleteMin(root);
        assert check();
    }

    private Node deleteMin(Node x) {
        // if the left link is empty,will link to the node of the right subtree.
        if (x.left == null) return x.right;

        x.left = deleteMin(x.left);
        x.size = size(x.left) + size(x.right) + 1;
        return x;
    }

    /**
     * Removes the largest key and associated value from the symbol table.
     *
     * @throws NoSuchElementException if the symbol table is empty
     */
    public void deleteMax() {
        if (isEmpty())
            throw new NoSuchElementException("Symbol table underflow.");
        root = deleteMax(root);
        assert check();
    }

    private Node deleteMax(Node x) {
        // if the right link is empty,will link to the node of the left subtree.
        if (x.right == null) return x.left;

        x.right = deleteMax(x.right);
        x.size = size(x.left) + size(x.right) + 1;
        return x;
    }
```

### 删除


----------


删除操作是二叉查找树中较为复杂的操作,假设我们要删除节点`x`(它是一个拥有两个子节点的节点),基本的实现逻辑如下.

 - 在删除节点`x`后用它的后继节点填补它的位置.
 
 - 找出`x`的**右子树中的最小节点**(这样替换仍能保证树的有序性,因为`x.key`和它的后继节点的键之间不存在其他的键)做为`x`的后继节点.


 - 将由此节点到根节点的路径上的所有节点的计数器减1(这里计数器的值仍然会被设为其所有子树中的节点总数加1).

![](http://algs4.cs.princeton.edu/32bst/images/bst-delete.png)

具体的过程如以下例子:

 1. 将指向即将被删除的节点的链接保存为`t`.


 2. 将`x`指向它的后继节点`min(t.right)`.


 3. 将`x`的右链接(原本指向一棵所有节点都大于`x.key`的二叉查找树)指向`deleteMin(t.right)`,也就是在删除后所有的节点仍然都大于`x.key`的子二叉查找树.


 4. 将`x`的左链接(本为空)指向`t.left`(其下所有的键都小于被删除的节点和它的后继节点).

以上的实现逻辑有一个缺点,即是**在某些实际应用场景下会产生性能问题**,主要原因**在于后继节点是一个随意的决定,且没有考虑树的对称性**.

```java
    public void delete(K key) {
        if (key == null)
            throw new IllegalArgumentException("called delete() with a null key.");
        root = delete(root, key);
        assert check();
    }

    private Node delete(Node x, K key) {
        if (x == null) return null;

        int cmp = key.compareTo(x.key);
        if (cmp < 0)
            x.left = delete(x.left, key);
        else if (cmp > 0)
            x.right = delete(x.right, key);
        else {
            if (x.right == null)
                return x.left;
            if (x.left == null)
                return x.right;
            Node t = x;
            x = min(t.right);
            x.right = deleteMin(t.right);
            x.left = t.left;
        }
        x.size = size(x.left) + size(x.right) + 1;
        return x;
    }
```

### 范围查找


----------


实现一个范围查找的思路可以是:**将所有落在给定范围以内的键放入一个队列`Queue`并跳过那些不可能含有所查找键的子树**.

```java
    public Iterable<K> keys(K lo, K hi) {
        if (lo == null)
            throw new IllegalArgumentException("called keys(lo,hi) first argument is null.");
        if (hi == null)
            throw new IllegalArgumentException("called keys(lo,hi) second argument is null.");

        Queue<K> queue = new Queue<K>();
        keys(root, queue, lo, hi);
        return queue;
    }

    private void keys(Node x, Queue<K> queue, K lo, K hi) {
        if (x == null) return;
        int cmp_lo = lo.compareTo(x.key);
        int cmp_hi = hi.compareTo(x.key);
        if (cmp_lo < 0)
            keys(x.left, queue, lo, hi);
        if (cmp_lo <= 0 && cmp_hi >= 0)
            queue.enqueue(x.key);
        if (cmp_hi > 0)
            keys(x.right, queue, lo, hi);
    }
```

### 总结


----------


二叉查找树的高度决定了它在最坏情况下的运行效率,但由于键的插入顺序不会是永远随机的,所以树的某一端高度可能会非常深,解决这个问题可以使用平衡二叉查找树,它能保证无论键的插入顺序如何,树的高度都将是总键数的对数.

本文中的实现皆采用递归的方式是为了提高可读性,二叉查找树可以使用非递归的方式实现且效率会更高.

二叉查找树结合了链表插入操作的灵活性和有序数组查找操作的高效性,且还有很多种优化的改进方案(例如平衡二叉查找树),总体来说二叉查找树是一种比较好的动态查找方法.

### end


----------


- Author : [SylvanasSun][1]


 - Email : sylvanassun_xtz@163.com


 - 文中的完整实现代码见我的[GitHub][5] & [Gist][6]


 - 本文参考资料引用自[<<Algorithms,4th Edition>>][3] & [WikiPedia][4]


  [1]: https://github.com/SylvanasSun
  [3]: http://algs4.cs.princeton.edu/32bst/
  [4]: https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%85%83%E6%90%9C%E5%B0%8B%E6%A8%B9
  [5]: https://github.com/SylvanasSun/algs4-study
  [6]: https://gist.github.com/SylvanasSun/c4bddf50d94148470c0836af64449e8b
