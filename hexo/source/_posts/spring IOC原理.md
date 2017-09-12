---
title: Spring IOC原理
date: 2017-07-12 18:54:25
tags: Spring
categories: Spring
---

## 简介

IOC全称是 Inversion of Control，控制反转，从字面上的意思理解，就是对象的控制权反转了。直接在对象内部通过new进行创建对象，是程序主动去创建依赖对象，此时被创建的对象的控制权在创建该对象的对象手中，而在Spring中，对象的创建由容器负责，在通过依赖注入(DI)将对象交给需要该对象的对象，因此控制权在容器，即发生了对象控制权的反转。那位，在Spring当中，容器是什么的？我觉得可以理解为是**配置文件**。xml ， properties文件等语义化配置文件。

在Spring中，由容器负责控制对象的**生命周期**和**对象间的关系**。

(DI)依赖注入，它主要实现的是**动态的向某个对象提供它所需要的其他对象**，它有三种注入方式：

1. 接口注入

2. Construct注入 

3. Setter注入

## BeanFactory

Spring当中，对象被表示为Bean，需要让对象被容器管理的，可以在xmle文件里定义<bean></bean>或在类定义中加上注解@Conponnet. **BeanFactory**是创建Bean的顶层接口, 也可以看做是容器

```
public interface BeanFactory {    
     
     //对FactoryBean的转义定义，因为如果使用bean的名字检索FactoryBean得到的对象是工厂生成的对象，    
     //如果需要得到工厂本身，需要转义           
     String FACTORY_BEAN_PREFIX = "&"; 
        
     //根据bean的名字，获取在IOC容器中得到bean实例    
     Object getBean(String name) throws BeansException;    
   
    //根据bean的名字和Class类型来得到bean实例，增加了类型安全验证机制。    
     Object getBean(String name, Class requiredType) throws BeansException;    
    
    //提供对bean的检索，看看是否在IOC容器有这个名字的bean    
     boolean containsBean(String name);    
    
    //根据bean名字得到bean实例，并同时判断这个bean是不是单例    
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;    
    
    //得到bean实例的Class类型    
    Class getType(String name) throws NoSuchBeanDefinitionException;    
    
    //得到bean的别名，如果根据别名检索，那么其原名也会被检索出来    
   String[] getAliases(String name);    
    
 }
```

### bean的生命周期

总的流程是：创建（实例化-初始化）-使用-销毁

1. BeanFactory读取Bean定义文件，生成各种实例

2. 对定义的属性进行注入，即设置对象成员变量的初始属性，可通过setter或者constructor进行注入

3. 如果Bean类实现了org.springframework.beans.factory.BeanNameAware接口，则执行其setBeanName()方法

4. 如果Bean类实现了org.springframework.beans.factory.BeanFactoryAware接口，则执行其setBeanFactory()方法

5. 容器中如果有实现org.springframework.beans.factory.BeanPostProcessors接口的实例，则任何Bean在初始化之前都会执行这个实例的processBeforeInitialization()方法

6. 如果Bean类实现了org.springframework.beans.factory.InitializingBean接口，则执行其afterPropertiesSet()方法

7. Bean定义文件中定义init-method，在Bean定义文件中使用“init-method”属性设定方法名称

```
<bean id="demoBean" class="com.yangsq.bean.DemoBean" init-method="initMethod">  ....... </bean>
```

8. 容器中如果有实现org.springframework.beans.factory.BeanPostProcessors接口的实例，则任何Bean在初始化之前都会执行这个实例的processAfterInitialization()方法

9.  在容器关闭时，如果Bean类实现了org.springframework.beans.factory.DisposableBean接口，则执行它的destroy()方法

10. Bean定义文件中定义destroy-method, 在容器关闭时，可以在Bean定义文件中使用“destory-method”定义的方法

```
<bean id="demoBean" class="com.yangsq.bean.DemoBean" destory-method="destroyMethod">  .......</bean>
```

### bean的一些属性

1. scope属性

singleton与prototype，默认值为singleton，表示创建bean是单例还是多例，可以使用xm配置或者注解配置
```
<bean id="loginService" class="com.springapp.mvc.service.impl.LoginServiceImpl" scope="prototype"/>
@Scope(value=ConfigurableBeanFactory.SCOPE_PROTOTYPE)
```

2. lazy-init属性

表示是否延迟加载，即默认false的话，容器创建时加载，否则，第一次使用时加载， **只对单例有效**



























