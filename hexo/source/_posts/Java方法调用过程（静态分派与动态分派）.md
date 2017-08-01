---
title: Java方法调用过程（静态分派与动态分派）
date: 2017-08-01
tags: Java
categories: Java
---

在任何程序中，方法的调用都是非常拼单的操作，而掌握Java方法的调用，需要综合多方面的知识，其中包括class文件的结构，类加载的过程，类型转换和多态等概念，下面就详细讲解一下Java方法调用的过程。

## Class文件的编译和加载

由之前的学习的Class文件结构和类加载过程可知，编译器将.java文件编译成.class文件的过程中，不包含传统的连接过程，方法在.class文件当中知识符号引用，指向常量池中方法的全限定名，并不是直接引用，即实际代码块。这个特性给Java带来了更强大的**动态拓展能力**。符号引用并不能让方法能够顺利的调用，即执行到代码块，因此，**符号引用**一定要转换成直接引用。这个转化的过程，可以在两个阶段完成，1）类加载阶段；2）运行期间

## 方法解析

对于方法的调用，虚拟机提供了四条方法调用的字节码指令，分别是：

1. `invokestatic`: 调用静态方法

2. `invokespecial`: 调用构造方法，私有方法，父类方法

3. `invokevirtual`: 调用虚方法

4. `invokeinterface`: 调用接口方法

其中，1和2都可以在类加载阶段确定方法的唯一版本，因此，在**类加载阶段**就可以把符号引用解析为直接引用，在调用时刻直接找到方法代码块的内存地址进行执行（编译时已经找到了，并且存在方法调用的入口）；3和4则是在运行期间动态绑定方法的直接引用。注意，final修饰的方法也属于虚方法。

## 分派

下面，继续沿着方法调用，讲解分派的概念

假设类Man继承了类Human，定义一个类型为HUman的变量p如下：

```
Human p = new Man();
```
其中，Human是变量p的静态类型或外观类型，而Man是变量p的实际类型；**p的静态类型在编译期就可以确定**，而**实际类型则是在运行期才能确定**

### 静态分派

现在，有如下代码：

```
class Annimal{
  
}

class Cat extends Annimal{
}

class Dog extends Annimal{
}

class Test{
  public void say(Cat c){
    System.out.println("miao");
  }
  
  public void say(Dog d){
    System.out.println("wang");
  }
  
  public void say(Annimal a){
    System.out.println("wuwu");
  }
  public static void main{
    Annimal a1 = new Cat();
    Annimal a2 = new Dog();
    
    say(a1);
    say((Dog)a2);
  }
}

```

代码输出：
```
wuwu
wang
```

**方法重载是静态分配典型的应用**。重载函数的调用取决于参数的类型和个数，因此，在编译期间的方法调用`say(a1);`和`say((Dog)a2);`时，参数的类型取决于参数的**静态类型**， 即Annimal和Dog，因此在编译期间已经为方法的调用绑定了直接应用。

依赖静态类型来绑定方法的直接引用的反派行为，称为静态分派，**它是在编译阶段完成的**

### 动态分派

它和方法的重写相关，可以说就是是多态的概念，相信多态的例子都很熟悉了，变现出来就是方法的调用取决于变量的实际类型，而不是静态类型。这类的方法调用是由`invokevirtual`指令来执行的。以下是网上摘抄的例子：

```
public class DynamicDispatch {  
    static abstract class Human {  
        protected abstract void sayHello();  
    }  
    static class Man extends Human {  
        @Override  
        protected void sayHello() {  
            System.out.println("man say hello");              
        }  
    }  
    static class Woman extends Human {  
        @Override  
        protected void sayHello() {  
            System.out.println("woman say hello");  
        }  
    }  
      
    public static void main(String[] args) {  
        Human man = new Man();  
        Human woman = new Woman();  
        man.sayHello();  
        woman.sayHello();  
        man = new Woman();  
        man.sayHello();  
    }  
}  
```
代码反编译的结果如下：

