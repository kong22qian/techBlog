[TOC]

# Java mult thread base info

## 基本概念

### 多线程有什么用？

单核：比如一个线程使用的时候CPU计算，IO空闲，IO操作时候CPU空闲，当CPU空闲时，执行另一个线程IO操作对方迟迟没有返回，这时CPU就空闲，而用户阻塞。为了防止阻塞，我们开启多线程。这样提高CPU利用率。
多核，提高CPU利用率，多核分别执行多个线程。比如一个复杂任务，让多核并行，提高效率。

**问题：**内存泄漏、上下文切换、死锁还有受限于硬件和软件的资源闲置问题。

### 线程、进程、协程的区别

进程：是系统分配资源都基本单元，有独立的内存空间 比如一个迅雷应用程序，既不共享堆，亦不共享栈。一个进程中至少有一个线程。
线程：线程是操作系统调度的最小单元线程拥有自己独立的栈和共享的堆，共享堆，不共享栈比如为在迅雷程序中开启多个下载任务下载就是多线程。如果只有单个线程，那么我们就只能下载一个任务，然后等待他下载完了再下载，而开启多个线程就可以同时下载多个任务。

协程和线程一样共享堆，不共享栈，协程由程序员在协程的代码里显示调度。而线程是CPU进行调度。

协程和线程的区别是：协程避免了无意义的调度，由此可以提高性能，但也因此，**程序员必须自己承担调度的责任，**同时，协程也失去了标准线程使用多CPU的能力。
在线程里面可以开启协程，让程序在特定的时间内运行。也就是说一个线程执行不同的协程。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190610063610738.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nYmluMDA3,size_16,color_FFFFFF,t_70)

在单核上，CPU决定线程A和B的执行权。
在双核上，并行，一个核执行线程A一个核执行线程B。
而协程则是由程序员控制执行权，比如我可以让一个协程A同时运行在两个核上，执行完之后再使用双核同时运行协程B。

### 什么是多线程上下文切换？

每个线程都是竞争CPU执行权的，比如CPU只有单核
CPU在执行一个已经运行的线程时候切换到另一个在等待获取CPU执行权的线程。，当切换执行权的时候就叫做多线程上下文切换。

上下文切换通常是计算密集型的。上下文切换对系统来说意味着消耗大量的 CPU 时间，事实上，可能是操作系统中时间消耗最大的操作。

Linux 相比与其他操作系统（包括其他类 Unix 系统）有很多的优点，其中有一项就是，其上下文切换和模式切换的时间消耗非常少。

#### 如何减少上下文切换，提高操作系统效率

1. 无**锁**并发：多线程**竞争锁**时，会引起上下文切换，所以多线程处理数据时，可以用一些办法来避免使用锁。
2. 使用**最少**线程：**避免创建不需要的线程**，比如任务很少，但是创建了很多线程来处理，这样会造成大量线程都处于等待状态
3. **协程**：在单线程实现多任务的调度，并**在单线程里维持多个任务间的切换**

### 什么是线程安全

如果你的代码在多线程下执行和在单线程下执行永远都能获得一样的结果，那么你的代码就是线程安全的。
这里可以举例说明不安全的一个实例，就是多线程处理共享数据比如仅剩一张票时，多线程操作会造成重复卖票情况。也就是说在多线程环境中，能够正确地处理多个线程之间的共享变量，使程序功能正确完成。

### 一个线程终止，程序会终止吗

子线程被创建后就跟母线程没什么关系了。每个线程都要去处理自己的事情，包括异常。当所有的线程都结束的时候才说明程序运行Over了。
如果线程的异常没有被捕获的话，这个线程就停止执行了。另外重要的一点是：如果这个线程持有某个某个对象的监视器，那么这个对象监视器会被立即释放
若有线程调用system.exit（）则整个程序终止

### 一个线程如果出现了运行时异常会怎么样

Exception分为RuntimeException和非运行时异常。
非运行时异常必须处理，比如thread中sleep()时，必须处理InterruptedException异常，才能通过编译。
运行时异常可以抛出，然后线程继续运行，如果不抛出或者不处理，则该线程则会停止。

