---
title: Java 集合类
date: 2017-09-03 18:54:25
tags: Java
categories: Java
---

Java的集合类有两个大分支：`Collection` 和 `Map`

## Collection

容器是用于持有对象的。最好的持有对象的方法是**数组**，但是数组的长度固定。容器的顶层接口是Collections, 其中包含List，set接口， Map接口不属于Collections接口的。

### ArrayList

是动态数组，从类的定义上看
```
public class ArrayList<E> extends AbstractList<E>  implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

有几个特点:

+ 支持泛型，RandomAccess是一个标记接口，接口内没有定义任何内容。

+ 实现了Cloneable接口的类，可以调用Object.clone方法返回该对象的浅拷贝。

+ 通过实现 java.io.Serializable接口以启用其序列化功能。未实现此接口的类将无法使其任何状态序列化或反序列化。序列化接口没有方法或字段，仅用于标识可序列化的语义。

#### 属性

两个**私有属性**：
```
/**  
      * The array buffer into which the elements of the ArrayList are stored.  
      * The capacity of the ArrayList is the length of this array buffer.  
      */    
     private transient Object[] elementData;    
     
     /**  
      * The size of the ArrayList (the number of elements it contains).  
      *  
      * @serial  
      */    
     private int size;    
```

transient是关键字，用来表示一个域不是该对象串行化的一部分。

#### 构造函数

共有3个:
```
/**  
      * Constructs an empty list with the specified initial capacity.  
      */    
     public ArrayList(int initialCapacity) {    
     super();    
         if (initialCapacity < 0)    
             throw new IllegalArgumentException("Illegal Capacity: "+    
                                                initialCapacity);    
     this.elementData = new Object[initialCapacity];    
     }    
     
     /**  
      * Constructs an empty list with an initial capacity of ten.  
      */    
     public ArrayList() {    
     this(10);    
     }    
     
     /**  
      * Constructs a list containing the elements of the specified  
      * collection, in the order they are returned by the collection's  
      * iterator.  
      */    
     public ArrayList(Collection<? extends E> c) {    
     elementData = c.toArray();    
     size = elementData.length;    
     // c.toArray might (incorrectly) not return Object[] (see 6260652)    
     if (elementData.getClass() != Object[].class)    
         elementData = Arrays.copyOf(elementData, size, Object[].class);    
     }
```

1：初始化一个指定大小的数组
2：无参数构造函数，默认数组大小是10
3：用一个Collection创建。

#### 成员方法

+ 添加一个元素：
```
public boolean add(E e) {    
    ensureCapacity(size + 1);  // Increments modCount!!    
    elementData[size++] = e;    
    return true;    
    }

/**  
      * Increases the capacity of this <tt>ArrayList</tt> instance, if  
      * necessary, to ensure that it can hold at least the number of elements  
      * specified by the minimum capacity argument.  
      *  
      * @param   minCapacity   the desired minimum capacity  
      */    
     public void ensureCapacity(int minCapacity) {    
     modCount++;    
     int oldCapacity = elementData.length;    
     if (minCapacity > oldCapacity) {    
         Object oldData[] = elementData;    
         int newCapacity = (oldCapacity * 3)/2 + 1;    
             if (newCapacity < minCapacity)    
         newCapacity = minCapacity;    
             // minCapacity is usually close to size, so this is a win:    
             elementData = Arrays.copyOf(elementData, newCapacity);    
     }    
     } 	
```

**如果超出数组目前的大小，则数组的长度变为原来的*3/2 + 1；**

+ 指定位置插入
```
public void add(int index, E element) {    
     if (index > size || index < 0)    
         throw new IndexOutOfBoundsException(    
         "Index: "+index+", Size: "+size);    
     
     ensureCapacity(size+1);  // Increments modCount!!    
     System.arraycopy(elementData, index, elementData, index + 1,    
              size - index);    
     elementData[index] = element;    
     size++;    
     }  
