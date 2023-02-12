[TOC]

# Java的两大、三类代理模式

## 简述

代理，是一种设计模式，主要作用是为其他对象提供一种代理，以控制对这个对象的访问。在某些情况下，一个对象不想或者不能直接引用另一个对象，而代理对象可以在客户端和目标对象之间起到中介的作用。

==主要分为两大代理：静态代理、动态代理（JDK动态代理、CGLIB动态代理）==

以下面的例子为例，卖房人、代理方、买房人，买卖人双方不需要打交道，通过和代理方打交道，卖房人提供房子给代理方，不需要自己去找买方，代理方有一堆卖房信息，买方直接找代理方即可

----------------------------------------------------------------------------------------------------

## 代码实现

接口

```java
public interface IHome {
	void say();
}
```

被代理类

```java
public class ForlanHome implements IHome{
	@Override
	public void say() {
		System.out.println("卖程序员Forlan房子啦！！！");
	}
}
```

> ==**2.1 静态代理**==

```java
public class HomeProxy implements IHome {
	IHome home;
	public HomeProxy(IHome home) {
		this.home = home;
	}
	@Override
	public void say() {
		System.out.println("This is HomeProxy");
		home.say();
	}
}
```

```java
public static void main(String[] args) {
		ForlanHome forlanHome = new ForlanHome();
		HomeProxy homeProxy = new HomeProxy(forlanHome);
		homeProxy.say();
	}
```

==**2.2 动态代理**==

>  2.2.1 ==JDK动态代理==

==普通版:==实现InvocationHandler接口重写invoke方法

```java
public class HomeInvocationHandler implements InvocationHandler {
	IHome home;
	public HomeInvocationHandler(IHome home) {
		this.home = home;
	}
	/**
	 * 重写invoke方法
	 *
	 * @param proxy  生成的代理对象
	 * @param method 调用的方法
	 * @param args 方法入参
	 * @return
	 */
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println("method " + method.getName() + " start!");
		Object o = method.invoke(home, args);
		System.out.println("method " + method.getName() + " end!");
		return o;
	}
}
```

测试方法

```java
public static void main(String[] args) {
	IHome forlanHome = new ForlanHome();
	IHome home = (IHome) Proxy.newProxyInstance(IHome.class.getClassLoader(),
			new Class[]{IHome.class},
			new HomeInvocationHandler(forlanHome)
	);
	home.say();
}
```

> 2.2.1 ==JDK动态代理==

-==反射版:==引入反射封装，方便调用

```java
public class CommonInvocationHandler<T> implements InvocationHandler {
	T obj;
	public T getProxyObj(T t) {
		obj = t;
		T proxyInstance = (T) Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(), this);
		return proxyInstance;
	}
	/**
	 * 重写invoke方法
	 *
	 * @param proxy  生成的代理对象
	 * @param method 调用的方法
	 * @param args   方法入参
	 * @return
	 */
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println("method " + method.getName() + " start..");
		Object o = method.invoke(obj, args);
		System.out.println("method " + method.getName() + " end!");
		return o;
	}
}
```

测试方法

```java
public static void main(String[] args) {
	IHome forlanHome = new ForlanHome();
	CommonInvocationHandler<IHome> commonInvocationHandler = new CommonInvocationHandler<>();
	IHome proxyObj = commonInvocationHandler.getProxyObj(forlanHome);
	proxyObj.say();
}
```

>  2.2.2 ==CGLIB动态代理==

第三方，需要导入jar包

```xml
<dependency>
     <groupId>cglib</groupId>
     <artifactId>cglib</artifactId>
     <version>2.2.2</version>
 </dependency>
```

==普通版：==实现MethodInterceptor，重写intercept方法

```java
public class HomeMethodInterceptor implements MethodInterceptor {
	@Override
	public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
		System.out.println("method " + method.getName() + " start!");
		Object result = null;
		result = methodProxy.invokeSuper(o, objects);
		System.out.println("method " + method.getName() + " end!");
		return result;
	}
}
```

测试方法

```java
public static void main(String[] args) {
	Enhancer enhancer = new Enhancer();
	enhancer.setSuperclass(ForlanHome.class); // 被代理对象的class
	enhancer.setCallback(new HomeMethodInterceptor()); // 设置回调，方法拦截器
	ForlanHome forlanHome = (ForlanHome)enhancer.create(); // 生成动态代理类
	forlanHome.say();
}
```

==泛型版：==使用泛型，封装Enhancer代码，方便调用

```java
public class CommonMethodInterceptor<T> implements MethodInterceptor {
	public T getProxyObj(Class<T> tClass) {
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(tClass); // 被代理对象的class
		enhancer.setCallback(this); // 设置回调，方法拦截器
		return (T) enhancer.create(); // 生成动态代理类
	}
	@Override
	public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
		System.out.println("method " + method.getName() + " start!");
		Object result = null;
		result = methodProxy.invokeSuper(o, objects);
		System.out.println("method " + method.getName() + " end!");
		return result;
	}
}
```

测试方法

```java
public static void main(String[] args) {
	CommonMethodInterceptor<ForlanHome> commonMethodInterceptor = new CommonMethodInterceptor<>();
	ForlanHome proxyObj = commonMethodInterceptor.getProxyObj(ForlanHome.class);
	proxyObj.say();
}
```

## 总结

静态代理：

> 代理对象实现被代理对象的接口，通过调用代理接口的接口，实现代理功能
>
> - 代理对象/被代理对象共同实现接口
> - 多态的一种应用

动态代理：

> 分离代理行为和被代理对象，替被代理对象干某件事
>
> - JDK动态代理
>   面向接口的动态代理，代理一个对象去增强面向某个接口中定义的方法
>   缺点：被代理对象必须实现接口
> - CGLIB动态代理
>   面向类的动态代理，重写父类方法，增强使用
>   缺点： final类不能被代理

区别：

> - 静态代理
>   a、自己写生成代理类
>   b、代理类和委托类的关系在运行前就确定了
> - 动态代理
>   a、在程序运行期间动态生成代理类
>   b、代理类和委托类的关系是在程序运行时确定

link:

>  https://www.bbsmax.com/A/pRdBD6pGJn/
