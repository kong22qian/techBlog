[TOC]

# JAVA BASE INFO

##  JDK 和 JRE 有什么区别？

> JDK（Java Development Kit），Java开发工具包
>
> JRE（Java Runtime Environment），Java运行环境
>
> JDK中包含JRE，JDK中有一个名为jre的目录，里面包含两个文件夹bin和lib，bin就是JVM，lib就是JVM工作所需要的类库。

## == 和 [equals](https://so.csdn.net/so/search?q=equals&spm=1001.2101.3001.7020) 的区别是什么？

> 1. 对于基本类型，==比较的是值；
> 2. 对于引用类型，==比较的是地址；
> 3. equals不能用于基本类型的比较；
> 4. 如果没有重写equals，equals就相当于==；
> 5. 如果重写了equals方法，equals比较的是对象的内容

## final 在 java 中有什么作用？

> 1. final修饰的成员变量，必须在声明的同时赋值，一旦创建不可修改；
> 2. final修饰的方法，不能被子类重写；
> 3. final类中的方法默认是final的；
> 4. private类型的方法默认是final的；

## Object有哪些公用方法

> Object是所有类的父类，任何类都默认继承Object
>
> clone 保护方法，实现对象的浅复制，只有实现了Cloneable接口才可以调用该方法，否则抛出CloneNotSupportedException异常。
>
> equals 在Object中与==是一样的，子类一般需要重写该方法。
>
> hashCode 该方法用于哈希查找，重写了equals方法一般都要重写hashCode方法。这个方法在一些具有哈希功能的Collection中用到。
>
> getClass final方法，获得运行时类型
>
> wait 使当前线程等待该对象的锁，当前线程必须是该对象的拥有者，也就是具有该对象的锁。 wait() 方法一直等待，直到获得锁或者被中断。 wait(long timeout) 设定一个超时间隔，如果在规定时间内没有获得锁就返回。
>
> 调用该方法后当前线程进入睡眠状态，直到以下事件发生
>
> 1、其他线程调用了该对象的notify方法。 2、其他线程调用了该对象的notifyAll方法。 3、其他线程调用了interrupt中断该线程。 4、时间间隔到了。 5、此时该线程就可以被调度了，如果是被中断的话就抛出一个InterruptedException异常。
>
> notify 唤醒在该对象上等待的某个线程。
>
> notifyAll 唤醒在该对象上等待的所有线程。
>
> toString 转换成字符串，一般子类都有重写，否则打印句柄。

## java 中的 Math.round(-1.5) 等于多少？

> Math提供了三个与取整有关的方法：ceil、floor、round
>
> 1、ceil：向上取整；
>
> Math.ceil(11.3) = 12;
>
> Math.ceil(-11.3) = 11;
>
> 2、floor：向下取整；
>
> Math.floor(11.3) = 11;
>
> Math.floor(-11.3) = -12;
>
> 3、round：四舍五入；
>
> 加0.5然后向下取整。
>
> Math.round(11.3) = 11;
>
> Math.round(11.8) = 12;
>
> Math.round(-11.3) = -11;
>
> Math.round(-11.8) = -12;

## String 属于基础的数据类型吗？

> 不属于。
>
> 八种基本数据类型：byte、short、char、int、long、double、float、boolean。

## 八种基本数据类型的大小，以及他们的封装类

> 八种基本数据类型：int、short、float、double、long、boolean、byte、char。
>
> 封装类分别是：Integer、Short、Float、Double、Long、Boolean、Byte、Character。

## String str="i"与 String str=new String(“i”)一样吗？

> String str="i"会将起分配到常量池中，常量池中没有重复的元素，如果常量池中存中i，就将i的地址赋给变量，如果没有就创建一个再赋给变量。
>
> String str=new String(“i”)会将对象分配到堆中，即使内存一样，还是会重新创建一个新的对象。

## 如何将字符串反转？

> 将对象封装到stringBuilder中，调用reverse方法反转。