### 你对线程优先级的理解是什么？

每一个线程都是有优先级的，一般来说，高优先级的线程在运行时会具有优先权，但这依赖于线程调度的实现，这个实现是和操作系统相关的(OS dependent)。我们可以定义线程的优先级，但是这并不能保证高优先级的线程会在低优先级的线程前执行。线程优先级是一个int变量(从1-10)，1代表最低优先级，10代表最高优先级。
一般无需设置线程优先级。

## java多线程

### Java中用的是什么线程调度算法

有两种调度模型：==分时调度模型和抢占式调度模型。==
分时调度模型是指让所有的线程轮流获得cpu的使用权,并且平均分配每个线程占用的CPU的时间片这个也比较好理解。

==java虚拟机采用抢占式调度模型==，是指优先让可运行池中优先级高的线程占用CPU，如果可运行池中的线程优先级相同，那么就随机选择一个线程，使其占用CPU。处于运行状态的线程会一直运行，一个线程用完CPU之后，操作系统会根据线程优先级，饥饿程度等数据计算出优先级并执行下一个线程。

### 创建线程的方式

三种：

1. 继承Thread类
2. 实现Runnable接口
3. 通过Callable和Future创建线程JDK5升级
4. 创建线程池（当前最普遍）

逐渐升级：

5. Java不支持多继承.因此扩展[Thread类](https://so.csdn.net/so/search?q=Thread类&spm=1001.2101.3001.7020)就不能继承其他类.而实现Runnable接口的类可以继承其他类
6. 好处可以有返回值，和获取线程状态

### Runnable和Callable的区别什么是Future

Runnable接口中的run()方法的返回值是void，它做的事情只是纯粹地去执行run()方法中的代码而已；
Callable接口中的call()方法是有返回值的，是一个泛型，和Future、FutureTask配合可以用来拿到异步执行任务的返回值。。Callable用于产生结果，Future用于获取结果。

而Callable+Future/FutureTask却可以方便获取多线程运行的结果和线程运行状态等信息

### Thread类中的start()和run()方法有什么区别?

start()方法被用来启动新创建的线程，而且start()内部调用了run()方法，所以他会开启新线程并执行run方法
当你调用run()方法的时候，只会是在原来的线程中调用方法，没有新的线程启动。所以还是单线程的。

### JAVA中++操作符是线程安全的吗？

不是，他涉及到多个指令，如变量值的**读取，增加，存储**回内存，这个过程可能会出现多个线程交叉执行。

### 什么是线程dump（介绍线程状态）

Java Thread dump记录了线程在jvm中的执行信息，可以看成是线程活动的日志。Java线程转储文件有助于分析应用程序和死锁情况中的瓶颈。使用一些工具比如jstack。可以查看。

> 新建状态（New）
> 就绪状态（Runnable）
> 等待获得CPU的使用权。
> 运行状态（Running）
> 阻塞状态（Blocked）
> 阻塞状态是指线程因为某些原因放弃CPU，暂时停止运行。

阻塞状态可分为以下3种：

- 位于对象等待池中的阻塞状态（Blocked in object’s wait pool）：当线程处于运行状态时，如果执行了某个对象的wait()方法，Java虚拟机就会把线程放到这个对象的等待池中，这涉及到“线程通信”的内容。使用Lock配合Condition的 await方法 signal唤醒
- 位于对象锁池中的阻塞状态（Blocked in object’s lock pool）：当一个线程想要获取其他的锁时，锁被其他线程占用，则该线程阻塞，如果该对象的同步锁已经被其他线程占用，Java虚拟机就会把这个线程放到这个对象的锁池中，这涉及到“线程同步”的内容。
- 其他阻塞状态（Otherwise Blocked）当前线程执行了sleep()方法

### 如何实现多线程通信和协作（停止一个正在运行的线程）

1. 使用共享变量方式
2. 使用interrupt方法终止

### sleep() 方法和 wait() 方法区别?

释放锁：sleep 方法没有释放锁，而 wait 方法释放了锁 。
作用：Wait 通常被用于线程间通信，sleep 用于暂停执行。
苏醒：wait() 方法被调用后，线程不会自动苏醒，需要别的线程调用同一个对象上的 notify() 或者 notifyAll() 方法。sleep() 方法执行完成后，线程会自动苏醒。或者可以使用wait(long timeout)超时后线程会自动苏醒。

### 守护线程

- 任何线程都可以设置为守护线程，使用Thread.setDaemon()方法。
- 也称后台线程，守护线程类似于一个后台服务，比如GC线程。守护线程和前台线程一起运行，但是当所有前台线程执行完了，守护线程也就自动结束，。
- JVM会等待所有非守护线程执行完之后关闭，但是不会等待守护线程。

### 如何在两个线程之间共享数据，线程间通信

通过在线程之间共享对象/变量，然后通过wait/notify/notifyAll、await/signal/signalAll等待唤醒机制。
rescource 就是共享对象。

``` java
public class SynchronizedSample {
    public static void main(String[] args) {
        Resource resource = new Resource();  // 实例化资源

        Producer producer = new Producer(resource);  // 实例化生产者和消费者类，它们取得同一个资源
        Comsumer comsumer = new Comsumer(resource);

        Thread threadProducer = new Thread(producer); // 创建1个生产者线程
        Thread threadComsumer = new Thread(comsumer); // 创建1个消费者线程

        // 分别开启线程
        threadProducer.start();
        threadComsumer.start();
    }
}
```

### ThreadLocal有什么用

而ThreadLocal采用了“以空间换时间”的方式。他为每一个线程都提供了一份变量副本，因此可以同时访问而互不影响，所以肯定线程安全。
与局部变量相比，他可以更好的解偶，因为外部设置定义这个对象。

### Thread.Sleep(0)的作用

因此，==Thread.Sleep(0)的作用，就是“触发操作系统立刻重新进行一次CPU竞争”==。给其他线程获得CPU执行一次的机会。
竞争的结果也许是当前线程仍然获得CPU控制权，也许会换成别的线程获得CPU控制权。这也是我们在大循环里面经常会写一句Thread.Sleep(0) ，因为这样就给了其他线程比如Paint线程获得CPU控制权的权力。

### 不可变对象对多线程有何帮助

不可变对象即对象一旦被创建他的状态就不能改变。
不可变对象的类为不可变类。如String等
不可变对象永远线程安全。Erlang中定义不可变变量也是为了此种目的。

### 什么是可重入锁

线程可以进入任何一个他已经拥有的锁的同步代码块。
synchronized. ReetrantLock都是可重入的锁。

## JMM内存模型与（多线程）三大特性

### JMM内存模型

抽象概念，并不真实存在。
JMM三大特性（也即多线程三大特性）：可见性，原子性，有序性。
JMM内存模型要求保证这三个特性。

#### 可见性

JMM规定所有共享变量都存储在主内存。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007121901556.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nYmluMDA3,size_16,color_FFFFFF,t_70)

