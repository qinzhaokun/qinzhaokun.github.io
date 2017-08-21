---
title: Java 动态代理
date: 2017-08-21
tags: Java
categories: Java
---

*动态代理*是Java语言一个非常重要的特性，也是Java相比其他语言一个重要的优势，下面就来学习一下动态代理。

## 静态代理

首先，要从静态代理开始讲起。在设计模式中，就有[代理模式](https://qinzhaokun.github.io/2017/08/01/%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F/)这个概念。总结一下，代理模式能有用一个类代理另一个类，使得调用者和被调用者的分离，降低系统耦合度，同时，屏蔽被调用者一些功能，达到对调用者权限控制的功能。此外，还有一个非常重要的功能，就是**不修改代码的情况下，对方法进行增强**。
 
但是，在使用静态代理时，对于一个被代理对象的一个方法，我们往往要重新写一个新的方法去代理它，当方法不断增多，我们就要不断的重写新的方法，数量是原来的两倍，造成代码量直线上升。由此引出**动态代理**，在程序运行时动态生成新的方法，该方法能够代理目标对象的目标方法。

## 动态代理

动态代理的概念上面已经说了比较清楚了，只要记住：它的目的是减少了因为代理造成方法数量的增多，使用一种*关键技术*，在运行时动态生成方法即可。下面就具体看看动态代理这种技术是如何实现的。

### invocationHander接口

在静态代理中，有代理类和被代理类两个角色，而动态代理中，引入了中介类，它要求实现invocationHandler,接口的定义如下：
```
public interface InvocationHandler { 
  
Object invoke(Object proxy, Method method, Object[] args); 
  
}
```

其中，`invoke`是一个关键的方法，从它的参数定义可知，它接收三个参数，分别是代理对象，方法和方法参数。**当我们调用代理对象的某个方法时，会转而进入invoke方法**。**当我们调用代理对象的某个方法时，会转而进入invoke方法**。

下面我们定义一个被代理类`Cat`

```
public class Cat implemtents Eat{
	public void eat{
		System.out.println("I am eating");
	}
	
	public void drink{
		System.out.println("I am drinking");
	}
}
```

中介类需要实现指定接口：
```
public class DynamicProxy implements InvocationHandler { 
  
	private Object obj; //obj为委托类对象; 
  
	public DynamicProxy(Object obj) { 
  
	this.obj = obj; 
  
	} 
  
	@Override
  
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable { 
  
		System.out.println("before"); 
  
		Object result = method.invoke(obj, args); 
  
		System.out.println("after"); 
  
		return result; 
  
	} 
} 
```

中介类持有一个被代理类的对象，并且在关键的`invoke`方法中，调用真正的代理方法method前后，对方法进行了增强。中介类与被代理类构成了静态代理关系，动态代理关系由两组静态代理关系组成，这就是动态代理的原理。

接着就是最重要的部分，体现何谓动态。

动态生成代理类

```
public class Main { 
  
	public static void main(String[] args) { 
  
	//创建中介类实例 
  
	DynamicProxy inter = new DynamicProxy(new Cat()); 
  
	//加上这句将会产生一个$Proxy0.class文件，这个文件即为动态生成的代理类文件 
  
	System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles","true"); 
  
	//获取代理类实例sell 
  
	Eat catEat = (Eat)(Proxy.newProxyInstance(Eat.class.getClassLoader(), new Class[] {Eat.class}, inter)); 
  
	//通过代理类对象调用代理类方法，实际上会转到invoke方法调用 
  
	catEat.eat(); 
  
	catEat.drink(); 
  
	} 
  
} 
```

创建中介类实例后，使用Proxy类的静态方法`newProxyInstance`方法创建代理类，该方法的参数也很重要，有三个：
```
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException
```

分别是代理类类加载器，代理类实现的接口和中介类的实例。

## 实现原理

这部分一定是最难的，JDK提供了一个动态生成类的方法，并且它完全有能力生成这样的方法。整个机制的实现关键就在`newProxyInstance`这，源码没有仔细阅读，毕竟读不懂，看了网上博客的总结，原理其实不难。它主要有三个步骤，1. 生成代理类；2. 得到代理类的构造方法；3. 调用构造方法生成代理对象返回。简单浏览了一下，在生成代理类中，使用了concurrentMap实现类的缓存，使用了CAS实现了线程安全等，最终生成类的字节码。

得到代理对象后，调用指定方法，是怎样一个调用过程呢？

以下是网上有人反编译动态生成的代理类得到代码：

```
final class $Proxy0 extends Proxy implements pro {
        //fields    
        private static Method m1;
        private static Method m2;
        private static Method m3;
        private static Method m0;

        public $Proxy0(InvocationHandler var1) throws  {
            super(var1);
        }

        public final boolean equals(Object var1) throws  {
            try {
                return ((Boolean)super.h.invoke(this, m1, new Object[]{var1})).booleanValue();
            } catch (RuntimeException | Error var3) {
                throw var3;
            } catch (Throwable var4) {
                throw new UndeclaredThrowableException(var4);
            }
        }

        public final String toString() throws  {
            try {
                return (String)super.h.invoke(this, m2, (Object[])null);
            } catch (RuntimeException | Error var2) {
                throw var2;
            } catch (Throwable var3) {
                throw new UndeclaredThrowableException(var3);
            }
        }

        public final void text() throws  {
            try {
                //实际就是调用中介类的invoke方法 
                super.h.invoke(this, m3, (Object[])null);
            } catch (RuntimeException | Error var2) {
                throw var2;
            } catch (Throwable var3) {
                throw new UndeclaredThrowableException(var3);
            }
        }

        public final int hashCode() throws  {
            try {
                return ((Integer)super.h.invoke(this, m0, (Object[])null)).intValue();
            } catch (RuntimeException | Error var2) {
                throw var2;
            } catch (Throwable var3) {
                throw new UndeclaredThrowableException(var3);
            }
        }

        static {
            try {
                //这里每个方法对象 和类的实际方法绑定
                m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[]{Class.forName("java.lang.Object")});
                m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
                m3 = Class.forName("spring.commons.api.study.CreateModel.pro").getMethod("text", new Class[0]);
                m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
            } catch (NoSuchMethodException var2) {
                throw new NoSuchMethodError(var2.getMessage());
            } catch (ClassNotFoundException var3) {
                throw new NoClassDefFoundError(var3.getMessage());
            }
        }
    }
```

其中，代理类中`text()`函数的调用，最终是中介类的`invoke`函数，而方法m3是接口方法，即上面代码的Eat接口的方法。

以上代码最关键的就是：

```
super.h.invoke(this, m3, (Object[])null);
```

我的理解是，当执行代理对象的eat方法时，该方法会调用父类的h实例的invoke方法，而父类是Proxy公共类，而h是一个`InvocationHandler`的指针，它指向了`DynamicProxy`实例，近而调用了我们自己在`DynamicProxy`中编写的`invoke`方法。而传进去的三个参数，第一个是代理类本身this指针，个人认为没用；第二是被代理类实现的接口方法；第三个是参数列表。它告诉了我们`DynamicProxy`的invoke，该调用什么方法，参数是什么？而在我们自己编写的`DynamicProxy`的`invoke`方法中，有这样一句关键调用

```
Object result = method.invoke(obj, args);
```

此处就是用了*反射*机制，obj是被代理类，而method是代理类创建来的接口方法，这样我们就能调用被代理类的method方法。当然前提是被代理类要实现该接口方法。

### 总结

通过动态生成代理类，代理类继承字Proxy，而在Proxy中，有一个h的成员变量指向了`DynamicProxy`实例。当我们要调用代理类的方法时，通过`super.h.invoke(this,method,Object[] args)`告诉`DynamicProxy`中的invoke要调用被代理类的哪个方法，方法的参数是什么？而最终由`DynamicProxy`去使用反射进行真正的调用，同时在前后自定义方法增强操作。