![img](https://img-blog.csdnimg.cn/20210528233452290.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

## String 类的常用方法都有那些？

> 1、常见String类的获取功能
>
> length：获取字符串长度；
> charAt(int index)：获取指定索引位置的字符；
> indexOf(int ch)：返回指定字符在此字符串中第一次出现处的索引；
> substring(int start)：从指定位置开始截取字符串,默认到末尾；
> substring(int start,int end)：从指定位置开始到指定位置结束截取字符串;
>
> 2、常见String类的判断功能
>
> equals(Object obj)： 比较字符串的内容是否相同,区分大小写；
> contains(String str): 判断字符串中是否包含传递进来的字符串；
> startsWith(String str): 判断字符串是否以传递进来的字符串开头；
> endsWith(String str): 判断字符串是否以传递进来的字符串结尾；
> isEmpty(): 判断字符串的内容是否为空串""；
>
> 3、常见String类的转换功能
>
> byte[] getBytes(): 把字符串转换为字节数组；
> char[] toCharArray(): 把字符串转换为字符数组；
> String valueOf(char[] chs): 把字符数组转成字符串。valueOf可以将任意类型转为字符串；
> toLowerCase(): 把字符串转成小写；
> toUpperCase(): 把字符串转成大写；
> concat(String str): 把字符串拼接；
>
> 4、常见String类的其他常用功能
>
> replace(char old,char new) 将指定字符进行互换
> replace(String old,String new) 将指定字符串进行互换
> trim() 去除两端空格
> int compareTo(String str) 会对照ASCII 码表 从第一个字母进行减法运算 返回的就是这个减法的结果，如果前面几个字母一样会根据两个字符串的长度进行减法运算返回的就是这个减法的结果，如果连个字符串一摸一样 返回的就是0。

## 普通类和抽象类有哪些区别？

> 1. 抽象类不能被实例化；
> 2. 抽象类可以有抽象方法，只需申明，无须实现；
> 3. 有抽象方法的类一定是抽象类；
> 4. 抽象类的子类必须实现抽象类中的所有抽象方法，否则子类仍然是抽象类；
> 5. 抽象方法不能声明为静态、不能被static、final修饰。

## 接口和抽象类有什么区别？

> 1、接口
>
> 接口使用interface修饰；
> 接口不能实例化；
> 类可以实现多个接口；
> ①java8之前，接口中的方法都是抽象方法，省略了public abstract。②java8之后；接口中可以定义静态方法，静态方法必须有方法体，普通方法没有方法体，需要被实现；
> 2、抽象类
>
> 抽象类使用abstract修饰；
> 抽象类不能被实例化；
> 抽象类只能单继承；
> 抽象类中可以包含抽象方法和非抽象方法，非抽象方法需要有方法体；
> 如果一个类继承了抽象类，①如果实现了所有的抽象方法，子类可以不是抽象类；②如果没有实现所有的抽象方法，子类仍然是抽象类。

## java 中 IO 流分为几种？

> 1、按流划分，可以分为输入流和输出流；
>
> 2、按单位划分，可以分为字节流和字符流；
>
> 字节流：inputStream、outputStream；
>
> 字符流：reader、writer；

## BIO、NIO、AIO 有什么区别？

> 1、同步阻塞BIO
>
> 一个连接一个线程。
>
> JDK1.4之前，建立网络连接的时候采用BIO模式，先在启动服务端socket，然后启动客户端socket，对服务端通信，客户端发送请求后，先判断服务端是否有线程响应，如果没有则会一直等待或者遭到拒绝请求，如果有的话会等待请求结束后才继续执行。
>
> 2、同步非阻塞NIO
>
> NIO主要是想解决BIO的大并发问题，BIO是每一个请求分配一个线程，当请求过多时，每个线程占用一定的内存空间，服务器瘫痪了。
>
> JDK1.4开始支持NIO，适用于连接数目多且连接比较短的架构，比如聊天服务器，并发局限于应用中。
>
> 一个请求一个线程。
>
> 3、异步非阻塞AIO
>
> 一个有效请求一个线程。
>
> JDK1.7开始支持AIO，适用于连接数目多且连接比较长的结构，比如相册服务器，充分调用OS参与并发操作。

## Files的常用方法都有哪些？

> 1. exist
> 2. createFile
> 3. createDirectory
> 4. write
> 5. read
> 6. copy
> 7. size
> 8. delete
> 9. move

## 什么是反射？

> 所谓反射，是java在运行时进行自我观察的能力，通过class、constructor、field、method四个方法获取一个类的各个组成部分。
>
> 在Java运行时环境中，对任意一个类，可以知道类有哪些属性和方法。这种动态获取类的信息以及动态调用对象的方法的功能来自于反射机制。

## 什么是 java 序列化？什么情况下需要序列化？

> 序列化就是一种用来处理对象流的机制。将对象的内容流化，将流化后的对象传输于网络之间。
>
> 序列化是通过实现serializable接口，该接口没有需要实现的方法，implement Serializable只是为了标注该对象是可被序列化的，使用一个输出流（FileOutputStream）来构造一个ObjectOutputStream对象，接着使用ObjectOutputStream对象的writeObejct（Object object）方法就可以将参数的obj对象到磁盘，需要恢复的时候使用输入流。
>
> 序列化是将对象转换为容易传输的格式的过程。
>
> 例如，可以序列化一个对象，然后通过HTTP通过Internet在客户端和服务器之间传输该对象。在另一端，反序列化将从流中心构造成对象。
>
> 一般程序在运行时，产生对象，这些对象随着程序的停止而消失，但我们想将某些对象保存下来，这时，我们就可以通过序列化将对象保存在磁盘，需要使用的时候通过反序列化获取到。
>
> 对象序列化的最主要目的就是传递和保存对象，保存对象的完整性和可传递性。
>
> 譬如通过网络传输或者把一个对象保存成本地一个文件的时候，需要使用序列化。

## 为什么要使用克隆？如何实现对象克隆？深拷贝和浅拷贝区别是什么？

> 1、什么要使用克隆？
>
> 想对一个对象进行复制，又想保留原有的对象进行接下来的操作，这个时候就需要克隆了。
>
> 2、如何实现对象克隆？
>
> 实现Cloneable接口，重写clone方法；
> 实现Serializable接口，通过对象的序列化和反序列化实现克隆，可以实现真正的深克隆。
> BeanUtils，apache和Spring都提供了bean工具，只是这都是浅克隆。
> 3、深拷贝和浅拷贝区别是什么？
>
> 浅拷贝：仅仅克隆基本类型变量，不克隆引用类型变量；
> 深克隆：既克隆基本类型变量，又克隆引用类型变量；

## throw 和 throws 的区别？

> 1、throw
>
> - 作用在方法内，表示抛出具体异常，由方法体内的语句处理；
> - 一定抛出了异常；
>
> 2、throws
>
> - 作用在方法的声明上，表示抛出异常，由调用者来进行异常处理；
> - 可能出现异常，不一定会发生异常；

## final、finally、finalize 有什么区别？

> final可以修饰类，变量，方法，修饰的类不能被继承，修饰的变量不能重新赋值，修饰的方法不能被重写
>
> finally用于抛异常，finally代码块内语句无论是否发生异常，都会在执行finally，常用于一些流的关闭。
>
> finalize方法用于垃圾回收。
>
> 一般情况下不需要我们实现finalize，当对象被回收的时候需要释放一些资源，比如socket链接，在对象初始化时创建，整个生命周期内有效，那么需要实现finalize方法，关闭这个链接。
>
> 但是当调用finalize方法后，并不意味着gc会立即回收该对象，所以有可能真正调用的时候，对象又不需要回收了，然后到了真正要回收的时候，因为之前调用过一次，这次又不会调用了，产生问题。所以，不推荐使用finalize方法。

## try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？

![img](https://img-blog.csdnimg.cn/20210528233602741.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d1b3J1aV9qYXZh,size_16,color_FFFFFF,t_70)

## 常见的异常类有哪些？

> NullPointerException：空指针异常；
> SQLException：数据库相关的异常；
> IndexOutOfBoundsException：数组下角标越界异常；
> FileNotFoundException：打开文件失败时抛出；
> IOException：当发生某种IO异常时抛出；
> ClassCastException：当试图将对象强制转换为不是实例的子类时，抛出此异常；
> NoSuchMethodException：无法找到某一方法时，抛出；
> ArrayStoreException：试图将错误类型的对象存储到一个对象数组时抛出的异常；
> NumberFormatException：当试图将字符串转换成数字时，失败了，抛出；
> IllegalArgumentException 抛出的异常表明向方法传递了一个不合法或不正确的参数。
> ArithmeticException当出现异常的运算条件时，抛出此异常。例如，一个整数“除以零”时，抛出此类的一个实例。 

## hashcode是什么？有什么作用？

> Java中Object有一个方法：
>
> public native int hashcode();
>
> 1、hashcode()方法的作用
>
> hashcode()方法主要配合基于散列的集合一起使用，比如HashSet、HashMap、HashTable。
>
> 当集合需要添加新的对象时，先调用这个对象的hashcode()方法，得到对应的hashcode值，实际上hashmap中会有一个table保存已经存进去的对象的hashcode值，如果table中没有改hashcode值，则直接存入，如果有，就调用equals方法与新元素进行比较，相同就不存了，不同就存入。
>
> 2、equals和hashcode的关系
>
> 如果equals为true，hashcode一定相等； 
>
> 如果equals为false，hashcode不一定不相等；
>
> 如果hashcode值相等，equals不一定相等；
>
> 如果hashcode值不等，equals一定不等；
>
> 3、重写equals方法时，一定要重写hashcode方法

```1.hashcode是用来查找的，如果你学过数据结构就应该知道，在查找和排序这一章有
例如内存中有这样的位置
0  1  2  3  4  5  6  7  
而我有个类，这个类有个字段叫ID,我要把这个类存放在以上8个位置之一，如果不用hashcode而任意存放，那么当查找时就需要到这八个位置里挨个去找，或者用二分法一类的算法。
但如果用hashcode那就会使效率提高很多。
我们这个类中有个字段叫ID,那么我们就定义我们的hashcode为ID％8，然后把我们的类存放在取得得余数那个位置。比如我们的ID为9，9除8的余数为1，那么我们就把该类存在1这个位置，如果ID是13，求得的余数是5，那么我们就把该类放在5这个位置。这样，以后在查找该类时就可以通过ID除 8求余数直接找到存放的位置了。
```

```2.但是如果两个类有相同的hashcode怎么办那（我们假设上面的类的ID不是唯一的），例如9除以8和17除以8的余数都是1，那么这是不是合法的，回答是：可以这样。那么如何判断呢？在这个时候就需要定义 equals了。
也就是说，我们先通过 hashcode来判断两个类是否存放某个桶里，但这个桶里可能有很多类，那么我们就需要再通过 equals 来在这个桶里找到我们要的类。
那么。重写了equals()，为什么还要重写hashCode()呢？
想想，你要在一个桶里找东西，你必须先要找到这个桶啊，你不通过重写hashcode()来找到桶，光重写equals()有什么用啊。
```

## java 中操作字符串都有哪些类？它们之间有什么区别？

> 1、String
>
> String是不可变对象，每次对String类型的改变时都会生成一个新的对象。
>
> 2、StringBuilder
>
> 线程不安全，效率高，多用于单线程。
>
> 3、StringBuffer
>
> 线程安全，由于加锁的原因，效率不如StringBuilder，多用于多线程。
>
> 不频繁的字符串操作使用String，操作频繁的情况不建议使用String。
>
> StringBuilder > StringBuffer > String。

## 在 Java 中，为什么不允许从静态方法中访问非静态变量？

> 1. 静态变量属于类本身，在类加载的时候就会分配内存，可以通过类名直接访问；
> 2. 非静态变量属于类的对象，只有在类的对象产生时，才会分配内存，通过类的实例去访问；
> 3. 静态方法也属于类本身，但是此时没有类的实例，内存中没有非静态变量，所以无法调用。

## 在 Java 中，什么时候用重载，什么时候用重写？

> 1、重载是多态的集中体现，在类中，要以统一的方式处理不同类型数据的时候，可以用重载。
>
> 2、重写的使用是建立在继承关系上的，子类在继承父类的基础上，增加新的功能，可以用重写。
>
> 3、简单总结：
>
> 重载是多样性，重写是增强剂；
> 目的是提高程序的多样性和健壮性，以适配不同场景使用时，使用重载进行扩展；
> 目的是在不修改原方法及源代码的基础上对方法进行扩展或增强时，使用重写；

## 举例说明什么情况下会更倾向于使用抽象类而不是接口？

> 接口和抽象类都遵循”面向接口而不是实现编码”设计原则，它可以增加代码的灵活性，可以适应不断变化的需求。下面有几个点可以帮助你回答这个问题：在 Java 中，你只能继承一个类，但可以实现多个接口。所以一旦你继承了一个类，你就失去了继承其他类的机会了。
>
> 接口通常被用来表示附属描述或行为如： Runnable 、 Clonable 、 Serializable 等等，因此当你使用抽象类来表示行为时，你的类就不能同时是 Runnable 和 Clonable( 注：这里的意思是指如果把 Runnable 等实现为抽象类的情况 ) ，因为在 Java 中你不能继承两个类，但当你使用接口时，你的类就可以同时拥有多个不同的行为。
>
> 在一些对时间要求比较高的应用中，倾向于使用抽象类，它会比接口稍快一点。如果希望把一系列行为都规范在类继承层次内，并且可以更好地在同一个地方进行编码，那么抽象类是一个更好的选择。有时，接口和抽象类可以一起使用，接口中定义函数，而在抽象类中定义默认的实现。

## Hashcode的作用

### **HashCode的特性**

> （1）HashCode的存在主要是用于查找的快捷性，如Hashtable，HashMap等，HashCode经常用于确定对象的存储地址。
>
> （2）如果两个对象相同，?equals方法一定返回true，并且这两个对象的HashCode一定相同。
>
> （3）两个对象的HashCode相同，并不一定表示两个对象就相同，即equals()不一定为true，只能够说明这两个对象在一个散列存储结构中。
>
> （4）如果对象的equals方法被重写，那么对象的HashCode也尽量重写。

### **HashCode作用**

> Java中的集合有两类，一类是List，再有一类是Set。前者集合内的元素是有序的，元素可以重复；后者元素无序，但元素不可重复。
>
> equals方法可用于保证元素不重复，但如果每增加一个元素就检查一次，若集合中现在已经有1000个元素，那么第1001个元素加入集合时，就要调用1000次equals方法。这显然会大大降低效率。?于是，Java采用了哈希表的原理。
>
> 哈希算法也称为散列算法，是将数据依特定算法直接指定到一个地址上。
>
> 这样一来，当集合要添加新的元素时，先调用这个元素的HashCode方法，就一下子能定位到它应该放置的物理位置上。
>
> （1）如果这个位置上没有元素，它就可以直接存储在这个位置上，不用再进行任何比较了。
>
> （2）如果这个位置上已经有元素了，就调用它的equals方法与新元素进行比较，相同的话就不存了。
>
> （3）不相同的话，也就是发生了Hash key相同导致冲突的情况，那么就在这个Hash key的地方产生一个链表，将所有产生相同HashCode的对象放到这个单链表上去，串在一起（很少出现）。
>
> 这样一来实际调用equals方法的次数就大大降低了，几乎只需要一两次。

#### **如何理解HashCode的作用：**

> 从Object角度看，JVM每new一个Object，它都会将这个Object丢到一个Hash表中去，这样的话，下次做Object的比较或者取这个对象的时候（读取过程），它会根据对象的HashCode再从Hash表中取这个对象。这样做的目的是提高取对象的效率。若HashCode相同再去调用equal

### **HashCode实践（如何用来查找）**

> HashCode是用于查找使用的，而equals是用于比较两个对象是否相等的。
>
> （1）例如内存中有这样的位置
>
> 0  1  2  3  4  5  6  7
> 而我有个类，这个类有个字段叫ID，我要把这个类存放在以上8个位置之一，如果不用HashCode而任意存放，那么当查找时就需要到这八个位置里挨个去找，或者用二分法一类的算法。
>
> 但以上问题如果用HashCode就会使效率提高很多 定义我们的HashCode为ID％8，比如我们的ID为9，9除8的余数为1，那么我们就把该类存在1这个位置，如果ID是13，求得的余数是5，那么我们就把该类放在5这个位置。依此类推。
>
> （2）但是如果两个类有相同的HashCode，例如9除以8和17除以8的余数都是1，也就是说，我们先通过?HashCode来判断两个类是否存放某个桶里，但这个桶里可能有很多类，那么我们就需要再通过equals在这个桶里找到我们要的类。

### HashMap的hashcode的作用

> hashCode的存在主要是用于查找的快捷性，如Hashtable，HashMap等，hashCode是用来在散列存储结构中确定对象的存储地址的。
>
> 如果两个对象相同，就是适用于equals(java.lang.Object) 方法，那么这两个对象的hashCode一定要相同。
>
> 如果对象的equals方法被重写，那么对象的hashCode也尽量重写，并且产生hashCode使用的对象，一定要和equals方法中使用的一致，否则就会违反上面提到的第2点。
>
> 两个对象的hashCode相同，并不一定表示两个对象就相同，也就是不一定适用于equals(java.lang.Object) 方法，只能够说明这两个对象在散列存储结构中，如Hashtable，他们“存放在同一个篮子里”。

### 为什么重载hashCode方法？

> 一般的地方不需要重载hashCode，只有当类需要放在HashTable、HashMap、HashSet等等hash结构的集合时才会重载hashCode，那么为什么要重载hashCode呢？
>
> 如果你重写了equals，比如说是基于对象的内容实现的，而保留hashCode的实现不变，那么很可能某两个对象明明是“相等”，而hashCode却不一样。
>
> 这样，当你用其中的一个作为键保存到hashMap、hasoTable或hashSet中，再以“相等的”找另一个作为键值去查找他们的时候，则根本找不到。

## ArrayList、LinkedList、Vector的区别

> List的三个子类的特点
>
> ArrayList:
>
> 底层数据结构是数组，查询快，增删慢。
> 线程不安全，效率高。
> Vector:
>
> 底层数据结构是数组，查询快，增删慢。
> 线程安全，效率低。
> Vector相对ArrayList查询慢(线程安全的)。
> Vector相对LinkedList增删慢(数组结构)。
> LinkedList
>
> 底层数据结构是链表，查询慢，增删快。
> 线程不安全，效率高。
> Vector和ArrayList的区别
>
> Vector是线程安全的,效率低。
> ArrayList是线程不安全的,效率高。
> 共同点:底层数据结构都是数组实现的,查询快,增删慢。
> ArrayList和LinkedList的区别
>
> ArrayList底层是数组结果,查询和修改快。
> LinkedList底层是链表结构的,增和删比较快,查询和修改比较慢。
> 共同点:都是线程不安全的
>
> List有三个子类使用
>
> 查询多用ArrayList。
> 增删多用LinkedList。
> 如果都多ArrayList。

## Map、Set、List、Queue、Stack的特点与用法

> Map
>
> Map是键值对，键Key是唯一不能重复的，一个键对应一个值，值可以重复。
> TreeMap可以保证顺序。
> HashMap不保证顺序，即为无序的。
> Map中可以将Key和Value单独抽取出来，其中KeySet()方法可以将所有的keys抽取正一个Set。而Values()方法可以将map中所有的values抽取成一个集合。
> Set
>
> 不包含重复元素的集合，set中最多包含一个null元素。
> 只能用Lterator实现单项遍历，Set中没有同步方法。
> List
>
> 有序的可重复集合。
> 可以在任意位置增加删除元素。
> 用Iterator实现单向遍历，也可用ListIterator实现双向遍历。
> Queue
>
> Queue遵从先进先出原则。
> 使用时尽量避免add()和remove()方法,而是使用offer()来添加元素，使用poll()来移除元素，它的优点是可以通过返回值来判断是否成功。
> LinkedList实现了Queue接口。
> Queue通常不允许插入null元素。
> Stack
>
> Stack遵从后进先出原则。
> Stack继承自Vector。
> 它通过五个操作对类Vector进行扩展，允许将向量视为堆栈，它提供了通常的push和pop操作，以及取堆栈顶点的peek()方法、测试堆栈是否为空的empty方法等。
> 用法
>
> 如果涉及堆栈，队列等操作，建议使用List。
> 对于快速插入和删除元素的，建议使用LinkedList。
> 如果需要快速随机访问元素的，建议使用ArrayList。

## Collection

> Collection 是对象集合， Collection 有两个子接口 List 和 Set
>
> List 可以通过下标 (1,2..) 来取得值，值可以重复。 Set 只能通过游标来取值，并且值是不能重复的。
>
> ArrayList ， Vector ， LinkedList 是 List 的实现类
>
> ArrayList 是线程不安全的， Vector 是线程安全的，这两个类底层都是由数组实现的。
> LinkedList 是线程不安全的，底层是由链表实现的。
>
> Map 是键值对集合
>
> HashTable 和 HashMap 是 Map 的实现类。
> HashTable 是线程安全的，不能存储 null 值。
> HashMap 不是线程安全的，可以存储 null 值。
> Stack类：继承自Vector，实现一个后进先出的栈。提供了几个基本方法，push、pop、peak、empty、search等。
>
> Queue接口：提供了几个基本方法，offer、poll、peek等。已知实现类有LinkedList、PriorityQueue等。

## HashMap和HashTable的区别

> Hashtable是基于陈旧的Dictionary类的，HashMap是Java 1.2引进的Map接口的一个实现，它们都是集合中将数据无序存放的。
>
> 1、hashMap去掉了HashTable?的contains方法，但是加上了containsValue()和containsKey()方法
>
> HashTable Synchronize同步的，线程安全，HashMap不允许空键值为空?，效率低。 HashMap 非Synchronize线程同步的，线程不安全，HashMap允许空键值为空?，效率高。 Hashtable是基于陈旧的Dictionary类的，HashMap是Java 1.2引进的Map接口的一个实现，它们都是集合中将数据无序存放的。
>
> Hashtable的方法是同步的，HashMap未经同步，所以在多线程场合要手动同步HashMap这个区别就像Vector和ArrayList一样。
>
> 查看Hashtable的源代码就可以发现，除构造函数外，Hashtable的所有 public 方法声明中都有 synchronized 关键字，而HashMap的源代码中则连 synchronized 的影子都没有，当然，注释除外。
>
> 2、Hashtable不允许 null 值(key 和 value 都不可以)，HashMap允许 null 值(key和value都可以)。
>
> 3、两者的遍历方式大同小异，Hashtable仅仅比HashMap多一个elements方法。

## JDK7与JDK8中HashMap的实现

### **JDK7中的HashMap**

> ==HashMap底层维护一个数组，数组中的每一项都是一个Entry。==
>
> ```java
> transient` `Entry<K,V>[] table;
> ```

> 我们向 HashMap 中所放置的对象实际上是存储在该数组当中。 而Map中的key，value则以Entry的形式存放在数组中。
>
> ```java
> static` `class` `Entry<K,V> ``implements` `Map.Entry<K,V> {
>     ``final` `K key;
>     ``V value;
>     ``Entry<K,V> next;
>     ``int` `hash;
> ```

> **总结一下map.put后的过程**：
>
> ==当向 HashMap 中 put 一对键值时，它会根据 key的 hashCode 值计算出一个位置， 该位置就是此对象准备往数组中存放的位置。==
>
> ==如果该位置没有对象存在，就将此对象直接放进数组当中；如果该位置已经有对象存在了，则顺着此存在的对象的链开始寻找(为了判断是否是否值相同，map不允许<key,value>键值对重复)， 如果此链上有对象的话，再去使用 equals方法进行比较，如果对此链上的每个对象的 equals 方法比较都为 false，则将该对象放到数组当中，然后将数组中该位置以前存在的那个对象链接到此对象的后面.==

### **JDK8中的HashMap**

> JDK8中采用的是==位桶+链表/红黑树==（有关红黑树请查看红黑树）的方式，也是非线程安全的。当某个位桶的链表的长度达到某个阀值的时候，这个链表就将转换成红黑树。
>
> JDK8中，当同一个hash值的节点数不小于8时，将不再以单链表的形式存储了，会被调整成一颗红黑树（上图中null节点没画）。这就是JDK7与JDK8中HashMap实现的最大区别。
>
> 接下来，我们来看下JDK8中HashMap的源码实现。
>
> JDK中Entry的名字变成了Node，原因是和红黑树的实现TreeNode相关联。
>
> transient Node<K,V>[] table;
> 当冲突节点数不小于8-1时，转换成红黑树。
>
> static final int TREEIFY_THRESHOLD = 8;

## HashMap和ConcurrentHashMap的区别，HashMap的底层源码

> ==为了线程安全从ConcurrentHashMap代码中可以看出，它引入了一个“分段锁”的概念，具体可以理解为把一个大的Map拆分成N个小的HashTable，根据key.hashCode()来决定把key放到哪个HashTable中。==
>
> ==**Hashmap本质是数组加链表。根据key取得hash值，然后计算出数组下标，如果多个key对应到同一个下标，就用链表串起来，新插入的在前面**==。
>
> ConcurrentHashMap：在hashMap的基础上，ConcurrentHashMap将数据分为多个segment，默认16个（concurrency level），然后每次操作对一个segment加锁，避免多线程锁的几率，提高并发效率。

### 总结

> ==JDK6,7中的ConcurrentHashmap主要使用Segment来实现减小锁粒度，把HashMap分割成若干个Segment，在put的时候需要锁住Segment，get时候不加锁，使用volatile来保证可见性，当要统计全局时（比如size），首先会尝试多次计算modcount来确定，这几次尝试中，是否有其他线程进行了修改操作，如果没有，则直接返回size。如果有，则需要依次锁住所有的Segment来计算。==
>
> jdk7中ConcurrentHashmap中，当长度过长碰撞会很频繁，链表的增改删查操作都会消耗很长的时间，影响性能。
>
> jdk8 中完全重写了concurrentHashmap,代码量从原来的1000多行变成了 6000多 行，实现上也和原来的分段式存储有很大的区别。
>
> JDK8中采用的是位桶+链表/红黑树（有关红黑树请查看红黑树）的方式，也是非线程安全的。当某个位桶的链表的长度达到某个阀值[16+8]的时候，这个链表就将转换成红黑树。
>
> JDK8中，当同一个hash值的节点数不小于8时，将不再以单链表的形式存储了，会被调整成一颗红黑树（上图中null节点没画）。这就是JDK7与JDK8中HashMap实现的最大区别。
>
> 主要设计上的变化有以下几点
>
> 1.==jdk8不采用segment而采用node，锁住node来实现减小锁粒度。 2.设计了MOVED状态 当resize的中过程中 线程2还在put数据，线程2会帮助resize。 3.使用3个CAS操作来确保node的一些操作的原子性，这种方式代替了锁。 4.sizeCtl的不同值来代表不同含义，起到了控制的作用。==
>
> 至于为什么JDK8中使用synchronized而不是ReentrantLock，我猜是因为JDK8中对synchronized有了足够的优化吧

### ConcurrentHashMap能完全替代HashTable吗

> hashTable虽然性能上不如ConcurrentHashMap，但并不能完全被取代，两者的迭代器的一致性不同的，hash table的迭代器是强一致性的，而concurrenthashmap是弱一致的。
>
> ConcurrentHashMap的get，clear，iterator 都是弱一致性的。 Doug Lea 也将这个判断留给用户自己决定是否使用ConcurrentHashMap。
>
> ConcurrentHashMap与HashTable都可以用于多线程的环境，但是当Hashtable的大小增加到一定的时候，性能会急剧下降，因为迭代时需要被锁定很长的时间。因为ConcurrentHashMap引入了分割(segmentation)，不论它变得多么大，仅仅需要锁定map的某个部分，而其它的线程不需要等到迭代完成才能访问map。简而言之，在迭代的过程中，==ConcurrentHashMap仅仅锁定map的某个部分，而Hashtable则会锁定整个map。==

### **那么什么是强一致性和弱一致性呢？**

> get方法是弱一致的，是什么含义？可能你期望往ConcurrentHashMap底层数据结构中加入一个元素后，立马能对get可见，但ConcurrentHashMap并不能如你所愿。换句话说，put操作将一个元素加入到底层数据结构后，get可能在某段时间内还看不到这个元素，若不考虑内存模型，单从代码逻辑上来看，却是应该可以看得到的。
>
> 下面将结合代码和java内存模型相关内容来分析下put/get方法。put方法我们只需关注Segment#put，get方法只需关注Segment#get，在继续之前，先要说明一下Segment里有两个volatile变量：count和table；HashEntry里有一个volatile变量：value。

`ConcurrentHashMap的弱一致性主要是为了提升效率，是一致性与效率之间的一种权衡。要成为强一致性，就得到处使用锁，甚至是全局锁，这就与Hashtable和同步的HashMap一样了。`

## 为什么HashMap是线程不安全的

> HashMap 在并发执行 put 操作时会引起死循环，导致 CPU 利用率接近100%。因为多线程会导致 HashMap 的 Node 链表形成环形数据结构，一旦形成环形数据结构，Node 的 next 节点永远不为空，就会在获取 Node 时产生死循环。

> ConcurrentHashMap
>
> ==ConcurrentHashMap 于 Java 7 的，和8有区别,在8中 CHM 摒弃了 Segment（锁段）的概念，而是启用了一种全新的方式实现,利用 CAS 算法，有时间会重新总结一下。==
>
> SynchronizedMap
>
> synchronizedMap() 方法后会返回一个 SynchronizedMap 类的对象，而在 SynchronizedMap 类中使用了 synchronized 同步关键字来保证对 Map 的操作是线程安全的。

## TreeMap、HashMap、LindedHashMap的区别

> LinkedHashMap可以保证HashMap集合有序，存入的顺序和取出的顺序一致。
>
> TreeMap实现SortMap接口，能够把它保存的记录根据键排序,默认是按键值的升序排序，也可以指定排序的比较器，当用Iterator遍历TreeMap时，得到的记录是排过序的。
>
> HashMap不保证顺序，即为无序的，具有很快的访问速度。 HashMap最多只允许一条记录的键为Null;允许多条记录的值为 Null。 HashMap不支持线程的同步。
>
> 我们在开发的过程中使用HashMap比较多，在Map中在Map 中插入、删除和定位元素，HashMap 是最好的选择。
>
> 但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。
>
> 如果需要输出的顺序和输入的相同,那么用LinkedHashMap 可以实现,它还可以按读取顺序来排列。

## Collection包结构，与Collections的区别

> **Collection** 是集合类的上级接口，子接口主要有Set、List 、Map。
>
> **Collecions** 是针对集合类的一个帮助类， 提供了操作集合的工具方法，一系列静态方法实现对各**种集合的搜索、排序线性、线程安全化**等操作。

## Override和Overload的含义去区别

> Override（重写）
>
> 方法名、参数、返回值相同。
> 子类方法不能缩小父类方法的访问权限。
> 子类方法不能抛出比父类方法更多的异常(但子类方法可以不抛出异常)。
> 存在于父类和子类之间。
> 方法被定义为final不能被重写。
>
> Overload（重载）
>
> 参数类型、个数、顺序至少有一个不相同。
> 不能重载只有返回值不同的方法名。
> 存在于父类和子类、同类中。