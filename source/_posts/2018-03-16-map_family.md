---
title:         Map大家族的那点事儿
date:        2018-03-16 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - Java
tags:
    - 后端
    - Java
    - 数据结构
    - 2018
---


### Map


----------



Map是一种用于快速查找的数据结构，它以键值对的形式存储数据，每一个键都是唯一的，且对应着一个值，如果想要查找Map中的数据，只需要传入一个键，Map会对键进行匹配并返回键所对应的值，可以说Map其实就是一个存放键值对的集合。Map被各种编程语言广泛使用，只不过在名称上可能会有些混淆，像Python中叫做字典（Dictionary），也有些语言称其为关联数组（Associative Array），但其实它们都是一样的，都是一个存放键值对的集合。至于Java中经常用到的HashMap也是Map的一种，它被称为散列表，关于散列表的细节我会在本文中解释HashMap的源码时提及。

Java还提供了一种与Map密切相关的数据结构：Set，它是数学意义上的集合，特性如下：

 - 无序性：一个集合中，每个元素的地位都是相同的，元素之间也都是无序的。不过Java中也提供了有序的Set，这点倒是没有完全遵循。

 - 互异性：一个集合中，任何两个元素都是不相同的。

 - 确定性：给定一个集合以及其任一元素，该元素属于或者不属于该集合是必须可以确定的。

很明显，Map中的key就很符合这些特性，Set的实现其实就是在内部使用Map。例如，HashSet就定义了一个类型为HashMap的成员变量，向HashSet添加元素a，等同于向它内部的HashMap添加了一个key为a，value为一个Object对象的键值对，这个Object对象是HashSet的一个常量，它是一个虚拟值，没有什么实际含义，源码如下：

```java
    private transient HashMap<E,Object> map;

    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();

    public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }
```

小插曲过后，让我们接着说Map，它是JDK的一个顶级接口，提供了三种集合视图（Collection Views）：包含所有key的集合、包含所有value的集合以及包含所有键值对的集合，Map中的元素顺序与它所返回的集合视图中的元素的迭代顺序相关，也就是说，Map本身是不保证有序性的，当然也有例外，比如TreeMap就对有序性做出了保证，这主要因为它是基于红黑树实现的。

所谓的集合视图就是由集合本身提供的一种访问数据的方式，同时对视图的任何修改也会影响到集合。好比`Map.keySet()`返回了它包含的key的集合，如果你调用了`Map.remove(key)`那么`keySet.contains(key)`也将返回`false`，再比如说`Arrays.asList(T)`可以把一个数组封装成一个List，这样你就可以通过List的API来访问和操作这些数据，如下列示例代码：

```java
String[] strings = {"a", "b", "c"};
List<String> list = Arrays.asList(strings);
System.out.println(list.get(0)); // "a"
strings[0] = "d";
System.out.println(list.get(0)); // "d"
list.set(0, "e");
System.out.println(strings[0]); // "e"
```

是不是感觉很神奇，其实`Arrays.asList()`只是将传入的数组与`Arrays`中的一个内部类`ArrayList`（注意，它与`java.util`包下的`ArrayList`不是同一个）做了一个”绑定“，在调用`get()`时会直接根据下标返回数组中的元素，而调用`set()`时也会直接修改数组中对应下标的元素。相对于直接复制来说，集合视图的优点是内存利用率更高，假设你有一个数组，又很想使用List的API来操作它，那么你不用new一个`ArrayList`以拷贝数组中的元素，只需要一点额外的内存（通过`Arrays.ArrayList`对数组进行封装），原始数据依然是在数组中的，并不会复制成多份。

Map接口规范了Map数据结构的通用API（也含有几个用于简化操作的default方法，default是JDK8的新特性，它是接口中声明的方法的默认实现，即非抽象方法）并且还在内部定义了Entry接口（键值对的实体类），在JDK中提供的所有Map数据结构都实现了Map接口，下面为Map接口的源码（代码中的注释太长了，基本都是些实现的规范，为了篇幅我就尽量省略了）。

```java
package java.util;

import java.util.function.BiConsumer;
import java.util.function.BiFunction;
import java.util.function.Function;
import java.io.Serializable;

public interface Map<K,V> {
	
	// 查询操作

    /**
     * 返回这个Map中所包含的键值对的数量，如果大于Integer.MAX_VALUE，
     * 则应该返回Integer.MAX_VALUE。
     */
    int size();

    /**
     * Map是否为空。
     */
    boolean isEmpty();

    /**
 	 * Map中是否包含key，如果是返回true，否则false。
     */
    boolean containsKey(Object key);

    /**
     * Map中是否包含value，如果是返回true，否则false。
     */
    boolean containsValue(Object value);

    /**
     * 根据key查找value，如果Map不包含该key，则返回null。
     */
    V get(Object key);

    // 修改操作

    /**
     * 添加一对键值对，如果Map中已含有这个key，那么新value将覆盖掉旧value，
     * 并返回旧value，如果Map中之前没有这个key，那么返回null。
     */
    V put(K key, V value);

    /**
     * 删除指定key并返回之前的value，如果Map中没有该key，则返回null。
     */
    V remove(Object key);


    // 批量操作

    /**
     * 将指定Map中的所有键值对批量添加到当前Map。
     */
    void putAll(Map<? extends K, ? extends V> m);

    /**
     * 删除Map中所有的键值对。
     */
    void clear();


    // 集合视图

    /**
     * 返回包含Map中所有key的Set，对该视图的所有修改操作会对Map产生同样的影响，反之亦然。
     */
    Set<K> keySet();

    /**
     * 返回包含Map中所有value的集合，对该视图的所有修改操作会对Map产生同样的影响，反之亦然。
     */
    Collection<V> values();

    /**
     * 返回包含Map中所有键值对的Set，对该视图的所有修改操作会对Map产生同样的影响，反之亦然。
     */
    Set<Map.Entry<K, V>> entrySet();

    /**
     * Entry代表一对键值对，规范了一些基本函数以及几个已实现的类函数（各种比较器）。
     */
    interface Entry<K,V> {
       
        K getKey();

        V getValue();

        V setValue(V value);

        boolean equals(Object o);

        int hashCode();

        public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> c1.getKey().compareTo(c2.getKey());
        }

        public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> c1.getValue().compareTo(c2.getValue());
        }

        public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
            Objects.requireNonNull(cmp);
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
        }

        public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {
            Objects.requireNonNull(cmp);
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
        }
    }

    // 比较和hashing

    /**
     * 将指定的对象与此Map进行比较是否相等。
     */
    boolean equals(Object o);

    /**
     * 返回此Map的hash code。
     */
    int hashCode();

    // 默认方法（非抽象方法）

    /**
     * 根据key查找value，如果该key不存在或等于null则返回defaultValue。
     */
    default V getOrDefault(Object key, V defaultValue) {
        V v;
        return (((v = get(key)) != null) || containsKey(key)) ? v : defaultValue;
    }

    /**
     * 遍历Map并对每个键值对执行指定的操作（action）。
     * BiConsumer是一个函数接口（具有一个抽象方法的接口，用于支持Lambda），
     * 它代表了一个接受两个输入参数的操作，且不返回任何结果。
     * 至于它奇怪的名字，根据Java中的其他函数接口的命名规范，Bi应该是Binary的缩写，意思是二元的。
     */
    default void forEach(BiConsumer<? super K, ? super V> action) {
        Objects.requireNonNull(action);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
            action.accept(k, v);
        }
    }

    /** 
     * 遍历Map，然后调用传入的函数function生成新value对旧value进行替换。
     * BiFunction同样是一个函数接口，它接受两个输入参数并且返回一个结果。
     */
    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
        Objects.requireNonNull(function);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }

            // ise thrown from function is not a cme.
            v = function.apply(k, v);

            try {
                entry.setValue(v);
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
        }
    }

    /**
     * 如果指定的key不存在或者关联的value为null，则添加键值对。
     */
    default V putIfAbsent(K key, V value) {
        V v = get(key);
        if (v == null) {
            v = put(key, value);
        }

        return v;
    }

    /**
     * 当指定key关联的value与传入的参数value相等时删除该key。
     */
    default boolean remove(Object key, Object value) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, value) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        remove(key);
        return true;
    }

    /**
     * 当指定key关联的value与oldValue相等时，使用newValue进行替换。
     */
    default boolean replace(K key, V oldValue, V newValue) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, oldValue) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        put(key, newValue);
        return true;
    }

    /**
     * 当指定key关联到某个value时进行替换。
     */
    default V replace(K key, V value) {
        V curValue;
        if (((curValue = get(key)) != null) || containsKey(key)) {
            curValue = put(key, value);
        }
        return curValue;
    }

    /**
     * 当指定key没有关联到一个value或者value为null时，调用mappingFunction生成值并添加键值对到Map。
     * Function是一个函数接口，它接受一个输入参数并返回一个结果，如果mappingFunction返回的结果
     * 也为null，那么将不会调用put。
     */
    default V computeIfAbsent(K key,
            Function<? super K, ? extends V> mappingFunction) {
        Objects.requireNonNull(mappingFunction);
        V v;
        if ((v = get(key)) == null) {
            V newValue;
            if ((newValue = mappingFunction.apply(key)) != null) {
                put(key, newValue);
                return newValue;
            }
        }

        return v;
    }

    /**
     * 当指定key关联到一个value并且不为null时，调用remappingFunction生成newValue，
     * 如果newValue不为null，那么进行替换，否则删除该key。
     */
    default V computeIfPresent(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue;
        if ((oldValue = get(key)) != null) {
            V newValue = remappingFunction.apply(key, oldValue);
            if (newValue != null) {
                put(key, newValue);
                return newValue;
            } else {
                remove(key);
                return null;
            }
        } else {
            return null;
        }
    }

    /**
     * remappingFunction根据key与其相关联的value生成newValue，
     * 当newValue等于null时删除该key，否则添加或者替换旧的映射。
     */
    default V compute(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue = get(key);

        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue == null) {
            // delete mapping
            if (oldValue != null || containsKey(key)) {
                // something to remove
                remove(key);
                return null;
            } else {
                // nothing to do. Leave things as they were.
                return null;
            }
        } else {
            // add or replace old mapping
            put(key, newValue);
            return newValue;
        }
    }

    /**
     * 当指定key没有关联到一个value或者value为null，将它与传入的参数value
     * 进行关联。否则，调用remappingFunction生成newValue并进行替换。
     * 如果，newValue等于null，那么删除该key。
     */
    default V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        Objects.requireNonNull(value);
        V oldValue = get(key);
        V newValue = (oldValue == null) ? value :
                   remappingFunction.apply(oldValue, value);
        if(newValue == null) {
            remove(key);
        } else {
            put(key, newValue);
        }
        return newValue;
    }
}
```

需要注意一点，这些default方法都是非线程安全的，任何保证线程安全的扩展类都必须重写这些方法，例如ConcurrentHashMap。

下图为Map的继承关系结构图，它也是本文接下来将要分析的Map实现类的大纲，这些实现类都是比较常用的，在JDK中Map的实现类有几十个，大部分都是我们用不到的，限于篇幅原因就不一一讲解了（本文包含许多源码与对实现细节的分析，建议读者抽出一段连续的空闲时间静下心来慢慢阅读）。

