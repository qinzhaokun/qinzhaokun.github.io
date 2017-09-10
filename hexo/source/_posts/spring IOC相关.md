---
title: Spring IOC���
date: 2017-07-12 18:54:25
tags: Spring
categories: Spring
---

## ���

IOCȫ���� Inversion of Control�����Ʒ�ת���������ϵ���˼��⣬���Ƕ���Ŀ���Ȩ��ת�ˡ�ֱ���ڶ����ڲ�ͨ��new���д��������ǳ�������ȥ�����������󣬴�ʱ�������Ķ���Ŀ���Ȩ�ڴ����ö���Ķ������У�����Spring�У�����Ĵ���������������ͨ������ע��(DI)�����󽻸���Ҫ�ö���Ķ�����˿���Ȩ���������������˶������Ȩ�ķ�ת����λ����Spring���У�������ʲô�ģ��Ҿ��ÿ������Ϊ��**�����ļ�**��xml �� properties�ļ������廯�����ļ���

��Spring�У�������������ƶ����**��������**��**�����Ĺ�ϵ**��

(DI)����ע�룬����Ҫʵ�ֵ���**��̬����ĳ�������ṩ������Ҫ����������**����������ע�뷽ʽ��

1. �ӿ�ע��

2. Constructע�� 

3. Setterע��

## BeanFactory

Spring���У����󱻱�ʾΪBean����Ҫ�ö�����������ģ�������xmle�ļ��ﶨ��<bean></bean>�����ඨ���м���ע��@Conponnet. **BeanFactory**�Ǵ���Bean�Ķ���ӿ�, Ҳ���Կ���������

```
public interface BeanFactory {    
     
     //��FactoryBean��ת�嶨�壬��Ϊ���ʹ��bean�����ּ���FactoryBean�õ��Ķ����ǹ������ɵĶ���    
     //�����Ҫ�õ�����������Ҫת��           
     String FACTORY_BEAN_PREFIX = "&"; 
        
     //����bean�����֣���ȡ��IOC�����еõ�beanʵ��    
     Object getBean(String name) throws BeansException;    
   
    //����bean�����ֺ�Class�������õ�beanʵ�������������Ͱ�ȫ��֤���ơ�    
     Object getBean(String name, Class requiredType) throws BeansException;    
    
    //�ṩ��bean�ļ����������Ƿ���IOC������������ֵ�bean    
     boolean containsBean(String name);    
    
    //����bean���ֵõ�beanʵ������ͬʱ�ж����bean�ǲ��ǵ���    
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;    
    
    //�õ�beanʵ����Class����    
    Class getType(String name) throws NoSuchBeanDefinitionException;    
    
    //�õ�bean�ı�����������ݱ�����������ô��ԭ��Ҳ�ᱻ��������    
   String[] getAliases(String name);    
    
 }
```

### bean����������

�ܵ������ǣ�������ʵ����-��ʼ����-ʹ��-����

1. BeanFactory��ȡBean�����ļ������ɸ���ʵ��

2. �Զ�������Խ���ע�룬�����ö����Ա�����ĳ�ʼ���ԣ���ͨ��setter����constructor����ע��

3. ���Bean��ʵ����org.springframework.beans.factory.BeanNameAware�ӿڣ���ִ����setBeanName()����

4. ���Bean��ʵ����org.springframework.beans.factory.BeanFactoryAware�ӿڣ���ִ����setBeanFactory()����

5. �����������ʵ��org.springframework.beans.factory.BeanPostProcessors�ӿڵ�ʵ�������κ�Bean�ڳ�ʼ��֮ǰ����ִ�����ʵ����processBeforeInitialization()����

6. ���Bean��ʵ����org.springframework.beans.factory.InitializingBean�ӿڣ���ִ����afterPropertiesSet()����

7. Bean�����ļ��ж���init-method����Bean�����ļ���ʹ�á�init-method�������趨��������

```
<bean id="demoBean" class="com.yangsq.bean.DemoBean" init-method="initMethod">  ....... </bean>
```

8. �����������ʵ��org.springframework.beans.factory.BeanPostProcessors�ӿڵ�ʵ�������κ�Bean�ڳ�ʼ��֮ǰ����ִ�����ʵ����processAfterInitialization()����

9.  �������ر�ʱ�����Bean��ʵ����org.springframework.beans.factory.DisposableBean�ӿڣ���ִ������destroy()����

10. Bean�����ļ��ж���destroy-method, �������ر�ʱ��������Bean�����ļ���ʹ�á�destory-method������ķ���

```
<bean id="demoBean" class="com.yangsq.bean.DemoBean" destory-method="destroyMethod">  .......</bean>
```

### bean��һЩ����

1. scope����

singleton��prototype��Ĭ��ֵΪsingleton����ʾ����bean�ǵ������Ƕ���������ʹ��xm���û���ע������
```
<bean id="loginService" class="com.springapp.mvc.service.impl.LoginServiceImpl" scope="prototype"/>
@Scope(value=ConfigurableBeanFactory.SCOPE_PROTOTYPE)
```

2. lazy-init����

��ʾ�Ƿ��ӳټ��أ���Ĭ��false�Ļ�����������ʱ���أ����򣬵�һ��ʹ��ʱ���أ� **ֻ�Ե�����Ч**

## AOP

AOPȫ������������ı�̡����ܰ�һЩɢ���ڸ���ҵ�������е�ϵͳ�������ܳ�һ�����档ʹ��AOP������һЩ���к������ʵ�ϵͳ����������������ȫ��顢���桢����ع���ȣ��Ѿ���Ϊһ�ַǳ����õĽ������