开始是共享变量25在主内存，每个线程都拿到共享变量都拷贝25。当一个线程修改主内存值37时（不能直接修改主内存中的值），然后拷贝回主内存，这时其他线程是不知道主内存值已经改变了。这个就是可见性。如果其他线程能够看到该共享变量值已经改变，则为可见。volatile和synchronized保证可见性。

线程间不能够直接访问其他线程，必须通过主内存共享变量来完成。
是指当一个线程修改了某一个共享变量的值，其他线程是否能够立即知道这个修改。(volitle)就是做这个保证的。

#### 原子性

**是指一个操作是不可中断的**。即使是多个线程一起执行的时候，一个操作一旦开始，就不会被其他线程干扰。当一个线程执行操作的时候，不要被其他线程加塞。比如三个线程操作n++，当一个线程写1时候，这时第二个线程和第三个线程都写为1，也就是写丢了两次。这就没有保证原子性。（写丢失）

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007161738658.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nYmluMDA3,size_16,color_FFFFFF,t_70)

解决：

1. 使用synchronized
2. 使用AtomicInteger，底层使用CAS。

#### 有序性

处理器在处理并发时，会进行指令重排优化导致结果无法预测。程序的执行可能**会出现乱序**。给人的直观感觉就是：写在前面的代码，会在后面执行。重排后的指令与原指令的顺序未必一致。

