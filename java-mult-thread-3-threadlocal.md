[TOC]

# ThreadLocal 原理

## 概述

在java学习生涯中可能很多人都会听到ThreadLocal变量，从字面上理解ThreadLocal就是“线程局部变量”的意思。简单的说就是，==一个ThreadLocal在一个线程中是共享的，在不同线程之间又是隔离的==（每个线程都只能看到自己线程的值）。可能一开始把这句话放出来很难理解，那我们就继续往后面看吧。

## API介绍

再学习一个类之前我们需要了解一个类的API，这也是我们学习类的入口。而ThreadLocal类的API相当简单。

![img](https://img-blog.csdnimg.cn/20190102094342330.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RpYW5qaW5kb25nMDgwNA==,size_16,color_FFFFFF,t_70)

在这里面比较重要的就是，get、set、remove了，这三个方法是对这个变量进行操作的关键。set用于赋值操作，get用于获取变量中的值，remove就是删除当前这个变量的值。需要注意的是initialValue方法会在第一次调用时被触发，用于初始化当前变量值，例如在下列代码中我们需要创建一个ThreadLocal<Connection>，用于创建一个与线程绑定的Connection对象：

```java
ThreadLocal<Connection> connection = new ThreadLocal<Connection>(){
    public Connection initialValue(){
        return DriverManager.getConnection(...);
    }
};
```

> 为什么我们将ThreadLocal说成变量，我们姑且可以这么理解，每个ThreadLocal实例中都可以保存一个值（基本数据类型值或者引用类型的引用值），而内部保存的值是可以修改的，而这样的特性与变量的特性及其相似，变量不就是用来保存一个值的吗？
>
> 也就是说每一个ThreadLocal实例就类似于一个变量名，不同的ThreadLocal实例就是不同的变量名，它们内部会存有一个值（暂时这么理解）在后面的描述中所说的“ThreadLocal变量或者是线程变量”代表的就是ThreadLocal类的实例。
>
> 这里还需要介绍一下initialValue方法，我么都知道在Java中成员变量都会有默认值，而ThreadLocal做变量也会有默认值，那我们可以通过重写initialValue方法指定ThreadLocal变量的初始值。默认情况下initialValue返回的是null。

## ThreadLocal的理解

说完了ThreadLocal类的API了，那我们就来动手实践一下了，来理解前面没有理解的那句话：一个ThreadLocal在一个线程中是共享的，在不同线程之间又是隔离的（每个线程都只能看到自己线程的值）

```java
public class ThreadLocalTest {
 
	private static ThreadLocal<Integer> num = new ThreadLocal<Integer>() {
		// 重写这个方法，可以修改“线程变量”的初始值，默认是null
		@Override
		protected Integer initialValue() {
			return 0;
		}
	};
 
	public static void main(String[] args) {
		// 创建一号线程
		new Thread(new Runnable() {
			@Override
			public void run() {
				// 在一号线程中将ThreadLocal变量设置为1
				num.set(1);
				System.out.println("一号线程中ThreadLocal变量中保存的值为：" + num.get());
			}
		}).start();
 
		// 创建二号线程
		new Thread(new Runnable() {
			@Override
			public void run() {
				num.set(2);
				System.out.println("二号线程中ThreadLocal变量中保存的值为：" + num.get());
			}
		}).start();
 
		//为了让一二号线程执行完毕，让主线程睡500ms
		try {
			Thread.sleep(500);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		System.out.println("主线程中ThreadLocal变量中保存的值：" + num.get());
	}
}
```

**稍微解释一下上面的代码：**

在类中创建了一个静态的“ThreadLocal变量”，在主线程中创建两个线程，在这两个线程中分别设置ThreadLocal变量为1和2。然后等待一号和二号线程执行完毕后，在主线程中查看ThreadLocal变量的值。

**程序结果及分析**

![img](https://img-blog.csdnimg.cn/20190102100906847.png)

程序结果重点看的是主线程输出的是0，如果是一个普通变量，在一号线程和二号线程中将普通变量设置为1和2，那么在一二号线程执行完毕后在打印这个变量，输出的值肯定是1或者2（到底输出哪一个由操作系统的线程调度逻辑有关）。但使用ThreadLocal变量通过两个线程赋值后，在主线程程中输出的却是初始值0。在这也就是为什么“一个ThreadLocal在一个线程中是共享的，在不同线程之间又是隔离的”，每个线程都只能看到自己线程的值，这也就是ThreadLocal的核心作用：实现线程范围的局部变量。

## **ThreadLocal的原理分析**

 老规矩我们还是将最后结论摆在前面，每个Thread对象都有一个ThreadLocalMap，当创建一个ThreadLocal的时候，就会将该ThreadLocal对象添加到该Map中，其中键就是ThreadLocal，值可以是任意类型。这句话看不懂很正常，等我们一起看完源码以后就明白了。

此时就需要纠正前面提到的错误观点了，前面我们的理解是所有的常量值或者是引用类型的引用都是保存在ThreadLocal实例中的，但实际上不是的，这种说法只是让我们更好的理解ThreadLocal变量这个概念。向ThreadLocal存入一个值，实际上是向当前线程对象中的ThreadLocalMap存入值，ThreadLocalMap我们可以简单的理解成一个Map，而向这个Map存值的key就是ThreadLocal实例本身。

![img](https://img-blog.csdnimg.cn/20190102103356116.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3RpYW5qaW5kb25nMDgwNA==,size_16,color_FFFFFF,t_70)

也就是说，想要存入的ThreadLocal中的数据实际上并没有存到ThreadLocal对象中去，而是以这个ThreadLocal实例作为key存到了当前线程中的一个Map中去了，获取ThreadLocal的值时同样也是这个道理。这也就是为什么ThreadLocal可以实现线程之间隔离的原因了。

> ==ThreadLocal的作用：实现线程范围内的局部变量，即ThreadLocal在一个线程中是共享的，在不同线程之间是隔离的。==
>
> ==ThreadLocal的原理：ThreadLocal存入值时使用当前ThreadLocal实例作为key，存入当前线程对象中的Map中去。最开始在看源码之前，我以为是以当前线程对象作为key将对象存入到ThreadLocal中的Map中去....==

## ThreadLocal防止内存泄露

> 我们知道，TheadLocal这种实现体系下，Thread会持有一个Map，并且该Map中会大量使用TheadLocal作为key，Map引用关系默认是强引用。在线程池或连接池这种会对线程进行复用的场景下，为了防止TheadLocal被强引用导致一直不能被回收，所以Thead中对TheadLocal的维护Map使用的是一个简单实现的WeakReferenceMap。该Map有以下特点：
>
> 1. 该Map的哈希冲突使用跳跃式处理，而不是HashMap中的桶。原因应该是把该空间当做栈来思考的话，使用跳跃式可以减少更多的空间分配，并且更好的利用cpu缓存。
> 2. 该Map的Entry，由继承了WeakReference的Entry实现，该Entry本身为TheadLocal的一个弱引用，当TheadLocal的强引用失去，被gc回收后。该Map在其各方法中都增加了清理Map节点的步骤，根据该弱引用对应的弱引用回收队列中的节点，清理Map中对应的键值对。

## ThreadLocal为什么会内存泄露

我们日常中一般使用TheadLocal的方式都如下，比如使用一个Util类来维护请求的TraceId，进行链路追踪的效果：

```java
public class TraceIDUtils {
	private static final ThreadLocal<String> SEQUENCE_ID = new ThreadLocal<String>();
}
```

那么我们会发现，该TheadLocal对象，本身就是一个static声明，永远存在来源于该类的强引用，那么该TheadLocal本身就不会被回收，Thead中的Map是否使用弱引用好像毫无意义。

当时也是思考了比较久，后面才发现钻了牛角尖，当时大牛们设计出来，应该是考虑类似情况：

```java
public class Test {
    ThreadLocal<String> threadLocal;
    public void run(){
        this.threadLocal = new ThreadLocal<>();
        this.threadLocal.set("abc");
        //do something
        //……
        //end
    }
    public static void main(String[] args) throws Exception {
        new Test().run();
    }
}
```

我们现在当然知道，不使用static定义，肯定是不合适的，但是Java的大牛们作为Api的设计者，肯定是要考虑到其他人在任何情况下使用该工具都应是安全的。而在这种使用方式下，在线程被销毁前，显然如果线程中的Map对该TheadLocal为强引用，那么在该方法执行完毕后该方法内的强引用消失后，依旧在Map上会直接造成内存泄露。
这也能使人明白为什么Java不选择在TheadLocal中维护一个以Thead为Key的Map来实现。