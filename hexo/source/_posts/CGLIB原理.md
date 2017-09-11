---
title: CGLIB原理
date: 2017-09-11
tags: Java
categories: Java
---

## 理论
JDK原生的方式实现动态代理，它要求被代理的类实现某一个接口，那么并不是每个类需要实现接口，对于那些没有实现接口的类，难道就不能使用动态代理了吗？这里我们就使用本文要讲到的CGlib，它采用非常底层的字节码技术，通过字节码技术为被代理的类创建一个子类，并在子类中采用方法拦截的技术拦截了父类的方法调用，织入了横切逻辑。JDK的动态带了和CGLIB都被应用到了Spring AOP中。

CGLib创建的动态代理对象性能比JDK创建的动态代理对象的性能高不少，但是CGLib在创建代理对象时所花费的时间却比JDK多得多，所以对于单例的对象，因为无需频繁创建对象，用CGLib合适，反之，使用JDK方式要更为合适一些。同时，由于CGLib由于是采用动态创建子类的方法，对于final方法，无法进行代理。

JDK动态代理在调用方法时，使用了反射技术来调用被拦截的方法，效率低下，CGLib底层采用ASM字节码生成框架，使用字节码技术生成代理类，比使用Java反射效率要高。并且CGLIB采用`fastclass`机制来进行调用，对一个类的方法建立索引，通过索引来直接调用相应的方法。唯一需要注意的是，CGLib不能对声明为final的方法进行代理。

## 实践

下面是具体的使用：

```
public class Cat{
  public void say(){
    System.out.println("Hello");
  }
}
```
一直会说Hello的猫。

```
public calss CglibProxy implements MethodInterceptor{
//实例化一个增强器，也就是cglib中的一个class generator
  private Enhancer enhancer = new Enhancer();
  
  public Object getProxy(Class clazz){
    //设置目标类
    enhancer.setSuperclass(clazz);
    //设置拦截对象
    enhancer.setCallback(this);
    //生成代理类实例
    return enhancer.create();
  }
  
  public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable{
    System.out.println("before method was called");
    //进行被代理类的方法调用
    Object res = proxy.invokeSuper(obj,args);
    System.out.println("after method was called");
  }
}
```

```
public class Test{
  public static void main(String [] args) {
    CglibProxy proxy = new CglibProxy();
    
    Cat catProxy = (Cat)proxy.getProxy(Cat.class);
    
    catProxy.say();
  }
}
```