### Volatile关键字的作用

JVM提供的轻量级同步机制。
volatile 关键字的主要作用就是保证变量的可见性然后还有一个作用是和有序性，不保证原子性。

使用volatile关键字修饰的共享变量，保证了其在多线程之间的可见性，即多个线程每次读取到volatile变量，一定是最新的数据，就是一个线程修改共享变量volatile值之后及时通知其他线程。
不保证原子性
现实中，为了获取更好的性能JVM可能会对指令进行重排序，多线程下就会产生一些问题。使用volatile则会对禁止语义重排序避免问题，但是这也一定程度上降低了代码执行效率。

#### volatile与synchronized区别

volatile关键字能保证数据的可见性，但不能保证数据的原子性。synchronized关键字两者都能保证，也就是保证资源的同步。
volatile关键字只能用于变量而synchronized关键字可以修饰方法以及代码块。

### synchronized关键字

保证**可见性**和**原子性**。
synchronized关键字解决的是多个线程之间访问资源的同步性，可见性。
synchronized关键字可以保证被它修饰的方法或者代码块在任意时刻只能有一个线程执行，原子性。

## 多线程同步和锁

### 什么是多线程的同步和异步

一个进程启动的多个不相干线程，它们相互之间关系为**异步**。
**同步**必须执行到底之后才能执行其他操作，而异步可以任意操作，是多个线程同时访问同一资源，等待资源访问结束

#### 同步的好处与弊端

好处：解决了线程的安全问题。
弊端：每次都有判断锁，降低了效率。

但是在安全与效率之间，首先考虑的是安全。

### 谈谈 synchronized和ReentrantLock 的区别

==synchronized==

- 关键字：synchronized 他主要是在JVM层面实现而没有直接暴露给程序员。底层使用monitor来完成
- synchronized关键字与wait()和notify()/notifyAll()方法相结合可以实现等待/通知机制，不需要手动释放
- 而synchronized关键字就相当于整个Lock对象中只有一个Condition实例，所有的线程都注册在它一个身上。如果执行notifyAll()方法的话就会通知所有处于等待状态的线程这样会造成很大的效率问题

==ReentrantLock==

- 类：而ReentrantLock是API层面是一个类，底层调用Unsafe的park方法加锁。比如说我们可以显示的调用Lock和unlock来实现。加锁和解锁。类就更灵活，可以继承，有方法和变量。
- 使用condition的await和signalALl来通知。需要手动释放。
- 用ReentrantLock类结合Condition实例可以实现选择性通知 ，这个功能非常重要。可以实现多路通知功能也就是在一个Lock对象中可以创建多个Condition条件实例。比如生产者消费者，可以指定解锁消费者线程。
  

### 为什么wait,nofity和nofityAll这些方法不放在Thread类当中

JAVA提供的锁是对象级的而不是线程级的，每个对象都有锁，通过线程获得。如果线程需要等待某些锁那么调用对象中的wait()方法就有意义了。如果wait()方法定义在Thread类中，线程正在等待的是哪个锁就不明显了。简单的说，由于wait，notify和notifyAll都是锁级别的操作，所以把他们定义在Object类中因为锁属于对象。
Thread类属于类级别的，不是对象级别

### 生产者消费者模型的作用是什么

一个程序是面包工场，开启多个线程来生产面包，多个线程来消费面包。每生产一个就wait然后notify消费者去消费。往复循环。

通过平衡生产者的生产能力和消费者的消费能力来提升整个系统的运行效率
解耦，解耦意味着生产者和消费者之间的联系少，联系越少越可以独自发展而不需要收到相互的制约

### 为何需要等待唤醒机制？

