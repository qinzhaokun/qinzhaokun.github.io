---
title: Java ��̬����
date: 2017-08-21
tags: Java
categories: Java
---

*��̬����*��Java����һ���ǳ���Ҫ�����ԣ�Ҳ��Java�����������һ����Ҫ�����ƣ��������ѧϰһ�¶�̬����

## ��̬����

���ȣ�Ҫ�Ӿ�̬����ʼ���������ģʽ�У�����[����ģʽ](https://qinzhaokun.github.io/2017/08/01/%E4%BB%A3%E7%90%86%E6%A8%A1%E5%BC%8F/)�������ܽ�һ�£�����ģʽ������һ���������һ���࣬ʹ�õ����ߺͱ������ߵķ��룬����ϵͳ��϶ȣ�ͬʱ�����α�������һЩ���ܣ��ﵽ�Ե�����Ȩ�޿��ƵĹ��ܡ����⣬����һ���ǳ���Ҫ�Ĺ��ܣ�����**���޸Ĵ��������£��Է���������ǿ**��
 
���ǣ���ʹ�þ�̬����ʱ������һ������������һ����������������Ҫ����дһ���µķ���ȥ���������������������࣬���Ǿ�Ҫ���ϵ���д�µķ�����������ԭ������������ɴ�����ֱ���������ɴ�����**��̬����**���ڳ�������ʱ��̬�����µķ������÷����ܹ�����Ŀ������Ŀ�귽����

## ��̬����

��̬����ĸ��������Ѿ�˵�˱Ƚ�����ˣ�ֻҪ��ס������Ŀ���Ǽ�������Ϊ������ɷ������������࣬ʹ��һ��*�ؼ�����*��������ʱ��̬���ɷ������ɡ�����;��忴����̬�������ּ��������ʵ�ֵġ�

### invocationHander�ӿ�

�ھ�̬�����У��д�����ͱ�������������ɫ������̬�����У��������н��࣬��Ҫ��ʵ��invocationHandler,�ӿڵĶ������£�
```
public interface InvocationHandler { 
  
Object invoke(Object proxy, Method method, Object[] args); 
  
}
```

���У�`invoke`��һ���ؼ��ķ����������Ĳ��������֪�������������������ֱ��Ǵ�����󣬷����ͷ���������**�����ǵ��ô�������ĳ������ʱ����ת������invoke����**��**�����ǵ��ô�������ĳ������ʱ����ת������invoke����**��

�������Ƕ���һ����������`Cat`

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

�н�����Ҫʵ��ָ���ӿڣ�
```
public class DynamicProxy implements InvocationHandler { 
  
	private Object obj; //objΪί�������; 
  
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

�н������һ����������Ķ��󣬲����ڹؼ���`invoke`�����У����������Ĵ�����methodǰ�󣬶Է�����������ǿ���н����뱻�����๹���˾�̬�����ϵ����̬�����ϵ�����龲̬�����ϵ��ɣ�����Ƕ�̬�����ԭ��

���ž�������Ҫ�Ĳ��֣����ֺ�ν��̬��

��̬���ɴ�����

```
public class Main { 
  
	public static void main(String[] args) { 
  
	//�����н���ʵ�� 
  
	DynamicProxy inter = new DynamicProxy(new Cat()); 
  
	//������佫�����һ��$Proxy0.class�ļ�������ļ���Ϊ��̬���ɵĴ������ļ� 
  
	System.getProperties().put("sun.misc.ProxyGenerator.saveGeneratedFiles","true"); 
  
	//��ȡ������ʵ��sell 
  
	Eat catEat = (Eat)(Proxy.newProxyInstance(Eat.class.getClassLoader(), new Class[] {Eat.class}, inter)); 
  
	//ͨ�������������ô����෽����ʵ���ϻ�ת��invoke�������� 
  
	catEat.eat(); 
  
	catEat.drink(); 
  
	} 
  
} 
```

�����н���ʵ����ʹ��Proxy��ľ�̬����`newProxyInstance`�������������࣬�÷����Ĳ���Ҳ����Ҫ����������
```
public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h) throws IllegalArgumentException
```

�ֱ��Ǵ��������������������ʵ�ֵĽӿں��н����ʵ����

## ʵ��ԭ��

�ⲿ��һ�������ѵģ�JDK�ṩ��һ����̬������ķ�������������ȫ���������������ķ������������Ƶ�ʵ�ֹؼ�����`newProxyInstance`�⣬Դ��û����ϸ�Ķ����Ͼ����������������ϲ��͵��ܽᣬԭ����ʵ���ѡ�����Ҫ���������裬1. ���ɴ����ࣻ2. �õ�������Ĺ��췽����3. ���ù��췽�����ɴ�����󷵻ء��������һ�£������ɴ������У�ʹ����concurrentMapʵ����Ļ��棬ʹ����CASʵ�����̰߳�ȫ�ȣ�������������ֽ��롣

�õ��������󣬵���ָ��������������һ�����ù����أ�

�������������˷����붯̬���ɵĴ�����õ����룺

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
                //ʵ�ʾ��ǵ����н����invoke���� 
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
                //����ÿ���������� �����ʵ�ʷ�����
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

���У���������`text()`�����ĵ��ã��������н����`invoke`������������m3�ǽӿڷ���������������Eat�ӿڵķ�����

���ϴ�����ؼ��ľ��ǣ�

```
super.h.invoke(this, m3, (Object[])null);
```

�ҵ�����ǣ���ִ�д�������eat����ʱ���÷�������ø����hʵ����invoke��������������Proxy�����࣬��h��һ��`InvocationHandler`��ָ�룬��ָ����`DynamicProxy`ʵ�������������������Լ���`DynamicProxy`�б�д��`invoke`������������ȥ��������������һ���Ǵ����౾��thisָ�룬������Ϊû�ã��ڶ��Ǳ�������ʵ�ֵĽӿڷ������������ǲ����б�������������`DynamicProxy`��invoke���õ���ʲô������������ʲô�����������Լ���д��`DynamicProxy`��`invoke`�����У�������һ��ؼ�����

```
Object result = method.invoke(obj, args);
```

�˴���������*����*���ƣ�obj�Ǳ������࣬��method�Ǵ����ഴ�����Ľӿڷ������������Ǿ��ܵ��ñ��������method��������Ȼǰ���Ǳ�������Ҫʵ�ָýӿڷ�����

### �ܽ�

ͨ����̬���ɴ����࣬������̳���Proxy������Proxy�У���һ��h�ĳ�Ա����ָ����`DynamicProxy`ʵ����������Ҫ���ô�����ķ���ʱ��ͨ��`super.h.invoke(this,method,Object[] args)`����`DynamicProxy`�е�invokeҪ���ñ���������ĸ������������Ĳ�����ʲô����������`DynamicProxy`ȥʹ�÷�����������ĵ��ã�ͬʱ��ǰ���Զ��巽����ǿ������