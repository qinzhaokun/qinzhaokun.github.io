---
title: Java 反射机制
date: 2017-08-22
tags: Java
categories: Java
---

*反射*在Java中充满了魔力，它像一个JVM给程序员开的一个后门，让程序员在程序运行时能够对程序有更强的控制力。用一句话来概括Java的反射，那就是**可以在运行期间获任意一个类的字节码，其中包括接口，变量和方法等信息；同时可以通过反射创建对象，调用对象的方法，访问/操作对象的属性**。

## 获取类信息

对于一个对象，我们可以访问设置其属性，调用其方法，但有的时候，我们更想知道该对象类的信息，由其是在程序运行的时候，动态的去查看对象所属类的信息，在去执行不同的操作。例如，我们想知道一个对象所属的类一共有多少个成员变量，每个成员变量的访问权限，一共有多少种方法，每个方法的访问权限，该类的父类是哪个，实现了哪些接口等，这些信息存储在Java字节码中，而在程序运行时，则可以通过Java反射机制来获取这些信息。

### Class对象

在学习类加载的过程中，我们知道每个类在堆都对应了一个Class对象，它是连接对象和类信息的桥梁，它保存着类在方法区数据的入口。获取Class对象有两种方法：

1. Class.forName(String classname); 如果知道类的全限定名，如“java.lang.Object”，则可通过传入一个String字符串来获得一个类的Class对象

2. 如果持有一个类的对象，那么，可以直接使用getClass()函数返回类的Class对象

```
			Class t = Class.forName("java.util.concurrent.Semaphore");
			Semaphore sem = new Semaphore(4);
			Class t1 = sem.getClass();
			if(t == t1){
				System.out.println("yes");
			}
```

#### 获取类的基本信息

Class对象提供了一些方法，返回该类的信息：

+ `getName()`，返回类的全限定名

+ `getSimpleName()`, 返回简单类名

+ `getModifiers()`，类的修饰符，返回一个int，表示该类是什么修饰符修饰的，可使用`Modifier`来检查

```
Modifier.isAbstract(int modifiers);
Modifier.isFinal(int modifiers);
Modifier.isInterface(int modifiers);
Modifier.isNative(int modifiers);
Modifier.isPrivate(int modifiers);
Modifier.isProtected(int modifiers);
Modifier.isPublic(int modifiers);
Modifier.isStatic(int modifiers);
Modifier.isStrict(int modifiers);
Modifier.isSynchronized(int modifiers);
Modifier.isTransient(int modifiers);
Modifier.isVolatile(int modifiers);
```

+ `getPackage()`, 获取包信息

+ `getSuperClass()`, 获取父类的Class对象

+ `Class[] interfaces = aClass.getInterfaces()`, 获取实现的接口

#### 获取类的构造函数

`Constructor[] constructors = aClass.getConstructors()`, 可用于获取构造函数，它返回的是一个数组，所有的构造函数都在里面。如果我们只需要某个特定的构造函数呢？由于函数的区分是通过参数列表来区分的，所以只有知道该构造函数的参数列表即可，如下：

 `Constructor constructor = aClass.getConstructor(new Class[]{String.class})`
 
 返回具有指定参数的构造函数，该例子表示返回参数是一个String的构造函数,传入的*参数列表*是Class对象的数组

+此外，我们如果得到一个构造函数，想知道它的参数列表，可以通过如下调用：

`Class[] parameterTypes = constructor.getParameterTypes()`

下面是重点了，有了构造函数的对象`constructor`，可以创建该类的实例：

(SomeObject)constructor.newInstance("string");

具体的代码示例如下：
```
		try {
			Class t = Class.forName("java.util.concurrent.Semaphore");
			
			Constructor con = t.getConstructor(new Class[]{int.class});
			
			Semaphore sem = (Semaphore)con.newInstance(4);
			
			System.out.println(sem);
			
			
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (NoSuchMethodException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (SecurityException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (InstantiationException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IllegalArgumentException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (InvocationTargetException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
```

#### 获取类的方法

`Method [] methods = aClass.getMethods()`, 返回所有*public*的方法的集合，所有方法毕竟不方便，我们需要知道某个具体的方法，因此，只有知道方法名字和参数列表，就可以返回指定方法，如：
```
Method method = aClass.getMethod("hashcode",null);
```

如果要获取私有方法，使用`Class.getDeclaredMethod(String name, Class[] parameterTypes)或者 Class.getDeclaredMethods()` 来获取。

同样的，Method类还为我们提供了返回该方法的参数列表（method.getParameterTypes()）和返回该方法的返回类型：method.getReturnType();

**调用一个方法**，这是获取该方法的最终目的，使用`invoke`函数。值得注意的时，方法分为*静态方法*和*非静态方法*，前者属于类所有，后者属于对象，因此，在调用非静态方法的时候，要传入对象，如：

```
Object object = new Object();
Method method = object.getClass().getMethod("hashcode",null);
Object returnType = method.invoke(object,null);
```

Java中的动态代理就是使用了反射当中的invoke。动态生成的代理类调用某个方法时，会把对应的method和参数列表发送给中介类（实现了`InvocationHandler `的类），在中介类的invoke调用`method.invoke`，中介类持有被代理的对象。

代码示例如下：
```
		try {
			Class t = Class.forName("java.util.concurrent.Semaphore");
			
			Constructor con = t.getConstructor(new Class[]{int.class});
			
			Semaphore sem = (Semaphore)con.newInstance(4);
						
			Method method = t.getMethod("hashCode", null);
			
			int code = (int)method.invoke(sem, null);
			
			System.out.println(code);
			
			
		} catch (ClassNotFoundException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (NoSuchMethodException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (SecurityException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IllegalArgumentException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (InvocationTargetException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (InstantiationException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} 
```
#### 获取类的成员变量

`Field[] fields = aClass.getFields()`可以得到类的公有成员变量；而访问私有变量，可以通过`getDeclaredField()或getDeclaredField(String name)`来获取。

得到了成员变量后，我们如何访问呢？使用`field.get(Object)`方法；如果修改呢？使用`fields.set(Object, value)`。公有变量可以直接进行访问了，但是如果访问私有变量，回进行作用域检查，直接访问运行时会抛出异常：`java.lang.IllegalAccessException:`，要加上如下语句：
```
field.setAccessible(true);//为 true 则表示反射的对象在使用时取消 Java 语言访问检查
```

主要上面我们发现读/写一个field，都要传出该拥有变量的对象，对于普通成员函数，这是合理的，但如果想读写静态变量呢？只需要要传入对应的Class即可。具体一个简单的示例如下：
```
class A {
	private int c = 10;
	public static int d = 14;
	
	A(int c){
		this.c = c;
	}
	
	private void f(){
		c = 20;
	}
}

public class Test {

	public static void main(String[] args) throws InterruptedException {
		try {
		A a = new A(12);
		
		Class t = a.getClass();
		
		Field field = t.getDeclaredField("c");
		
		Field field1 = t.getField("d");
		
		field.setAccessible(true);
		field.set(a, 22);
		field1.set(t, 24);
		
		System.out.println(field.get(a) + " " + field1.get(t));
			
			
		} catch (NoSuchFieldException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IllegalArgumentException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IllegalAccessException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} 
	}

}

```