因为拿生产者消费者模型来说，如果只使用加锁同步synchronized，则我们生产者在生产的时候就不会释放锁一直生产，而消费者不能消费。使用等待唤醒，wait（）方法来让线程等待，比如生产完一个就阻塞，然后暂时释放锁给消费者。

### 如何判断一个线程是否持有某个对象的锁

线程有一个名为holdsLock（Object obj）的静态方法，它根据线程是否对传递的对象持有锁来返回true或false。

### 同步块还是同步方法效果更好

同步方法直接在方法上加synchronized实现加锁，范围更大，
同步代码块则在方法内部需要同步的语句加锁，同步代码块范围要小点
一般同步的范围越大，性能就越差，一般需要加锁进行同步的时候，肯定是范围越小越好，这样性能更好

### 什么是同步容器，什么是并发容器

同步容器：可以简单地理解为通过synchronized来实现同步的容器，如果有多个线程调用同步容器的方法，竞争锁。比如Vector，Hashtable，
可以通过查看Vector，Hashtable等这些同步容器的实现代码，在需要同步的方法上加上关键字synchronized。

并发容器使用了与同步容器完全不同的加锁策略来提供更高的并发性和伸缩性，例如在ConcurrentHashMap中采用了一种粒度更细的加锁机制，可以称为分段锁。

允许任意数量的读线程并发地访问map，并且执行读操作的线程和写操作的线程也可以并发的访问map，同时允许一定数量的写操作线程并发地修改map，所以它可以在并发环境下实现更高的吞吐量。

### 死锁

#### 什么是线程死锁?如何避免死锁?

同时持有资源又同时申请资源。是指两个或两个以上的进程（或线程）在执行过程中，因争夺对方线程的资源而造成的一种互相等待的现象，若无外力作用，它们都将无法推进下去。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190812191445178.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nYmluMDA3,size_16,color_FFFFFF,t_70)

死锁四个必备条件：

- 互请不行
  互斥条件：线程独占资源，资源
- 能供一个线程使用。
- 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
- 不剥夺条件: 线程已获得的资源不能被其他线程强行剥夺，使用完毕后释放。
- 循环等待条件: 若干进程之间形成一种头尾相接的循环等待资源关系。

#### 如何避免线程死锁?

破坏任意一个条件都可以防止死锁都产生。通常我们破坏循环等待。
破坏循环等待条件。
靠按序申请资源来预防。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813185400963.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nYmluMDA3,size_16,color_FFFFFF,t_70)

按某一顺序申请资源锁，释放资源则反序释放。破坏循环等待条件

#### Java实现一个死锁

```java
package CommonClass;

public class DeadLock {
    public final static Object lock1 = new Object();
    public final static Object lock2 = new Object();

    public static void main(String[] args) {
        DeadLock d = new DeadLock();
        d.setDeadlock();
    }

    // 建立死锁
    private void setDeadlock() {
        Thread A = new Thread(() -> {
            synchronized (DeadLock.lock1) {//拿住锁对象1
                System.out.println("A keeps lock1");
                sleep();  // 留出时间让线程B去锁住2
                synchronized (DeadLock.lock2) {//已有lock1想去申请lock2
                    System.out.println("A got lock2");
                    sleep();
                }

            }
        });
        Thread B = new Thread(() -> {
            synchronized (DeadLock.lock2) {//拿住锁对象2
                System.out.println("B keeps lock2");
                sleep();  // 留出时间让线程A去锁1
                synchronized (DeadLock.lock1) {//已有lock2 想去申请lock1
                    System.out.println("B got lock1");
                    sleep();
                }

            }
        });
        A.start();
        B.start();
    }

    // 让线程等待一段时间，以便使A,B都能先锁住一个资源
    private void sleep() {
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

}
```

#### 产生死锁之后如何查看

用JPS定位进程号，
用jstack查看线程死锁问题。

#### 活锁

任务或者线程没有被阻塞，由于某些条件没有满足，一直重复尝试但是失败。活锁会不断尝试然后失败，而死锁就一直等待。活锁可以自行解开而死锁不可用。

