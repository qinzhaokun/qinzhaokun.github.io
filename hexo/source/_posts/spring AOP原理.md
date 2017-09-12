---
title: Spring AOP原理
date: 2017-07-12 18:54:25
tags: Spring
categories: Spring
---

AOP全称是**面向切面的编程**。它能把一些散落在各个业务流程中的系统方法汇总成一个切面。使用AOP来灵活处理一些具有横切性质的系统级服务，如事务处理、安全检查、缓存、对象池管理等，已经成为一种非常适用的解决方案, 它解决了在OOP中大量的重复代码。在 OOP 中, 我们以类(class)作为我们的基本单元, 而 AOP 中的基本单元是 Aspect(切面)

AOP把影响多个类的公共行为封装到一个可重用的模块。它使用了动态代理技术，运行时在内存中动态生成代理类，调用代理类的构造函数生成代理对象，该对象包含被代理对象的所有方法，并在方法中回调原被代理对象的方法，同时在调用前后对方法进行增强。AOP中动态代理主要有两种方式，JDK动态代理和CGLIB动态代理。

+ JDK动态代理通过反射来接收被代理的类，并且要求被代理的类必须实现一个接口。JDK动态代理的核心是InvocationHandler接口和Proxy类。

+ 如果被代理的类没有实现接口，那么就采用CGLIB代理技术，它主要是动态生成继承自被代理类的子类，从而进行相应的调用。

## 相关概念

### 通知（Advice）

通知分为五种类型，分别是：`Before`,`After`,`After-returning`, `After-throwing` 和`Around` .它定义了切面做什么，什么时候做。

### 连接点(Join point)

这是一个抽象的概念，它表示程序运行的任意一个时间点，可以是方法的执行，也可以是异常的处理，而在Spring AOP中，join point 总是方法的执行点

> a point during the execution of a program, such as the execution of a method or the handling of an exception. In Spring AOP, a join point always represents a method execution.

### 切入点(PointCut)

对于程序中有很多个连接点，那么那些节点是要被拦截的呢？pointcut指明了哪些方法需要被拦截，通常由`Point Expression`来描述。由于在spring当中每一个方法都是join point，我们并不希望每个方法都加入advice，因此使用pointcut来匹配jointpoint，定义需要被拦截的方法。

> 在 Spring AOP 中, 所有的方法执行都是 join point. 而 point cut 是一个描述信息, 它修饰的是 join point, 通过 point cut, 我们就可以确定哪些 join point 可以被织入 Advice. 因此 join point 和 point cut 本质上就是两个不同纬度上的东西. advice 是在 join point 上执行的, 而 point cut 规定了哪些 join point 可以执行哪些 advice.

### 方面(Aspect)

一个切面的抽象，它表示一个具体的功能，如事务管理，权限鉴定等，advice和pointcut的组合就是aspect。它完整定义了一个交叉业务要做什么，它是要在何时，何地weaving进主业务中

## 简单实例

### XML配置方式

1. 目标对象
```
package com.spring.service;

public class ServiceImpl implements Service {  
  
    public void say() {  
        System.out.println("ServiceImpl say hello");  
    }  
}  
```

2. 切面对象
```
package com.spring.aop;
public class Aspect {  
  
    public void doAfter(JoinPoint jp) {  
        System.out.println("log Ending method: " + jp.getTarget().getClass().getName() + "." + jp.getSignature().getName());  
    }  
  
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {  
        long time = System.currentTimeMillis();  
        Object retVal = pjp.proceed();  
        time = System.currentTimeMillis() - time;  
        System.out.println("process time: " + time + " ms");  
        return retVal;  
    }  
  
    public void doBefore(JoinPoint jp) {  
        System.out.println("log Begining method: " + jp.getTarget().getClass().getName() + "." + jp.getSignature().getName());  
    }  
  
    public void doThrowing(JoinPoint jp, Throwable ex) {  
        System.out.println("method " + jp.getTarget().getClass().getName() + "." + jp.getSignature().getName() + " throw exception");  
        System.out.println(ex.getMessage());  
    }  
}  
```

3. 配置文件
```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:aop="http://www.springframework.org/schema/aop"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:tx="http://www.springframework.org/schema/tx"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd  
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.1.xsd  
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.1.xsd">  
    <aop:config>  
        <aop:aspect id="TestAspect" ref="aspectBean">  
            <!--配置com.spring.service包下所有类或接口的所有方法-->  
            <aop:pointcut id="myService" expression="execution(* com.spring.service.*.*(..))" />  
            <aop:before pointcut-ref="myService" method="doBefore"/>  
            <aop:after pointcut-ref="myService" method="doAfter"/>  
            <aop:around pointcut-ref="myService" method="doAround"/>  
            <aop:after-throwing pointcut-ref="businessService" method="doThrowing" throwing="ex"/>  
        </aop:aspect>  
    </aop:config>  
      
    <bean id="aspectBean" class="com.spring.aop.Aspect" />  
    <bean id="Service" class="com.spring.service.ServiceImpl"></bean>    
</beans>  
```

### 注解方式
```
package com.spring.aop;  
  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.After;  
import org.aspectj.lang.annotation.AfterReturning;  
import org.aspectj.lang.annotation.AfterThrowing;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
import org.aspectj.lang.annotation.Before;  
import org.aspectj.lang.annotation.Pointcut;  
  
@Aspect  
public class Aspect {  
  
    @Pointcut("execution(* com.spring.service.*.*(..))")  
    private void pointCutMethod() {  
    }  
  
    //声明前置通知  
    @Before("pointCutMethod()")  
    public void doBefore() {  
        System.out.println("前置通知");  
    }  
  
    //声明后置通知  
    @AfterReturning(pointcut = "pointCutMethod()", returning = "result")  
    public void doAfterReturning(String result) {  
        System.out.println("后置通知");  
        System.out.println("---" + result + "---");  
    }  
  
    //声明例外通知  
    @AfterThrowing(pointcut = "pointCutMethod()", throwing = "e")  
    public void doAfterThrowing(Exception e) {  
        System.out.println("例外通知");  
        System.out.println(e.getMessage());  
    }  
  
    //声明最终通知  
    @After("pointCutMethod()")  
    public void doAfter() {  
        System.out.println("最终通知");  
    }  
  
    //声明环绕通知  
    @Around("pointCutMethod()")  
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {  
        System.out.println("进入方法---环绕通知");  
        Object o = pjp.proceed();  
        System.out.println("退出方法---环绕通知");  
        return o;  
    }  
}
```

```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
    xmlns:aop="http://www.springframework.org/schema/aop"  
    xmlns:context="http://www.springframework.org/schema/context"  
    xmlns:tx="http://www.springframework.org/schema/tx"  
    xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd  
        http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.1.xsd  
        http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd  
        http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-3.1.xsd">  
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>  
    <bean class="org.springframework.aop.aspectj.annotation.AnnotationAwareAspectJAutoProxyCreator" />  
  
    <bean id="aspectBean" class="com.spring.aop.Aspect" />  
    <bean id="Service" class="com.spring.service.ServiceImpl"></bean>   
</beans>  
```
