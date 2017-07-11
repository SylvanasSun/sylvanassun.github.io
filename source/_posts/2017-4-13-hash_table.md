---
layout:     post
title:         《Algorithms,4th Edition》读书笔记-散列表
subtitle:   "介绍散列表与如何处理碰撞冲突"
date:       2017-04-13 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - Algorithms
    - 数据结构
    - HashTable
tags:
    - 数据结构
    - HashTable
    - Algorithms
    - 2017
---


### 概述


----------



`散列表`(`Hash Table`,也叫`哈希表`),它是根据键而直接访问在内存存储位置的数据结构.也可以说是用一个数组来实现的**无序的符号表**,将键作为数组的索引而数组中键`i`处存储的就是它对应的值.

`散列表`通过`散列函数`将键转化为数组的索引来访问数组中的键值对.

在`散列表`的算法中,最重要的两个操作如下.

 1. 使用`散列函数`将被查找的键转化为数组的一个索引.


 2. 处理散列表中的`碰撞冲突`问题.

![](http://algs4.cs.princeton.edu/34hash/images/hashing-crux.png)

### 性质


----------



 - 若关键字为`k`,则其值存放于<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/c36f16f5357aeb5b0fa2fe3040e74282d62f8881">的存储位置上.由此,不需要比较便可直接取得所查记录.称这个对应关系<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/132e57acb643253e7810ee9702d9581f159a1c61">为`散列函数`,按照这个思想建立的符合表为`散列表`.


 - 对不同的键可能会得到同一个`散列地址`,即<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/f2b910a452063a4769272110d8d22cab053d433d">,而<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/fa1d43b27a17bf57baf12626ad7cfbf8ee9bb96d">,这种现象被称为`碰撞冲突`.具有相同函数值的键对该`散列函数`来说称做同义词.综上所述,根据散列函数<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/c36f16f5357aeb5b0fa2fe3040e74282d62f8881">和处理`碰撞冲突`的方法将一组键映射到一个有限的连续的地址集(区间)上,这种表称为`散列表`,这一映射过程称为`散列`,所得的存储位置称为`散列地址`.


 - 若对于键集合中的任一个键,经`散列函数`映射到地址集合中任何一个地址的概率是相等的,则这个`散列函数`被称为`均匀散列函数`,它可以减少`碰撞冲突`.

### 散列函数


----------


`散列函数`用于将键转化为数组的索引.如果我们有一个能够保存M个键值对的数组,那么我们就需要一个能够将任意键转化为该数组范围内的索引([0,M-1]范围内的整数)的`散列函数`

`散列函数`与键的类型有关,对于每种类型的键都需要一个与之对应的`散列函数`.

#### 实现散列函数的几种方法

 - 直接定址法 : 取`key`或者`key`的某个线性函数值为`散列地址`.即<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/989ebc7db55ece5d29e2a8baa005e876ef486e4e">或<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/989ebc7db55ece5d29e2a8baa005e876ef486e4e">其中a,b为常数(这种`散列函数`叫做自身函数).


 - 数字分析法 : 假设`key`是以`r`为基的数,并且`散列表`中可能出现的`key`都是事先知道的,则可取`key`的若干数位组成`散列地址`.


 - 平方取中法 : 取`key`平方后的中间几位为`散列地址`.通常在选定`散列函数`时不一定能知道`key`的全部情况,取其中的哪几位也不一定合适,而一个数平方后的中间几位数和数的每一位都相关,由此使随机分布的`key`得到的`散列地址`也是随机的.取的位数由表长决定.


 - 折叠法 : 将`key`分割成位数相同的几部分(最后一部分的位数可以不同),然后取这几部分的叠加和(舍去进位)作为`散列地址`.


 - 除留余数法 : 取`key`被某个不大于`散列表`长度`m`的数`p`除后所得的余数为`散列地址`.即<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/bc04a0c2f72156976761fa24dd4ba098855b7dca">,<img src="https://wikimedia.org/api/rest_v1/media/math/render/svg/3aad2b022083cbc8aef0745526f3a448e7d96160">.不仅可以对`key`直接取模，也可在`折叠法`、`平方取中法`等运算之后取模。对`p`的选择很重要，一般取`素数`或`m`，若`p`选择不好，容易产生`碰撞冲突`.


#### 正整数

将正整数`散列`一般使用的是`除留余数法`.我们选择大小为**素数**`M`的数组,对于任意正整数`k`,计算`k`除以`M`的余数(即`k%M`).它能够有效地将`key`散布在0到M-1的范围内.

如果`M`不是**素数**,可能无法利用`key`中包含的所有信息,这**可能导致无法均匀地散列散列值**.

![](http://algs4.cs.princeton.edu/34hash/images/modular-hashing.png)

#### 浮点数

对浮点数进行散列一般是将`key`表示为二进制数然后再使用`除留余数法`.

#### 字符串

`除留余数法`也可以处理较长的`key`,例如字符串,我们只需将它们当成大整数即可.

```java
	int hash = 0;
	
	for (int i = 0; i < s.length(); i++) {
    	hash = (R * hash + s.charAt(i)) % M;
	}	
```

Java的`charAt()`函数能够返回一个char值,即一个非负16位整数.如果`R`比任何字符的值都大,这种计算相当于将字符串当作一个N位的`R`进制值,将它除以`M`并取余.只要`R`足够小,不造成溢出,那么结果就能够落在0至M-1之间.可以使用一个较小的素数,例如31.

#### 组合键

如果`key`的类型含有多个整型变量,我们可以和字符串类型一样将它们混合起来.

例如,`key`的类型为Date,其中含有几个整型的域 : day(两个数字表示的日),month(两个数字表示的月),year(四个数字表示的年).我们可以这样计算它的散列值: 

```java
int hash = (((day * R + month) % M) * R + year) % M; 
```

#### Java中的约定

在Java中如果要为自定义的数据类型定义散列函数,需要同时重写`hashCode()`和`equals()`两个函数,并要遵守以下规则.

 - `hashCode()`与`equals()`的结果必须保持一致性.即`a.equals(b)`返回true,则`a.hashCode()`的返回值也必然和`b.hashCode()`的返回值相同.


 - 但如果两个对象的`hashCode()`函数的返回值相同,这两个对象也有可能不同,还需要用`equals()`函数进行判断.

一个使用`除留余数法`的简单`散列函数`如下,它会将符号位屏蔽(将一个32位整数变为一个31位非负整数).

```java
private int hash(Key key) {
   return (key.hashCode() & 0x7fffffff) % M;
}
```

#### 软缓存

由于`散列函数`的计算有可能会很耗时,我们可以进行缓存优化,将每个`key`的散列值缓存起来(可以在每个`key`中使用一个hash变量来保存它的`hashCode()`的返回值).

当第一次调用`hashCode()`时,需要计算对象的散列值,但之后对`hashCode()`方法的调用会直接返回hash变量的值.

#### 总结

总之,要想实现一个优秀的`散列函数`需要满足以下的条件.

 1. 一致性,等价的`key`必然产生相等的散列值.


 2. .高效性,计算简便.


 3. 均匀性,均匀地散列所有的`key`.

### 基于拉链法的散列表


----------


拉链法是解决`碰撞冲突`的一种策略,它的核心思想是 : 将大小为`M`的**数组中的每个元素指向一条链表**,链表中的每个节点都存储了散列值为该元素的索引的键值对.

拉链法的实现一般分为以下两种: 

 1. 使用一个原始的链表数据类型来表示数组中的每个元素.


 2. 使用一个符号表实现来表示数组中的每个元素(这个方法实现简单但效率偏低).

![](http://algs4.cs.princeton.edu/34hash/images/separate-chaining.png)

```java
public class SeparateChainingHashST<K, V> {

    private static final int INIT_CAPACITY = 4;

    private int n; // the number of key-value pairs in the symbol table
    private int m; // the number of size of separate chaining table
    private Node<K, V>[] table; // array of linked-list symbol tables

    private class Node<K, V> {
        private K key;
        private V value;
        private Node<K,V> next;

        public Node() {

        }

        public Node(K key, V value, Node next) {
            this.key = key;
            this.value = value;
            this.next = next;
        }
    }
	
	private int hash(K key) {
        return ((key.hashCode()) & 0x7fffffff) % m;
    }
}	
```

#### 查找、插入、删除

基于拉链法的`散列表`的查找、插入、删除算法基本分为两步:

 1. 首先根据散列值找到对应的链表.


 2. 然后沿着这条链表进行相应的操作.

```java
    public V get(K key) {
        if (key == null)
            throw new IllegalArgumentException("called get() with key is null.");
        int i = hash(key);
        Node x = table[i];
        while (x != null) {
            if (key.equals(x.key))
                return (V) x.value;
            x = x.next;
        }
        return null;
    }
	
    public void put(K key, V value) {
        if (key == null)
            throw new IllegalArgumentException("called put() with key is null.");

        if (value == null) {
            remove(key);
            return;
        }

        // double table size if average length of list >= 10
        if (n >= 10 * m)
            resize(2 * m);
        int i = hash(key);
        Node x = table[i];
        Node p = null;
        while (x != null) {
            if (key.equals(x.key)) {
                x.value = value;
                return;
            }
            p = x;
            x = x.next;
        }
        if (p == null) {
            table[i] = new Node(key, value, null);
            n++;
        } else {
            p.next = new Node(key, value, null);
            n++;
        }
    }
	
    public V remove(K key) {
        if (key == null)
            throw new IllegalArgumentException("called remove() with key is null.");
        if (isEmpty())
            throw new NoSuchElementException("called remove() with empty symbol table.");

        if (!contains(key))
            return null;
        int i = hash(key);
        Node x = table[i];
        Node p = null;
        V oldValue = null;
        while (x != null) {
            if (key.equals(x.key)) {
                oldValue = (V) x.value;
                if (p == null) {
                    table[i] = x.next;
                } else {
                    p.next = x.next;
                }
                n--;
                break;
            }
            p = x;
            x = x.next;
        }

        // halve table size if average length of list <= 2
        if (m > INIT_CAPACITY && n <= 2 * m)
            resize(m / 2);
        return oldValue;
    }	
```

### 基于线性探测法的散列表


----------


解决`碰撞冲突`的另一种策略是使用线性探测法.它的核心思想是: 使用大小为`M`的数组保存`N`个键值对,其中`M>N`.这种方法**需要依靠数组中的空位来解决`碰撞冲突`**,基于这种策略的所有方法被统称为`开放地址散列表`.

`开放地址散列表`中最简单的方法就是线性探测法: 当发生`碰撞冲突`时,我们直接检查`散列表`中的下一个位置(将索引值加1).它可能会产生三种结果: 

 1. 命中,该位置的`key`和被查找的`key`相同.


 2. 未命中,`key`为空(该位置没有`key`).


 3. 继续查找,该位置的`key`和被查找的`key`不同.

我们使用`散列函数`找到`key`在数组中的索引,检查其中的`key`和被查找的`key`是否相同.如果不同则继续查找(将索引值加1,到达数组结尾时折回数组的开头),直到找到该`key`或者遇到一个空元素.

`开放地址散列表`的核心思想是: 与其将内存用作链表,不如将它们作为在`散列表`的空元素(这些空元素可以作为查找结束的标识).

![](http://algs4.cs.princeton.edu/34hash/images/linear-probing.png)

```java
public class LinearProbingHashST<K, V> {

    private static final int INIT_CAPACITY = 4;

    private int n; // the number of key-value pairs in the symbol table
    private int m; // the number of size of linear probing table
    private K[] keys; // the keys
    private V[] vals; // the values

    /**
     * Initializes an empty symbol table.
     */
    public LinearProbingHashST() {
        this(INIT_CAPACITY);
    }

    /**
     * Initializes an empty symbol table with the specified initial capacity.
     *
     * @param capacity the initial capacity
     */
    public LinearProbingHashST(int capacity) {
        m = capacity;
        n = 0;
        keys = (K[]) new Object[m];
        vals = (V[]) new Object[m];
    }
	
    public V get(K key) {
        if (key == null)
            throw new IllegalArgumentException("called get() with key is null.");
        for (int i = hash(key); keys[i] != null; i = (i + 1) % m) {
            if (keys[i].equals(key))
                return vals[i];
        }
        return null;
    }
	
    public void put(K key, V value) {
        if (key == null)
            throw new IllegalArgumentException("called put() with key is null.");
        if (value == null) {
            delete(key);
            return;
        }

        // double table size if 50% full
        if (n >= m / 2) resize(2 * m);

        int i;
        for (i = hash(key); keys[i] != null; i = (i + 1) % m) {
            if (keys[i].equals(key)) {
                vals[i] = value;
                return;
            }
        }
        keys[i] = key;
        vals[i] = value;
        n++;
    }	
}	
```

#### 删除

基于线性探测法的`散列表`的删除操作较为复杂,我们不能直接将`key`所在的位置设为`null`,这样会使在此位置之后的元素无法被查找到.

因此,我们需要**将被删除键的右侧的所有键重新插入到`散列表`中**.

```java
    public V delete(K key) {
        if (key == null)
            throw new IllegalArgumentException("called delete() with key is null.");
        if (isEmpty())
            throw new NoSuchElementException("called delete() with empty symbol table.");

        if (!contains(key))
            return null;
        // find position i of key
        int i = hash(key);
        while (!key.equals(keys[i])) {
            i = (i + 1) % m;
        }

        V oldValue = vals[i];
        // delete key and associated value
        keys[i] = null;
        vals[i] = null;

        // rehash all keys in same cluster
        i = (i + 1) % m;
        while (keys[i] != null) {
            // delete keys[i] an vals[i] and reinsert
            K keyToRehash = keys[i];
            V valToRehash = vals[i];
            keys[i] = null;
            vals[i] = null;
            n--;
            put(keyToRehash, valToRehash);
            i = (i + 1) % m;
        }
        n--;

        // halves size of array if it's 12.5% full or less
        if (n > 0 && n <= m / 8) resize(m / 2);
        assert check();
        return oldValue;
    }
```

#### 键簇

线性探测法的平均成本取决于元素在插入数组后聚集成的一组连续的条目,也叫作`键簇`.

显然,短小的`键簇`才能保证较高的效率.随着插入的`key`越来越多,这个要求会很难满足,较长的`键簇`会越来越多.`长键簇`的可能性要比`短键簇`更大,因为新键的散列值无论落在`键簇`的任何位置都会使它的长度加1.

### 总结


----------


`散列表`使用了适度的空间和时间并在这两个极端之间找到了一种平衡,所以它可以在一般应用中实现拥有(均摊后)常数级别的查找和插入操作的`符号表`.

但`散列表`是很难实现有序操作的,这是因为散列最主要的目的在于均匀地将键散布开来,因此在计算散列后键的顺序信息就已经丢失了.

同时,`散列表`的性能也依赖于`α=N/M`的比值,其中`α`称为`散列表`的使用率.对于`拉链法`来说,`α`是每条链表的长度,因此一般大于1.对于`线性探测法`来说,`α`是表中已被占用的空间的比例,它是不可能大于1的.

`散列表`的性能虽然高效,但它也有以下的局限性: 

 - 每种类型的键都需要一个优秀的`散列函数`.


 - 性能保证来自于`散列函数`的质量.


 - `散列函数`的计算可能复杂而且昂贵.


 - 难以支持有序性相关的操作.


### end


----------


 - Author : [SylvanasSun][1]


 - Email : sylvanassun_xtz@163.com


 - 文中的完整实现代码见我的[GitHub][2] & [Gist][3]


 - 文中参考资料引用自[<<Algorithms,4th Edition>>][4] & [Wikepedia][5]


[1]: https://github.com/SylvanasSun
[2]: https://github.com/SylvanasSun/algs4-study
[3]: https://gist.github.com/SylvanasSun/6872abd0fad061de28466cb775a84cea
[4]: http://algs4.cs.princeton.edu/34hash/
[5]: https://en.wikipedia.org/wiki/Hash_table