## [线程池](https://so.csdn.net/so/search?q=线程池&spm=1001.2101.3001.7020)

学到Spring就不再new对象了，利用依赖反转，缺什么拿什么。
在多线程也是这个思想，我们不用一个一个new，线程池给我new好，我直接用。

==思想==

也即底层实现，是**队列**；**底层源码**实现类为**ThreadPoolExecutor**这个类中一个参数就是阻塞队列。

将待处理任务放入阻塞队列，然后在线程创建后执行队列中任务。如果任务超出线程数量，则等线程执行完之后再从队列中取出，这样的思想。

### 为何要使用线程池（Executor框架）（优势）	

1. 管理线程：线程池可以**统一分配线程**，管理和监控。
2. 控制最大并发数：所以创建一个线程池是个更好的解决方案，因为**可以更好控制线程的数量**，避免过多线程竞争。而不是无限的创建，可能导致应用内存溢出。
3. 线程复用：**可以回收再利用这些线程**避免频繁的创建和销毁。提高资源使用率。

### 创建线程池

创建ThreadPoolExecutor 的方式，有三种类型的线程池，

- 创建只含单独线程的线程池ExecutorService service1 = Executors.newSingleThreadExecutor();
- 固定数量的线程池ExecutorService service3 = Executors.newFixedThreadPool(10);用阻塞队列放任务，然后固定数量线程池执行阻塞队列中的。适用于：长期执行任务，性能好。
- 创建可调整数量的线程池。ExecutorService service2 = Executors.newCacheThreadPool(); 适用于：执行短期小程序。方便扩容。

### 线程池底层实现，七大参数（重点）

举例可以为银行窗口和候客区。

> corePoolSize: 线程中核心线程数
> maximumPoolSize：最大线程数。最大上限，当常用线程池线程满了，阻塞队列中任务也满了，还有任务过来，则这时可以扩容，临时加班到最大上限。
> keepalive： 多余线程存活时间，也就是临时线程存活的时间，比如临时加班一个小时后，执行完任务，就下班。从max退回到核心线程数
> unit 时间单位
> workQueue：阻塞队列，待执行存放任务区域。
> threadFatory：线程工厂，一般默认
> 拒绝策略：窗口最大数量满了和阻塞队列也满了，那么我们就拒绝一些任务。

==步骤：==

- 来任务，开线程干活
- 大于核心线程数，放入阻塞队列
- 队列满了，开线程到最大线程数
- 最大线程数也满了，拒绝策略
- 设置临时线程存活时间，超过时间收缩到核心线程大小。

### 如何合理配置线程池（配置线程数）

看任务是CPU密集型还是IO密集型。

1. 要知道自己到服务器是几核的。了解自己的情况。
2. 如果是CPU密集型，大量运算，一般就是CPU核数+1 。尽量减少切换。
3. 若IO密集型。因为CPU不是一直在执行任务，则可以尽可能多配置线程数，一般为CPU*2

### Java线程池中submit() 和 execute()方法有什么区别？

两个方法都可以向线程池提交任务，
execute()方法的返回类型是void，类似于run方法，它定义在Executor接口中。
而submit()方法可以返回持有计算结果的Future对象，它定义在ExecutorService接口中，它扩展了Executor接口

## CAS

### 何谓悲观锁与乐观锁

悲观锁：总是假设最坏的情况，每次拿到数据时都会上锁，共享资源每次只给一个线程使用，其它线程阻塞，用完后再把资源转让给其它线程。Java中synchronized和ReentrantLock等独占锁就是悲观锁思想的实现。

乐观锁：总是假设最好的情况，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下是否有人更新过这个数据
乐观锁是一种思想。CAS是这种思想的一种实现方式。CAS（compare and swap）比较和替换，如果是预期的值，则替换，如果不是，则什么都不做返回false。是非阻塞算法的一种常见实现。当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，但失败的线程不会像悲观锁那样阻塞，而是被告知这次竞争中失败，并可以再次尝试。

在Java中java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式CAS实现的。