```
public static void main(java.lang.String[]);  
  flags: ACC_PUBLIC, ACC_STATIC  
  Code:  
    stack=2, locals=3, args_size=1  
       0: new           #16                 // class com/xtayfjpk/jvm/chapter8/DynamicDispatch$Man  
       3: dup  
       4: invokespecial #18                 // Method com/xtayfjpk/jvm/chapter8/DynamicDispatch$Man."<init>":()V  
       7: astore_1  
       8: new           #19                 // class com/xtayfjpk/jvm/chapter8/DynamicDispatch$Woman  
      11: dup  
      12: invokespecial #21                 // Method com/xtayfjpk/jvm/chapter8/DynamicDispatch$Woman."<init>":()V  
      15: astore_2  
      16: aload_1  
      17: invokevirtual #22                 // Method com/xtayfjpk/jvm/chapter8/DynamicDispatch$Human.sayHello:()V  
      20: aload_2  
      21: invokevirtual #22                 // Method com/xtayfjpk/jvm/chapter8/DynamicDispatch$Human.sayHello:()V  
      24: new           #19                 // class com/xtayfjpk/jvm/chapter8/DynamicDispatch$Woman  
      27: dup  
      28: invokespecial #21                 // Method com/xtayfjpk/jvm/chapter8/DynamicDispatch$Woman."<init>":()V  
      31: astore_1  
      32: aload_1  
      33: invokevirtual #22                 // Method com/xtayfjpk/jvm/chapter8/DynamicDispatch$Human.sayHello:()V  
      36: return  
```

其中，17和21是两次sayHello()方法的调用，对应着不同的实际类型man和woman，它们都对应着`invokevirtual #22`这个相同的指令，但是执行结果却不同。这是由于方法的实际类型不同导致的。这个22实际是在编译期间确定的。`sayHello()`这个函数是重写自Human中，每个类都有一个**方法表**，这个方法表有如下特性:

1. 方法表的没一项是一个指针，指向实际的方法区的代码块的内存首地址

2. 父类的方法表在前，自己的在后面，由此可看出Object的方法会出现在所有类的最前面

3. 子类重写父类的方法，会把方法表中对应方法的指针由指向父类的代码块指向自身的代码块。

由以上我们可以知道，对于重写的方法`sayHello()`，在父类（Human）和子类（Man和Woman）中的方法表的**偏移量都是相同的**，即22，但里面的指针指向不同代码块。在编译期间，由于静态类型是Human，因此，查找Human方法表中的`sayHello()`，定位在22，得到了`invokevirtual #22`这个指令；而在运行期间，根据对象的this指针知道实际的类型，找到实际类型的方法表，索引到22这个位置，由于方法表`sayHello()`的偏移量都相同，因此22仍是对应`sayHello()`方法；继而找到指向子类的代码块的指针进行运行。

以上就是虚拟机执行`invokevirtual`指令，虚拟机实现多态，动态分派的原理。

### 方法表的内存模型

![](/images/Java方法调用_01.png)

由图可知，Girl和Boy的方法表的前面几项都是来自父类的，并且顺序都是相同的，如果没有重写，指向的都是父类的方法区代码块，toString()除外。

![](/images/Java方法调用_02.png)

执行方法调用的过程中，根据索引值，能够快速的找到对应的方法代码块的指针，不用重新遍历，这一切都取决于固定的索引值。

![](/images/Java方法调用_03.png)

对于接口的重载，情况变得不一样，由图可以看出，同一方法，在不同类的位置是不同的，因此不能使用索引快速定位，要采用遍历的方式进行搜索，效率没有类继承方式实现多态的情况快。

### 单分派和多分派

方法的接收者和方法的参数统一称为方法**宗量**。基于宗量，分派可以划分为**单分派**和**多分派**；单分派是根据一个宗量来对目标方法进行选择，多分派则是根据多于一个宗量对目标方法进行选择。

结论：**Java是静态多分派，动态单分派的语言**。

1. 静态分派中，选择目标方法的依据有两点：一是静态类型；二是方法参数

2. 动态分派过程只能接收者的实际类型一个宗量作为目标方法选择依据

## Java和C++在实现多态上的区别

C++也是非常流行的语言，不同的是，它在编译期间已经找到绑定了方法的直接引用，而不是Java的符号引用。C++中是可以**单继承**和**多继承**的。

单继承的情况下，函数的调用和Java类似，通过方法表。C++的对象的头部都有一个指针，指向一个虚函数表，这个虚函数表就和Java的方法表功能一样，存储的是每个方法的具体代码块的指针。**不同的是**，只有被`vitural`关键字修饰的方法才会出现虚函数表中。正常的方法，则是在编译时确定函数的调用的地址，称为早期绑定，而虚函数，则通过虚表来进行迟绑定。

多继承的情况下，对象的头有多个虚指针，指向不同的虚表。因此C++不用像Java那样搜索方法表