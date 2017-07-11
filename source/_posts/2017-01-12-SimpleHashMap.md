---
layout:     post
title:      "实现一个简单的HashMap"
subtitle:   "Simple HashMap"
date:       2017-01-12 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - Algorithms
    - 数据结构
    - HashTable
tags:
    - HashTable
    - Algorithms
    - 数据结构
---


### 概述


----------


&nbsp;&nbsp;HashMap是基于哈希表的Map接口的实现,以key-value的形式存在,在HashMap中,key-value会被当成一个整体(Entry)来处理,HashMap内部维护了一个链表数组,会根据hash算法来计算key-value在数组中的存储位置.

### HashMap的内部结构


----------


![](http://wx3.sinaimg.cn/mw690/63503acbly1fbo5dcppj6j20hs0hkwmv.jpg)

&nbsp;&nbsp;HashMap内部使用了桶(bucket)来存储键值对,桶就是一个存储key-value的链表.而HashMap中维护了一个数组,这个数组的每个元素就是一个桶(bucket),在HashMap中,桶是使用链表实现的.

 -  HashMap使用散列函数(hash)将给定键转化为一个数组索引(桶号),不同的key会被转化为不同的索引,实际中有几率会把不同的键转化为同一个索引,这种情况叫做hash碰撞,HashMap使用了拉链法解决hash碰撞.

 - 当发生hash碰撞时,会生成一个新的key-value对象,并将新对象挂在链表头部.

 -  得到数组索引后,就可以遍历这个链表来进行各种操作了.

&nbsp;&nbsp;在HashMap中有2个重要的参数:capaCity(容量),loadFactor(负载因子).

 - 容量:它表示HashMap中桶的数量.

 - 负载因子:它是HashMap在其容量自动扩容之前可以达到多满的一种尺度,它衡量的是一个散列表的空间使用程度,负载因子越大表示散列表的空间利用率越大,反之越小.对于使用拉链法的散列表来说,查找一个元素的平均时间是O(1+a),因此负载因子越大,对空间的利用越充分,但是查找效率就会越低,如果负载因子很小,那么散列表的空间利用率将会很小,对空间造成严重浪费,但是查找效率则会变快.HashMap中负载因子的默认值为0.75.

 - 阈值:容量自动扩展的阈值,当HashMap中的键值对数量到达阈值时,将会进行自动扩容(当前容量x2),阈值通常的计算方法为 capaCity * loadFactor.

### 简单实现


----------

#### construction

```java
    /**
     * 构造一个空的SimpleHashMap,使用默认的capacity和负载因子
     */
    public SimpleHashMap(int initialiCapacity, float loadFactor) {
        if (initialiCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: "
                    + initialiCapacity);
        if (initialiCapacity > MAXIMUM_CAPACITY)
            initialiCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: "
                    + loadFactor);

        //计算出大于initialCapacity的最小的2的n次方值
        int capacity = 1;
        while (capacity < initialiCapacity) {
            capacity <<= 1;
        }

        this.loadFactor = loadFactor;
        //设置HashMap的扩容阈值,当到达这个阈值时会进行自动扩容
        threshold = (int) (capacity * loadFactor);
        //初始化table数组
        table = new Node[capacity];
    }
```

&nbsp;&nbsp;其中table是一个Node<K,V>数组,它是由链表实现的.

```java
static class Node<K, V> implements Map.Entry<K, V> {
        final int hash;
        final K key;
        V value;
        Node<K, V> next;

        Node(int hash, K key, V value, Node<K, V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey() {
            return key;
        }

        public final V getValue() {
            return value;
        }

        public final String toString() {
            return key + "=" + value;
        }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?, ?> e = (Entry<?, ?>) o;
                if (Objects.equals(key, e.getKey()) &&
                        Objects.equals(value, e.getValue())) {
                    return true;
                }
            }
            return false;
        }
    }
```

#### put 

```java
public V put(K key, V value) {
        //如果key为null,调用putForNullKey()向null key存入value
        if (key == null)
            return putForNullKey(value);
        //计算key的hash
        int hash = hash(key.hashCode());   ---1
        //计算key的hash在table数组中的索引
        int i = indexFor(hash, table.length);  --2
        //遍历table
        for (Node<K, V> e = table[i]; e != null; e = e.next) {
            Object k;
            //如果有相同的key,直接覆盖value,返回oldValue
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                return oldValue;
            }
        }
        modCount++; //修改次数++
        //将key,value添加至i处
        addEntry(hash, key, value, i);  --3
        return null;
    }
```

&nbsp;&nbsp;看以上代码1、2处,这2个函数计算了hash值和bucket索引.

```java
    /**
     * 预处理hash值，避免较差的离散hash序列，导致桶没有充分利用.
     */
    static int hash(int h) {
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

    /**
     * 返回对应hash值得索引
     */
    static int indexFor(int h, int length) {
        /**
         * 由于length是2的n次幂，所以h & (length-1)相当于h % length。
         * 对于length，其2进制表示为1000...0，那么length-1为0111...1。
         * 那么对于任何小于length的数h，该式结果都是其本身h。
         * 对于h = length，该式结果等于0。
         * 对于大于length的数h，则和0111...1位与运算后，
         * 比0111...1高或者长度相同的位都变成0，
         * 相当于减去j个length，该式结果是h-j*length，
         * 所以相当于h % length。
         * 其中一个很常用的特例就是h & 1相当于h % 2。
         * 这也是为什么length只能是2的n次幂的原因，为了优化。
         */
        return h % (length - 1);
    }
```

&nbsp;&nbsp;代码3的addEntry函数向数组添加了一对key-value.

```java
    /**
     * 添加一对key-value,如果当前索引上已有桶(发生hash碰撞),则将新元素放入链表头
     */
    void addEntry(int hash, K key, V value, int bucketIndex) {
        //保存对应table的值
        Node<K, V> e = table[bucketIndex];
        //用新桶链住旧桶
        table[bucketIndex] = new Node<K, V>(hash, key, value, e);
        //如果HashMap中元素的个数已经超过阈值,则扩容两倍
        if (size++ >= threshold)
            resize(2 * table.length);
    }
```

#### get

```java
 public V get(Object key) {
        //如果key为null,则调用getForNullkey()获得key为null的值
        if (key == null)
            return getForNullKey();
        //根据key的hashCode计算它的hash
        int hash = hash(key.hashCode());
        //取出table中指定索引处的值
        for (Node<K, V> e = table[indexFor(hash, table.length)]; e != null; e = e.next) {
            Object k;
            //如果查找相同的key,返回其对应的value
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                return e.value;
            }
        }
        return null;
    }
```

#### all

```java
/**
 * 一个简单的HashMap,内部使用拉链法解决hash碰撞.
 * <p>
 * Created by sylvanasp on 2017/1/12.
 */
public class SimpleHashMap<K, V> extends AbstractMap<K, V>
        implements Map<K, V>, Cloneable, Serializable {

    private static final long serialVersionUID = 6623475452522370065L;

    /**
     * 默认的容量(bucket数量),1 << 4(16).
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;

    /**
     * 最大的容量,1 << 30 (2^30)
     */
    static final int MAXIMUM_CAPACITY = 1 << 30;

    /**
     * 默认的负载因子,0.75.
     */
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

    /**
     * KV链表
     */
    static class Node<K, V> implements Map.Entry<K, V> {
        final int hash;
        final K key;
        V value;
        Node<K, V> next;

        Node(int hash, K key, V value, Node<K, V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey() {
            return key;
        }

        public final V getValue() {
            return value;
        }

        public final String toString() {
            return key + "=" + value;
        }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?, ?> e = (Entry<?, ?>) o;
                if (Objects.equals(key, e.getKey()) &&
                        Objects.equals(value, e.getValue())) {
                    return true;
                }
            }
            return false;
        }
    }

    /**
     * 链表数组,数组中的每一个元素代表了一个链表的头部.
     */
    transient Node[] table;

    /**
     * 当前map的key-value映射数，也就是当前size
     */
    transient int size;

    /**
     * 代表这个HashMap修改key-value的次数.
     */
    transient int modCount;

    /**
     * 自动扩展的阈值(capacity * loadfactor)
     */
    int threshold;

    /**
     * 负载因子:
     * 它是哈希表在其容量自动增加之前可以达到多满的一种尺度，
     * 它衡量的是一个散列表的空间的使用程度，负载因子越大表示散列表的装填程度越高，
     * 反之愈小。对于使用链表法的散列表来说，查找一个元素的平均时间是O(1+a)，
     * 因此如果负载因子越大，对空间的利用更充分，然而后果是查找效率的降低；
     * 如果负载因子太小，那么散列表的数据将过于稀疏，对空间造成严重浪费.
     */
    final float loadFactor;

    /**
     * 构造一个空的SimpleHashMap,使用默认的capacity和负载因子
     */
    public SimpleHashMap(int initialiCapacity, float loadFactor) {
        if (initialiCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: "
                    + initialiCapacity);
        if (initialiCapacity > MAXIMUM_CAPACITY)
            initialiCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: "
                    + loadFactor);

        //计算出大于initialCapacity的最小的2的n次方值
        int capacity = 1;
        while (capacity < initialiCapacity) {
            capacity <<= 1;
        }

        this.loadFactor = loadFactor;
        //设置HashMap的扩容阈值,当到达这个阈值时会进行自动扩容
        threshold = (int) (capacity * loadFactor);
        //初始化table数组
        table = new Node[capacity];
    }

    public SimpleHashMap() {
        this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
    }

    /**
     * 预处理hash值，避免较差的离散hash序列，导致桶没有充分利用.
     */
    static int hash(int h) {
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }

    /**
     * 返回对应hash值得索引
     */
    static int indexFor(int h, int length) {
        /**
         * 由于length是2的n次幂，所以h & (length-1)相当于h % length。
         * 对于length，其2进制表示为1000...0，那么length-1为0111...1。
         * 那么对于任何小于length的数h，该式结果都是其本身h。
         * 对于h = length，该式结果等于0。
         * 对于大于length的数h，则和0111...1位与运算后，
         * 比0111...1高或者长度相同的位都变成0，
         * 相当于减去j个length，该式结果是h-j*length，
         * 所以相当于h % length。
         * 其中一个很常用的特例就是h & 1相当于h % 2。
         * 这也是为什么length只能是2的n次幂的原因，为了优化。
         */
        return h % (length - 1);
    }

    /**
     * 获得key为null的值
     */
    private V getForNullKey() {
        //遍历table[0]
        for (Node<K, V> e = table[0]; e != null; e = e.next) {
            //如果找到key为null,则返回对应的值
            if (e.key == null) {
                return e.value;
            }
        }
        return null;
    }

    /**
     * 当Key为Null时如何放入值
     */
    private V putForNullKey(V value) {
        //遍历table[0]
        for (Node<K, V> e = table[0]; e != null; e = e.next) {
            if (e.key == null) {
                //取出oldValue,并存入newValue
                V oldValue = e.value;
                e.value = value;
                //返回oldValue
                return oldValue;
            }
        }
        modCount++;
        addEntry(0, null, value, 0);
        return null;
    }

    /**
     * 添加一对key-value,如果当前索引上已有桶(发生hash碰撞),则将新元素放入链表头
     */
    void addEntry(int hash, K key, V value, int bucketIndex) {
        //保存对应table的值
        Node<K, V> e = table[bucketIndex];
        //用新桶链住旧桶
        table[bucketIndex] = new Node<K, V>(hash, key, value, e);
        //如果HashMap中元素的个数已经超过阈值,则扩容两倍
        if (size++ >= threshold)
            resize(2 * table.length);
    }

    /**
     * 扩充容量
     */
    void resize(int newCapacity) {
        //保存oldTable
        Node[] oldTable = table;
        //保存旧容量
        int oldCapacity = oldTable.length;
        //如果旧的容量已经是系统默认最大容量了，那么将阈值设置成整形的最大值
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        //根据newCapacity创建一个table
        Node[] newTable = new Node[newCapacity];
        //将table转换为newTable
        transfer(newTable);
        table = newTable;
        //设置阈值
        threshold = (int) (newCapacity * loadFactor);
    }

    // 将所有格子里的桶都放到新的table中
    void transfer(Node[] newTable) {
        // 得到旧的table
        Node[] src = table;
        // 得到新的容量
        int newCapacity = newTable.length;
        // 遍历src里面的所有格子
        for (int j = 0; j < src.length; j++) {
            // 取到格子里的桶（也就是链表）
            Node<K, V> e = src[j];
            // 如果e不为空
            if (e != null) {
                // 将当前格子设成null
                src[j] = null;
                // 遍历格子的所有桶
                do {
                    // 取出下个桶
                    Node<K, V> next = e.next;
                    // 寻找新的索引
                    int i = indexFor(e.hash, newCapacity);
                    // 设置e.next为newTable[i]保存的桶（也就是链表连接上）
                    e.next = newTable[i];
                    // 将e设成newTable[i]
                    newTable[i] = e;
                    // 设置e为下一个桶
                    e = next;
                } while (e != null);
            }
        }
    }

    @Override
    public V get(Object key) {
        //如果key为null,则调用getForNullkey()获得key为null的值
        if (key == null)
            return getForNullKey();
        //根据key的hashCode计算它的hash
        int hash = hash(key.hashCode());
        //取出table中指定索引处的值
        for (Node<K, V> e = table[indexFor(hash, table.length)]; e != null; e = e.next) {
            Object k;
            //如果查找相同的key,返回其对应的value
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                return e.value;
            }
        }
        return null;
    }

    @Override
    public V put(K key, V value) {
        //如果key为null,调用putForNullKey()向null key存入value
        if (key == null)
            return putForNullKey(value);
        //计算key的hash
        int hash = hash(key.hashCode());
        //计算key的hash在table数组中的索引
        int i = indexFor(hash, table.length);
        //遍历table
        for (Node<K, V> e = table[i]; e != null; e = e.next) {
            Object k;
            //如果有相同的key,直接覆盖value,返回oldValue
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                return oldValue;
            }
        }
        modCount++; //修改次数++
        //将key,value添加至i处
        addEntry(hash, key, value, i);
        return null;
    }

    @Override
    public Set<Entry<K, V>> entrySet() {
        return null;
    }

    public static void main(String[] args) {
        Map<String, String> map = new SimpleHashMap<>();
        map.put("hello", "world");
        System.out.println(map.get("hello"));
    }
}
```