适用场景：
像乐观锁适用于写比较少的情况下（多读情况）冲突会比较少。这样可以省去了锁的开销，提高吞吐量。
一般如果冲突产生比较多，多写的场景下用悲观锁就比较合适。

### 懂不懂AtomicInteger

保证原子性，替代n++
底层使用CAS

### 什么是CAS（compare and set)

简单来讲就是比较，交换。CAS是这种思想的一种实现方式。CAS（compare and swap）比较和替换，如果是预期的值，则替换，如果不是，则什么都不做返回false。类似于github版本号，当我比较的时候如果一样那么我就可以提交修改，如果不一样有冲突，那么我就要重新获得主物理内存真实值。
比如为创建一个主物理内存AtomicInteger是5，第一个线程得到副本拷贝为5，期望值是5，则可以更新为6。
这时主物理内存为6，而第二个线程再进行比较，期望值是5，而内存值为6，就返回false，线程需要重新获取内存值。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019100716462275.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nYmluMDA3,size_16,color_FFFFFF,t_70)

**CAS底层原理**

1. 自旋锁：使用无锁机制，性能很高。
2. unsafe类：其内部方法可以像C指针一样操作内存，所有方法都用native修饰，可以直接操作系统底层资源。

CAS 利用 CPU 指令保证了操作的原子性，以达到锁的效果

**CAS缺点**

> 不加锁保证一致性，只能保证一个共享变量的原子性操作。

**引出ABA问题：**

> 假设如下事件序列：
>
> 线程 1 从内存位置V中取出A。
> 线程 2 从位置V中取出A。
> 线程 2 进行了一些操作，将B写入位置V。
> 线程 2 将A再次写入位置V。
> 线程 1 进行CAS操作，发现位置V中仍然是A，操作成功。
> 尽管线程 1 的CAS操作成功，但不代表这个过程没有问题——对于线程 1 ，线程 2 的修改已经丢失。
>
> 优化方向：CAS不能只比对“值”，还必须确保的是原来的数据，才能修改成功。
>
> 常见实践：版本号 getStamp 进行的比对，一个数据一个版本，版本变化，即使值相同，也不应该修改成功。

## 集合类不安全问题

### ArrayList
vector 线程安全，数据一致性可以保证但是并发性下降，使用了synchronized。
Arraylist，并发性上升。多线程并发add向list中写入数据操作，当一个线程写的时候，另一个线程竞争写，就会出现并发报错问题。
解决并发问题：

包一个Collections.synchronizedList(new ArrayList<>());
使用CopyOnwriteArrayList()。读写分离，也就是使用读写锁，写都时候该类使用了lock锁，操作都放在lock里面。读的时候并发去读取数据。提高效率。

### Set
HashSet底层数据结构就是HashMap，HashSet放一个数据而HashMap放两个数据，如何解释？就是调用HashMap的put方法，而key是放入的值，value设置为一个常量不管put多少个都是恒定值。

