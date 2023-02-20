[TOC]

# NETTY BASE INFO

## Netty概述

### Netty简介

> 首先netty是什么，官网是这样定义的。
> Netty is an asynchronous event-driven network application framework for rapid development of maintainable high performance protocol servers & clients.
> Netty是一个异步事件驱动的网络应用程序框架用于快速开发可维护的高性能协议服务器和客户端。
>
> Netty是目前所有NIO框架中性能最好的框架，Java本身提供的有NIO工具，但是在实际操作过程中，复杂繁琐，且对编程人员要求比较高，多线程环境下不容易定位问题，且容易出现问题，所以netty的出现极大的简化了这种操作，且性能稳定高效，其架构设计也非常优秀，值得深入学习总结。

netty官网：https://netty.io/

![在这里插入图片描述](https://img-blog.csdnimg.cn/6689b9a86e9a4267b03be17281a3242b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_19,color_FFFFFF,t_70,g_se,x_16)

### 原生NIO问题

> 1.NIO 的类库和 API 繁杂，使用麻烦：需要熟练掌握 Selector、ServerSocketChannel、SocketChannel、ByteBuffer等。
>
> 2.需要具备其他的额外技能：要熟悉 Java 多线程编程，因为 NIO 编程涉及到 Reactor 模式，你必须对多线程和网络编程非常熟悉，才能编写出高质量的 NIO 程序。
>
> 3.开发工作量和难度都非常大：例如客户端面临断连重连、网络闪断、半包读写、失败缓存、网络拥塞和异常流的处理等等。
>
> 4.JDK NIO 的 Bug：例如臭名昭著的 Epoll Bug，它会导致 Selector 空轮询，最终导致 CPU100%。直到 JDK1.7 版本该问题仍旧存在，没有被根本解决。

### Netty特点

Netty 对 JDK 自带的 NIO 的 API 进行了封装，解决了上述问题。

> 1.设计优雅：适用于各种传输类型的统一 API 阻塞和非阻塞 Socket；基于灵活且可扩展的事件模型，可以清晰地分离关注点；高度可定制的线程模型-单线程，一个或多个线程池。
> 2.使用方便：详细记录的 Javadoc，用户指南和示例；没有其他依赖项，JDK5（Netty3.x）或 6（Netty4.x）就足够了。
> 3.高性能、吞吐量更高：延迟更低；减少资源消耗；最小化不必要的内存复制。
> 4.安全：完整的 SSL/TLS 和 StartTLS 支持。
> 5.社区活跃、不断更新：社区活跃，版本迭代周期短，发现的 Bug 可以被及时修复，同时，更多的新功能会被加入。

### Netty应用场景

互联网行业

> 1.互联网行业：在分布式系统中，各个节点之间需要远程服务调用，高性能的 RPC 框架必不可少，Netty 作为异步高性能的通信框架，往往作为基础通信组件被这些 RPC 框架使用。
> 2.典型的应用有：阿里分布式服务框架 Dubbo 的 RPC 框 架使用 Dubbo 协议进行节点间通信，Dubbo 协议默认使用 Netty 作为基础通信组件，用于实现各进程节点之间的内部通信。

游戏行业

> 1.无论是手游服务端还是大型的网络游戏，Java 语言得到了越来越广泛的应用。
> 2.Netty 作为高性能的基础通信组件，提供了 TCP/UDP 和 HTTP 协议栈，方便定制和开发私有协议栈，账号登录服务器。
> 3.地图服务器之间可以方便的通过 Netty 进行高性能的通信。

大数据领域

> 1.经典的 Hadoop 的高性能通信和序列化组件 Avro 的 RPC 框架，默认采用 Netty 进行跨界点通信。
> 2.它的 NettyService 基于 Netty 框架二次封装实现。

### Netty版本说明

Netty 版本分为 Netty 3.x 和 Netty 4.x、Netty 5.x（已经被舍弃）。
目前在官网可下载的版本 Netty 3.x、Netty 4.0.x 和 Netty 4.1.x。
netty下载地址：https://netty.io/downloads.html

Netty 5.0为啥被舍弃问题
Netty 5.0以前是发布alpha版，市面上也有一些基于Netty 5.0的书籍，突然间舍弃的原因作者是这样阐述的，大概意思是，使用ForkJoinPool增加了复杂性，并且没有显示出明显的性能优势。同时保持所有的分支同步是相当多的工作，没有必要。
原因具体答复链接， https://github.com/netty/netty/issues/4466

**3.X与4.X不兼容问题**
需要注意的问题是，3.X与4.X不兼容，需要升级的需要特别注意，具体见，[Netty版本升级血泪史之线程篇](https://blog.csdn.net/guolong1983811/article/details/78798046)

## Java IO模型

### IO模型
I/O 模型简单的理解：就是用什么样的通道进行数据的发送和接收，很大程度上决定了程序通信的性能。

Java 共支持 3 种网络编程模型 I/O 模式：BIO、NIO、AIO。

Java BIO：同步并阻塞（传统阻塞型），服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销.

![在这里插入图片描述](https://img-blog.csdnimg.cn/8cf30881bb9344149485f70d05dcac07.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_17,color_FFFFFF,t_70,g_se,x_16)

Java NIO：同步非阻塞，服务器实现模式为一个线程处理多个请求（连接），即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有 I/O 请求就进行处理。

![在这里插入图片描述](https://img-blog.csdnimg.cn/4d6c87b91a0b485babe4d0fa9b20e7b0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_20,color_FFFFFF,t_70,g_se,x_16)

Java AIO(NIO.2)：异步非阻塞，AIO 引入异步通道的概念，采用了 Proactor 模式，简化了程序编写，有效的请求才启动线程，它的特点是先由操作系统完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时间较长的应用。

### BIO,NIO,AIO适用场景

> 1.BIO 方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4 以前的唯一选择，但程序简单易理解。
> 2.NIO 方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，弹幕系统，服务器间通讯等。编程比较复杂，JDK1.4 开始支持。
> 3.AIO 方式使用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用 OS 参与并发操作，编程比较复杂，JDK7 开始支持。

## Java BIO

### Java BIO 基本说明

> 1.Java BIO 就是传统的 Java I/O 编程，其相关的类和接口在 java.io。
> 2.BIO(BlockingI/O)：同步阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，可以通过线程池机制改善（实现多个客户连接服务器）。
> 3.BIO 方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4 以前的唯一选择，程序简单易理解。

### Java BIO 工作机制

![在这里插入图片描述](https://img-blog.csdnimg.cn/53df1d24a7e942ef9802935cb7a13ee1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_20,color_FFFFFF,t_70,g_se,x_16)

> 1.服务器端启动一个 ServerSocket。
> 2.客户端启动 Socket 对服务器进行通信，默认情况下服务器端需要对每个客户建立一个线程与之通讯。
> 3.客户端发出请求后，先咨询服务器是否有线程响应，如果没有则会等待，或者被拒绝。
> 4.如果有响应，客户端线程会等待请求结束后，在继续执行。
>
> BIO模型是一对一的，一个服务端对应一个客户端，对应一个线程。

### Java BIO 问题分析

> 1.每个请求都需要创建独立的线程，与对应的客户端进行数据 Read，业务处理，数据 Write。
> 2.当并发数较大时，需要创建大量线程来处理连接，系统资源占用较大。
> 3.连接建立后，如果当前线程暂时没有数据可读，则线程就阻塞在 Read 操作上，造成线程资源浪费。

## Java NIO

### Java NIO 基本说明

> 1.Java NIO 全称 Java non-blocking IO ，是指 JDK 提供的新 API。从 JDK1.4 开始，Java 提供了一系列改进的输入/输出的新特性，被统称为 NIO（即 NewIO），是同步非阻塞的。
> 2.NIO 相关类都被放在 java.nio 包及子包下，并且对原 java.io 包中的很多类进行改写。
> 3.NIO 有三大核心部分: Channel（通道）、Buffer（缓冲区）、Selector（选择器） 。
> 4.NIO 是面向缓冲区，或者面向块编程的。数据读取到一个它稍后处理的缓冲区，需要时可在缓冲区中前后移动，这就增加了处理过程中的灵活性，使用它可以提供非阻塞式的高伸缩性网络。
> 5.Java NIO 的非阻塞模式，使一个线程从某通道发送请求或者读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他的事情。非阻塞写也是如此，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情。
> 6.通俗理解：NIO 是可以做到用一个线程来处理多个操作的。假设有 10000 个请求过来,根据实际情况，可以分配 50 或者 100 个线程来处理。不像之前的阻塞 IO 那样，非得分配 10000 个。
> 7.HTTP 2.0 使用了多路复用的技术，做到同一个连接并发处理多个请求，而且并发请求的数量比 HTTP 1.1 大了好几个数量级。

### NIO 和 BIO 的比较

> 1.BIO 以流的方式处理数据，而 NIO 以块的方式处理数据，块 I/O 的效率比流 I/O 高很多。
> 2.BIO 是阻塞的，NIO 则是非阻塞的。
> 3.BIO 基于字节流和字符流进行操作，而 NIO 基于 Channel（通道）和 Buffer（缓冲区）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector（选择器）用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用单个线程就可以监听多个客户端通道。

### NIO 核心组件

NIO 三大核心组件selector，channel，buffer。

![在这里插入图片描述](https://img-blog.csdnimg.cn/abf52cf8ea724322ae3a0328416e4e14.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_15,color_FFFFFF,t_70,g_se,x_16)

> 1.每个Thread对应一个selector选择器，一个selector选择器对应多个通道channel，每个Channel 都会对应一个 Buffer。
> 2.Selector 对应一个线程，一个线程对应多个 Channel（连接）。
> 3.该图反应了有三个 Channel 注册到该 Selector //程序
> 4.程序切换到哪个 Channel 是由事件决定的，Event 就是一个重要的概念。
> 5.Selector 会根据不同的事件，在各个通道上切换。
> 6.Buffer 就是一个内存块，底层是有一个数组。
> 7.数据的读取写入是通过 Buffer，这个和 BIO，BIO 中要么是输入流，或者是输出流，不能双向，但是 NIO 的 Buffer 是可以读也可以写，需要 flip 方法切换 Channel 是双向的，可以返回底层操作系统的情况，比如 Linux，底层的操作系统通道就是双向的

### Selector（选择器）

> 1.Java 的 NIO，用非阻塞的 IO 方式。可以用一个线程，处理多个的客户端连接，就会使用到 Selector（选择器）。
> 2.Selector 能够检测多个注册的通道上是否有事件发生（注意：多个 Channel 以事件的方式可以注册到同一个 Selector），如果有事件发生，便获取事件然后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个通道，也就是管理多个连接和请求。【示意图】
> 3.只有在连接/通道真正有读写事件发生时，才会进行读写，就大大地减少了系统开销，并且不必为每个连接都创建一个线程，不用去维护多个线程。
> 4.避免了多线程之间的上下文切换导致的开销。

### Channel（通道）

> 1.NIO 的通道类似于流，但有些区别如下：
> 通道可以同时进行读写，而流只能读或者只能写
> 通道可以实现异步读写数据
> 通道可以从缓冲读数据，也可以写数据到缓冲:
> 2.BIO 中的 Stream 是单向的，例如 FileInputStream 对象只能进行读取数据的操作，而 NIO 中的通道（Channel）是双向的，可以读操作，也可以写操作。
> 3.Channel 在 NIO 中是一个接口 public interface Channel extends Closeable{}
> 4.常用的 Channel 类有: FileChannel、DatagramChannel、ServerSocketChannel 和 SocketChannel 。【ServerSocketChanne 类似 ServerSocket、SocketChannel 类似 Socket】
> 5.FileChannel 用于文件的数据读写，DatagramChannel 用于 UDP 的数据读写，ServerSocketChannel 和 SocketChannel 用于 TCP 的数据读写

### Buffer（缓冲区）

缓冲区（Buffer）：缓冲区本质上是一个可以读写数据的内存块，可以理解成是一个容器对象（含数组），该对象提供了一组方法，可以更轻松地使用内存块，，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变化情况。Channel 提供从文件、网络读取数据的渠道，但是读取或写入的数据都必须经由 Buffer，如图:【后面举例说明】

![在这里插入图片描述](https://img-blog.csdnimg.cn/a959d16db5ce4732b4bf89d7e6dce302.png)

### NIO 非阻塞网络编程原理分析图

NIO 非阻塞网络编程相关的（Selector、SelectionKey、ServerScoketChannel 和 SocketChannel）关系梳理图。

![在这里插入图片描述](https://img-blog.csdnimg.cn/d20745a1256f4ef993377ef08be094b4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_20,color_FFFFFF,t_70,g_se,x_16)

### NIO与零拷贝

**1.零拷贝基本介绍**

> 1.零拷贝是网络编程的关键，很多性能优化都离不开。
> 2.在 Java 程序中，常用的零拷贝有 mmap（内存映射）和 sendFile。

**2.传统IO数据读写**

![在这里插入图片描述](https://img-blog.csdnimg.cn/bcf97ba2243547aa8e1afa0d7d787649.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_20,color_FFFFFF,t_70,g_se,x_16)

> 调用流程说明
> 1.read 调用导致用户态到内核态的一次变化，同时，第一次复制开始：DMA（Direct Memory Access，直接内存存取，即不使用 CPU 拷贝数据到内存，而是 DMA 引擎传输数据到内存，用于解放 CPU） 引擎从磁盘读取 index.html 文件，并将数据放入到内核缓冲区。
> 2.发生第二次数据拷贝，即：将内核缓冲区的数据拷贝到用户缓冲区，同时，发生了一次用内核态到用户态的上下文切换。
> 3.发生第三次数据拷贝，我们调用 write 方法，系统将用户缓冲区的数据拷贝到 Socket 缓冲区。此时，又发生了一次用户态到内核态的上下文切换。
> 4.第四次拷贝，数据异步的从 Socket 缓冲区，使用 DMA 引擎拷贝到网络协议引擎。这一段，不需要进行上下文切换。
> 5.write 方法返回，再次从内核态切换到用户态。
>
> 传统IO数据读写总共经历了4次拷贝，4次上下文切换（用户态与内核态切换）。

**3.mmap数据读写**
mmap 通过内存映射，将文件映射到内核缓冲区，同时，用户空间可以共享内核空间的数据。这样，在进行网络传输时，就可以减少内核空间到用户控件的拷贝次数。如下图

![在这里插入图片描述](https://img-blog.csdnimg.cn/f0a34664c1134ec2a1820320bf2460f8.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_20,color_FFFFFF,t_70,g_se,x_16)

如上图，user buffer 和 kernel buffer 共享 index.html。如果你想把硬盘的 index.html 传输到网络中，再也不用拷贝到用户空间，再从用户空间拷贝到 Socket 缓冲区。
现在，你只需要从内核缓冲区拷贝到 Socket 缓冲区即可，这将减少一次内存拷贝（从 4 次变成了 3 次），但不减少上下文切换次数。

**mmap 数据读写总共经历了3次拷贝，4次上下文切换。**

**4.sendFile数据读写**

![在这里插入图片描述](https://img-blog.csdnimg.cn/c4e09d829a454389baf6dffdd8e63644.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_20,color_FFFFFF,t_70,g_se,x_16)

> Linux 2.1 版本 提供了 sendFile 函数，其基本原理如下：数据根本不经过用户态，直接从内核缓冲区进入到 Socket Buffer，同时，由于和用户态完全无关，就减少了一次上下文切换。
> 如上图，我们进行 sendFile 系统调用时，数据被 DMA 引擎从文件复制到内核缓冲区，然后调用，然后掉一共 write 方法时，从内核缓冲区进入到 Socket，这时，是没有上下文切换的，因为在一个用户空间。
>
> Linux 在 2.4 版本中sendFile ，做了一些修改，避免了从内核缓冲区拷贝到 Socket buffer 的操作，直接拷贝到协议栈，从而再一次减少了数据拷贝。现在，index.html 要从文件进入到网络协议栈，只需 2 次拷贝：第一次使用 DMA 引擎从文件拷贝到内核缓冲区，第二次从内核缓冲区将数据拷贝到网络协议栈；内核缓存区只会拷贝一些 offset 和 length 信息到 SocketBuffer，基本无消耗。
>
> Linux 在 2.1版本中sendFile ，数据读写总共经历了3次拷贝，3次上下文切换。
> Linux 在 2.4版本中sendFile ，数据读写总共经历了2次拷贝，3次上下文切换。

所以，根据传统IO，mmap，sendFile三种方式数据读写的操作真正的零拷贝不是没有拷贝，最少也需要2次拷贝，3次上下文切换。
==首先我们说零拷贝，是从操作系统的角度来说的。因为内核缓冲区之间，没有数据是重复的（只有 kernel buffer 有一份数据，sendFile 2.1 版本实际上有 2 份数据，算不上零拷贝）。例如我们刚开始的例子，内核缓存区和 Socket 缓冲区的数据就是重复的。而零拷贝不仅仅带来更少的数据复制，还能带来其他的性能优势，例如更少的上下文切换，更少的 CPU 缓存伪共享以及无 CPU 校验和计算。==

### Java AIO 基本介绍

> 1.JDK7 引入了 AsynchronousI/O，即 AIO。在进行 I/O 编程中，常用到两种模式：Reactor 和 Proactor。Java 的 NIO 就是 Reactor，当有事件触发时，服务器端得到通知，进行相应的处理
> 2.AIO 即 NIO2.0，叫做异步不阻塞的 IO。AIO 引入异步通道的概念，采用了 Proactor 模式，简化了程序编写，有效的请求才启动线程，它的特点是先由操作系统完成后才通知服务端程序启动线程去处理，一般适用于连接数较多且连接时间较长的应用
> 3.目前 AIO 还没有广泛应用，Netty 也是基于 NIO，而不是 AIO，因此我们就不详解 AIO 了

[《Java新一代网络编程模型AIO原理及Linux系统AIO介绍》](http://www.52im.net/thread-306-1-1.html)

### BIO、NIO、AIO 对比表

|          | BIO      | NIO                    | AIO        |
| -------- | -------- | ---------------------- | ---------- |
| IO模型   | 同步阻塞 | 同步非阻塞（多路复用） | 异步非阻塞 |
| 编程难度 | 简单     | 复杂                   | 复杂       |
| 可靠性   | 差       | 好                     | 好         |
| 吞吐量   | 低       | 高                     | 高         |

## Netty高性能架构

### 线程模型说明

> 1.不同的线程模式，对程序的性能有很大影响。
> 2.目前存在的线程模型有：传统阻塞 I/O 服务模型 Reactor 模式
> 3.根据 Reactor 的数量和处理资源池线程的数量不同，有 3 种典型的实现单 Reactor 单线程；单 Reactor多线程；主从 Reactor多线程
> 4.Netty 线程模式（Netty 主要基于主从 Reactor 多线程模型做了一定的改进，其中主从 Reactor 多线程模型有多个 Reactor）

### 传统阻塞IO模型

![在这里插入图片描述](https://img-blog.csdnimg.cn/3429dd1744fc4362a734679c545f71b5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_16,color_FFFFFF,t_70,g_se,x_16)

1.采用阻塞 IO 模式获取输入的数据
2.每个连接都需要独立的线程完成数据的输入，业务处理，数据返回

问题分析

> 1.当并发数很大，就会创建大量的线程，占用很大系统资源
> 2.连接创建后，如果当前线程暂时没有数据可读，该线程会阻塞在 read 操作，造成线程资源浪费

### Reactor模式

> 针对传统阻塞 I/O 服务模型的 2 个缺点，解决方案：
> 1.IO复用
> 基于 I/O 复用模型：多个连接共用一个阻塞对象，应用程序只需要在一个阻塞对象等待，无需阻塞等待所有连接。当某个连接有新的数据可以处理时，操作系统通知应用程序，线程从阻塞状态返回，开始进行业务处理 Reactor 对应的叫法：
> 反应器模式
> 分发者模式（Dispatcher）
> 通知者模式（notifier）
>
> 2.线程复用
> 基于线程池复用线程资源：不必再为每个连接创建线程，将连接完成后的业务处理任务分配给线程进行处理，一个线程可以处理多个连接的业务。

I/O 复用结合线程池，就是 Reactor 模式基本设计思想，如图

![在这里插入图片描述](https://img-blog.csdnimg.cn/43632ba973044068b242e1c59fdc575c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_20,color_FFFFFF,t_70,g_se,x_16)

Reactor 模式中核心组成

> 1.Reactor：Reactor 在一个单独的线程中运行，负责监听和分发事件，分发给适当的处理程序来对 IO 事件做出反应。它就像公司的电话接线员，它接听来自客户的电话并将线路转移到适当的联系人；
> 2.Handlers：处理程序执行 I/O 事件要完成的实际事件，类似于客户想要与之交谈的公司中的实际官员。Reactor 通过调度适当的处理程序来响应 I/O 事件，处理程序执行非阻塞操作。

Reactor 模式分类
根据 Reactor 的数量和处理资源池线程的数量不同，有 3 种典型的实现。

> 1.单 Reactor 单线程
> 2.单 Reactor 多线程
> 3.主从 Reactor 多线程

#### 单Reactor单线程

![在这里插入图片描述](https://img-blog.csdnimg.cn/d66638b6d41f42b8942c3eb590c18f69.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_20,color_FFFFFF,t_70,g_se,x_16)

> 1.Select 是前面 I/O 复用模型介绍的标准网络编程 API，可以实现应用程序通过一个阻塞对象监听多路连接请求
> 2.Reactor 对象通过 Select 监控客户端请求事件，收到事件后通过 Dispatch 进行分发
> 3.如果是建立连接请求事件，则由 Acceptor 通过 Accept 处理连接请求，然后创建一个 Handler 对象处理连接完成后的后续业务处理
> 4.如果不是建立连接事件，则 Reactor 会分发调用连接对应的 Handler 来响应
> 5.Handler 会完成 Read → 业务处理 → Send 的完整业务流程
>
> 结合实例：服务器端用一个线程通过多路复用搞定所有的 IO 操作（包括连接，读、写等），编码简单，清晰明了，但是如果客户端连接数量较多，将无法支撑，前面的 NIO 案例就属于这种模型。

#### 单Reactor多线程

![在这里插入图片描述](https://img-blog.csdnimg.cn/6a55052b0b4f46b28418a2834bb95692.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_18,color_FFFFFF,t_70,g_se,x_16)

> 1.Reactor 对象通过 Select 监控客户端请求事件，收到事件后，通过 Dispatch 进行分发
> 2.如果建立连接请求，则右 Acceptor 通过 accept 处理连接请求，然后创建一个 Handler 对象处理完成连接后的各种事件
> 3.如果不是连接请求，则由 Reactor 分发调用连接对应的 handler 来处理
> 4.handler 只负责响应事件，不做具体的业务处理，通过 read 读取数据后，会分发给后面的 worker 线程池的某个线程处理业务
> 5.worker 线程池会分配独立线程完成真正的业务，并将结果返回给 handler
> 6.handler 收到响应后，通过 send 将结果返回给 client

#### 主从Reactor多线程

![在这里插入图片描述](https://img-blog.csdnimg.cn/c5b9b084450140d6926d7e27545d69f7.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_17,color_FFFFFF,t_70,g_se,x_16)

> 1.Reactor 主线程 MainReactor 对象通过 select 监听连接事件，收到事件后，通过 Acceptor 处理连接事件
> 2.当 Acceptor 处理连接事件后，MainReactor 将连接分配给 SubReactor
> 3.subreactor 将连接加入到连接队列进行监听，并创建 handler 进行各种事件处理
> 4.当有新事件发生时，subreactor 就会调用对应的 handler 处理
> 5.handler 通过 read 读取数据，分发给后面的 worker 线程处理
> 6.worker 线程池分配独立的 worker 线程进行业务处理，并返回结果
> 7.handler 收到响应的结果后，再通过 send 将结果返回给 client
> 8.Reactor 主线程可以对应多个 Reactor 子线程，即 MainRecator 可以关联多个 SubReactor

### Netty模型

![在这里插入图片描述](https://img-blog.csdnimg.cn/8eaad81cf4904a03b883abfea9ba4472.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_20,color_FFFFFF,t_70,g_se,x_16)

> 1.Netty 抽象出两组线程池 BossGroup 专门负责接收客户端的连接，WorkerGroup 专门负责网络的读写
> 2.BossGroup 和 WorkerGroup 类型都是 NioEventLoopGroup
> 3.NioEventLoopGroup 相当于一个事件循环组，这个组中含有多个事件循环，每一个事件循环是 NioEventLoop
> 4.NioEventLoop 表示一个不断循环的执行处理任务的线程，每个 NioEventLoop 都有一个 Selector，用于监听绑定在其上的 socket 的网络通讯
> 5.NioEventLoopGroup 可以有多个线程，即可以含有多个 NioEventLoop
> 6.每个 BossNioEventLoop 循环执行的步骤有 3 步
>
> ```
> 轮询 accept 事件
> 处理 accept 事件，与 client 建立连接，生成 NioScocketChannel，并将其注册到某个 worker NIOEventLoop 上的 Selector
> 处理任务队列的任务，即 runAllTasks
> ```
>
> 7.每个 Worker NIOEventLoop 循环执行的步骤
>
> ```
> 轮询 read，write 事件
> 处理 I/O 事件，即 read，write 事件，在对应 NioScocketChannel 处理
> 处理任务队列的任务，即 runAllTasks
> ```
>
> 8.每个 Worker NIOEventLoop 处理业务时，会使用 pipeline（管道），pipeline 中包含了 channel，即通过 pipeline 可以获取到对应通道，管道中维护了很多的处理器。

###  异步模型

> 1.异步的概念和同步相对。当一个异步过程调用发出后，调用者不能立刻得到结果。实际处理这个调用的组件在完成后，通过状态、通知和回调来通知调用者。
> 2.Netty 中的 I/O 操作是异步的，包括 Bind、Write、Connect 等操作会简单的返回一个 ChannelFuture。
> 3.调用者并不能立刻获得结果，而是通过 Future-Listener 机制，用户可以方便的主动获取或者通过通知机制获得 IO 操作结果。
> 4.Netty 的异步模型是建立在 future 和 callback 的之上的。callback 就是回调。重点说 Future，它的核心思想是：假设一个方法 fun，计算过程可能非常耗时，等待 fun 返回显然不合适。那么可以在调用 fun 的时候，立马返回一个 Future，后续可以通过 Future 去监控方法 fun 的处理过程（即：Future-Listener 机制）

![在这里插入图片描述](https://img-blog.csdnimg.cn/90f748e7b02e48978e65c474ba734248.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_20,color_FFFFFF,t_70,g_se,x_16)

## Netty核心组件说明

### Bootstrap、ServerBootstrap

```java
Bootstrap 意思是引导，一个 Netty 应用通常由一个 Bootstrap 开始，主要作用是配置整个 Netty 程序，串联各个组件，Netty 中 Bootstrap 类是客户端程序的启动引导类，ServerBootstrap 是服务端启动引导类。
常见的方法有
    public ServerBootstrap group(EventLoopGroup parentGroup, EventLoopGroup childGroup)，该方法用于服务器端，用来设置两个 EventLoop
    public B group(EventLoopGroup group)，该方法用于客户端，用来设置一个 EventLoop
    public B channel(Class<? extends C> channelClass)，该方法用来设置一个服务器端的通道实现
    public <T> B option(ChannelOption<T> option, T value)，用来给 ServerChannel 添加配置
    public <T> ServerBootstrap childOption(ChannelOption<T> childOption, T value)，用来给接收到的通道添加配置
    public ServerBootstrap childHandler(ChannelHandler childHandler)，该方法用来设置业务处理类（自定义的handler）
    public ChannelFuture bind(int inetPort)，该方法用于服务器端，用来设置占用的端口号
    public ChannelFuture connect(String inetHost, int inetPort)，该方法用于客户端，用来连接服务器端
```

### Future、ChannelFuture

Netty 中所有的 IO 操作都是异步的，不能立刻得知消息是否被正确处理。但是可以过一会等它执行完成或者直接注册一个监听，具体的实现就是通过 Future 和 ChannelFutures，他们可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件

常见的方法有

```java
Channel channel()，返回当前正在进行 IO 操作的通道
ChannelFuture sync()，等待异步操作执行完毕
```

### Channel

```java
Netty 网络通信的组件，能够用于执行网络 I/O 操作。
通过 Channel 可获得当前网络连接的通道的状态
通过 Channel 可获得网络连接的配置参数（例如接收缓冲区大小）
Channel 提供异步的网络 I/O 操作(如建立连接，读写，绑定端口)，异步调用意味着任何 I/O 调用都将立即返回，并且不保证在调用结束时所请求的 I/O 操作已完成
调用立即返回一个 ChannelFuture 实例，通过注册监听器到 ChannelFuture 上，可以 I/O 操作成功、失败或取消时回调通知调用方
支持关联 I/O 操作与对应的处理程序
不同协议、不同的阻塞类型的连接都有不同的 Channel 类型与之对应，常用的 Channel 类型：
    NioSocketChannel，异步的客户端 TCP Socket 连接。
    NioServerSocketChannel，异步的服务器端 TCP Socket 连接。
    NioDatagramChannel，异步的 UDP 连接。
    NioSctpChannel，异步的客户端 Sctp 连接。
    NioSctpServerChannel，异步的 Sctp 服务器端连接，这些通道涵盖了 UDP 和 TCP 网络 IO 以及文件 IO。
```

### Selector

```java
Netty 基于 Selector 对象实现 I/O 多路复用，通过 Selector 一个线程可以监听多个连接的 Channel 事件。
当向一个 Selector 中注册 Channel 后，Selector 内部的机制就可以自动不断地查询（Select）这些注册的 Channel 是否有已就绪的 I/O 事件（例如可读，可写，网络连接完成等），这样程序就可以很简单地使用一个线程高效地管理多个 Channel
```

### ChannelHandler 及其实现类

``` ChannelHandler 是一个接口，处理 I/O 事件或拦截 I/O 操作，并将其转发到其 ChannelPipeline（业务处理链）中的下一个处理程序。
ChannelHandler 本身并没有提供很多方法，因为这个接口有许多的方法需要实现，方便使用期间，可以继承它的子类
ChannelHandler 及其实现类一览图（后）
我们经常需要自定义一个 Handler 类去继承 ChannelInboundHandlerAdapter，然后通过重写相应方法实现业务逻辑，我们接下来看看一般都需要重写哪些方法。
```

### Pipeline 和 ChannelPipeline

ChannelPipeline 是一个重点：

```java
ChannelPipeline 是一个 Handler 的集合，它负责处理和拦截 inbound 或者 outbound 的事件和操作，相当于一个贯穿 Netty 的链。（也可以这样理解：ChannelPipeline 是保存 ChannelHandler 的 List，用于处理或拦截 Channel 的入站事件和出站操作）
ChannelPipeline 实现了一种高级形式的拦截过滤器模式，使用户可以完全控制事件的处理方式，以及 Channel 中各个的 ChannelHandler 如何相互交互
在 Netty 中每个 Channel 都有且仅有一个 ChannelPipeline 与之对应，它们的组成关系如下。
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/b8f9699d8cf14275a9f19dad97f3a3aa.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_20,color_FFFFFF,t_70,g_se,x_16)

### ChannelHandlerContext

```java
保存 Channel 相关的所有上下文信息，同时关联一个 ChannelHandler 对象
即 ChannelHandlerContext 中包含一个具体的事件处理器 ChannelHandler，同时 ChannelHandlerContext 中也绑定了对应的 pipeline 和 Channel 的信息，方便对 ChannelHandler 进行调用。
常用方法
    ChannelFuture close()，关闭通道
    ChannelOutboundInvoker flush()，刷新
    ChannelFuture writeAndFlush(Object msg)，将数据写到
    ChannelPipeline 中当前 ChannelHandler 的下一个 ChannelHandler 开始处理（出站）
```

### ChannelOption

```java
Netty 在创建 Channel 实例后，一般都需要设置 ChannelOption 参数。
ChannelOption 参数如下：
```

### EventLoopGroup 和其实现类 NioEventLoopGroup

```java
EventLoopGroup 是一组 EventLoop 的抽象，Netty 为了更好的利用多核 CPU 资源，一般会有多个 EventLoop 同时工作，每个 EventLoop 维护着一个 Selector 实例。
EventLoopGroup 提供 next 接口，可以从组里面按照一定规则获取其中一个 EventLoop 来处理任务。在 Netty 服务器端编程中，我们一般都需要提供两个 EventLoopGroup，例如：BossEventLoopGroup 和 WorkerEventLoopGroup。
通常一个服务端口即一个 ServerSocketChannel 对应一个 Selector 和一个 EventLoop 线程。BossEventLoop 负责接收客户端的连接并将 SocketChannel 交给 WorkerEventLoopGroup 来进行 IO 处理。
```

### Unpooled 类

```java
Netty 提供一个专门用来操作缓冲区（即 Netty 的数据容器）的工具类
常用方法如下所示
```

## Google Protobuf

### 编码和解码的基本介绍

1.编写网络应用程序时，因为数据在网络中传输的都是二进制字节码数据，在发送数据时就需要编码，接收数据时就需要解码[示意图]

2.codec（编解码器）的组成部分有两个：decoder（解码器）和 encoder（编码器）。encoder 负责把业务数据转换成字节码数据，decoder 负责把字节码数据转换成业务数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/02c47c200072465796c1e86307bc677b.png)

### Netty 本身的编码解码的机制和问题分析

```java
Netty 自身提供了一些 codec(编解码器)

Netty 提供的编码器 StringEncoder，对字符串数据进行编码 ObjectEncoder，对Java对象进行编码...

Netty 提供的解码器 StringDecoder,对字符串数据进行解码 ObjectDecoder，对 Java 对象进行解码...

Netty 本身自带的 ObjectDecoder 和 ObjectEncoder 可以用来实现 POJO 对象或各种业务对象的编码和解码，底层使用的仍是Java序列化技术,而Java序列化技术本身效率就不高，存在如下问题
    无法跨语言
    序列化后的体积太大，是二进制编码的5倍多。
    序列化性能太低

=>引出新的解决方案[Google 的 Protobuf]
```

### Protobuf

```java
Protobuf 基本介绍和使用示意图
Protobuf 是 Google 发布的开源项目，全称 Google Protocol Buffers，是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 RPC [远程过程调用 remote procedure call ]数据交换格式。目前很多公司 http + json tcp + protobuf
参考文档：https://developers.google.com/protocol-buffers/docs/proto 语言指南
Protobuf 是以 message 的方式来管理数据的.
支持跨平台、跨语言，即[客户端和服务器端可以是不同的语言编写的]（支持目前绝大多数语言，例如 C++、C#、Java、python 等）
高性能，高可靠性
使用 protobuf 编译器能自动生成代码，Protobuf 是将类的定义使用 .proto 文件进行描述。说明，在 idea 中编写 .proto 文件时，会自动提示是否下载 .ptoto 编写插件.可以让语法高亮。
然后通过 protoc.exe 编译器根据 .proto 自动生成 .java 文件
protobuf 使用示意图
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/b4893065162a4b6fbe0e7c37545ca95b.png)

## Netty 编解码器和 Handler 调用机制

### 基本说明

```java
Netty 的组件设计：Netty 的主要组件有 Channel、EventLoop、ChannelFuture、ChannelHandler、ChannelPipe 等
ChannelHandler 充当了处理入站和出站数据的应用程序逻辑的容器。例如，实现 ChannelInboundHandler 接口（或 ChannelInboundHandlerAdapter），你就可以接收入站事件和数据，这些数据会被业务逻辑处理。当要给客户端发送响应时，也可以从 ChannelInboundHandler 冲刷数据。业务逻辑通常写在一个或者多个 ChannelInboundHandler 中。ChannelOutboundHandler 原理一样，只不过它是用来处理出站数据的
ChannelPipeline 提供了 ChannelHandler 链的容器。以客户端应用程序为例，如果事件的运动方向是从客户端到服务端的，那么我们称这些事件为出站的，即客户端发送给服务端的数据会通过 pipeline 中的一系列 ChannelOutboundHandler，并被这些 Handler 处理，反之则称为入站的
```

### 编码解码器

```java
当 Netty 发送或者接受一个消息的时候，就将会发生一次数据转换。入站消息会被解码：从字节转换为另一种格式（比如 java 对象）；如果是出站消息，它会被编码成字节。
Netty 提供一系列实用的编解码器，他们都实现了 ChannelInboundHadnler 或者 ChannelOutboundHandler 接口。在这些类中，channelRead 方法已经被重写了。以入站为例，对于每个从入站 Channel 读取的消息，这个方法会被调用。随后，它将调用由解码器所提供的 decode() 方法进行解码，并将已经解码的字节转发给 ChannelPipeline 中的下一个 ChannelInboundHandler。
```

### 解码器 - ByteToMessageDecoder

```java
由于不可能知道远程节点是否会一次性发送一个完整的信息，tcp 有可能出现粘包拆包的问题，这个类会对入站数据进行缓冲，直到它准备好被处理.
一个关于 ByteToMessageDecoder 实例分析
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/0399056a804947e2ba10f584c7b96441.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_16,color_FFFFFF,t_70,g_se,x_16)

### Netty 的 handler 链的调用机制

使用自定义的编码器和解码器来说明 Netty 的 handler 调用机制 客户端发送 long -> 服务器 服务端发送 long -> 客户端。

![在这里插入图片描述](https://img-blog.csdnimg.cn/d45fac15092b473a9941ffa23080e23a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_16,color_FFFFFF,t_70,g_se,x_16)

### 解码器 - ReplayingDecoder

1.public abstract class ReplayingDecoder< S > extends ByteToMessageDecoder
2.ReplayingDecoder 扩展了 ByteToMessageDecoder 类，使用这个类，我们不必调用 readableBytes() 方法。参数 S 指定了用户状态管理的类型，其中 Void 代表不需要状态管理。

## TCP 粘包和拆包及解决方案

### TCP 粘包和拆包基本介绍

1.TCP 是面向连接的，面向流的，提供高可靠性服务。收发两端（客户端和服务器端）都要有一一成对的 socket，因此，发送端为了将多个发给接收端的包，更有效的发给对方，使用了优化方法（Nagle 算法），将多次间隔较小且数据量小的数据，合并成一个大的数据块，然后进行封包。这样做虽然提高了效率，但是接收端就难于分辨出完整的数据包了，因为面向流的通信是无消息保护边界的。
2.由于 TCP 无消息保护边界,需要在接收端处理消息边界问题，也就是我们所说的粘包、拆包问题,看一张图
3.示意图 TCP 粘包、拆包图解

![在这里插入图片描述](https://img-blog.csdnimg.cn/ff51236470ed456e9a3766ffe385338f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6Z2W6IqC5YWI55Sf,size_18,color_FFFFFF,t_70,g_se,x_16)

对图的说明: 假设客户端分别发送了两个数据包 D1 和 D2 给服务端，由于服务端一次读取到字节数是不确定的，故可能存在以下四种情况：

```java
服务端分两次读取到了两个独立的数据包，分别是 D1 和 D2，没有粘包和拆包
服务端一次接受到了两个数据包，D1 和 D2 粘合在一起，称之为 TCP 粘包
服务端分两次读取到了数据包，第一次读取到了完整的 D1 包和 D2 包的部分内容，第二次读取到了 D2 包的剩余内容，这称之为 TCP 拆包
服务端分两次读取到了数据包，第一次读取到了 D1 包的部分内容 D1_1，第二次读取到了 D1 包的剩余部分内容 D1_2 和完整的 D2 包。
```

### TCP 粘包和拆包解决方案

> 1.使用自定义协议+编解码器来解决
> 2.==关键就是要解决服务器端每次读取数据长度的问题==，这个问题解决，就不会出现服务器多读或少读数据的问题，从而避免的 TCP 粘包、拆包。

## **Netty Hello World**

### **引入java包**

```xml
<dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>4.1.20.Final</version>
</dependency>
```

### **netty server 端编写**

```java
package com.zpb.netty.netty.helloWorld;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoop;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

/**
 * @dec : netty入门
 * @Date: 2019/11/24
 * @Auther: pengbo.zhao
 * @version: 1.0
 * @demand:
 *
 *    {@link #main(String[] args)}
 *
 */
public class NettyServer {

        public static void main(String[] args) throws  Exception{

        //1.创建BossGroup 和 WorkerGroup
        //1.1 创建2个线程组
        //bossGroup只处理连接请求
        //workerGroup 处理客户端的业务逻辑
        //2个都是无限循环
        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();

        //2.创建服务端的启动对象,可以为服务端启动配置一些服务参数
        ServerBootstrap bootStrap = new ServerBootstrap();

        //2.1使用链式编程来配置服务参数
        bootStrap.group(bossGroup,workerGroup)                          //设置2个线程组
                .channel(NioServerSocketChannel.class)                 //使用NioServerSocketChannel作为服务器的通道
                .option(ChannelOption.SO_BACKLOG,128)            //设置线程等待的连接个数
                .childOption(ChannelOption.SO_KEEPALIVE,Boolean.TRUE) //设置保持活动连接状态
                .childHandler(new ChannelInitializer<SocketChannel>() {
                    //给PipeLine设置处理器
                    @Override
                    protected void initChannel(SocketChannel socketChannel) throws Exception {
                        //通过socketChannel得到pipeLine，然后向pipeLine中添加处理的handle
                        socketChannel.pipeline().addLast(new NettyServerHandle());
                    }
                }); //给workerGroup 的EventLoop对应的管道设置处理器(可以自定义/也可使用netty的)
        System.err.println("server is ready......");

        //启动服务器，并绑定1个端口且同步生成一个ChannelFuture 对象
        ChannelFuture channelFuture = bootStrap.bind(8888).sync();

        //对关闭通道进行监听(netty异步模型)
        //当通道进行关闭时，才会触发这个关闭动作
        channelFuture.channel().closeFuture().sync();

    }
}
```

### **netty server handler编写**

```java
package com.zpb.netty.netty.helloWorld;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.Channel;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.channel.ChannelPipeline;
import io.netty.util.CharsetUtil;

/**
 * @dec :
 * @Date: 2019/11/24
 * @Auther: pengbo.zhao
 * @version: 1.0
 * @demand:
 */
public class NettyServerHandle extends ChannelInboundHandlerAdapter {
    /**
     * 读取数据
     *
     * @param: 1.ChannelHandlerContext ctx:上下文对象, 含有 管道 pipeline , 通道 channel, 地址
     * @param: 2. Object msg: 就是客户端发送的数据 默认 Object
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.err.println("服务器读取线程 " + Thread.currentThread().getName());
        System.out.println("server ctx =" + ctx);
        System.out.println("看看 channel 和 pipeline 的关系");

        Channel channel = ctx.channel();
        ChannelPipeline pipeline = ctx.pipeline(); //本质是一个双向链接, 出站入站

        //将 msg 转成一个 ByteBuf,ByteBuf 是 Netty 提供的，不是 NIO 的 ByteBuffer.
        ByteBuf buf = (ByteBuf) msg;
        System.out.println("客户端发送消息是:" + buf.toString(CharsetUtil.UTF_8));
        System.out.println("客户端地址:" + channel.remoteAddress());
    }

    /**
     * 读取数据完成后
     *
     * @param:
     * @return:
     * @auther:
     * @date:
     */
    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        //writeAndFlush 是 write + flush
        //将数据写入到缓存，并刷新
        //一般讲，我们对这个发送的数据进行编码
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello, 客户端~(>^ω^<)喵", CharsetUtil.UTF_8));
    }

    //处理异常, 一般是需要关闭通道
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        ctx.close();
    }
}
```

### **netty client端编写**

```java
package com.zpb.netty.netty.helloWorld;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.EventLoopGroup;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;


/**
 * @dec :
 * @Date: 2019/11/24
 * @Auther: pengbo.zhao
 * @version: 1.0
 * @demand:
 */
public class NettyClient {

    public static void main(String[] args) throws Exception {

        //1.客户端定义一个循环事件组
        EventLoopGroup group = new NioEventLoopGroup();

        try {

            //2.创建客户端启动对象
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)                      //设置线程组
                    .channel(NioSocketChannel.class)   //设置客户端通道实现类
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel socketChannel) throws Exception {
                            socketChannel.pipeline().addLast(new NettyClientHandle());
                        }
                    });
            System.err.println("client is ready......");

            //3.启动客户端去连接服务端
            ChannelFuture channelFuture = bootstrap.connect("127.0.0.1", 8888).sync();

            //4.设置通道关闭监听(当监听到通道关闭时，关闭client)
            channelFuture.channel().closeFuture().sync();
        } finally {
            group.shutdownGracefully();
        }
    }
}
```

### **netty client handler端编写**

```java
package com.zpb.netty.netty.helloWorld;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;
import io.netty.util.CharsetUtil;

/**
 * @dec :
 * @Date: 2019/11/24
 * @Auther: pengbo.zhao
 * @version: 1.0
 * @demand:
 */
public class NettyClientHandle extends ChannelInboundHandlerAdapter{

    //如果client 端服务启动完成后
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {

        System.err.println("client "+ctx);
        ctx.writeAndFlush(Unpooled.copiedBuffer("hello,netty server...",CharsetUtil.UTF_8));
    }

    //当通道有读事件时
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {

        ByteBuf byteBuf = (ByteBuf) msg;
        System.err.println("服务器端回复消息："+byteBuf.toString(CharsetUtil.UTF_8));
        System.err.println("服务器端地址是："+ctx.channel().remoteAddress());
    }

    //当通道有异常时

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

