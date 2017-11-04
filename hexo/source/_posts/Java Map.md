---
title: Java Map
date: 2017-09-26
tags: Java
categories: Java
---

Map是一个集合类的接口。

## HashMap

它实现了Map接口，存储键值对。底层实现是**散列表**，即数组加链表的结构。有几个重要的参数：

+ Size 当前Hashmap存储元素的个数

+ Capacity 容量，当前Hashmap最多能够发现多少元素

+ Load factor，决定什么时候对Hashmap进行拓容，当Size/Capacity > Load factor时，进行拓容。

+ ModCount, **fail fast**机制相关的参数。

什么是**快速失败**机制，java.util下的都是快速失败的，即如果当使用迭代器遍历一个集合的时候，其他线程修改了改集合，那么，他会马上抛出异常，与之相对应的是**快速安全**，其意思是修改不会使迭代器抛出异常（迭代器应该拷贝了一份集合来遍历）。那么，它是如何实现的？原理是每当对集合进行修改操作时，都会`ModCount++`，而在迭代器调用`next()`时，都会检查ModCount是否是预期的值。

### put

1. 对key的hashCode()做hash，然后再计算index;

2. 如果没碰撞直接放到bucket里；

3. 如果碰撞了，以链表的形式存在buckets后；

4. 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树；（在JDK8里，新增默认为8的閥值[TREEIFY_THRESHOLD]，当一个桶里的Entry超过閥值，就不以单向链表而以红黑树来存放以加快Key的查找速度）

5. 如果节点已经存在就替换old value(保证key的唯一性)

6. 如果bucket满了(超过load factor*current capacity)，就要resize

### get

1. bucket里的第一个节点，直接命中；

2. 如果有冲突，则通过key.equals(k)去查找对应的entry

3. 若为树，则在树中通过key.equals(k)查找，O(logn)；

4. 若为链表，则在链表中通过key.equals(k)查找，O(n)。

### hash函数

```
h = hashCode(); //调用key的hashCode函数
hash = h ^ (h>>>16); //计算hash值，高16bit不变，低16bit和高16bit做了一个异或。
index = (n - 1) & hash; //计算数组下标
```

n其实就是capacity，数组的元素个数，这里保证它是的2的n次方，使用与操作比求余操作性能更好

HashMap是不是线程安全的，多线程下回出现死锁的。而`Hashtable`是线程安全的，它的get, put函数都使用了`synchronzied`关键字修饰，由于要加锁，它的效率很低，一般都不适用，基本被废弃。

## TreeMap

同样是存储键值对，`TreeMap`底层使用了红黑树，最大的特点就是它能够基于键值的自然顺序进行拍讯。使得树具有不错的平衡性，这样操作的速度就可以达到log(n)的水平了。TreeMap的基本操作 containsKey、get、put 和 remove 的时间复杂度是 log(n) 

## LinkedHashMap

LinkedHashMap是Hash表和链表的实现，**并且依靠着双向链表保证了迭代顺序是插入的顺序**

## ConcurrentHashMap

以上的Map都是非线程安全的，当设计多线程时，需要手动实现互斥同步，而ConcurrentHashMap从名字上来看就知道它能够实现线程安全的Map

它的原理也是散列表，只不过进行了封装，新增了Segment。它的结构包含Segment 的数组，在默认的并发级别会创建包含 16 个 Segment 对象的数组。通过我们上面的知识，我们知道每个 Segment 又包含若干个散列表的桶，每个桶是由 HashEntry 链接起来的一个链表。

因此，在不同Segment里操作数据，可以并发，不用加锁，而在相同的Segment，自动实现了互斥同步。

## WeakHashmap

以弱键 实现的基于哈希表的 Map。在 WeakHashMap 中，当某个键不再正常使用时，将自动移除其条目。更精确地说，对于一个给定的键，其映射的存在并不阻止垃圾回收器对该键的丢弃，这就使该键成为可终止的，被终止，然后被回收。**WeekHashMap 的这个特点特别适用于需要缓存的场景**。