Collections.synchronizedSet(new HashSet<>());`
CopyOnwriteArraySet() 他的底层数据结构使用CopyOnwriteArrayList()

### Map
Hashtable线程安全，但是对整个表都加了锁，范围太大。性能下降
HashMap线程不安全。
解决：
使用ConcurrentHashMap
Java8中的ConcurrentHashMap的底层结构：数组+链表+红黑树。
每次在put()的时候我们只给链表头部元素加锁。所以我们可以只对数组中存的那个元素加锁即可。

## 锁

**公平锁和非公平锁**
公平锁，指多个线程按照申请顺序来获取锁，先来后到但是效率低。
非公平锁：多个线程不按照顺序来获取锁，有都可能先获得锁。性能更好，吞吐量大。ReentrantLock默认非公平锁。Synchronized也是非公平锁。

**可重入锁（递归锁）**
外层方法获取一个锁后，内层方法自动获取该锁。
ReentrantLock和。Synchronized都是可重入锁
作用：避免死锁。

**自旋锁==**
unsafe类+CAS思想 就是自旋锁
获取锁的线程获取不到的时候不会立即阻塞，而是采用循环的方式去尝试获取锁。

既然用锁或 synchronized 关键字可以实现原子操作，那么为什么还要用 CAS 呢，因为加锁或使用 synchronized 关键字带来的性能损耗较大，而用 CAS 可以实现乐观锁使用的是无锁机制，它实际上是直接利用了 CPU 层面的指令，所以性能很高。

上面也说了，CAS 是实现自旋锁的基础，CAS 利用 CPU 指令保证了操作的原子性，以达到锁的效果，至于自旋呢，一般是用一个无限循环实现。这样一来，一个无限循环中，执行一个 CAS 操作，当操作成功，返回 true 时，循环结束；当返回 false 时，接着执行循环，继续尝试 CAS 操作，直到返回 true。
好处是不用阻塞，坏处是cpu消耗较大。

**独占锁（写锁），共享锁（读锁）**
独占锁：该锁一次只有一个线程持有，ReentrantLock和Synchronized都是独占锁。
共享锁：指该所可被多个线程持有。
ReentrantReadWriteLock 写锁时独占，读锁时共享。
因为写资源时必须一个线程写，其他线程不能写，而读的时候可以并发的去读。不会出现问题。

## CyclicBarrier](https://so.csdn.net/so/search?q=CyclicBarrier&spm=1001.2101.3001.7020)和CountDownLatch，和Semaphore

**CountDownLatch倒计数**
CountDownLatch， 一个同步辅助类计数，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。
比如五个线程去执行任务，每个线程都执行，然后CountDownLatch-1，让主线程等待，等到五个线程都执行完之后，也就是countDownLatch为0 的时候，主线程才开始执行。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007215436789.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nYmluMDA3,size_16,color_FFFFFF,t_70)

**CyclicBarrier 循环屏障**

CyclicBarrier，和CountDownLatch相反的，CountDown是做减法，到0。而CyclicBarrier是做加法。
先到的被阻塞。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2019100722093429.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nYmluMDA3,size_16,color_FFFFFF,t_70)

**Semaphore信号量计数**

比如有三个停车位，六个车进来停。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20191007222708658.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2Nob25nYmluMDA3,size_16,color_FFFFFF,t_70)

控制三个车停，这时没有车位，其他线程阻塞。然后停三秒钟后有车离开，其他线程进来补位。
相当于一个资源池：一个固定长度的资源池，当池为空时，请求资源会失败。或者对容器施加边界。

## 什么是AQS

简单说一下AQS，AQS全称为AbstractQueuedSychronizer，翻译过来应该是抽象队列同步器。

如果说java.util.concurrent的基础是CAS的话，那么AQS就是整个Java并发包的核心了，ReentrantLock、CountDownLatch、Semaphore等等都用到了它。AQS实际上以双向队列的形式连接所有的Entry，比方说ReentrantLock，所有等待的线程都被放在一个Entry中并连成双向队列，前面一个线程使用ReentrantLock好了，则双向队列实际上的第一个Entry开始运行。

AQS定义了对双向队列所有的操作，而只开放了tryLock和tryRelease方法给开发者使用，开发者可以根据自己的实现重写tryLock和tryRelease方法，以实现自己的并发功能。

## 阻塞队列

### 什么是阻塞队列BlockingQueue？如何使用阻塞队列来实现生产者和消费者模型？

阻塞队列：在队列为空时，获取元素的线程会等待队列中放入元素。当队列满时，放入元素的线程会等待队列可用。
他是MQ消息队列核心底层原理。

生产者消费者，阻塞队列就是仓库存储的蛋糕。
BlockingQueue接口，当生产者试图向BlockingQueue放入元素时，如果队列满，则生产者线程被阻塞。当消费者线程试图取出元素时，如果BlockingQueue为空，则消费者线程被阻塞。利用这个特性多个线程交替向队列中存放和取出元素，实现控制线程通信。

同步阻塞队列，即单个元素阻塞队列。产生一个消费一个。

阻塞队列常用场景就是socket客户端数据操作，线程不断将数据放入队列，然后有线程不断从队列中取出数据。