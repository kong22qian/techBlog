[TOC]

# Common

## 自我介绍

## 网络

### 网络协议层次

> 七层划分为：应用层、表示层、会话层、[传输层](https://so.csdn.net/so/search?q=传输层&spm=1001.2101.3001.7020)、网络层、数据链路层、物理层。
>
> 五层划分为[TCP/IP]：应用层、传输层、网络层、数据链路层、物理层。
>
> 四层划分为：应用层、传输层、网络层、网络接口层。

![在这里插入图片描述](https://img-blog.csdn.net/20180802094638614)

### http三次握手 四次挥手

**三次握手：**

![c29f153929b2e6d6b5c92262848de5c4.png](https://img-blog.csdnimg.cn/img_convert/c29f153929b2e6d6b5c92262848de5c4.png)

> 1. 客户端向服务端发送一个数据包，告诉服务端需要建立连接;
> 2. 服务端收到客户端发送的数据包之后，会返回一个数据包，通知客户端，我已经收到你的连接请求;
> 3. 客户端收到服务端返回的信息后，知道服务端已经准备好建立连接，但是还需要再发送一个数据包给服务端，用于告诉服务端我已经收到你的回复了。

**四次挥手：**

![09e1892c5dd97c0ea841192122b0e4ee.png](https://img-blog.csdnimg.cn/img_convert/09e1892c5dd97c0ea841192122b0e4ee.png)

> 首先进行关闭的一方将执行主动关闭，而另一方则执行被动关闭。
>
> 1. 第一次挥手：client发送一个FIN，用来关闭client到server的数据传输，client进入FIN_WAIT_1状态；
> 2. 第二次挥手：server收到FIN后，发送一个ACK给client，确认序号为收到序号+1，server进入CLOSE_WAIT状态；
> 3. 第三次挥手：server发送一个FIN，用来关闭server到client的数据传输，server进入LAST_ACK状态；
> 4. 第四次挥手：client收到FIN后，client进入TIME_WAIT状态，接着发送一个ACK给server，确认序号为收到序号+1，server进入CLOSE状态。
>
> 完成四次挥手。

### HTTP 与 HTTPS 区别

> - HTTP 明文传输，数据都是未加密的，安全性较差，HTTPS（SSL+HTTP） 数据传输过程是加密的，安全性较好。
> - 使用 HTTPS 协议需要到 CA（Certificate Authority，数字证书认证机构） 申请证书，一般免费证书较少，因而需要一定费用。证书颁发机构如：Symantec、Comodo、GoDaddy 和 GlobalSign 等。
> - HTTP 页面响应速度比 HTTPS 快，主要是因为 HTTP 使用 TCP 三次握手建立连接，客户端和服务器需要交换 3 个包，而 HTTPS除了 TCP 的三个包，还要加上 ssl 握手需要的 9 个包，所以一共是 12 个包。
> - http 和 https 使用的是完全不同的连接方式，用的端口也不一样，前者是 80，后者是 443。
> - HTTPS 其实就是建构在 SSL/TLS 之上的 HTTP 协议，所以，要比较 HTTPS 比 HTTP 要更耗费服务器资源。

### HTTPS 的工作原理

我们都知道 HTTPS 能够加密信息，以免敏感信息被第三方获取，所以很多银行网站或电子邮箱等等安全级别较高的服务都会采用 HTTPS 协议。

![img](https://www.runoob.com/wp-content/uploads/2018/09/https-intro.png)

## Java 基础

### == 和 [equals](https://so.csdn.net/so/search?q=equals&spm=1001.2101.3001.7020) 的区别是什么？

> 1. 对于基本类型，==比较的是值；
> 2. 对于引用类型，==比较的是地址；
> 3. equals不能用于基本类型的比较；
> 4. 如果没有重写equals，equals就相当于==；
> 5. 如果重写了equals方法，equals比较的是对象的内容

### java 中 IO 流分为几种？

> 1、按流划分，可以分为输入流和输出流；
>
> 2、按单位划分，可以分为字节流和字符流；
>
> 字节流：inputStream、outputStream；
>
> 字符流：reader、writer；

### ArrayList、LinkedList、Vector的区别

> List的三个子类的特点
>
> - ArrayList:
>
> ```
> 底层数据结构是数组，查询快，增删慢。
> 线程不安全，效率高。
> ```
>
> - Vector:
>
> ```
> 底层数据结构是数组，查询快，增删慢。
> 线程安全，效率低。
> Vector相对ArrayList查询慢(线程安全的)。
> Vector相对LinkedList增删慢(数组结构)。
> ```
>
> - LinkedList
>
> ```
> 底层数据结构是链表，查询慢，增删快。
> 线程不安全，效率高。
> ```
>
> **Vector和ArrayList的区别:**
>
> Vector是线程安全的,效率低。
> ArrayList是线程不安全的,效率高。
> 共同点:底层数据结构都是数组实现的,查询快,增删慢。
>
> **ArrayList和LinkedList的区别:**
>
> ArrayList底层是数组结果,查询和修改快。
> LinkedList底层是链表结构的,增和删比较快,查询和修改比较慢。
> 共同点:都是线程不安全的
>
> **List有三个子类使用:**
>
> ```
> 查询多用ArrayList。
> 增删多用LinkedList。
> 如果都多ArrayList
> ```

### Map、Set、List、Queue、Stack的特点与用法

> - Map
>
> ```
> Map是键值对，键Key是唯一不能重复的，一个键对应一个值，值可以重复。
> TreeMap可以保证顺序。
> HashMap不保证顺序，即为无序的。
> Map中可以将Key和Value单独抽取出来，其中KeySet()方法可以将所有的keys抽取正一个Set。而Values()方法可以将map中所有的values抽取成一个集合。
> ```
>
> - Set
>
> ```
> 不包含重复元素的集合，set中最多包含一个null元素。
> 只能用Lterator实现单项遍历，Set中没有同步方法。
> ```
>
> - List
>
> ```
> 有序的可重复集合。
> 可以在任意位置增加删除元素。
> 用Iterator实现单向遍历，也可用ListIterator实现双向遍历。
> ```
>
> - Queue
>
> ```
> Queue遵从先进先出原则。
> 使用时尽量避免add()和remove()方法,而是使用offer()来添加元素，使用poll()来移除元素，它的优点是可以通过返回值来判断是否成功。
> LinkedList实现了Queue接口。
> Queue通常不允许插入null元素。
> ```
>
> - Stack
>
> ```
> Stack遵从后进先出原则。
> Stack继承自Vector。
> 它通过五个操作对类Vector进行扩展，允许将向量视为堆栈，它提供了通常的push和pop操作，以及取堆栈顶点的peek()方法、测试堆栈是否为空的empty方法等。
> ```
>
> **用法**
>
> 如果涉及堆栈，队列等操作，建议使用List。
> 对于快速插入和删除元素的，建议使用LinkedList。
> 如果需要快速随机访问元素的，建议使用ArrayList。

### Collection

**Collection 是对象集合， Collection 有两个子接口 List 和 Set**

```
List 可以通过下标 (1,2..) 来取得值，值可以重复。 Set 只能通过游标来取值，并且值是不能重复的。

ArrayList ， Vector ， LinkedList 是 List 的实现类

ArrayList 是线程不安全的， Vector 是线程安全的，这两个类底层都是由数组实现的。
LinkedList 是线程不安全的，底层是由链表实现的。
```

**Map 是键值对集合**

```
HashTable 和 HashMap 是 Map 的实现类。
HashTable 是线程安全的，不能存储 null 值。
HashMap 不是线程安全的，可以存储 null 值。
Stack类：继承自Vector，实现一个后进先出的栈。提供了几个基本方法，push、pop、peak、empty、search等。
```

### **JDK8中的HashMap**

JDK8中采用的是==位桶+链表/红黑树==（有关红黑树请查看红黑树）的方式，也是非线程安全的。当某个位桶的链表的长度达到某个阀值的时候，这个链表就将转换成红黑树。

JDK8中，当同一个hash值的节点数不小于8时，将不再以单链表的形式存储了，会被调整成一颗红黑树（上图中null节点没画）。这就是JDK7与JDK8中HashMap实现的最大区别。

### Override和Overload的含义去区别

**Override（重写）**

```
方法名、参数、返回值相同。
子类方法不能缩小父类方法的访问权限。
子类方法不能抛出比父类方法更多的异常(但子类方法可以不抛出异常)。
存在于父类和子类之间。
方法被定义为final不能被重写。
```

**Overload（重载）**

```
参数类型、个数、顺序至少有一个不相同。
不能重载只有返回值不同的方法名。
存在于父类和子类、同类中
```

### Java8(JDK1.8)新特性

> 1、Lamdba表达式
>
> 2、[函数式接口](https://so.csdn.net/so/search?q=函数式接口&spm=1001.2101.3001.7020)
>
> 3、[方法引用](https://so.csdn.net/so/search?q=方法引用&spm=1001.2101.3001.7020)和构造引用
>
> 4、Stream API
>
> 5、接口中的默认方法和静态方法
>
> 6、新时间日期API
>
> 7、OPtional

优点：

> 1、速度快；
>
> 2、代码少、简洁（新增特性:lamdba表达式）；
>
> 3、强大的Stream API；
>
> 4、使用并行流和串行流；
>
> 5、最大化减少[空指针异常](https://so.csdn.net/so/search?q=空指针异常&spm=1001.2101.3001.7020)Optional；
>
> 其中最为核心的是**Lambda表达式和Stream API**

### jdk8内置函数式接口

**Function、Consumer、Supplier、Predicate**

### 什么是双亲委派模型？

**双亲委派模型（Parents Delegation Model）要求除了顶层的启动类加载器外，其余加载器都应当有自己的父类加载器。类加载器之间的父子关系，通过组合关系复用。
工作过程：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把这个请求委派给父类加载器完成。每个层次的类加载器都是如此，因此所有的加载请求最终都应该传送到顶层的启动类加载器中，只有到父加载器反馈自己无法完成这个加载请求（它的搜索范围没有找到所需的类）时，子加载器才会尝试自己去加载。**



## 多线程

### 创建线程的方式

> 1. 继承Thread类
> 2. 实现Runnable接口
> 3. 通过Callable和Future创建线程JDK5升级
> 4. 创建线程池（当前最普遍）

### Runnable和Callable的区别什么是Future

> Runnable接口中的run()方法的返回值是void，它做的事情只是纯粹地去执行run()方法中的代码而已；
> Callable接口中的call()方法是有返回值的，是一个泛型，和Future、FutureTask配合可以用来拿到异步执行任务的返回值。。Callable用于产生结果，Future用于获取结果。
>
> 而Callable+Future/FutureTask却可以方便获取多线程运行的结果和线程运行状态等信息

### Thread类中的start()和run()方法有什么区别?

> start()方法被用来启动新创建的线程，而且start()内部调用了run()方法，所以他会开启新线程并执行run方法
> 当你调用run()方法的时候，只会是在原来的线程中调用方法，没有新的线程启动。所以还是单线程的。

### sleep() 方法和 wait() 方法区别?

> 释放锁：sleep 方法没有释放锁，而 wait 方法释放了锁 。
> 作用：Wait 通常被用于线程间通信，sleep 用于暂停执行。
> 苏醒：wait() 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 notify() 或者 notifyAll() 方法。sleep() 方法执行完成后，线程会自动苏醒。或者可以使用wait(long timeout)超时后线程会自动苏醒。

### 线程六种状态

[java多线程](https://so.csdn.net/so/search?q=java多线程&spm=1001.2101.3001.7020)包含六种状态，分别是NEW，RUNNABLE，BLOCKED，WAITING，TIMED_WAITING，TERMINATED。

> - NEW:初始状态；在thread被new出来，但并没有调用start方法时，线程将处于这个状态。
> - RUNNABLE：运行状态（又具体分成正在运行状态和准备状态）；当我们执行start方法后，线程状态变成该状态。
> - BLOCKED：阻塞状态；如果有其他线程持有锁，当该线程也要尝试拿锁的时候，就会进入到阻塞状态。
> - WAITING：等待状态；执行wait，join，park等方法时，线程就会进入等待状态。进行特定的操作才能从等待状态回到RUNNABLE状态。
> - TIMED_WAITING：有时间的等待状态；除了进行特定的操作解除等待外，还会因为时间到了自动解除等待。
> - TERMINATED：结束状态。正常运行完，或者终止都会进入到这个状态。

### JMM三大特性（也即多线程三大特性）

JMM三大特性（也即多线程三大特性）：可见性，原子性，有序性。

## JVM基础

### Java内存区域（JVM内存区域）

![img](https://images2018.cnblogs.com/blog/1332556/201805/1332556-20180530105521301-2013126296.png)

 

Java虚拟机在运行程序时会把其自动管理的内存划分为以上几个区域，每个区域都有的用途以及创建销毁的时机，其中蓝色部分代表的是所有线程共享的数据区域，而绿色部分代表的是每个线程的私有数据区域。

- 方法区（Method Area）：

  方法区属于线程共享的内存区域，又称Non-Heap（非堆），主要用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等数据，根据Java 虚拟机规范的规定，当方法区无法满足内存分配需求时，将抛出OutOfMemoryError 异常。值得注意的是在方法区中存在一个叫运行时常量池(Runtime Constant Pool）的区域，它主要用于存放编译器生成的各种字面量和符号引用，这些内容将在类加载后存放到运行时常量池中，以便后续使用。

- JVM堆（Java Heap）：

  Java 堆也是属于线程共享的内存区域，它在虚拟机启动时创建，是Java 虚拟机所管理的内存中最大的一块，主要用于存放对象实例，几乎所有的对象实例都在这里分配内存，注意Java 堆是垃圾收集器管理的主要区域，因此很多时候也被称做GC 堆，如果在堆中没有内存完成实例分配，并且堆也无法再扩展时，将会抛出OutOfMemoryError 异常。

- 程序计数器(Program Counter Register)：

  属于线程私有的数据区域，是一小块内存空间，主要代表当前线程所执行的字节码行号指示器。字节码解释器工作时，通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等基础功能都需要依赖这个计数器来完成。

- 虚拟机栈(Java Virtual Machine Stacks)：

  属于线程私有的数据区域，与线程同时创建，总数与线程关联，代表Java方法执行的内存模型。栈中只保存基础数据类型和自定义对象的引用(不是对象)，对象都存放在堆区中。每个方法执行时都会创建一个栈桢来存储方法的的变量表、操作数栈、动态链接方法、返回值、返回地址等信息。每个方法从调用直结束就对于一个栈桢在虚拟机栈中的入栈和出栈过程，如下（图有误，应该为栈桢）：

  ![img](https://img-blog.csdn.net/20170608151435751?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvamF2YXplamlhbg==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- 本地方法栈(Native Method Stacks)：

  本地方法栈属于线程私有的数据区域，这部分主要与虚拟机用到的 Native 方法相关，一般情况下，我们无需关心此区域。

 ### happens-before具体规则

> - 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作。
> - 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁。
> - volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读。 
> - 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。
> - start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A线程的ThreadB.start()操作happens-before于线程B中的任意操作。
> - join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before于线程A从ThreadB.join()操作成功返回。

### 如何判断对象是否“死去”？

1. 引用计数法

   > 给对象添加一个引用计数器，每当有一个地方引用它，计数器就+1,；当引用失效时，计数器就-1；任何时刻计数器都为0的对象就是不能再被使用的。
   >
   > ==引用计数法的缺点？==
   >
   > 很难解决对象之间的循环引用问题。

2. 根搜索算法

   > 通过一系列的名为==“GC Roots”==的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链==（Reference Chain）==，当一个对象到 GC Roots 没有任何引用链相连（用图论的话来说就是从 GC Roots 到这个对象不可达）时，则证明此对象是不可用的。

### 有哪些垃圾收集算法？

> 1. 标记-清除算法
> 2. 复制算法
> 3. 标记-整理算法
> 4. 分代收集算法

### Minor GC 和 Full GC有什么区别？

==Minor GC：新生代 GC，指发生在新生代的垃圾收集动作，因为 Java 对象大多死亡频繁，所以 Minor GC 非常频繁，一般回收速度较快。
Full GC：老年代 GC，也叫 Major GC，速度一般比 Minor GC 慢 10 倍以上。==

### 线上常用的 JVM 参数有哪些

> ==数据区设置==
> Xms：初始堆大小
> Xmx：最大堆大小
> Xss:Java 每个线程的Stack大小
> XX:NewSize=n：设置年轻代大小
> XX:NewRatio=n：设置年轻代和年老代的比值。如：为 3，表示年轻代与年老代比值为 1:3，年轻代占整个年轻代年老代和的 1/4。
> XX：SurvivorRatio=n：年轻代中 Eden 区与两个 Survivor 区的比值。注意 Survivor 区有两个。如：3，表示 Eden：Survivor=3：2，一个 Survivor 区占整个年轻代的 1/5。
> XX：MaxPermSize=n：设置持久代大小。
>
> ==收集器设置==
> XX:+UseSerialGC：设置串行收集器
> XX:+UseParallelGC:：设置并行收集器
> XX:+UseParalledlOldGC：设置并行年老代收集器
> XX:+UseConcMarkSweepGC：设置并发收集器
>
> ==GC日志打印设置==
> XX:+PrintGC：打印 GC 的简要信息
> XX:+PrintGCDetails：打印 GC 详细信息
> XX:+PrintGCTimeStamps：输出 GC 的时间戳

## Redis

全文

## rabbitMq

全文

## Mysql

### 日常工作中你是怎么优化SQL的？

> - 加索引
> - 避免返回不必要的数据
> - 适当分批量进行
> - 优化sql结构
> - 分库分表
> - 读写分离

### InnoDB与MyISAM的区别

> **InnoDB支持事务，MyISAM不支持事务**
> InnoDB支持外键，MyISAM不支持外键
> InnoDB 支持 MVCC(多版本并发控制)，MyISAM 不支持
> select count(*) from table时，MyISAM更快，因为它有一个变量保存了整个表的总行数，可以直接读取，InnoDB就需要全表扫描。
> Innodb不支持全文索引，而MyISAM支持全文索引（5.7以后的InnoDB也支持全文索引）
> InnoDB支持表、行级锁，而MyISAM支持表级锁。
> InnoDB表必须有主键，而MyISAM可以没有主键
> Innodb表需要更多的内存和存储，而MyISAM可被压缩，存储空间较小，。
> Innodb按主键大小有序插入，MyISAM记录插入顺序是，按记录插入顺序保存。
> InnoDB 存储引擎提供了具有提交、回滚、崩溃恢复能力的事务安全，与 MyISAM 比 InnoDB 写的效率差一些，并且会占用更多的磁盘空间以保留数据和索引

### 事务的隔离级别有哪些？MySQL的默认隔离级别是什么

- 读未提交（Read Uncommitted）
- 读已提交（Read Committed）
- 可重复读（Repeatable Read）
- 串行化（Serializable）

### [脏读](https://so.csdn.net/so/search?q=脏读&spm=1001.2101.3001.7020)、幻读和[不可重复读](https://so.csdn.net/so/search?q=不可重复读&spm=1001.2101.3001.7020)

> 1、脏读：脏读就是指当一个事务正在访问数据，并且对数据进行了修改，而这种修改还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。
> 例如：
> 张三的工资为5000,事务A中把他的工资改为8000,但事务A尚未提交。
> 与此同时，
> 事务B正在读取张三的工资，读取到张三的工资为8000。
> 随后，
> 事务A发生异常，而回滚了事务。张三的工资又回滚为5000。
> 最后，
> 事务B读取到的张三工资为8000的数据即为脏数据，事务B做了一次脏读。
>
> 2、不可重复读：是指在一个事务内，多次读同一数据。在这个事务还没有结束时，另外一个事务也访问该同一数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改，那么第一个事务两次读到的的数据可能是不一样的。这样就发生了在一个事务内两次读到的数据是不一样的，因此称为是不可重复读。
> 例如：
> 在事务A中，读取到张三的工资为5000，操作没有完成，事务还没提交。
> 与此同时，
> 事务B把张三的工资改为8000，并提交了事务。
> 随后，
> 在事务A中，再次读取张三的工资，此时工资变为8000。在一个事务中前后两次读取的结果并不致，导致了不可重复读。
>
> 3、幻读：是指当事务不是独立执行时发生的一种现象，例如第一个事务对一个表中的数据进行了修改，这种修改涉及到表中的全部数据行。同时，第二个事务也修改这个表中的数据，这种修改是向表中插入一行新数据。那么，以后就会发生操作第一个事务的用户发现表中还有没有修改的数据行，就好象发生了幻觉一样。
> 例如：
> 目前工资为5000的员工有10人，事务A读取所有工资为5000的人数为10人。
> 此时，
> 事务B插入一条工资也为5000的记录。
> 这是，事务A再次读取工资为5000的员工，记录为11人。此时产生了幻读。

### SQL优化的一般步骤是什么，怎么看执行计划（explain），如何理解其中各个字段的含义。

- show status 命令了解各种 sql 的执行频率
- 通过慢查询日志定位那些执行效率较低的 sql 语句
- explain 分析低效 sql 的执行计划（这点非常重要，日常开发中用它分析Sql，会大大降低Sql导致的线上事故）

### MySQL事务得四大特性以及实现原理

![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC81LzIxLzE3MjM3ZjQ1YTY2MWNiYjc?x-oss-process=image/format,png)

> - 原子性： 事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行。
> - 一致性： 指在事务开始之前和事务结束以后，数据不会被破坏，假如A账户给B账户转10块钱，不管成功与否，A和B的总金额是不变的。
> - 隔离性： 多个事务并发访问时，事务之间是相互隔离的，即一个事务不影响其它事务运行效果。简言之，就是事务之间是进水不犯河水的。
> - 持久性： 表示事务完成以后，该事务对数据库所作的操作更改，将持久地保存在数据库之中。

==**事务ACID特性的实现思想**==

> - 原子性：是使用 undo log来实现的，如果事务执行过程中出错或者用户执行了rollback，系统通过undo log日志返回事务开始的状态。
> - 持久性：使用 redo log来实现，只要redo log日志持久化了，当系统崩溃，即可通过redo log把数据恢复。
> - 隔离性：通过锁以及MVCC,使事务相互隔离开。
> - 一致性：通过回滚、恢复，以及并发情况下的隔离性，从而实现一致性。

### UNION与UNION ALL的区别？

> - Union：对两个结果集进行并集操作，不包括重复行，同时进行默认规则的排序；
> - Union All：对两个结果集进行并集操作，包括重复行，不进行排序；
> - UNION的效率高于 UNION ALL

### Mysql数据结构是B+树，查询效率快

## Springboot

### Spring的 IOC和[AOP](https://so.csdn.net/so/search?q=AOP&spm=1001.2101.3001.7020)机制 ？

**IOC**

> 控制反转（Inversion of Control，缩写为IoC），是面向对象编程中的一种设计原则，可以用来减低计算机代码之间的耦合度。其中最常见的方式叫做依赖注入（Dependency Injection，简称DI），还有一种方式叫“依赖查找”（Dependency Lookup）。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体将其所依赖的对象的引用传递给它。也可以说，依赖被注入到对象中。

简单总结一下IOC的作用就是降低代码之间的耦合程度，而在Spring中大多使用了依赖注入，依赖查找使用较少。

**AOP**

> 在软件业，AOP为Aspect Oriented Programming的缩写，意为：面向切面编程，通过预编译方式和运行期间动态代理实现程序功能的统一维护的一种技术。AOP是OOP的延续，是软件开发中的一个热点，也是Spring框架中的一个重要内容，是函数式编程的一种衍生范型。利用AOP可以对业务逻辑的各个部分进行隔离，从而使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，同时提高了开发的效率。

### 解释一下 Spring bean的生命周期

**什么是 Spring Bean 的生命周期**

> 对于普通的 Java 对象，当 new 的时候创建对象，然后该对象就能够使用了。一旦该对象不再被使用，则由 Java 自动进行垃圾回收。
>
> 而 Spring 中的对象是 bean，bean 和普通的 Java 对象没啥大的区别，只不过 Spring 不再自己去 new 对象了，而是由 IoC 容器去帮助我们实例化对象并且管理它，我们需要哪个对象，去问 IoC 容器要即可。IoC 其实就是解决对象之间的耦合问题，Spring Bean 的生命周期完全由容器控制。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210707225212729.png?)

### springboot自动装配原理

1.springboot自动装配主要是基于注解编程，和预定优于配置的思想来进行设计的

> 自动装配就是自动地把其他组件中的Bean装载到IOC容器中，不需要开发人员再去配置文件中添加大量的配置，
>
> 我们只需要在springboot的启动类上添加一个SptingBootApplication的一个注解，这样就可以开启自动装配
>
> 这种自动装配的思想在spring3.x以后就支持了，我们只需要在类上添加一个叫做@Enable的注解就可以了，只是spring没有向SpringBoot这样全面去设计，
>
> 因此**Spring和SpringBoot最大的区别就是在于SpringBoot的自动装配**

2.自动装配的原理又是什么？

@SptingBootApplication这个注解是暴露给用户使用的一个入口，它的底层其实是由**@EnableAutoConfiguration这个注解来实现**的，

自动装配的实现，归纳为以下三个核心的步骤：

- 第一步：

> 启动依赖组件的时候
>
> 组件中必须要包含@Configuration的配置类，在这个配置类里面声明为Bean注解，然后将方法的返回值或者是属性注入到IOC容器中

- 第二部：

> 第三方jar包，SpringBoot会采用SPI机制，在/META-INF/目录下增加**spring.factories**文件，然后SpringBoot会自动根据约定，自动使用SpringFactoriesLoader来加载配置文件中的内容

- 第三步：

> Spring获取到第三方jar中的配置以后会调用**ImportSelector**接口来完成动态加载，
>
> 这样设计的好处，在于大幅度减少了臃肿的配置文件，而各模块之间的依赖，也深度的解耦，
>
> 比如我们使用Spring创建Web程序的时候需要引用非常多的Maven依赖，而SpringBoot中只需要引用一个Maven依赖就可以来创建Web程序
>
> 并且SpringBoot把我们常用的依赖都放在了一起，，我们只需要去引入spring-boot-starter-web这个依赖就可以去完成一个简单的Web应用
>
> 以前我们使用Spring的时候需要xml文件来配置开启一些功能，现在使用SpringBoot就不需要xml文件了，
>
> 只需要一个加了@Configuration注解的类，或者是实现了对因接口的配置类就可以了

## Springcloud使用过吗，有哪些组件

> 这就有很多了，我讲几个开发中最重要的
>
> - Spring Cloud Eureka：服务注册与发现
> - Spring Cloud Zuul：服务网关
> - Spring Cloud Ribbon：客户端负载均衡
> - Spring Cloud Feign：声明性的Web服务客户端
> - Spring Cloud Hystrix：断路器
> - Spring Cloud Config：分布式统一配置管理
> - 等20几个框架，开源一直在更新

## 生产过程中遇到过的问题，怎么解决的

短信网关超频pdf