![](http://wx4.sinaimg.cn/large/63503acbly1fpu3a4q9ewj20hc08wjre.jpg)

> 本文作者为[SylvanasSun(sylvanas.sun@gmail.com)][1]，首发于[SylvanasSun’s Blog][2]。
> 原文链接：https://sylvanassun.github.io/2018/03/16/2018-03-16-map_family/
> （转载请务必保留本段声明，并且保留超链接。）


### AbstractMap


----------


AbstractMap是一个抽象类，它是Map接口的一个骨架实现，最小化实现了此接口提供的抽象函数。在Java的Collection框架中基本都遵循了这一规定，骨架实现在接口与实现类之间构建了一层抽象，其目的是为了复用一些比较通用的函数以及方便扩展，例如List接口拥有骨架实现AbstractList、Set接口拥有骨架实现AbstractSet等。

下面我们按照不同的操作类型来看看AbstractMap都实现了什么，首先是查询操作：

```java
package java.util;
import java.util.Map.Entry;

public abstract class AbstractMap<K,V> implements Map<K,V> {
    
    protected AbstractMap() {
    }

    // Query Operations

    public int size() {
        return entrySet().size();
    }

    // 键值对的集合视图留给具体的实现类实现
    public abstract Set<Entry<K,V>> entrySet();

    public boolean isEmpty() {
        return size() == 0;
    }

    /**
     * 遍历entrySet，然后逐个进行比较。
     */
    public boolean containsValue(Object value) {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        if (value==null) {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getValue()==null)
                    return true;
            }
        } else {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (value.equals(e.getValue()))
                    return true;
            }
        }
        return false;
    }

    /**
     * 跟containsValue()同理，只不过比较的是key。
     */
    public boolean containsKey(Object key) {
        Iterator<Map.Entry<K,V>> i = entrySet().iterator();
        if (key==null) {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getKey()==null)
                    return true;
            }
        } else {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (key.equals(e.getKey()))
                    return true;
            }
        }
        return false;
    }

    /**
     * 遍历entrySet，然后根据key取出关联的value。
     */
    public V get(Object key) {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        if (key==null) {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getKey()==null)
                    return e.getValue();
            }
        } else {
            while (i.hasNext()) {
                Entry<K,V> e = i.next();
                if (key.equals(e.getKey()))
                    return e.getValue();
            }
        }
        return null;
    }
}
```

可以发现这些操作都是依赖于函数`entrySet()`的，它返回了一个键值对的集合视图，由于不同的实现子类的Entry实现可能也是不同的，所以一般是在内部实现一个继承于AbstractSet且泛型为`Map.Entry`的内部类作为EntrySet，接下来是修改操作与批量操作：

```java
    // Modification Operations

    /**
     * 没有提供实现，子类必须重写该方法，否则调用put()会抛出异常。
     */
    public V put(K key, V value) {
        throw new UnsupportedOperationException();
    }

    /**
     * 遍历entrySet，先找到目标的entry，然后删除。
     *（还记得之前说过的吗，集合视图中的操作也会影响到实际数据）
     */
    public V remove(Object key) {
        Iterator<Entry<K,V>> i = entrySet().iterator();
        Entry<K,V> correctEntry = null;
        if (key==null) {
            while (correctEntry==null && i.hasNext()) {
                Entry<K,V> e = i.next();
                if (e.getKey()==null)
                    correctEntry = e;
            }
        } else {
            while (correctEntry==null && i.hasNext()) {
                Entry<K,V> e = i.next();
                if (key.equals(e.getKey()))
                    correctEntry = e;
            }
        }

        V oldValue = null;
        if (correctEntry !=null) {
            oldValue = correctEntry.getValue();
            i.remove();
        }
        return oldValue;
    }


    // Bulk Operations

    /**
     * 遍历参数m，然后将每一个键值对put到该Map中。
     */
    public void putAll(Map<? extends K, ? extends V> m) {
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
            put(e.getKey(), e.getValue());
    }

    /**
     * 清空entrySet等价于清空该Map。
     */
    public void clear() {
        entrySet().clear();
    }

```

AbstractMap并没有实现`put()`函数，这样做是为了考虑到也许会有不可修改的Map实现子类继承它，而对于一个可修改的Map实现子类则必须重写`put()`函数。

AbstractMap没有提供`entrySet()`的实现，但是却提供了`keySet()`与`values()`集合视图的默认实现，它们都是依赖于`entrySet()`返回的集合视图实现的，源码如下：

```java
    /**
     * keySet和values是lazy的，它们只会在第一次请求视图时进行初始化，
     * 而且它们是无状态的，所以只需要一个实例（初始化一次）。
     */
    transient Set<K>        keySet;
    transient Collection<V> values;

    /**
     * 返回一个AbstractSet的子类，可以发现它的行为都委托给了entrySet返回的集合视图
     * 与当前的AbstractMap实例，所以说它自身是无状态的。
     */
    public Set<K> keySet() {
        Set<K> ks = keySet;
        if (ks == null) {
            ks = new AbstractSet<K>() {
                public Iterator<K> iterator() {
                    return new Iterator<K>() {
                        private Iterator<Entry<K,V>> i = entrySet().iterator();

                        public boolean hasNext() {
                            return i.hasNext();
                        }

                        public K next() {
                            return i.next().getKey();
                        }

                        public void remove() {
                            i.remove();
                        }
                    };
                }

                public int size() {
                    return AbstractMap.this.size();
                }

                public boolean isEmpty() {
                    return AbstractMap.this.isEmpty();
                }

                public void clear() {
                    AbstractMap.this.clear();
                }

                public boolean contains(Object k) {
                    return AbstractMap.this.containsKey(k);
                }
            };
            keySet = ks;
        }
        return ks;
    }

    /**
     * 与keySet()基本一致，唯一的区别就是返回的是AbstractCollection的子类，
     * 主要是因为value不需要保持互异性。
     */
    public Collection<V> values() {
        Collection<V> vals = values;
        if (vals == null) {
            vals = new AbstractCollection<V>() {
                public Iterator<V> iterator() {
                    return new Iterator<V>() {
                        private Iterator<Entry<K,V>> i = entrySet().iterator();

                        public boolean hasNext() {
                            return i.hasNext();
                        }

                        public V next() {
                            return i.next().getValue();
                        }

                        public void remove() {
                            i.remove();
                        }
                    };
                }

                public int size() {
                    return AbstractMap.this.size();
                }

                public boolean isEmpty() {
                    return AbstractMap.this.isEmpty();
                }

                public void clear() {
                    AbstractMap.this.clear();
                }

                public boolean contains(Object v) {
                    return AbstractMap.this.containsValue(v);
                }
            };
            values = vals;
        }
        return vals;
    }
```

它还提供了两个Entry的实现类：SimpleEntry与SimpleImmutableEntry，这两个类的实现非常简单，区别也只是前者是可变的，而后者是不可变的。

```java
    private static boolean eq(Object o1, Object o2) {
        return o1 == null ? o2 == null : o1.equals(o2);
    }

    public static class SimpleEntry<K,V>
        implements Entry<K,V>, java.io.Serializable
    {
        private static final long serialVersionUID = -8499721149061103585L;

        private final K key;
        private V value;

        public SimpleEntry(K key, V value) {
            this.key   = key;
            this.value = value;
        }

        public SimpleEntry(Entry<? extends K, ? extends V> entry) {
            this.key   = entry.getKey();
            this.value = entry.getValue();
        }

        public K getKey() {
            return key;
        }

        public V getValue() {
            return value;
        }

        public V setValue(V value) {
            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }

        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            return eq(key, e.getKey()) && eq(value, e.getValue());
        }

        public int hashCode() {
            return (key   == null ? 0 :   key.hashCode()) ^
                   (value == null ? 0 : value.hashCode());
        }

        public String toString() {
            return key + "=" + value;
        }

    }

    /**
     * 它与SimpleEntry的区别在于它是不可变的，value被final修饰，并且不支持setValue()。
     */
    public static class SimpleImmutableEntry<K,V>
        implements Entry<K,V>, java.io.Serializable
    {
        private static final long serialVersionUID = 7138329143949025153L;

        private final K key;
        private final V value;

        public SimpleImmutableEntry(K key, V value) {
            this.key   = key;
            this.value = value;
        }

        public SimpleImmutableEntry(Entry<? extends K, ? extends V> entry) {
            this.key   = entry.getKey();
            this.value = entry.getValue();
        }

        public K getKey() {
            return key;
        }

        public V getValue() {
            return value;
        }

        public V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            return eq(key, e.getKey()) && eq(value, e.getValue());
        }

        public int hashCode() {
            return (key   == null ? 0 :   key.hashCode()) ^
                   (value == null ? 0 : value.hashCode());
        }

        public String toString() {
            return key + "=" + value;
        }

    }
```

我们通过阅读上述的源码不难发现，AbstractMap实现的操作都依赖于`entrySet()`所返回的集合视图。剩下的函数就没什么好说的了，有兴趣的话可以自己去看看。


### TreeMap


----------


TreeMap是基于红黑树（一种自平衡的二叉查找树）实现的一个保证有序性的Map，在继承关系结构图中可以得知TreeMap实现了NavigableMap接口，而该接口又继承了SortedMap接口，我们先来看看这两个接口定义了一些什么功能。

#### SortedMap


----------


首先是SortedMap接口，实现该接口的实现类应当按照自然排序保证key的有序性，所谓自然排序即是根据key的`compareTo()`函数（需要实现Comparable接口）或者在构造函数中传入的Comparator实现类来进行排序，集合视图遍历元素的顺序也应当与key的顺序一致。SortedMap接口还定义了以下几个有效利用有序性的函数：

```java
package java.util;

public interface SortedMap<K,V> extends Map<K,V> {
    /**
     * 用于在此Map中对key进行排序的比较器，如果为null，则使用key的compareTo()函数进行比较。
     */
    Comparator<? super K> comparator();

    /**
     * 返回一个key的范围为从fromKey到toKey的局部视图（包括fromKey，不包括toKey，包左不包右），
     * 如果fromKey和toKey是相等的，则返回一个空视图。
     * 返回的局部视图同样是此Map的集合视图，所以对它的操作是会与Map互相影响的。
     */
    SortedMap<K,V> subMap(K fromKey, K toKey);

    /**
     * 返回一个严格地小于toKey的局部视图。
     */
    SortedMap<K,V> headMap(K toKey);

    /**
     * 返回一个大于或等于fromKey的局部视图。
     */
    SortedMap<K,V> tailMap(K fromKey);

    /**
     * 返回当前Map中的第一个key（最小）。
     */
    K firstKey();

    /**
     * 返回当前Map中的最后一个key（最大）。
     */
    K lastKey();

    Set<K> keySet();

    Collection<V> values();

    Set<Map.Entry<K, V>> entrySet();
}
```

#### NavigableMap


----------



然后是SortedMap的子接口NavigableMap，该接口扩展了一些用于导航（Navigation）的方法，像函数`lowerEntry(key)`会根据传入的参数key返回一个小于key的最大的一对键值对，例如，我们如下调用`lowerEntry(6)`，那么将返回key为5的键值对，如果没有key为5，则会返回key为4的键值对，以此类推，直到返回null（实在找不到的情况下）。

```java
    public static void main(String[] args) {
        NavigableMap<Integer, Integer> map = new TreeMap<>();
        for (int i = 0; i < 10; i++)
            map.put(i, i);
        
        assert map.lowerEntry(6).getKey() == 5;
        assert map.lowerEntry(5).getKey() == 4;
        assert map.lowerEntry(0).getKey() == null;
    }
```

NavigableMap定义的都是一些类似于`lowerEntry(key)`的方法和以逆序、升序排序的集合视图，这些方法利用有序性实现了相比SortedMap接口更加灵活的操作。

```java
package java.util;

public interface NavigableMap<K,V> extends SortedMap<K,V> {
    /**
     * 返回一个小于指定key的最大的一对键值对，如果找不到则返回null。
     */
    Map.Entry<K,V> lowerEntry(K key);

    /**
     * 返回一个小于指定key的最大的一个key，如果找不到则返回null。
     */
    K lowerKey(K key);

    /**
     * 返回一个小于或等于指定key的最大的一对键值对，如果找不到则返回null。
     */
    Map.Entry<K,V> floorEntry(K key);

    /**
     * 返回一个小于或等于指定key的最大的一个key，如果找不到则返回null。
     */
    K floorKey(K key);

    /**
     * 返回一个大于或等于指定key的最小的一对键值对，如果找不到则返回null。
     */
    Map.Entry<K,V> ceilingEntry(K key);

    /**
     * 返回一个大于或等于指定key的最小的一个key，如果找不到则返回null。
     */
    K ceilingKey(K key);

    /**
     * 返回一个大于指定key的最小的一对键值对，如果找不到则返回null。
     */
    Map.Entry<K,V> higherEntry(K key);

    /**
     * 返回一个大于指定key的最小的一个key，如果找不到则返回null。
     */
    K higherKey(K key);

    /**
     * 返回该Map中最小的键值对，如果Map为空则返回null。
     */
    Map.Entry<K,V> firstEntry();

    /**
     * 返回该Map中最大的键值对，如果Map为空则返回null。
     */
    Map.Entry<K,V> lastEntry();

    /**
     * 返回并删除该Map中最小的键值对，如果Map为空则返回null。
     */
    Map.Entry<K,V> pollFirstEntry();

    /**
     * 返回并删除该Map中最大的键值对，如果Map为空则返回null。
     */
    Map.Entry<K,V> pollLastEntry();

    /**
     * 返回一个以当前Map降序（逆序）排序的集合视图
     */
    NavigableMap<K,V> descendingMap();

    /**
     * 返回一个包含当前Map中所有key的集合视图，该视图中的key以升序（正序）排序。
     */
    NavigableSet<K> navigableKeySet();

    /**
     * 返回一个包含当前Map中所有key的集合视图，该视图中的key以降序（逆序）排序。
     */
    NavigableSet<K> descendingKeySet();

    /**
     * 与SortedMap.subMap基本一致，区别在于多的两个参数fromInclusive和toInclusive，
     * 它们代表是否包含from和to，如果fromKey与toKey相等，并且fromInclusive与toInclusive
     * 都为true，那么不会返回空集合。
     */
    NavigableMap<K,V> subMap(K fromKey, boolean fromInclusive,
                             K toKey,   boolean toInclusive);

    /**
     * 返回一个小于或等于（inclusive为true的情况下）toKey的局部视图。
     */
    NavigableMap<K,V> headMap(K toKey, boolean inclusive);

    /**
     * 返回一个大于或等于（inclusive为true的情况下）fromKey的局部视图。
     */
    NavigableMap<K,V> tailMap(K fromKey, boolean inclusive);

    /**
     * 等价于subMap(fromKey, true, toKey, false)。
     */
    SortedMap<K,V> subMap(K fromKey, K toKey);

    /**
     * 等价于headMap(toKey, false)。
     */
    SortedMap<K,V> headMap(K toKey);

    /**
     * 等价于tailMap(fromKey, true)。
     */
    SortedMap<K,V> tailMap(K fromKey);
}
```

NavigableMap接口相对于SortedMap接口来说灵活了许多，正因为TreeMap也实现了该接口，所以在需要数据有序而且想灵活地访问它们的时候，使用TreeMap就非常合适了。

#### 红黑树


----------




上文我们提到TreeMap的内部实现基于红黑树，而红黑树又是二叉查找树的一种。二叉查找树是一种有序的树形结构，优势在于查找、插入的时间复杂度只有`O(log n)`，特性如下：

 - 任意节点最多含有两个子节点。

 - 任意节点的左、右节点都可以看做为一棵二叉查找树。

 - 如果任意节点的左子树不为空，那么左子树上的所有节点的值均小于它的根节点的值。

 - 如果任意节点的右子树不为空，那么右子树上的所有节点的值均大于它的根节点的值。

 - 任意节点的key都是不同的。

![二叉查找树](https://upload.wikimedia.org/wikipedia/commons/d/da/Binary_search_tree.svg)

尽管二叉查找树看起来很美好，但事与愿违，二叉查找树在极端情况下会变得并不是那么有效率，假设我们有一个有序的整数序列：`1,2,3,4,5,6,7,8,9,10,...`，如果把这个序列按顺序全部插入到二叉查找树时会发生什么呢？二叉查找树会产生倾斜，序列中的每一个元素都大于它的根节点（前一个元素），左子树永远是空的，那么这棵二叉查找树就跟一个普通的链表没什么区别了，查找操作的时间复杂度只有`O(n)`。

为了解决这个问题需要引入自平衡的二叉查找树，所谓自平衡，即是在树结构将要倾斜的情况下进行修正，这个修正操作被称为旋转，通过旋转操作可以让树趋于平衡。

红黑树是平衡二叉查找树的一种实现，它的名字来自于它的子节点是着色的，每个子节点非黑即红，由于只有两种颜色（两种状态），一般使用boolean来表示，下面为TreeMap中实现的Entry，它代表红黑树中的一个节点：

```java
    // Red-black mechanics

    private static final boolean RED   = false;
    private static final boolean BLACK = true;

    /**
     * Node in the Tree.  Doubles as a means to pass key-value pairs back to
     * user (see Map.Entry).
     */

    static final class Entry<K,V> implements Map.Entry<K,V> {
        K key;
        V value;
        Entry<K,V> left;
        Entry<K,V> right;
        Entry<K,V> parent;
        boolean color = BLACK;

        /**
         * Make a new cell with given key, value, and parent, and with
         * {@code null} child links, and BLACK color.
         */
        Entry(K key, V value, Entry<K,V> parent) {
            this.key = key;
            this.value = value;
            this.parent = parent;
        }

        /**
         * Returns the key.
         *
         * @return the key
         */
        public K getKey() {
            return key;
        }

        /**
         * Returns the value associated with the key.
         *
         * @return the value associated with the key
         */
        public V getValue() {
            return value;
        }

        /**
         * Replaces the value currently associated with the key with the given
         * value.
         *
         * @return the value associated with the key before this method was
         *         called
         */
        public V setValue(V value) {
            V oldValue = this.value;
            this.value = value;
            return oldValue;
        }

        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;

            return valEquals(key,e.getKey()) && valEquals(value,e.getValue());
        }

        public int hashCode() {
            int keyHash = (key==null ? 0 : key.hashCode());
            int valueHash = (value==null ? 0 : value.hashCode());
            return keyHash ^ valueHash;
        }

        public String toString() {
            return key + "=" + value;
        }
    }
```

任何平衡二叉查找树的查找操作都是与二叉查找树是一样的，因为查找操作并不会影响树的结构，也就不需要进行修正，代码如下：

```java
    public V get(Object key) {
        Entry<K,V> p = getEntry(key);
        return (p==null ? null : p.value);
    }

    final Entry<K,V> getEntry(Object key) {
        // 使用Comparator进行比较
        if (comparator != null)
            return getEntryUsingComparator(key);
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        Entry<K,V> p = root;
        // 从根节点开始，不断比较key的大小进行查找
        while (p != null) {
            int cmp = k.compareTo(p.key);
            if (cmp < 0) // 小于，转向左子树
                p = p.left;
            else if (cmp > 0) // 大于，转向右子树
                p = p.right;
            else
                return p;
        }
        return null; // 没有相等的key，返回null
    }
```

而插入和删除操作与平衡二叉查找树的细节是息息相关的，关于红黑树的实现细节，我之前写过的一篇博客[红黑树的那点事儿][3]已经讲的很清楚了，对这方面不了解的读者建议去阅读一下，就不在这里重复叙述了。

#### 集合视图


----------



最后看一下TreeMap的集合视图的实现，集合视图一般都是实现了一个封装了当前实例的类，所以对集合视图的修改本质上就是在修改当前实例，TreeMap也不例外。

TreeMap的`headMap()`、`tailMap()`以及`subMap()`函数都返回了一个静态内部类AscendingSubMap<K, V>，从名字上也能猜出来，为了支持倒序，肯定也还有一个DescendingSubMap<K, V>，它们都继承于NavigableSubMap<K, V>，一个继承AbstractMap<K, V>并实现了NavigableMap<K, V>的抽象类：

```java
    abstract static class NavigableSubMap<K,V> extends AbstractMap<K,V>
        implements NavigableMap<K,V>, java.io.Serializable {
        private static final long serialVersionUID = -2102997345730753016L;

        final TreeMap<K,V> m;

        /**
         * (fromStart, lo, loInclusive) 与 (toEnd, hi, hiInclusive)代表了两个三元组，
         * 如果fromStart为true，那么范围的下限（绝对）为map（被封装的TreeMap）的起始key，
         * 其他值将被忽略。
         * 如果loInclusive为true，lo将会被包含在范围内，否则lo是在范围外的。
         * toEnd与hiInclusive与上述逻辑相似，只不过考虑的是上限。
         */
        final K lo, hi;
        final boolean fromStart, toEnd;
        final boolean loInclusive, hiInclusive;

        NavigableSubMap(TreeMap<K,V> m,
                        boolean fromStart, K lo, boolean loInclusive,
                        boolean toEnd,     K hi, boolean hiInclusive) {
            if (!fromStart && !toEnd) {
                if (m.compare(lo, hi) > 0)
                    throw new IllegalArgumentException("fromKey > toKey");
            } else {
                if (!fromStart) // type check
                    m.compare(lo, lo);
                if (!toEnd)
                    m.compare(hi, hi);
            }

            this.m = m;
            this.fromStart = fromStart;
            this.lo = lo;
            this.loInclusive = loInclusive;
            this.toEnd = toEnd;
            this.hi = hi;
            this.hiInclusive = hiInclusive;
        }

        // internal utilities

        final boolean tooLow(Object key) {
            if (!fromStart) {
                int c = m.compare(key, lo);
                // 如果key小于lo，或等于lo（需要lo不包含在范围内）
                if (c < 0 || (c == 0 && !loInclusive))
                    return true;
            }
            return false;
        }

        final boolean tooHigh(Object key) {
            if (!toEnd) {
                int c = m.compare(key, hi);
                // 如果key大于hi，或等于hi（需要hi不包含在范围内）
                if (c > 0 || (c == 0 && !hiInclusive))
                    return true;
            }
            return false;
        }

        final boolean inRange(Object key) {
            return !tooLow(key) && !tooHigh(key);
        }

        final boolean inClosedRange(Object key) {
            return (fromStart || m.compare(key, lo) >= 0)
                && (toEnd || m.compare(hi, key) >= 0);
        }

        // 判断key是否在该视图的范围之内
        final boolean inRange(Object key, boolean inclusive) {
            return inclusive ? inRange(key) : inClosedRange(key);
        }

        /*
         * 以abs开头的函数为关系操作的绝对版本。
         */

        /*
         * 获得最小的键值对：
         * 如果fromStart为true，那么直接返回当前map实例的第一个键值对即可，
         * 否则，先判断lo是否包含在范围内，
         * 如果是，则获得当前map实例中大于或等于lo的最小的键值对，
         * 如果不是，则获得当前map实例中大于lo的最小的键值对。
         * 如果得到的结果e超过了范围的上限，那么返回null。
         */
        final TreeMap.Entry<K,V> absLowest() {
            TreeMap.Entry<K,V> e =
                (fromStart ?  m.getFirstEntry() :
                 (loInclusive ? m.getCeilingEntry(lo) :
                                m.getHigherEntry(lo)));
            return (e == null || tooHigh(e.key)) ? null : e;
        }

        // 与absLowest()相反
        final TreeMap.Entry<K,V> absHighest() {
            TreeMap.Entry<K,V> e =
                (toEnd ?  m.getLastEntry() :
                 (hiInclusive ?  m.getFloorEntry(hi) :
                                 m.getLowerEntry(hi)));
            return (e == null || tooLow(e.key)) ? null : e;
        }

        // 下面的逻辑就都很简单了，注意会先判断key是否越界，
        // 如果越界就返回绝对值。

        final TreeMap.Entry<K,V> absCeiling(K key) {
            if (tooLow(key))
                return absLowest();
            TreeMap.Entry<K,V> e = m.getCeilingEntry(key);
            return (e == null || tooHigh(e.key)) ? null : e;
        }

        final TreeMap.Entry<K,V> absHigher(K key) {
            if (tooLow(key)) 
                return absLowest();
            TreeMap.Entry<K,V> e = m.getHigherEntry(key);
            return (e == null || tooHigh(e.key)) ? null : e;
        }

        final TreeMap.Entry<K,V> absFloor(K key) {
            if (tooHigh(key))
                return absHighest();
            TreeMap.Entry<K,V> e = m.getFloorEntry(key);
            return (e == null || tooLow(e.key)) ? null : e;
        }

        final TreeMap.Entry<K,V> absLower(K key) {
            if (tooHigh(key))
                return absHighest();
            TreeMap.Entry<K,V> e = m.getLowerEntry(key);
            return (e == null || tooLow(e.key)) ? null : e;
        }

        /** 返回升序遍历的绝对上限 */
        final TreeMap.Entry<K,V> absHighFence() {
            return (toEnd ? null : (hiInclusive ?
                                    m.getHigherEntry(hi) :
                                    m.getCeilingEntry(hi)));
        }

        /** 返回降序遍历的绝对下限 */
        final TreeMap.Entry<K,V> absLowFence() {
            return (fromStart ? null : (loInclusive ?
                                        m.getLowerEntry(lo) :
                                        m.getFloorEntry(lo)));
        }

        // 剩下的就是实现NavigableMap的方法以及一些抽象方法
		// 和NavigableSubMap中的集合视图函数。
        // 大部分操作都是靠当前实例map的方法和上述用于判断边界的方法提供支持
        .....
    }
```

一个局部视图最重要的是要能够判断出传入的key是否属于该视图的范围内，在上面的代码中可以发现NavigableSubMap提供了非常多的辅助函数用于判断范围，接下来我们看看NavigableSubMap的迭代器是如何实现的：

```java
        /**
         * Iterators for SubMaps
         */
        abstract class SubMapIterator<T> implements Iterator<T> {
            TreeMap.Entry<K,V> lastReturned;
            TreeMap.Entry<K,V> next;
            final Object fenceKey;
            int expectedModCount;

            SubMapIterator(TreeMap.Entry<K,V> first,
                           TreeMap.Entry<K,V> fence) {
                expectedModCount = m.modCount; 
                lastReturned = null;
                next = first;
                // UNBOUNDED是一个虚拟值（一个Object对象），表示无边界。
                fenceKey = fence == null ? UNBOUNDED : fence.key;
            }

            // 只要next不为null并且没有超过边界
            public final boolean hasNext() {
                return next != null && next.key != fenceKey;
            }

            final TreeMap.Entry<K,V> nextEntry() {
                TreeMap.Entry<K,V> e = next;
                // 已经遍历到头或者越界了
                if (e == null || e.key == fenceKey)
                    throw new NoSuchElementException();
                // modCount是一个记录操作数的计数器
                // 如果与expectedModCount不一致
                // 则代表当前map实例在遍历过程中已被修改过了（从其他线程）
                if (m.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                // 向后移动next指针
                // successor()返回指定节点的继任者
                // 它是节点e的右子树的最左节点
                // 也就是比e大的最小的节点
                // 如果e没有右子树，则会试图向上寻找
                next = successor(e);
                lastReturned = e; // 记录最后返回的节点
                return e;
            }

            final TreeMap.Entry<K,V> prevEntry() {
                TreeMap.Entry<K,V> e = next;
                if (e == null || e.key == fenceKey)
                    throw new NoSuchElementException();
                if (m.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                // 向前移动next指针
                // predecessor()返回指定节点的前任
                // 它与successor()逻辑相反。
                next = predecessor(e);
                lastReturned = e;
                return e;
            }

            final void removeAscending() {
                if (lastReturned == null)
                    throw new IllegalStateException();
                if (m.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                // 被删除的节点被它的继任者取代
                // 执行完删除后，lastReturned实际指向了它的继任者
                if (lastReturned.left != null && lastReturned.right != null)
                    next = lastReturned;
                m.deleteEntry(lastReturned);
                lastReturned = null;
                expectedModCount = m.modCount;
            }

            final void removeDescending() {
                if (lastReturned == null)
                    throw new IllegalStateException();
                if (m.modCount != expectedModCount)
                    throw new ConcurrentModificationException();
                m.deleteEntry(lastReturned);
                lastReturned = null;
                expectedModCount = m.modCount;
            }

        }

        final class SubMapEntryIterator extends SubMapIterator<Map.Entry<K,V>> {
            SubMapEntryIterator(TreeMap.Entry<K,V> first,
                                TreeMap.Entry<K,V> fence) {
                super(first, fence);
            }
            public Map.Entry<K,V> next() {
                return nextEntry();
            }
            public void remove() {
                removeAscending();
            }
        }

        final class DescendingSubMapEntryIterator extends SubMapIterator<Map.Entry<K,V>> {
            DescendingSubMapEntryIterator(TreeMap.Entry<K,V> last,
                                          TreeMap.Entry<K,V> fence) {
                super(last, fence);
            }

            public Map.Entry<K,V> next() {
                return prevEntry();
            }
            public void remove() {
                removeDescending();
            }
        }
```

到目前为止，我们已经针对集合视图讨论了许多，想必大家也能够理解集合视图的概念了，由于SortedMap与NavigableMap的缘故，TreeMap中的集合视图是非常多的，包括各种局部视图和不同排序的视图，有兴趣的读者可以自己去看看源码，后面的内容不会再对集合视图进行过多的解释了。


### HashMap


----------



光从名字上应该也能猜到，HashMap肯定是基于hash算法实现的，这种基于hash实现的map叫做散列表（hash table）。

散列表中维护了一个数组，数组的每一个元素被称为一个桶（bucket），当你传入一个`key = "a"`进行查询时，散列表会先把key传入散列（hash）函数中进行寻址，得到的结果就是数组的下标，然后再通过这个下标访问数组即可得到相关联的值。

![](http://wx1.sinaimg.cn/large/63503acbly1fpu3a5d3p7j20j00bpwex.jpg)

我们都知道数组中数据的组织方式是线性的，它会直接分配一串连续的内存地址序列，要找到一个元素只需要根据下标来计算地址的偏移量即可（查找一个元素的起始地址为：数组的起始地址加上下标乘以该元素类型占用的地址大小）。因此散列表在理想的情况下，各种操作的时间复杂度只有`O(1)`，这甚至超过了二叉查找树，虽然理想的情况并不总是满足的，关于这点之后我们还会提及。

#### 为什么是hash？


----------



hash算法是一种可以从任何数据中提取出其“指纹”的数据摘要算法，它将任意大小的数据（输入）映射到一个固定大小的序列（输出）上，这个序列被称为hash code、数据摘要或者指纹。比较出名的hash算法有MD5、SHA。

![](https://upload.wikimedia.org/wikipedia/commons/d/da/Hash_function.svg)

hash是具有唯一性且不可逆的，唯一性指的是相同的输入产生的hash code永远是一样的，而不可逆也比较容易理解，数据摘要算法并不是压缩算法，它只是生成了一个该数据的摘要，没有将数据进行压缩。压缩算法一般都是使用一种更节省空间的编码规则将数据重新编码，解压缩只需要按着编码规则解码就是了，试想一下，一个几百MB甚至几GB的数据生成的hash code都只是一个拥有固定长度的序列，如果再能逆向解压缩，那么其他压缩算法该情何以堪？

我们上述讨论的仅仅是在密码学中的hash算法，而在散列表中所需要的散列函数是要能够将key寻址到buckets中的一个位置，散列函数的实现影响到整个散列表的性能。

一个完美的散列函数要能够做到均匀地将key分布到buckets中，每一个key分配到一个bucket，但这是不可能的。虽然hash算法具有唯一性，但同时它还具有重复性，唯一性保证了相同输入的输出是一致的，却没有保证不同输入的输出是不一致的，也就是说，完全有可能两个不同的key被分配到了同一个bucket（因为它们的hash code可能是相同的），这叫做碰撞冲突。总之，理想很丰满，现实很骨感，散列函数只能尽可能地减少冲突，没有办法完全消除冲突。

散列函数的实现方法非常多，一个优秀的散列函数要看它能不能将key分布均匀。首先介绍一种最简单的方法：除留余数法，先对key进行hash得到它的hash code，然后再用该hash code对buckets数组的元素数量取余，得到的结果就是bucket的下标，这种方法简单高效，也可以当做对集群进行负载均衡的路由算法。

```java
private int hash(Key key) {
   // & 0x7fffffff 是为了屏蔽符号位，M为bucket数组的长度
   return (key.hashCode() & 0x7fffffff) % M;
}
```

要注意一点，只有整数才能进行取余运算，如果hash code是一个字符串或别的类型，那么你需要将它转换为整数才能使用除留余数法，不过Java在Object对象中提供了`hashCode()`函数，该函数返回了一个int值，所以任何你想要放入HashMap的自定义的抽象数据类型，都必须实现该函数和`equals()`函数，这两个函数之间也遵守着一种约定：如果`a.equals(b) == true`，那么a与b的`hashCode()`也必须是相同的。

下面为String类的`hashCode()`函数，它先遍历了内部的字符数组，然后在每一次循环中计算hash code（将hash code乘以一个素数并加上当前循环项的字符）：

```java
    /** The value is used for character storage. */
    private final char value[];

    /** Cache the hash code for the string */
    private int hash; // Default to 0

    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
```

HashMap没有采用这么简单的方法，有一个原因是HashMap中的buckets数组的长度永远为一个2的幂，而不是一个素数，如果长度为素数，那么可能会更适合简单暴力的除留余数法（当然除留余数法虽然简单却并不是那么高效的），顺便一提，时代的眼泪Hashtable就使用了除留余数法，它没有强制约束buckets数组的长度。

HashMap在内部实现了一个`hash()`函数，首先要对`hashCode()`的返回值进行处理：

```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

该函数将`key.hashCode()`的低16位和高16位做了个异或运算，其目的是为了扰乱低位的信息以实现减少碰撞冲突。之后还需要把`hash()`的返回值与`table.length - 1`做与运算（`table`为buckets数组），得到的结果即是数组的下标。

`table.length - 1`就像是一个低位掩码（这个设计也优化了扩容操作的性能），它和`hash()`做与操作时必然会将高位屏蔽（因为一个HashMap不可能有特别大的buckets数组，至少在不断自动扩容之前是不可能的，所以`table.length - 1`的大部分高位都为0），只保留低位，看似没什么毛病，但这其实暗藏玄机，它会导致总是只有最低的几位是有效的，这样就算你的`hashCode()`实现得再好也难以避免发生碰撞。这时，`hash()`函数的价值就体现出来了，它对hash code的低位添加了随机性并且混合了高位的部分特征，显著减少了碰撞冲突的发生（关于`hash()`函数的效果如何，可以参考这篇文章[An introduction to optimising a hashing strategy][4]）。

HashMap的散列函数具体流程如下图：

![](http://wx2.sinaimg.cn/large/63503acbly1fpu3a5ojfoj20rx0dhdh4.jpg)

#### 解决冲突


----------



在上文中我们已经多次提到碰撞冲突，但是散列函数不可能是完美的，key分布完全均匀的情况是不存在的，所以碰撞冲突总是难以避免。

那么发生碰撞冲突时怎么办？总不能丢弃数据吧？必须要有一种合理的方法来解决这个问题，HashMap使用了叫做分离链接（Separate chaining，也有人翻译成拉链法）的策略来解决冲突。它的主要思想是每个bucket都应当是一个互相独立的数据结构，当发生冲突时，只需要把数据放入bucket中（因为bucket本身也是一个可以存放数据的数据结构），这样查询一个key所消耗的时间为访问bucket所消耗的时间加上在bucket中查找的时间。

HashMap的buckets数组其实就是一个链表数组，在发生冲突时只需要把Entry（还记得Entry吗？HashMap的Entry实现就是一个简单的链表节点，它包含了key和value以及hash code）放到链表的尾部，如果未发生冲突（位于该下标的bucket为null），那么就把该Entry做为链表的头部。而且HashMap还使用了Lazy策略，buckets数组只会在第一次调用`put()`函数时进行初始化，这是一种防止内存浪费的做法，像ArrayList也是Lazy的，它在第一次调用`add()`时才会初始化内部的数组。

![分离链接法](https://upload.wikimedia.org/wikipedia/commons/d/d0/Hash_table_5_0_1_1_1_1_1_LL.svg)

不过链表虽然实现简单，但是在查找的效率上只有`O(n)`，而且我们大部分的操作都是在进行查找，在`hashCode()`设计的不是非常良好的情况下，碰撞冲突可能会频繁发生，链表也会变得越来越长，这个效率是非常差的。Java 8对其实现了优化，链表的节点数量在到达阈值时会转化为红黑树，这样查找所需的时间就只有`O(log n)`了，阈值的定义如下：

```java
    /**
     * The bin count threshold for using a tree rather than list for a
     * bin.  Bins are converted to trees when adding an element to a
     * bin with at least this many nodes. The value must be greater
     * than 2 and should be at least 8 to mesh with assumptions in
     * tree removal about conversion back to plain bins upon
     * shrinkage.
     */
    static final int TREEIFY_THRESHOLD = 8;
```

如果在插入Entry时发现一条链表超过阈值，就会执行以下的操作，对该链表进行树化；相对的，如果在删除Entry（或进行扩容）时发现红黑树的节点太少（根据阈值UNTREEIFY_THRESHOLD），也会把红黑树退化成链表。

```java
    /**
     * 替换指定hash所处位置的链表中的所有节点为TreeNode，
     * 如果buckets数组太小，就进行扩容。
     */
    final void treeifyBin(Node<K,V>[] tab, int hash) {
        int n, index; Node<K,V> e;
        // MIN_TREEIFY_CAPACITY = 64，小于该值代表数组中的节点并不是很多
        // 所以选择进行扩容，只有数组长度大于该值时才会进行树化。
        if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
            resize();
        else if ((e = tab[index = (n - 1) & hash]) != null) {
            TreeNode<K,V> hd = null, tl = null;
            // 转换链表节点为树节点，注意要处理好连接关系
            do {
                TreeNode<K,V> p = replacementTreeNode(e, null);
                if (tl == null)
                    hd = p;
                else {
                    p.prev = tl;
                    tl.next = p;
                }
                tl = p;
            } while ((e = e.next) != null);
            if ((tab[index] = hd) != null)
                hd.treeify(tab); // 从头部开始构造树
        }
    }

        // 该函数定义在TreeNode中
        final void treeify(Node<K,V>[] tab) {
            TreeNode<K,V> root = null;
            for (TreeNode<K,V> x = this, next; x != null; x = next) {
                next = (TreeNode<K,V>)x.next;
                x.left = x.right = null;
                if (root == null) { // 初始化root节点
                    x.parent = null;
                    x.red = false;
                    root = x;
                }
                else {
                    K k = x.key;
                    int h = x.hash;
                    Class<?> kc = null;
                    for (TreeNode<K,V> p = root;;) {
                        int dir, ph;
                        K pk = p.key;
                        // 确定节点的方向
                        if ((ph = p.hash) > h)
                            dir = -1;
                        else if (ph < h)
                            dir = 1;
                        // 如果kc == null
                        // 并且k没有实现Comparable接口
                        // 或者k与pk是没有可比较性的（类型不同）
                        // 或者k与pk是相等的（返回0也有可能是相等）
                        else if ((kc == null &&
                                  (kc = comparableClassFor(k)) == null) ||
                                 (dir = compareComparables(kc, k, pk)) == 0)
                            dir = tieBreakOrder(k, pk);

                        // 确定方向后插入节点，修正红黑树的平衡
                        TreeNode<K,V> xp = p;
                        if ((p = (dir <= 0) ? p.left : p.right) == null) {
                            x.parent = xp;
                            if (dir <= 0)
                                xp.left = x;
                            else
                                xp.right = x;
                            root = balanceInsertion(root, x);
                            break;
                        }
                    }
                }
            }
            // 确保给定的root是该bucket中的第一个节点
            moveRootToFront(tab, root);
        }

        static int tieBreakOrder(Object a, Object b) {
            int d;
            if (a == null || b == null ||
                (d = a.getClass().getName().
                 compareTo(b.getClass().getName())) == 0)
                // System.identityHashCode()将调用并返回传入对象的默认hashCode()
                // 也就是说，无论是否重写了hashCode()，都将调用Object.hashCode()。
                // 如果传入的对象是null，那么就返回0
                d = (System.identityHashCode(a) <= System.identityHashCode(b) ?
                     -1 : 1);
            return d;
        }            
```

解决碰撞冲突的另一种策略叫做开放寻址法（Open addressing），它与分离链接法的思想截然不同。在开放寻址法中，所有Entry都会存储在buckets数组，一个明显的区别是，分离链接法中的每个bucket都是一个链表或其他的数据结构，而开放寻址法中的每个bucket就仅仅只是Entry本身。

开放寻址法是基于数组中的空位来解决冲突的，它的想法很简单，与其使用链表等数据结构，不如直接在数组中留出空位来当做一个标记，反正都要占用额外的内存。

当你查找一个key的时候，首先会从起始位置（通过散列函数计算出的数组索引）开始，不断检查当前bucket是否为目标Entry（通过比较key来判断），如果当前bucket不是目标Entry，那么就向后查找（查找的间隔取决于实现），直到碰见一个空位（null），这代表你想要找的key不存在。

如果你想要put一个全新的Entry（Map中没有这个key存在），依然会从起始位置开始进行查找，如果起始位置不是空的，则代表发生了碰撞冲突，只好不断向后查找，直到发现一个空位。

开放寻址法的名字也是来源于此，一个Entry的位置并不是完全由hash值决定的，所以也叫做Closed hashing，相对的，分离链接法也被称为Open hashing或Closed addressing。

根据向后探测（查找）的算法不同，开放寻址法有多种不同的实现，我们介绍一种最简单的算法：线性探测法（Linear probing），在发生碰撞时，简单地将索引加一，如果到达了数组的尾部就折回到数组的头部，直到找到目标或一个空位。

![线性探测法](https://upload.wikimedia.org/wikipedia/commons/9/90/HASHTB12.svg)

基于线性探测法的查找操作如下：

```java
    private K[] keys; // 存储key的数组
    private V[] vals; // 存储值的数组 

    public V get(K key) {
    	// m是buckets数组的长度，即keys和vals的长度。
    	// 当i等于m时，取模运算会得0（折回数组头部）
        for (int i = hash(key); keys[i] != null; i = (i + 1) % m) {
            if (keys[i].equals(key))
                return vals[i];
        }
        return null;
    }
```

插入操作稍微麻烦一些，需要在插入之前判断当前数组的剩余容量，然后决定是否扩容。数组的剩余容量越多，代表Entry之间的间隔越大以及越早碰见空位（向后探测的次数就越少），效率自然就会变高。代价就是额外消耗的内存较多，这也是在用空间换取时间。

```java
    public void put(K key, V value) {
        // n是Entry的数量，如果n超过了数组长度的一半，就扩容一倍
        if (n >= m / 2) resize(2 * m);

        int i;
        for (i = hash(key); keys[i] != null; i = (i + 1) % m) {
            if (keys[i].equals(key)) {
                vals[i] = value;
                return;
            }
        }

        // 没有找到目标，那么就插入一对新的Entry
        keys[i] = key;
        vals[i] = value;
        n++;
    }
```

接下来是删除操作，需要注意一点，我们不能简单地把目标key所在的位置（keys和vals数组）设置为null，这样会导致此位置之后的Entry无法被探测到，所以需要将目标右侧的所有Entry重新插入到散列表中：

```java
public V delete(K key) {
    int i = hash(key);
    // 先找到目标的索引
    while (!key.equals(keys[i])) {
        i = (i + 1) % m;
    }

    V oldValue = vals[i];
    // 删除目标key和value
    keys[i] = null;
    vals[i] = null;
    // 指针移动到下一个索引
    i = (i + 1) % m;
    while (keys[i] != null) {
        // 先删除然后重新插入
        K keyToRehash = keys[i];
        V valToRehash = vals[i];
        keys[i] = null;
        vals[i] = null;
        n--;
        put(keyToRehash, valToRehash);
        i = (i + 1) % m;
    }

    n--;
    // 当前Entry小于等于数组长度的八分之一时，进行缩容
    if (n > 0 && n <= m / 8) resize(m / 2);

    return oldValue;
}
```

#### 动态扩容


----------



散列表以数组的形式组织bucket，问题在于数组是静态分配的，为了保证查找的性能，需要在Entry数量大于一个临界值时进行扩容，否则就算散列函数的效果再好，也难免产生碰撞。

所谓扩容，其实就是用一个容量更大（在原容量上乘以二）的数组来替换掉当前的数组，这个过程需要把旧数组中的数据重新hash到新数组，所以扩容也能在一定程度上减缓碰撞。

HashMap通过负载因子（Load Factor）乘以buckets数组的长度来计算出临界值，算法：`threshold = load_factor * capacity`。比如，HashMap的默认初始容量为16（`capacity = 16`），默认负载因子为0.75（`load_factor = 0.75`），那么临界值就为`threshold = 0.75 * 16 = 12`，只要Entry的数量大于12，就会触发扩容操作。

还可以通过下列的构造函数来自定义负载因子，负载因子越小查找的性能就会越高，但同时额外占用的内存就会越多，如果没有特殊需要不建议修改默认值。

```java
    /**
     * 可以发现构造函数中根本就没初始化buckets数组。
     * （之前说过buckets数组会推迟到第一次调用put()时进行初始化）
     */
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        // tableSizeFor()确保initialCapacity必须为一个2的N次方
        this.threshold = tableSizeFor(initialCapacity);
    }
```

buckets数组的大小约束对于整个HashMap都至关重要，为了防止传入一个不是2次幂的整数，必须要有所防范。`tableSizeFor()`函数会尝试修正一个整数，并转换为离该整数最近的2次幂。

```java
    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

![tableSizeFor()工作流程](http://wx1.sinaimg.cn/large/63503acbly1fpu3a6fobzj20pq0oy0th.jpg)

还记得数组索引的计算方法吗？`index = (table.length - 1) & hash`，这其实是一种优化手段，由于数组的大小永远是一个2次幂，在扩容之后，一个元素的新索引要么是在原位置，要么就是在原位置加上扩容前的容量。这个方法的巧妙之处全在于&运算，之前提到过&运算只会关注n - 1（n = 数组长度）的有效位，当扩容之后，n的有效位相比之前会多增加一位（n会变成之前的二倍，所以确保数组长度永远是2次幂很重要），然后只需要判断hash在新增的有效位的位置是0还是1就可以算出新的索引位置，如果是0，那么索引没有发生变化，如果是1，索引就为原索引加上扩容前的容量。

![扩容前与扩容后的索引计算](http://wx1.sinaimg.cn/large/63503acbly1fpu3a7n647j20z00fqq48.jpg)

这样在每次扩容时都不用重新计算hash，省去了不少时间，而且新增有效位是0还是1是带有随机性的，之前两个碰撞的Entry又有可能在扩容时再次均匀地散布开。下面是`resize()`的源码：

```java
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table; // table就是buckets数组
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        // oldCap大于0，进行扩容，设置阈值与新的容量
        if (oldCap > 0) {
            // 超过最大值不会进行扩容，并且把阈值设置成Interger.MAX_VALUE
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            // 没超过最大值，扩容为原来的2倍
            // 向左移1位等价于乘2
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        // oldCap = 0，oldThr大于0，那么就把阈值做为新容量以进行初始化
        // 这种情况发生在用户调用了带有参数的构造函数（会对threshold进行初始化）
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        // oldCap与oldThr都为0，这种情况发生在用户调用了无参构造函数
        // 采用默认值进行初始化
        else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        // 如果newThr还没有被赋值，那么就根据newCap计算出阈值
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        // 如果oldTab != null，代表这是扩容操作
        // 需要将扩容前的数组数据迁移到新数组
        if (oldTab != null) {
            // 遍历oldTab的每一个bucket，然后移动到newTab
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    // 索引j的bucket只有一个Entry（未发生过碰撞）
                    // 直接移动到newTab
                    if (e.next == null)
                        newTab[e.hash & (newCap - 1)] = e;
                    // 如果是一个树节点（代表已经转换成红黑树了）
                    // 那么就将这个节点拆分为lower和upper两棵树
                    // 首先会对这个节点进行遍历
                    // 只要当前节点的hash & oldCap == 0就链接到lower树
                    // 注意这里是与oldCap进行与运算，而不是oldCap - 1(n - 1)
                    // oldCap就是扩容后新增有效位的掩码
                    // 比如oldCap=16，二进制10000，n-1 = 1111，扩容后的n-1 = 11111
                    // 只要hash & oldCap == 0，就代表hash的新增有效位为0
                    // 否则就链接到upper树（新增有效位为1）
                    // lower会被放入newTab[原索引j]，upper树会被放到newTab[原索引j + oldCap]
                    // 如果lower或者upper树的节点少于阈值，会被退化成链表
                    else if (e instanceof TreeNode)
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order
                        // 下面操作的逻辑与分裂树节点基本一致
                        // 只不过split()操作的是TreeNode
                        // 而且会将两条TreeNode链表组织成红黑树
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }
```

使用HashMap时还需要注意一点，它不会动态地进行缩容，也就是说，你不应该保留一个已经删除过大量Entry的HashMap（如果不打算继续添加元素的话），此时它的buckets数组经过多次扩容已经变得非常大了，这会占用非常多的无用内存，这样做的好处是不用多次对数组进行扩容或缩容操作。不过一般也不会出现这种情况，如果遇见了，请毫不犹豫地丢掉它，或者把数据转移到一个新的HashMap。

#### 添加元素


----------



我们已经了解了HashMap的内部实现与工作原理，它在内部维护了一个数组，每一个key都会经过散列函数得出在数组的索引，如果两个key的索引相同，那么就使用分离链接法解决碰撞冲突，当Entry的数量大于临界值时，对数组进行扩容。

接下来以一个添加元素（`put()`）的过程为例来梳理一下知识，下图是`put()`函数的流程图：

![put()流程](http://wx3.sinaimg.cn/large/63503acbly1fpu3a8bhl4j20xp13kn0l.jpg)

然后是源码：

```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // table == null or table.length == 0
        // 第一次调用put()，初始化table
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // 没有发生碰撞，直接放入到数组
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            // 发生碰撞（头节点就是目标节点）
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 节点为红黑树
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 节点为链表
            else {
                for (int binCount = 0; ; ++binCount) {
                    // 未找到目标节点，在链表尾部链接新节点
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            // 链表过长，转换为红黑树
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 找到目标节点，退出循环
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            // 节点已存在，替换value
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                // afterNodeXXXXX是提供给LinkedHashMap重写的函数
                // 在HashMap中没有意义
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 超过临界值，进行扩容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```


### WeakHashMap


----------



WeakHashMap是一个基于Map接口实现的散列表，实现细节与HashMap类似（都有负载因子、散列函数等等，但没有HashMap那么多优化手段），它的特殊之处在于每个key都是一个弱引用。

首先我们要明白什么是弱引用，Java将引用分为四类（从JDK1.2开始），强度依次逐渐减弱：

 - 强引用： 就是平常使用的普通引用对象，例如`Object obj = new Object()`，这就是一个强引用，强引用只要还存在，就不会被垃圾收集器回收。

 - 软引用： 软引用表示一个还有用但并非必需的对象，不像强引用，它还需要通过SoftReference类来间接引用目标对象（除了强引用都是如此）。被软引用关联的对象，在将要发生内存溢出异常之前，会被放入回收范围之中以进行第二次回收（如果第二次回收之后依旧没有足够的内存，那么就会抛出OOM异常）。

 - 弱引用： 同样是表示一个非必需的对象，但要比软引用的强度还要弱，需要通过WeakReference类来间接引用目标对象。被弱引用关联的对象只能存活到下一次垃圾回收发生之前，当触发垃圾回收时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象（如果这个对象还被强引用所引用，那么就不会被回收）。

 - 虚引用： 这是一种最弱的引用关系，需要通过PhantomReference类来间接引用目标对象。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用来获得对象实例。虚引用的唯一作用就是能在这个对象被回收时收到一个系统通知（结合ReferenceQueue使用）。基于这点可以通过虚引用来实现对象的析构函数，这比使用`finalize()`函数是要靠谱多了。

WeakHashMap适合用来当做一个缓存来使用。假设你的缓存系统是基于强引用实现的，那么你就必须以手动（或者用一条线程来不断轮询）的方式来删除一个无效的缓存项，而基于弱引用实现的缓存项只要没被其他强引用对象关联，就会被直接放入回收队列。

需要注意的是，只有key是被弱引用关联的，而value一般都是一个强引用对象。因此，需要确保value没有关联到它的key，否则会对key的回收产生阻碍。在极端的情况下，一个value对象A引用了另一个key对象D，而与D相对应的value对象C又反过来引用了与A相对应的key对象B，这就会产生一个引用循环，导致D与B都无法被正常回收。想要解决这个问题，就只能把value也变成一个弱引用，例如`m.put(key, new WeakReference(value))`，弱引用之间的互相引用不会产生影响。

查找操作的实现跟HashMap相比简单了许多，只要读懂了HashMap，基本都能看懂，源码如下：

```java
    /**
     * Value representing null keys inside tables.
     */
    private static final Object NULL_KEY = new Object();

    /**
     * Use NULL_KEY for key if it is null.
     */
    private static Object maskNull(Object key) {
        return (key == null) ? NULL_KEY : key;
    }

    /**
     * Returns index for hash code h.
     */
    private static int indexFor(int h, int length) {
        return h & (length-1);
    }

    public V get(Object key) {
        // WeakHashMap允许null key与null value
        // null key会被替换为一个虚拟值
        Object k = maskNull(key); 
        int h = hash(k);
        Entry<K,V>[] tab = getTable();
        int index = indexFor(h, tab.length);
        Entry<K,V> e = tab[index];
        // 遍历链表
        while (e != null) {
            if (e.hash == h && eq(k, e.get()))
                return e.value;
            e = e.next;
        }
        return null;
    }
```

尽管key是一个弱引用，但仍需手动地回收那些已经无效的Entry。这个操作会在`getTable()`函数中执行，不管是查找、添加还是删除，都需要调用`getTable()`来获得buckets数组，所以这是种防止内存泄漏的被动保护措施。

```java
    /**
     * The table, resized as necessary. Length MUST Always be a power of two.
     */
    Entry<K,V>[] table;

    /**
     * Reference queue for cleared WeakEntries
     */
    private final ReferenceQueue<Object> queue = new ReferenceQueue<>();

    /**
     * Expunges stale entries from the table.
     */
    private void expungeStaleEntries() {
        // 遍历ReferenceQueue，然后清理table中无效的Entry
        for (Object x; (x = queue.poll()) != null; ) {
            synchronized (queue) {
                @SuppressWarnings("unchecked")
                    Entry<K,V> e = (Entry<K,V>) x;
                int i = indexFor(e.hash, table.length);

                Entry<K,V> prev = table[i];
                Entry<K,V> p = prev;
                while (p != null) {
                    Entry<K,V> next = p.next;
                    if (p == e) {
                        if (prev == e)
                            table[i] = next;
                        else
                            prev.next = next;
                        // Must not null out e.next;
                        // stale entries may be in use by a HashIterator
                        e.value = null; // Help GC
                        size--;
                        break;
                    }
                    prev = p;
                    p = next;
                }
            }
        }
    }

    /**
     * Returns the table after first expunging stale entries.
     */
    private Entry<K,V>[] getTable() {
        expungeStaleEntries();
        return table;
    }
```

然后是插入操作与删除操作，实现都比较简单：

```java
    public V put(K key, V value) {
        Object k = maskNull(key);
        int h = hash(k);
        Entry<K,V>[] tab = getTable();
        int i = indexFor(h, tab.length);

        for (Entry<K,V> e = tab[i]; e != null; e = e.next) {
            if (h == e.hash && eq(k, e.get())) {
                V oldValue = e.value;
                if (value != oldValue)
                    e.value = value;
                return oldValue;
            }
        }

        modCount++;
        Entry<K,V> e = tab[i];
        // e被连接在new Entry的后面
        tab[i] = new Entry<>(k, value, queue, h, e);
        if (++size >= threshold)
            resize(tab.length * 2);
        return null;
    }

    public V remove(Object key) {
        Object k = maskNull(key);
        int h = hash(k);
        Entry<K,V>[] tab = getTable();
        int i = indexFor(h, tab.length);
        Entry<K,V> prev = tab[i];
        Entry<K,V> e = prev;

        while (e != null) {
            Entry<K,V> next = e.next;
            if (h == e.hash && eq(k, e.get())) {
                modCount++;
                size--;
                if (prev == e)
                    tab[i] = next;
                else
                    prev.next = next;
                return e.value;
            }
            prev = e;
            e = next;
        }

        return null;
    }
```

我们并没有在`put()`函数中发现key被转换成弱引用，这是怎么回事？key只有在第一次被放入buckets数组时才需要转换成弱引用，也就是`new Entry<>(k, value, queue, h, e)`，WeakHashMap的Entry实现其实就是WeakReference的子类。

```java
    /**
     * The entries in this hash table extend WeakReference, using its main ref
     * field as the key.
     */
    private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V> {
        V value;
        final int hash;
        Entry<K,V> next;

        /**
         * Creates new entry.
         */
        Entry(Object key, V value,
              ReferenceQueue<Object> queue,
              int hash, Entry<K,V> next) {
            super(key, queue);
            this.value = value;
            this.hash  = hash;
            this.next  = next;
        }

        @SuppressWarnings("unchecked")
        public K getKey() {
            return (K) WeakHashMap.unmaskNull(get());
        }

        public V getValue() {
            return value;
        }

        public V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            K k1 = getKey();
            Object k2 = e.getKey();
            if (k1 == k2 || (k1 != null && k1.equals(k2))) {
                V v1 = getValue();
                Object v2 = e.getValue();
                if (v1 == v2 || (v1 != null && v1.equals(v2)))
                    return true;
            }
            return false;
        }

        public int hashCode() {
            K k = getKey();
            V v = getValue();
            return Objects.hashCode(k) ^ Objects.hashCode(v);
        }

        public String toString() {
            return getKey() + "=" + getValue();
        }
    }
```

有关使用WeakReference的一个典型案例是ThreadLocal，感兴趣的读者可以参考我之前写的博客[聊一聊Spring中的线程安全性][5]。


### LinkedHashMap


----------



LinkedHashMap继承HashMap并实现了Map接口，同时具有可预测的迭代顺序（按照插入顺序排序）。它与HashMap的不同之处在于，维护了一条贯穿其全部Entry的双向链表（因为额外维护了链表的关系，性能上要略差于HashMap，不过集合视图的遍历时间与元素数量成正比，而HashMap是与buckets数组的长度成正比的），可以认为它是散列表与链表的结合。

```java
    /**
     * The head (eldest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> head;

    /**
     * The tail (youngest) of the doubly linked list.
     */
    transient LinkedHashMap.Entry<K,V> tail;

    /**
     * 迭代顺序模式的标记位，如果为true，采用访问排序，否则，采用插入顺序
     * 默认插入顺序（构造函数中默认设置为false）
     */
    final boolean accessOrder;

    /**
     * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
     * with the default initial capacity (16) and load factor (0.75).
     */
    public LinkedHashMap() {
        super();
        accessOrder = false;
    }
```

LinkedHashMap的Entry实现也继承自HashMap，只不过多了指向前后的两个指针。

```java
    /**
     * HashMap.Node subclass for normal LinkedHashMap entries.
     */
    static class Entry<K,V> extends HashMap.Node<K,V> {
        Entry<K,V> before, after;
        Entry(int hash, K key, V value, Node<K,V> next) {
            super(hash, key, value, next);
        }
    }
```

你也可以通过构造函数来构造一个迭代顺序为访问顺序（accessOrder设为true）的LinkedHashMap，这个访问顺序指的是按照最近被访问的Entry的顺序进行排序（从最近最少访问到最近最多访问）。基于这点可以简单实现一个采用LRU（Least Recently Used）策略的缓存。

```java
    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
```

LinkedHashMap复用了HashMap的大部分代码，所以它的查找实现是非常简单的，唯一稍微复杂点的操作是保证访问顺序。

```java
    public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
```

还记得这些afterNodeXXXX命名格式的函数吗？我们之前已经在HashMap中见识过了，这些函数在HashMap中只是一个空实现，是专门用来让LinkedHashMap重写实现的hook函数。

```java
    // 在HashMap.removeNode()的末尾处调用
    // 将e从LinkedHashMap的双向链表中删除
    void afterNodeRemoval(Node<K,V> e) { // unlink
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.before = p.after = null;
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a == null)
            tail = b;
        else
            a.before = b;
    }

    // 在HashMap.putVal()的末尾处调用
    // evict是一个模式标记，如果为false代表buckets数组处于创建模式
    // HashMap.put()函数对此标记设置为true
    void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        // LinkedHashMap.removeEldestEntry()永远返回false
        // 避免了最年长元素被删除的可能（就像一个普通的Map一样）
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }

    // HashMap.get()没有调用此函数，所以LinkedHashMap重写了get()
	// get()与put()都会调用afterNodeAccess()来保证访问顺序
    // 将e移动到tail，代表最近访问到的节点
    void afterNodeAccess(Node<K,V> e) { // move node to last
        LinkedHashMap.Entry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMap.Entry<K,V> p =
                (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
```

注意`removeEldestEntry()`默认永远返回false，这时它的行为与普通的Map无异。如果你把`removeEldestEntry()`重写为永远返回true，那么就有可能使LinkedHashMap处于一个永远为空的状态（每次`put()`或者`putAll()`都会删除头节点）。

一个比较合理的实现示例：

```java
protected boolean removeEldestEntry(Map.Entry eldest){
    return size() > MAX_SIZE;
}
```

LinkedHashMap重写了`newNode()`等函数，以初始化或连接节点到它内部的双向链表：

```java
     // 链接节点p到链表尾部（或初始化链表）
    private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
        LinkedHashMap.Entry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }

    // 用dst替换掉src
    private void transferLinks(LinkedHashMap.Entry<K,V> src,
                               LinkedHashMap.Entry<K,V> dst) {
        LinkedHashMap.Entry<K,V> b = dst.before = src.before;
        LinkedHashMap.Entry<K,V> a = dst.after = src.after;
        // src是头节点
        if (b == null)
            head = dst;
        else
            b.after = dst;
        // src是尾节点
        if (a == null)
            tail = dst;
        else
            a.before = dst;
    }   

    Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMap.Entry<K,V> p =
            new LinkedHashMap.Entry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }

    Node<K,V> replacementNode(Node<K,V> p, Node<K,V> next) {
        LinkedHashMap.Entry<K,V> q = (LinkedHashMap.Entry<K,V>)p;
        LinkedHashMap.Entry<K,V> t =
            new LinkedHashMap.Entry<K,V>(q.hash, q.key, q.value, next);
        transferLinks(q, t);
        return t;
    }

    TreeNode<K,V> newTreeNode(int hash, K key, V value, Node<K,V> next) {
        TreeNode<K,V> p = new TreeNode<K,V>(hash, key, value, next);
        linkNodeLast(p);
        return p;
    }

    TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
        LinkedHashMap.Entry<K,V> q = (LinkedHashMap.Entry<K,V>)p;
        TreeNode<K,V> t = new TreeNode<K,V>(q.hash, q.key, q.value, next);
        transferLinks(q, t);
        return t;
    }
```

遍历LinkedHashMap所需要的时间与Entry数量成正比，这是因为迭代器直接对双向链表进行迭代，而链表中只会含有Entry节点。迭代的顺序是从头节点开始一直到尾节点，插入操作会将新节点链接到尾部，所以保证了插入顺序，而访问顺序会通过`afterNodeAccess()`来保证，访问次数越多的节点越接近尾部。

```java
    abstract class LinkedHashIterator {
        LinkedHashMap.Entry<K,V> next;
        LinkedHashMap.Entry<K,V> current;
        int expectedModCount;

        LinkedHashIterator() {
            next = head;
            expectedModCount = modCount;
            current = null;
        }

        public final boolean hasNext() {
            return next != null;
        }

        final LinkedHashMap.Entry<K,V> nextNode() {
            LinkedHashMap.Entry<K,V> e = next;
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            if (e == null)
                throw new NoSuchElementException();
            current = e;
            next = e.after;
            return e;
        }

        public final void remove() {
            Node<K,V> p = current;
            if (p == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            current = null;
            K key = p.key;
            removeNode(hash(key), key, null, false, false);
            expectedModCount = modCount;
        }
    }

    final class LinkedKeyIterator extends LinkedHashIterator
        implements Iterator<K> {
        public final K next() { return nextNode().getKey(); }
    }

    final class LinkedValueIterator extends LinkedHashIterator
        implements Iterator<V> {
        public final V next() { return nextNode().value; }
    }

    final class LinkedEntryIterator extends LinkedHashIterator
        implements Iterator<Map.Entry<K,V>> {
        public final Map.Entry<K,V> next() { return nextNode(); }
    }
```

### ConcurrentHashMap


----------



我们上述所讲的Map都是非线程安全的，这意味着不应该在多个线程中对这些Map进行修改操作，轻则会产生数据不一致的问题，甚至还会因为并发插入元素而导致链表成环（插入会触发扩容，而扩容操作需要将原数组中的元素rehash到新数组，这时并发操作就有可能产生链表的循环引用从而成环），这样在查找时就会发生死循环，影响到整个应用程序。

`Collections.synchronizedMap(Map<K,V> m)`可以将一个Map转换成线程安全的实现，其实也就是通过一个包装类，然后把所有功能都委托给传入的Map实现，而且包装类是基于`synchronized`关键字来保证线程安全的（时代的眼泪Hashtable也是基于`synchronized`关键字），底层使用的是互斥锁（同一时间内只能由持有锁的线程访问，其他竞争线程进入睡眠状态），性能与吞吐量差强人意。

```java
    public static <K,V> Map<K,V> synchronizedMap(Map<K,V> m) {
        return new SynchronizedMap<>(m);
    }
    
    private static class SynchronizedMap<K,V>
        implements Map<K,V>, Serializable {
        private static final long serialVersionUID = 1978198479659022715L;

        private final Map<K,V> m;     // Backing Map
        final Object      mutex;        // Object on which to synchronize

        SynchronizedMap(Map<K,V> m) {
            this.m = Objects.requireNonNull(m);
            mutex = this;
        }

        SynchronizedMap(Map<K,V> m, Object mutex) {
            this.m = m;
            this.mutex = mutex;
        }

        public int size() {
            synchronized (mutex) {return m.size();}
        }
        public boolean isEmpty() {
            synchronized (mutex) {return m.isEmpty();}
        }

        ............
    }
```

然而ConcurrentHashMap的实现细节远没有这么简单，因此性能也要高上许多。它没有使用一个全局锁来锁住自己，而是采用了减少锁粒度的方法，尽量减少因为竞争锁而导致的阻塞与冲突，而且ConcurrentHashMap的检索操作是不需要锁的。

在Java 7中，ConcurrentHashMap把内部细分成了若干个小的HashMap，称之为段（Segment），默认被分为16个段。对于一个写操作而言，会先根据hash code进行寻址，得出该Entry应被存放在哪一个Segment，然后只要对该Segment加锁即可。

![](http://wx4.sinaimg.cn/large/63503acbly1fpu3a98u3fj20uf0i03z8.jpg)

理想情况下，一个默认的ConcurrentHashMap可以同时接受16个线程进行写操作（如果都是对不同Segment进行操作的话）。

分段锁对于`size()`这样的全局操作来说就没有任何作用了，想要得出Entry的数量就需要遍历所有Segment，获得所有的锁，然后再统计总数。事实上，ConcurrentHashMap会先试图使用无锁的方式统计总数，这个尝试会进行3次，如果在相邻的2次计算中获得的Segment的modCount次数一致，代表这两次计算过程中都没有发生过修改操作，那么就可以当做最终结果返回，否则，就要获得所有Segment的锁，重新计算size。

本文主要讨论的是Java 8的ConcurrentHashMap，它与Java 7的实现差别较大。完全放弃了段的设计，而是变回与HashMap相似的设计，使用buckets数组与分离链接法（同样会在超过阈值时树化，对于构造红黑树的逻辑与HashMap差别不大，只不过需要额外使用CAS来保证线程安全），锁的粒度也被细分到每个数组元素（个人认为这样做的原因是因为HashMap在Java 8中也实现了不少优化，即使碰撞严重，也能保证一定的性能，而且Segment不仅臃肿还有弱一致性的问题存在），所以它的并发级别与数组长度相关（Java 7则是与段数相关）。

```java
    /**
     * The array of bins. Lazily initialized upon first insertion.
     * Size is always a power of two. Accessed directly by iterators.
     */
    transient volatile Node<K,V>[] table;
```

#### 寻址


----------



ConcurrentHashMap的散列函数与HashMap并没有什么区别，同样是把key的hash code的高16位与低16位进行异或运算（因为ConcurrentHashMap的buckets数组长度也永远是一个2的N次方），然后将扰乱后的hash code与数组的长度减一（实际可访问到的最大索引）进行与运算，得出的结果即是目标所在的位置。

```java
    // 2^31 - 1，int类型的最大值
    // 该掩码表示节点hash的可用位，用来保证hash永远为一个正整数
    static final int HASH_BITS = 0x7fffffff;

    static final int spread(int h) {
        return (h ^ (h >>> 16)) & HASH_BITS;
    }
```

下面是查找操作的源码，实现比较简单。

```java
    public V get(Object key) {
        Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
        int h = spread(key.hashCode());
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (e = tabAt(tab, (n - 1) & h)) != null) {
            if ((eh = e.hash) == h) {
                // 先尝试判断链表头是否为目标，如果是就直接返回
                if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                    return e.val;
            }
            else if (eh < 0)
                // eh < 0代表这是一个特殊节点（TreeBin或ForwardingNode）
                // 所以直接调用find()进行遍历查找
                return (p = e.find(h, key)) != null ? p.val : null;
            // 遍历链表
            while ((e = e.next) != null) {
                if (e.hash == h &&
                    ((ek = e.key) == key || (ek != null && key.equals(ek))))
                    return e.val;
            }
        }
        return null;
    }
```

一个普通的节点（链表节点）的hash不可能小于0（已经在`spread()`函数中修正过了），所以小于0的只可能是一个特殊节点，它不能用while循环中遍历链表的方式来进行遍历。

TreeBin是红黑树的头部节点（红黑树的节点为TreeNode），它本身不含有key与value，而是指向一个TreeNode节点的链表与它们的根节点，同时使用CAS（ConcurrentHashMap并不是完全基于互斥锁实现的，而是与CAS这种乐观策略搭配使用，以提高性能）实现了一个读写锁，迫使Writer（持有这个锁）在树重构操作之前等待Reader完成。

ForwardingNode是一个在数据转移过程（由扩容引起）中使用的临时节点，它会被插入到头部。它与TreeBin（和TreeNode）都是Node类的子类。

为了判断出哪些是特殊节点，TreeBin和ForwardingNode的hash域都只是一个虚拟值：

```java
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        volatile V val;
        volatile Node<K,V> next;

        Node(int hash, K key, V val, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.val = val;
            this.next = next;
        }

        public final V setValue(V value) {
            throw new UnsupportedOperationException();
        }

        ......

        /**
         * Virtualized support for map.get(); overridden in subclasses.
         */
        Node<K,V> find(int h, Object k) {
            Node<K,V> e = this;
            if (k != null) {
                do {
                    K ek;
                    if (e.hash == h &&
                        ((ek = e.key) == k || (ek != null && k.equals(ek))))
                        return e;
                } while ((e = e.next) != null);
            }
            return null;
        }
    }

    /*
     * Encodings for Node hash fields. See above for explanation.
     */
    static final int MOVED     = -1; // hash for forwarding nodes
    static final int TREEBIN   = -2; // hash for roots of trees
    static final int RESERVED  = -3; // hash for transient reservations    

    static final class TreeBin<K,V> extends Node<K,V> {
        ....

        TreeBin(TreeNode<K,V> b) {
            super(TREEBIN, null, null, null);
            ....
        }   
        
        ....     
    }

    static final class ForwardingNode<K,V> extends Node<K,V> {
        final Node<K,V>[] nextTable;
        ForwardingNode(Node<K,V>[] tab) {
            super(MOVED, null, null, null);
            this.nextTable = tab;
        }

        .....
    }        
```


#### 可见性


----------



我们在`get()`函数中并没有发现任何与锁相关的代码，那么它是怎么保证线程安全的呢？一个操作`ConcurrentHashMap.get("a")`，它的步骤基本分为以下几步：

 - 根据散列函数计算出的索引访问table。

 - 从table中取出头节点。

 - 遍历头节点直到找到目标节点。

 - 从目标节点中取出value并返回。

所以只要保证访问table与节点的操作总是能够返回最新的数据就可以了。ConcurrentHashMap并没有采用锁的方式，而是通过`volatile`关键字来保证它们的可见性。在上文贴出的代码中可以发现，table、Node.val和Node.next都是被`volatile`关键字所修饰的。

`volatile`关键字保证了多线程环境下变量的可见性与有序性，底层实现基于内存屏障（Memory Barrier）。

为了优化性能，现代CPU工作时的指令执行顺序与应用程序的代码顺序其实是不一致的（有些编译器也会进行这种优化），也就是所谓的乱序执行技术。乱序执行可以提高CPU流水线的工作效率，只要保证数据符合程序逻辑上的正确性即可（遵循`happens-before`原则）。不过如今是多核时代，如果随便乱序而不提供防护措施那是会出问题的。每一个cpu上都会进行乱序优化，单cpu所保证的逻辑次序可能会被其他cpu所破坏。

内存屏障就是针对此情况的防护措施。可以认为它是一个同步点（但它本身也是一条cpu指令）。例如在`IA32`指令集架构中引入的`SFENCE`指令，在该指令之前的所有写操作必须全部完成，读操作仍可以乱序执行。`LFENCE`指令则保证之前的所有读操作必须全部完成，另外还有粒度更粗的`MFENCE`指令保证之前的所有读写操作都必须全部完成。

内存屏障就像是一个保护指令顺序的栅栏，保护后面的指令不被前面的指令跨越。将内存屏障插入到写操作与读操作之间，就可以保证之后的读操作可以访问到最新的数据，因为屏障前的写操作已经把数据写回到内存（根据缓存一致性协议，不会直接写回到内存，而是改变该cpu私有缓存中的状态，然后通知给其他cpu这个缓存行已经被修改过了，之后另一个cpu在读操作时就可以发现该缓存行已经是无效的了，这时它会从其他cpu中读取最新的缓存行，然后之前的cpu才会更改状态并写回到内存）。

例如，读一个被`volatile`修饰的变量V总是能够从JMM（Java Memory Model）主内存中获得最新的数据。因为内存屏障的原因，每次在使用变量V（通过JVM指令`use`，后面说的也都是JVM中的指令而不是cpu）之前都必须先执行`load`指令（把从主内存中得到的数据放入到工作内存），根据JVM的规定，`load`指令必须发生在`read`指令（从主内存中读取数据）之后，所以每次访问变量V都会先从主内存中读取。相对的，写操作也因为内存屏障保证的指令顺序，每次都会直接写回到主内存。

不过`volatile`关键字并不能保证操作的原子性，对该变量进行并发的连续操作是非线程安全的，所幸ConcurrentHashMap只是用来确保访问到的变量是最新的，所以也不会发生什么问题。

出于性能考虑，Doug Lea（`java.util.concurrent`包的作者）直接通过Unsafe类来对table进行操作。

Java号称是安全的编程语言，而保证安全的代价就是牺牲程序员自由操控内存的能力。像在C/C++中可以通过操作指针变量达到操作内存的目的（其实操作的是虚拟地址），但这种灵活性在新手手中也经常会带来一些愚蠢的错误，比如内存访问越界。

Unsafe从字面意思可以看出是不安全的，它包含了许多本地方法（在JVM平台上运行的其他语言编写的程序，主要为C/C++，由`JNI`实现），这些方法支持了对指针的操作，所以它才被称为是不安全的。虽然不安全，但毕竟是由C/C++实现的，像一些与操作系统交互的操作肯定是快过Java的，毕竟Java与操作系统之间还隔了一层抽象（JVM），不过代价就是失去了JVM所带来的多平台可移植性（本质上也只是一个c/cpp文件，如果换了平台那就要重新编译）。

对table进行操作的函数有以下三个，都使用到了Unsafe（在`java.util.concurrent`包随处可见）：

```java
    @SuppressWarnings("unchecked")
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
        // 从tab数组中获取一个引用，遵循Volatile语义
        // 参数2是一个在tab中的偏移量，用来寻找目标对象
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }

    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        // 通过CAS操作将tab数组中位于参数2偏移量位置的值替换为v
        // c是期望值，如果期望值与实际值不符，返回false
        // 否则，v会成功地被设置到目标位置，返回true
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }

    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
        // 设置tab数组中位于参数2偏移量位置的值，遵循Volatile语义
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
    }
```

如果对Unsafe感兴趣，可以参考这篇文章：[Java Magic. Part 4: sun.misc.Unsafe][6]

#### 初始化


----------



ConcurrentHashMap与HashMap一样是Lazy的，buckets数组会在第一次访问`put()`函数时进行初始化，它的默认构造函数甚至是个空函数。

```java
    /**
     * Creates a new, empty map with the default initial table size (16).
     */
    public ConcurrentHashMap() {
    }
```

但是有一点需要注意，ConcurrentHashMap是工作在多线程并发环境下的，如果有多个线程同时调用了`put()`函数该怎么办？这会导致重复初始化，所以必须要有对应的防护措施。

ConcurrentHashMap声明了一个用于控制table的初始化与扩容的实例变量sizeCtl，默认值为0。当它是一个负数的时候，代表table正处于初始化或者扩容的状态。`-1`表示table正在进行初始化，`-N`则表示当前有N-1个线程正在进行扩容。

在其他情况下，如果table还未初始化（`table == null`），sizeCtl表示table进行初始化的数组大小（所以从构造函数传入的initialCapacity在经过计算后会被赋给它）。如果table已经初始化过了，则表示下次触发扩容操作的阈值，算法`stzeCtl = n - (n >>> 2)`，也就是n的75%，与默认负载因子（0.75）的HashMap一致。

```java
    private transient volatile int sizeCtl;
```

初始化table的操作位于函数`initTable()`，源码如下：

```java
    /**
     * Initializes table, using the size recorded in sizeCtl.
     */
    private final Node<K,V>[] initTable() {
        Node<K,V>[] tab; int sc;
        while ((tab = table) == null || tab.length == 0) {
            // sizeCtl小于0，这意味着已经有其他线程进行初始化了
            // 所以当前线程让出CPU时间片
            if ((sc = sizeCtl) < 0)
                Thread.yield(); // lost initialization race; just spin
            // 否则，通过CAS操作尝试修改sizeCtl
            else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if ((tab = table) == null || tab.length == 0) {
                        // 默认构造函数，sizeCtl = 0，使用默认容量（16）进行初始化
                        // 否则，会根据sizeCtl进行初始化
                        int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                        @SuppressWarnings("unchecked")
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = tab = nt;
                        // 计算阈值，n的75%
                        sc = n - (n >>> 2);
                    }
                } finally {
                    // 阈值赋给sizeCtl
                    sizeCtl = sc;
                }
                break;
            }
        }
        return tab;
    }
```

sizeCtl是一个`volatile`变量，只要有一个线程CAS操作成功，sizeCtl就会被暂时地修改为-1，这样其他线程就能够根据sizeCtl得知table是否已经处于初始化状态中，最后sizeCtl会被设置成阈值，用于触发扩容操作。


#### 扩容


----------



ConcurrentHashMap触发扩容的时机与HashMap类似，要么是在将链表转换成红黑树时判断table数组的长度是否小于阈值（64），如果小于就进行扩容而不是树化，要么就是在添加元素的时候，判断当前Entry数量是否超过阈值，如果超过就进行扩容。

```java
    private final void treeifyBin(Node<K,V>[] tab, int index) {
        Node<K,V> b; int n, sc;
        if (tab != null) {
            // 小于MIN_TREEIFY_CAPACITY，进行扩容
            if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
                tryPresize(n << 1);
            else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
                synchronized (b) {
                    // 将链表转换成红黑树...
                }
            }
        }
    }

    ...

    final V putVal(K key, V value, boolean onlyIfAbsent) {
        ...
        addCount(1L, binCount); // 计数
        return null;
    }

    private final void addCount(long x, int check) {
        // 计数...
        if (check >= 0) {
            Node<K,V>[] tab, nt; int n, sc;
            // s(元素个数)大于等于sizeCtl，触发扩容
            while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
                   (n = tab.length) < MAXIMUM_CAPACITY) {
                // 扩容标志位
                int rs = resizeStamp(n);
                // sizeCtl为负数，代表正有其他线程进行扩容
                if (sc < 0) {
                    // 扩容已经结束，中断循环
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    // 进行扩容，并设置sizeCtl，表示扩容线程 + 1
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                // 触发扩容（第一个进行扩容的线程）
                // 并设置sizeCtl告知其他线程
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
                // 统计个数，用于循环检测是否还需要扩容
                s = sumCount();
            }
        }
    }
```

可以看到有关sizeCtl的操作牵涉到了大量的位运算，我们先来理解这些位运算的意义。首先是`resizeStamp()`，该函数返回一个用于数据校验的标志位，意思是对长度为n的table进行扩容。它将n的前导零（最高有效位之前的零的数量）和`1 << 15`做或运算，这时低16位的最高位为1，其他都为n的前导零。

```java
    static final int resizeStamp(int n) {
        // RESIZE_STAMP_BITS = 16
        return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
    }
```

初始化sizeCtl（扩容操作被第一个线程首次进行）的算法为`(rs << RESIZE_STAMP_SHIFT) + 2`，首先`RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS = 16`，那么`rs << 16`等于将这个标志位移动到了高16位，这时最高位为1，所以sizeCtl此时是个负数，然后加二（至于为什么是2，还记得有关sizeCtl的说明吗？1代表初始化状态，所以实际的线程个数是要减去1的）代表当前有一个线程正在进行扩容，

这样sizeCtl就被分割成了两部分，高16位是一个对n的数据校验的标志位，低16位表示参与扩容操作的线程个数 + 1。

可能会有读者有所疑惑，更新进行扩容的线程数量的操作为什么是`sc + 1`而不是`sc - 1`，这是因为对sizeCtl的操作都是基于位运算的，所以不会关心它本身的数值是多少，只关心它在二进制上的数值，而`sc + 1`会在低16位上加1。

![](http://wx1.sinaimg.cn/large/63503acbly1fpu3a9r1q0j20vz0gwta5.jpg)

`tryPresize()`函数跟`addCount()`的后半段逻辑类似，不断地根据sizeCtl判断当前的状态，然后选择对应的策略。

```java
    private final void tryPresize(int size) {
        // 对size进行修正
        int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
            tableSizeFor(size + (size >>> 1) + 1);
        int sc;
        // sizeCtl是默认值或正整数
        // 代表table还未初始化
        // 或还没有其他线程正在进行扩容
        while ((sc = sizeCtl) >= 0) {
            Node<K,V>[] tab = table; int n;
            if (tab == null || (n = tab.length) == 0) {
                n = (sc > c) ? sc : c;
                // 设置sizeCtl，告诉其他线程，table现在正处于初始化状态
                if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                    try {
                        if (table == tab) {
                            @SuppressWarnings("unchecked")
                            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                            table = nt;
                            // 计算下次触发扩容的阈值
                            sc = n - (n >>> 2);
                        }
                    } finally {
                        // 将阈值赋给sizeCtl
                        sizeCtl = sc;
                    }
                }
            }
            // 没有超过阈值或者大于容量的上限，中断循环
            else if (c <= sc || n >= MAXIMUM_CAPACITY)
                break;
            // 进行扩容，与addCount()后半段的逻辑一致
            else if (tab == table) {
                int rs = resizeStamp(n);
                if (sc < 0) {
                    Node<K,V>[] nt;
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                        sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
            }
        }
    }
```

扩容操作的核心在于数据的转移，在单线程环境下数据的转移很简单，无非就是把旧数组中的数据迁移到新的数组。但是这在多线程环境下是行不通的，需要保证线程安全性，在扩容的时候其他线程也可能正在添加元素，这时又触发了扩容怎么办？有人可能会说，这不难啊，用一个互斥锁把数据转移操作的过程锁住不就好了？这确实是一种可行的解决方法，但同样也会带来极差的吞吐量。	

互斥锁会导致所有访问临界区的线程陷入阻塞状态，这会消耗额外的系统资源，内核需要保存这些线程的上下文并放到阻塞队列，持有锁的线程耗时越长，其他竞争线程就会一直被阻塞，因此吞吐量低下，导致响应时间缓慢。而且锁总是会伴随着死锁问题，一旦发生死锁，整个应用程序都会因此受到影响，所以加锁永远是最后的备选方案。

Doug Lea没有选择直接加锁，而是基于CAS实现无锁的并发同步策略，令人佩服的是他不仅没有把其他线程拒之门外，甚至还邀请它们一起来协助工作。

那么如何才能让多个线程协同工作呢？Doug Lea把整个table数组当做多个线程之间共享的任务队列，然后只需维护一个指针，当有一个线程开始进行数据转移，就会先移动指针，表示指针划过的这片bucket区域由该线程负责。

这个指针被声明为一个`volatile`整型变量，它的初始位置位于table的尾部，即它等于`table.length`，很明显这个任务队列是逆向遍历的。

```java
    /**
     * The next table index (plus one) to split while resizing.
     */
    private transient volatile int transferIndex;

    /**
     * 一个线程需要负责的最小bucket数
     */
    private static final int MIN_TRANSFER_STRIDE = 16;
	
    /**
     * The next table to use; non-null only while resizing.
     */
    private transient volatile Node<K,V>[] nextTable;	
```

一个已经迁移完毕的bucket会被替换成ForwardingNode节点，用来标记此bucket已经被其他线程迁移完毕了。我们之前提到过ForwardingNode，它是一个特殊节点，可以通过hash域的虚拟值来识别它，它同样重写了`find()`函数，用来在新数组中查找目标。

数据迁移的操作位于`transfer()`函数，多个线程之间依靠sizeCtl与transferIndex指针来协同工作，每个线程都有自己负责的区域，一个完成迁移的bucket会被设置为ForwardingNode，其他线程遇见这个特殊节点就跳过该bucket，处理下一个bucket。

`transfer()`函数可以大致分为三部分，第一部分对后续需要使用的变量进行初始化：

```java
    /**
     * Moves and/or copies the nodes in each bin to new table. See
     * above for explanation.
     */
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
        int n = tab.length, stride;
        // 根据当前机器的CPU数量来决定每个线程负责的bucket数
        // 避免因为扩容线程过多，反而影响到性能
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        // 初始化nextTab，容量为旧数组的一倍
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n; // 初始化指针
        }
        int nextn = nextTab.length;
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        boolean advance = true;
        boolean finishing = false; // to ensure sweep before committing nextTab
```

第二部分为当前线程分配任务和控制当前线程的任务进度，这部分是`transfer()`的核心逻辑，描述了如何与其他线程协同工作：

```java
        // i指向当前bucket，bound表示当前线程所负责的bucket区域的边界
        for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            // 这个循环使用CAS不断尝试为当前线程分配任务
            // 直到分配成功或任务队列已经被全部分配完毕
            // 如果当前线程已经被分配过bucket区域
            // 那么会通过--i指向下一个待处理bucket然后退出该循环
            while (advance) {
                int nextIndex, nextBound;
                // --i表示将i指向下一个待处理的bucket
                // 如果--i >= bound，代表当前线程已经分配过bucket区域
                // 并且还留有未处理的bucket
                if (--i >= bound || finishing)
                    advance = false;
                // transferIndex指针 <= 0 表示所有bucket已经被分配完毕
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                // 移动transferIndex指针
                // 为当前线程设置所负责的bucket区域的范围
                // i指向该范围的第一个bucket，注意i是逆向遍历的
                // 这个范围为(bound, i)，i是该区域最后一个bucket，遍历顺序是逆向的
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
            // 当前线程已经处理完了所负责的所有bucket
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                // 如果任务队列已经全部完成
                if (finishing) {
                    nextTable = null;
                    table = nextTab;
                    // 设置新的阈值
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                // 工作中的扩容线程数量减1
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    // (resizeStamp << RESIZE_STAMP_SHIFT) + 2代表当前有一个扩容线程
                    // 相对的，(sc - 2) !=  resizeStamp << RESIZE_STAMP_SHIFT
                    // 表示当前还有其他线程正在进行扩容，所以直接返回
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    // 否则，当前线程就是最后一个进行扩容的线程
                    // 设置finishing标识
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            // 如果待处理bucket是空的
            // 那么插入ForwardingNode，以通知其他线程
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            // 如果待处理bucket的头节点是ForwardingNode
            // 说明此bucket已经被处理过了，跳过该bucket
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
```

最后一部分是具体的迁移过程（对当前指向的bucket），这部分的逻辑与HashMap类似，拿旧数组的容量当做一个掩码，然后与节点的hash进行与操作，可以得出该节点的新增有效位，如果新增有效位为0就放入一个链表A，如果为1就放入另一个链表B，链表A在新数组中的位置不变（跟在旧数组的索引一致），链表B在新数组中的位置为原索引加上旧数组容量。

这个方法减少了rehash的计算量，而且还能达到均匀分布的目的，如果不能理解请去看本文中HashMap扩容操作的解释。

```java
            else {
                // 对于节点的操作还是要加上锁的
                // 不过这个锁的粒度很小，只锁住了bucket的头节点
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        // hash code不为负，代表这是条链表
                        if (fh >= 0) {
                            // fh & n 获得hash code的新增有效位，用于将链表分离成两类
                            // 要么是0要么是1，关于这个位运算的更多细节
                            // 请看本文中有关HashMap扩容操作的解释
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            // 这个循环用于记录最后一段连续的同一类节点
                            // 这个类别是通过fh & n来区分的
                            // 这段连续的同类节点直接被复用，不会产生额外的复制
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            // 0被放入ln链表，1被放入hn链表
                            // lastRun是连续同类节点的起始节点
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            // 将最后一段的连续同类节点之前的节点按类别复制到ln或hn
                            // 链表的插入方向是往头部插入的，Node构造函数的第四个参数是next
                            // 所以就算遇到类别与lastRun一致的节点也只会被插入到头部
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            // ln链表被放入到原索引位置，hn放入到原索引 + 旧数组容量
                            // 这一点与HashMap一致，如果看不懂请去参考本文对HashMap扩容的讲解
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd); // 标记该bucket已被处理
                            advance = true;
                        }
                        // 对红黑树的操作，逻辑与链表一样，按新增有效位进行分类
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            // 元素数量没有超过UNTREEIFY_THRESHOLD，退化成链表
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
```

#### 计数


----------


在Java 7中ConcurrentHashMap对每个Segment单独计数，想要得到总数就需要获得所有Segment的锁，然后进行统计。由于Java 8抛弃了Segment，显然是不能再这样做了，而且这种方法虽然简单准确但也舍弃了性能。

Java 8声明了一个`volatile`变量baseCount用于记录元素的个数，对这个变量的修改操作是基于CAS的，每当插入元素或删除元素时都会调用`addCount()`函数进行计数。

```java
    private transient volatile long baseCount;

    private final void addCount(long x, int check) {
        CounterCell[] as; long b, s;
        // 尝试使用CAS更新baseCount失败
        // 转用CounterCells进行更新
        if ((as = counterCells) != null ||
            !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
            CounterCell a; long v; int m;
            boolean uncontended = true;
            // 在CounterCells未初始化
            // 或尝试通过CAS更新当前线程的CounterCell失败时
            // 调用fullAddCount()，该函数负责初始化CounterCells和更新计数
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
                !(uncontended =
                  U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
                fullAddCount(x, uncontended);
                return;
            }
            if (check <= 1)
                return;
            // 统计总数
            s = sumCount();
        }
        if (check >= 0) {
        	// 判断是否需要扩容，在上文中已经讲过了
        }
    }    
```

counterCells是一个元素为CounterCell的数组，该数组的大小与当前机器的CPU数量有关，并且它不会被主动初始化，只有在调用`fullAddCount()`函数时才会进行初始化。

CounterCell是一个简单的内部静态类，每个CounterCell都是一个用于记录数量的单元：

```java
    /**
     * Table of counter cells. When non-null, size is a power of 2.
     */
    private transient volatile CounterCell[] counterCells;

    /**
     * A padded cell for distributing counts.  Adapted from LongAdder
     * and Striped64.  See their internal docs for explanation.
     */
    @sun.misc.Contended static final class CounterCell {
        volatile long value;
        CounterCell(long x) { value = x; }
    }
```

注解`@sun.misc.Contended`用于解决伪共享问题。所谓伪共享，即是在同一缓存行（CPU缓存的基本单位）中存储了多个变量，当其中一个变量被修改时，就会影响到同一缓存行内的其他变量，导致它们也要跟着被标记为失效，其他变量的缓存命中率将会受到影响。解决伪共享问题的方法一般是对该变量填充一些无意义的占位数据，从而使它独享一个缓存行。

ConcurrentHashMap的计数设计与LongAdder类似。在一个低并发的情况下，就只是简单地使用CAS操作来对baseCount进行更新，但只要这个CAS操作失败一次，就代表有多个线程正在竞争，那么就转而使用CounterCell数组进行计数，数组内的每个ConuterCell都是一个独立的计数单元。

每个线程都会通过`ThreadLocalRandom.getProbe() & m`寻址找到属于它的CounterCell，然后进行计数。ThreadLocalRandom是一个线程私有的伪随机数生成器，每个线程的probe都是不同的（这点基于ThreadLocalRandom的内部实现，它在内部维护了一个probeGenerator，这是一个类型为AtomicInteger的静态常量，每当初始化一个ThreadLocalRandom时probeGenerator都会先自增一个常量然后返回的整数即为当前线程的probe，probe变量被维护在Thread对象中），可以认为每个线程的probe就是它在CounterCell数组中的hash code。

这种方法将竞争数据按照线程的粒度进行分离，相比所有竞争线程对一个共享变量使用CAS不断尝试在性能上要效率多了，这也是为什么在高并发环境下LongAdder要优于AtomicInteger的原因。

`fullAddCount()`函数根据当前线程的probe寻找对应的CounterCell进行计数，如果CounterCell数组未被初始化，则初始化CounterCell数组和CounterCell。该函数的实现与Striped64类（LongAdder的父类）的`longAccumulate()`函数是一样的，把CounterCell数组当成一个散列表，每个线程的probe就是hash code，散列函数也仅仅是简单的`(n - 1) & probe`。

CounterCell数组的大小永远是一个2的n次方，初始容量为2，每次扩容的新容量都是之前容量乘以二，处于性能考虑，它的最大容量上限是机器的CPU数量。

所以说CounterCell数组的碰撞冲突是很严重的，因为它的bucket基数太小了。而发生碰撞就代表着一个CounterCell会被多个线程竞争，为了解决这个问题，Doug Lea使用无限循环加上CAS来模拟出一个自旋锁来保证线程安全，自旋锁的实现基于一个被`volatile`修饰的整数变量，该变量只会有两种状态：0和1，当它被设置为0时表示没有加锁，当它被设置为1时表示已被其他线程加锁。这个自旋锁用于保护初始化CounterCell、初始化CounterCell数组以及对CounterCell数组进行扩容时的安全。

CounterCell更新计数是依赖于CAS的，每次循环都会尝试通过CAS进行更新，如果成功就退出无限循环，否则就调用`ThreadLocalRandom.advanceProbe()`函数为当前线程更新probe，然后重新开始循环，以期望下一次寻址到的CounterCell没有被其他线程竞争。

如果连着两次CAS更新都没有成功，那么会对CounterCell数组进行一次扩容，这个扩容操作只会在当前循环中触发一次，而且只能在容量小于上限时触发。

`fullAddCount()`函数的主要流程如下：

 - 首先检查当前线程有没有初始化过ThreadLocalRandom，如果没有则进行初始化。ThreadLocalRandom负责更新线程的probe，而probe又是在数组中进行寻址的关键。

 - 检查CounterCell数组是否已经初始化，如果已初始化，那么就根据probe找到对应的CounterCell。

	 - 如果这个CounterCell等于null，需要先初始化CounterCell，通过把计数增量传入构造函数，所以初始化只要成功就说明更新计数已经完成了。初始化的过程需要获取自旋锁。

	 - 如果不为null，就按上文所说的逻辑对CounterCell实施更新计数。

 - CounterCell数组未被初始化，尝试获取自旋锁，进行初始化。数组初始化的过程会附带初始化一个CounterCell来记录计数增量，所以只要初始化成功就表示更新计数完成。

 - 如果自旋锁被其他线程占用，无法进行数组的初始化，只好通过CAS更新baseCount。


```java
    private final void fullAddCount(long x, boolean wasUncontended) {
        int h;
        // 当前线程的probe等于0，证明该线程的ThreadLocalRandom还未被初始化
        // 以及当前线程是第一次进入该函数
        if ((h = ThreadLocalRandom.getProbe()) == 0) {
            // 初始化ThreadLocalRandom，当前线程会被设置一个probe
            ThreadLocalRandom.localInit();      // force initialization
            // probe用于在CounterCell数组中寻址
            h = ThreadLocalRandom.getProbe();
            // 未竞争标志
            wasUncontended = true;
        }
        // 冲突标志
        boolean collide = false;                // True if last slot nonempty
        for (;;) {
            CounterCell[] as; CounterCell a; int n; long v;
            // CounterCell数组已初始化
            if ((as = counterCells) != null && (n = as.length) > 0) {
                // 如果寻址到的Cell为空，那么创建一个新的Cell
                if ((a = as[(n - 1) & h]) == null) {
                    // cellsBusy是一个只有0和1两个状态的volatile整数
                    // 它被当做一个自旋锁，0代表无锁，1代表加锁
                    if (cellsBusy == 0) {            // Try to attach new Cell
                        // 将传入的x作为初始值创建一个新的CounterCell
                        CounterCell r = new CounterCell(x); // Optimistic create
                        // 通过CAS尝试对自旋锁加锁
                        if (cellsBusy == 0 &&
                            U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                            // 加锁成功，声明Cell是否创建成功的标志
                            boolean created = false;
                            try {               // Recheck under lock
                                CounterCell[] rs; int m, j;
                                // 再次检查CounterCell数组是否不为空
                                // 并且寻址到的Cell为空
                                if ((rs = counterCells) != null &&
                                    (m = rs.length) > 0 &&
                                    rs[j = (m - 1) & h] == null) {
                                    // 将之前创建的新Cell放入数组
                                    rs[j] = r;
                                    created = true;
                                }
                            } finally {
                                // 释放锁
                                cellsBusy = 0;
                            }
                            // 如果已经创建成功，中断循环
                            // 因为新Cell的初始值就是传入的增量，所以计数已经完毕了
                            if (created)
                                break;
                            // 如果未成功
                            // 代表as[(n - 1) & h]这个位置的Cell已经被其他线程设置
                            // 那么就从循环头重新开始
                            continue;           // Slot is now non-empty
                        }
                    }
                    collide = false;
                }
                // as[(n - 1) & h]非空
                // 在addCount()函数中通过CAS更新当前线程的Cell进行计数失败
                // 会传入wasUncontended = false，代表已经有其他线程进行竞争
                else if (!wasUncontended)       // CAS already known to fail
                    // 设置未竞争标志，之后会重新计算probe，然后重新执行循环
                    wasUncontended = true;      // Continue after rehash
                // 尝试进行计数，如果成功，那么就退出循环
                else if (U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))
                    break;
                // 尝试更新失败，检查counterCell数组是否已经扩容
                // 或者容量达到最大值（CPU的数量）
                else if (counterCells != as || n >= NCPU)
                    // 设置冲突标志，防止跳入下面的扩容分支
                    // 之后会重新计算probe
                    collide = false;            // At max size or stale
                // 设置冲突标志，重新执行循环
                // 如果下次循环执行到该分支，并且冲突标志仍然为true
                // 那么会跳过该分支，到下一个分支进行扩容
                else if (!collide)
                    collide = true;
                // 尝试加锁，然后对counterCells数组进行扩容
                else if (cellsBusy == 0 &&
                         U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                    try {
                        // 检查是否已被扩容
                        if (counterCells == as) {// Expand table unless stale
                            // 新数组容量为之前的1倍
                            CounterCell[] rs = new CounterCell[n << 1];
                            // 迁移数据到新数组
                            for (int i = 0; i < n; ++i)
                                rs[i] = as[i];
                            counterCells = rs;
                        }
                    } finally {
                        // 释放锁
                        cellsBusy = 0;
                    }
                    collide = false;
                    // 重新执行循环
                    continue;                   // Retry with expanded table
                }
                // 为当前线程重新计算probe
                h = ThreadLocalRandom.advanceProbe(h);
            }
            // CounterCell数组未初始化，尝试获取自旋锁，然后进行初始化
            else if (cellsBusy == 0 && counterCells == as &&
                     U.compareAndSwapInt(this, CELLSBUSY, 0, 1)) {
                boolean init = false;
                try {                           // Initialize table
                    if (counterCells == as) {
                        // 初始化CounterCell数组，初始容量为2
                        CounterCell[] rs = new CounterCell[2];
                        // 初始化CounterCell
                        rs[h & 1] = new CounterCell(x);
                        counterCells = rs;
                        init = true;
                    }
                } finally {
                    cellsBusy = 0;
                }
                // 初始化CounterCell数组成功，退出循环
                if (init)
                    break;
            }
            // 如果自旋锁被占用，则只好尝试更新baseCount
            else if (U.compareAndSwapLong(this, BASECOUNT, v = baseCount, v + x))
                break;                          // Fall back on using base
        }
    }
```

对于统计总数，只要能够理解CounterCell的思想，就很简单了。仔细想一想，每次计数的更新都会被分摊在baseCount和CounterCell数组中的某一CounterCell，想要获得总数，把它们统计相加就是了。

```java
    public int size() {
        long n = sumCount();
        return ((n < 0L) ? 0 :
                (n > (long)Integer.MAX_VALUE) ? Integer.MAX_VALUE :
                (int)n);
    }

     final long sumCount() {
        CounterCell[] as = counterCells; CounterCell a;
        long sum = baseCount;
        if (as != null) {
            for (int i = 0; i < as.length; ++i) {
                if ((a = as[i]) != null)
                    sum += a.value;
            }
        }
        return sum;
    }   
```

其实`size()`函数返回的总数可能并不是百分百精确的，试想如果前一个遍历过的CounterCell又进行了更新会怎么样？尽管只是一个估算值，但在大多数场景下都还能接受，而且性能上是要比Java 7好上太多了。

#### 添加元素


----------



添加元素的主要逻辑与HashMap没什么区别，有所区别的复杂操作如扩容和计数我们上文都已经深入解析过了，所以整体来说`putVal()`函数还是比较简单的，可能唯一需要注意的就是在对节点进行操作的时候需要通过互斥锁保证线程安全，这个互斥锁的粒度很小，只对需要操作的这个bucket加锁。

```java
    public V put(K key, V value) {
        return putVal(key, value, false);
    }

    /** Implementation for put and putIfAbsent */
    final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0; // 节点计数器，用于判断是否需要树化
        // 无限循环+CAS，无锁的标准套路
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            // 初始化table
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            // bucket为null，通过CAS创建头节点，如果成功就结束循环
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            // bucket为ForwardingNode
            // 当前线程前去协助进行扩容
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        // 节点是链表
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                // 找到目标，设置value
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                // 未找到节点，插入新节点到链表尾部
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        // 节点是红黑树
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                // 根据bucket中的节点数决定是否树化
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    // oldVal不等于null，说明没有新节点
                    // 所以直接返回，不进行计数
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        // 计数
        addCount(1L, binCount);
        return null;
    }
```

至于删除元素的操作位于函数`replaceNode(Object key, V value, Object cv)`，当`table[key].val`等于期望值cv时（或cv等于null），更新节点的值为value，如果value等于null，那么删除该节点。

`remove()`函数通过调用`replaceNode(key, null, null)`来达成删除目标节点的目的，`replaceNode()`的具体实现与`putVal()`没什么差别，只不过对链表的操作有所不同而已，所以就不多叙述了。


#### 并行计算


----------



Java 8除了对ConcurrentHashMap重新设计以外，还引入了基于Lambda表达式的Stream API。它是对集合对象功能上的增强（所以不止ConcurrentHashMap，其他集合也都实现了该API），以一种优雅的方式来批量操作、聚合或遍历集合中的数据。

最重要的是，它还提供了并行模式，充分利用了多核CPU的优势实现并行计算。让我们看看如下的示例代码：

```java
    public static void main(String[] args) {
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
        String keys = "ABCDEFG";
        for (int i = 1; i <= keys.length(); i++) {
            map.put(String.valueOf(keys.charAt(i - 1)), i);
        }

        map.forEach(2,
                (k, v) -> System.out.println("key-" + k + ":value-" + v + ". by thread->" + Thread.currentThread().getName()));
    }
```

这段代码通过两个线程（包括主线程）并行地遍历map中的元素，然后输出到控制台，输出如下：

```
key-A:value-1. by thread->main
key-D:value-4. by thread->ForkJoinPool.commonPool-worker-2
key-B:value-2. by thread->main
key-E:value-5. by thread->ForkJoinPool.commonPool-worker-2
key-C:value-3. by thread->main
key-F:value-6. by thread->ForkJoinPool.commonPool-worker-2
key-G:value-7. by thread->ForkJoinPool.commonPool-worker-2
```

很明显，有两个线程在进行工作，那么这是怎么实现的呢？我们先来看看`forEach()`函数：

```java
    public void forEach(long parallelismThreshold,
                        BiConsumer<? super K,? super V> action) {
        if (action == null) throw new NullPointerException();
        new ForEachMappingTask<K,V>
            (null, batchFor(parallelismThreshold), 0, 0, table,
             action).invoke();
    }
```

`parallelismThreshold`是需要并行执行该操作的线程数量，`action`则是回调函数（我们想要执行的操作）。`action`的类型为BiConsumer，是一个用于支持Lambda表达式的FunctionalInterface，它接受两个输入参数并返回0个结果。

```java
@FunctionalInterface
public interface BiConsumer<T, U> {

    /**
     * Performs this operation on the given arguments.
     *
     * @param t the first input argument
     * @param u the second input argument
     */
    void accept(T t, U u);
```

看来实现并行计算的关键在于ForEachMappingTask对象，通过它的继承关系结构图可以发现，ForEachMappingTask其实就是ForkJoinTask。

![](http://wx3.sinaimg.cn/large/63503acbly1fpu3aa1nokj20a70bs3ym.jpg)

集合的并行计算是基于Fork/Join框架实现的，工作线程交由ForkJoinPool线程池维护。它推崇分而治之的思想，将一个大的任务分解成多个小的任务，通过`fork()`函数（有点像Linux的`fork()`系统调用来创建子进程）来开启一个工作线程执行其中一个小任务，通过`join()`函数等待工作线程执行完毕（需要等所有工作线程执行完毕才能合并最终结果），只要所有的小任务都已经处理完成，就代表这个大的任务也完成了。

像上文中的示例代码就是将遍历这个大任务分解成了N个小任务，然后交由两个工作线程进行处理。

```java
    static final class ForEachMappingTask<K,V>
        extends BulkTask<K,V,Void> {
        final BiConsumer<? super K, ? super V> action;
        ForEachMappingTask
            (BulkTask<K,V,?> p, int b, int i, int f, Node<K,V>[] t,
             BiConsumer<? super K,? super V> action) {
            super(p, b, i, f, t);
            this.action = action;
        }
        public final void compute() {
            final BiConsumer<? super K, ? super V> action;
            if ((action = this.action) != null) {
                for (int i = baseIndex, f, h; batch > 0 &&
                         (h = ((f = baseLimit) + i) >>> 1) > i;) {
                    // 记录待完成任务的数量
                    addToPendingCount(1);
                    // 开启一个工作线程执行任务
                    // 其余参数是任务的区间以及table和回调函数
                    new ForEachMappingTask<K,V>
                        (this, batch >>>= 1, baseLimit = h, f, tab,
                         action).fork();
                }
                for (Node<K,V> p; (p = advance()) != null; )
                    // 调用回调函数
                    action.accept(p.key, p.val);
                // 与addToPendingCount()相反
                // 它会减少待完成任务的计数器
                // 如果该计数器为0，代表所有任务已经完成了
                propagateCompletion();
            }
        }
    }
```

其他并行计算函数的实现也都差不多，只不过具体的Task实现不同，例如`search()`：

```java
    public <U> U search(long parallelismThreshold,
                        BiFunction<? super K, ? super V, ? extends U> searchFunction) {
        if (searchFunction == null) throw new NullPointerException();
        return new SearchMappingsTask<K,V,U>
            (null, batchFor(parallelismThreshold), 0, 0, table,
             searchFunction, new AtomicReference<U>()).invoke();
    }
```

为了节省篇幅（说实话现在似乎很少有人能耐心看完一篇长文_(:з」∠)_），有关Stream API是如何使用Fork/Join框架进行工作以及实现细节就不多讲了，以后有机会再说吧。


### 参考文献


----------



- [Associative array - Wikiwand][7]

- [Hash table - Wikiwand][8]

- [Hash function - Wikiwand][9]

- [ConcurrentHashMap (Java Platform SE 8 )][10]

- [LongAdder (Java Platform SE 8 )][11]

- [Java Magic. Part 4: sun.misc.Unsafe][6]

- [ConcurrentHashMap in Java 8 - DZone Java][12]


[1]:https://github.com/SylvanasSun
[2]: https://sylvanassun.github.io/
[3]: https://sylvanassun.github.io/2017/06/16/2017-06-16-RedBlackTree/
[4]: http://vanillajava.blogspot.com/2015/09/an-introduction-to-optimising-hashing.html
[5]: https://sylvanassun.github.io/2017/11/06/2017-11-06-spring_and_thread-safe/
[6]: http://mishadoff.com/blog/java-magic-part-4-sun-dot-misc-dot-unsafe/
[7]: http://www.wikiwand.com/en/Associative_array
[8]: http://www.wikiwand.com/en/Hash_table
[9]: http://www.wikiwand.com/en/Hash_function
[10]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html
[11]: https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/atomic/LongAdder.html
[12]: https://dzone.com/articles/concurrenthashmap-in-java8