```

原理就是把后边的数组都向右移动一位。

+ 移除某个位置的元素
```
public E remove(int index) {    
     RangeCheck(index);    
     modCount++;    
     E oldValue = (E) elementData[index];    
     int numMoved = size - index - 1;    
     if (numMoved > 0)    
         System.arraycopy(elementData, index+1, elementData, index,    
                  numMoved);    
     elementData[--size] = null; <span style="color:#ff0000;">// Let gc do its work </span>   
     return oldValue;    
     }   
```

### LinkedList

底层是双向循环链表，head节点不存放数据
```
public class LinkedList<E>  
     extends AbstractSequentialList<E>  
     implements List<E>, Deque<E>, Cloneable, java.io.Serializable 
```

+ LinkedList 是一个继承于AbstractSequentialList的双向链表。它也可以被当作堆栈、队列或双端队列进行操作。

+ LinkedList 实现 List 接口，能对它进行队列操作。

+ LinkedList 实现 Deque 接口，即能将LinkedList当作双端队列使用。

+ LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。

+ LinkedList 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。

+ LinkedList 是非同步的

#### 私有属性
```
private transient Entry<E> header = new Entry<E>(null, null, null);  
private transient int size = 0; 

private static class Entry<E> {  
    E element;  
     Entry<E> next;  
     Entry<E> previous;  
   
     Entry(E element, Entry<E> next, Entry<E> previous) {  
         this.element = element;  
         this.next = next;  
         this.previous = previous;  
    }  
 }   
```
#### 构造方法
```
public LinkedList() {  
    header.next = header.previous = header;  
}  
public LinkedList(Collection<? extends E> c) {  
    this();  
   addAll(c);  
}  
```

第一个构造方法不接受参数，将header实例的previous和next全部指向header实例（注意，这个是一个双向循环链表，如果不是循环链表，空链表的情况应该是header节点的前一节点和后一节点均为null），这样整个链表其实就只有header一个节点，用于表示一个空的链表

#### 成员方法

+ 添加元素
```
 // 将元素(E)添加到LinkedList中  
 public boolean add(E e) {  
     // 将节点(节点数据是e)添加到表头(header)之前。  
     // 即，将节点添加到双向链表的末端。  
     addBefore(e, header);  
     return true;  
 }  
  
 public void add(int index, E element) {  
     addBefore(element, (index==size ? header : entry(index)));  
 }  
  
private Entry<E> addBefore(E e, Entry<E> entry) {  
     Entry<E> newEntry = new Entry<E>(e, entry, entry.previous);  
     newEntry.previous.next = newEntry;  
     newEntry.next.previous = newEntry;  
     size++;  
     modCount++;  
     return newEntry;  
}  
```

+ 删除元素：
```
 private E remove(Entry<E> e) {  
     if (e == header)  
         throw new NoSuchElementException();  
     // 保留将被移除的节点e的内容  
     E result = e.element;  
    // 将前一节点的next引用赋值为e的下一节点  
     e.previous.next = e.next;  
    // 将e的下一节点的previous赋值为e的上一节点  
     e.next.previous = e.previous;  
    // 上面两条语句的执行已经导致了无法在链表中访问到e节点，而下面解除了e节点对前后节点的引用  
    e.next = e.previous = null;  
   // 将被移除的节点的内容设为null  
   e.element = null;  
   // 修改size大小  
   size--;  
   modCount++;  
   // 返回移除节点e的内容  
   return result;  
 } 
```
+ 获取某个index的元素，get.remove的时候要调用
```
// 获取双向链表中指定位置的节点      
    private Entry<E> entry(int index) {      
        if (index < 0 || index >= size)      
            throw new IndexOutOfBoundsException("Index: "+index+      
                                                ", Size: "+size);      
        Entry<E> e = header;      
        // 获取index处的节点。      
        // 若index < 双向链表长度的1/2,则从前先后查找;      
        // 否则，从后向前查找。      
        if (index < (size >> 1)) {      
            for (int i = 0; i <= index; i++)      
                e = e.next;      
        } else {      
            for (int i = size; i > index; i--)      
                e = e.previous;      
        }      
        return e;      
    }  
```


