---
title: Java 序列化
date: 2017-08-28
tags: Java
categories: Java
---

对一个对象进行序列化，是java中一个非常重要应用，它是将对象持久化的重要手段，并且在对象的网络传输中也发挥中重要的作用。一般，**对象的生命周期不会比长**，如果在JVM运行结束后，需要将对象保存下来，就必须将其持久化。存入数据库是一个比较好的持久化方式，但是需要在数据库中建表，而将其序列化保存到磁盘或传输到远端网络则更方便些。**对象的序列化不会存储类的静态变量**

## 基本用法

序列化的基本用法比较简单，主要依赖以下几个步骤：

1. 对象的类必须实现`java.io.Serializable`接口，表示它能被序列化，这仅仅是一个标记接口

2. 通过`ObjectOutputStream`和`ObjectInputStream`对对象进行序列化和反序列化，调用其`writeObject`和`readOject`方法

3. 序列化ID, `private static final long serialVersionUID`

4. 无法序列化类的静态变量

5. 如果父类的成员也需要被序列化，则父类也实现现`java.io.Serializable`接口；如果父类实现了序列化的接口，子类自动继承了序列化的功能

6. 关键字`Transient`可以阻止变量被序列化；在反序列化中，被该关键字修饰的变量被赋为0或null


### 代码示例

```
class User implements Seiralizable{
  private String name;
  private int age;
  
  public String getName(){
    return name;
  }
  
  public void setName(String name){
    this.name = name;
  }
  
  public Int getAge(){
    return age;
  }
  
  public void setAge(int age){
    this.age = age;
  }
}

public class Test{
  public static void main(string [] args){
    User user = new User();
    user.setName("Jack");
    user.setAge(19);
    
     ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("tempFile"));
            oos.writeObject(user);
        } catch (IOException e) {
            e.printStackTrace();
        } 

        //Read Obj from File
     File file = new File("tempFile");
     ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream(file));
            User newUser = (User) ois.readObject();
            System.out.println(newUser);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } 
  }
}
```

## 自定义序列化方式

如上的基本用法，使用的是Java默认的序列化方式，调用是 ObjectOutputStream 的 defaultWriteObject 方法以及 ObjectInputStream 的 defaultReadObject 方法，对于一般的POJO，也是没有问题的，省时省力，但是如果对对于需要特殊处理的字段，拥有不同其他变量的序列化方式，则需要自定义`writeObject`和`readObject`

什么时候需要自定义序列化方式的的？下面总结了一下几种情况：

1. 成员变量一定要序列化，且需要加密处理

2. 存储优化，只想保存成员变量的部分数据，而不是全部数据，典型的是成员变量时一个数组，有很多null元素，如ArrayList

3. 成员变量是一个类的对象，该对象没有实现序列化接口

> 在序列化过程中，如果被序列化的类中定义了writeObject 和 readObject 方法，虚拟机会试图调用对象类里的 writeObject 和 readObject 方法，进行用户自定义的序列化和反序列化。

以下，我们使用ArrayList来讲解其序列化的过程：

### ArrayList类定义

```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    private static final long serialVersionUID = 8683452581122892189L;
    transient Object[] elementData; // non-private to simplify nested class access
    private int size;
}
```
elementData是被`transient`关键字修饰，表示不被序列化。这个说法极其不准确，**它只能表示不被Java默认序列化的方式序列化**

### ArrayList的writeObject
```
private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException{
        // Write out element count, and any hidden stuff
        int expectedModCount = modCount;
        s.defaultWriteObject();

        // Write out size as capacity for behavioural compatibility with clone()
        s.writeInt(size);

        // Write out all elements in the proper order.
        for (int i=0; i<size; i++) {
            s.writeObject(elementData[i]);
        }

        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

ArrayList自定义了序列化的方式，传入了`java.io.ObjectOutputStream s`变量，首先它使用了`defaultWriteObject()`对其进行序列化，除了elementData之外，都已经被序列化了；之后，它针对被tranisent修饰的变量单独序列化。这样的原因是elementData是个数组，在很多情况下不是满元素的，可能一个32长度的数组只有前几个是有数组的，其余的都是null，**出节约空间的考虑**，仅仅只需要序列化一个*size*和size个元素即可。

### ArrayList的readObject
```
private void readObject(java.io.ObjectInputStream s)
        throws java.io.IOException, ClassNotFoundException {
        elementData = EMPTY_ELEMENTDATA;

        // Read in size, and any hidden stuff
        s.defaultReadObject();

        // Read in capacity
        s.readInt(); // ignored

        if (size > 0) {
            // be like clone(), allocate array based upon size not capacity
            ensureCapacityInternal(size);

            Object[] a = elementData;
            // Read in all elements in the proper order.
            for (int i=0; i<size; i++) {
                a[i] = s.readObject();
            }
        }
    }
```

### 反射调用

没错，序列化和反序列化也使用了反射机制，当调用ObjectOutputStream的writeObject时，会调用：
writeObject ---> writeObject0 --->writeOrdinaryObject--->writeSerialData--->invokeWriteObject

最终到达`void invokeWriteObject(Object obj, ObjectOutputStream out)`,在该函数中会使用反射调用对应的writeObject
：
```
writeObjectMethod.invoke(obj, new Object[]{ out });
```

## Externalizable序列化

`Externalizable`是另一种序列化的方式，严格来说，它叫*外部化*。它继承自`Serializable`, 不同的是，它需要完全自定义序列化的方式，即它需要用户覆写`void writeExternal(ObjectOutput out) `和`public void readExternal(ObjectInput in)`。同时，当读取对象时，会调用被序列化类的无参构造器去创建一个新的对象，然后再将被保存对象的字段的值分别填充到新对象中。因此，实现`Externalizable`接口的类必须要**提供一个无参的构造器，且它的访问权限为public**。

## Serializable VS Externalizable

序列化（Serializable）构建起来比较方便，它使用内建API，存储类的所有相关信息，如类的信息，属性和这些属性的类型等都会被存储，因此存储空间大，速度相对慢；相反，外部化（Externalizable）构建比较麻烦，它要求序列化和反序列化完全由自己来实现，要自己实现如何存储信息，哪些信息需要存储，自动存储的基本信息较少，带来的好处就是存储空间占用小，速度相对快。
