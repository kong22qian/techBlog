[TOC]

# Hardware base info



## 笔记本硬件解读

![image-20230302225753518](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20230302225753518.png)

> - ==CPU 是一个人的心脏，所有行为都靠他进行启动。==
>
> - ==内存 是一个人的小脑，所有的现有的想法都在这里思考==
>
> - ==硬盘 是一个人的大脑，所有以前经历过的事情 都在这里存着，等需要的时候 ，再放到小脑里面思考。==
>
> - ==主板 就是一个人的躯干，让一个个设备都有一个自己放的地方。==
>
> - 光驱 就是一个书架，可以把知识（光盘）都放到上面去学习，当然也可以没有。
>
> - 鼠标和键盘，输入外面的命令传达方式，把外界的东西，传给大脑。
>
> - 显示器，是一人的脸，所有的喜怒哀乐都可以在上面显示。
>
> - ==数据线，就是一个人的血管，传输 数据（血液）。==
>
> - 电源 就是一个人的食物，通过电源线（食道 小肠）

```
- CPU计算速度 是  内存的  250倍左右
- 内存读取速度 是  硬盘的  10倍左右

因此，当有一个方法-伪代码：
public void test{
   int i =0;
   i++; // 使用CPU
   system.out.println(i);
   Thread.sleep(100_000); // 这一步程序阻塞100秒，代替内存向硬盘写大文件   
                          // 这一步CPU空闲,所以，为了最大化利用CPU, 想办法让CPU去执行别的方法，这引出                           // 多线程
                          // test是一个线程调用的方法， 在写文件期间， CPU切换执行test1方法
0-   system.out.println("CPU又回来执行test方法，这是就需要程序计数器  记录这个位置");
}

public void test1{
   system.out.println("耗时2秒");
}

// 当test1执行结束后，又切回的test方法执行，通过程序计数器知道要在0-位置继续执行
```

## JVM内存区域

![image-20230302231351249](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20230302231351249.png)

- **程序计数器**

　　程序计数器（Program Counter Register），也有称作为PC寄存器。想必学过汇编语言的朋友对程序计数器这个概念并不陌生，在汇编语言中，程序计数器是指CPU中的寄存器，它保存的是程序当前执行的指令的地址（也可以说保存下一条指令的所在存储单元的地址），当CPU需要执行指令时，需要从程序计数器中得到当前需要执行的指令所在存储单元的地址，然后根据得到的地址获取到指令，在得到指令之后，程序计数器便自动加1或者根据转移指针得到下一条指令的地址，如此循环，直至执行完所有的指令。

　　虽然JVM中的程序计数器并不像汇编语言中的程序计数器一样是物理概念上的CPU寄存器，但是JVM中的程序计数器的功能跟汇编语言中的程序计数器的功能在逻辑上是等同的，也就是说是用来指示 执行哪条指令的。

　　由于在JVM中，多线程是通过线程轮流切换来获得CPU执行时间的，因此，在任一具体时刻，一个CPU的内核只会执行一条线程中的指令，因此，为了能够使得每个线程都在线程切换后能够恢复在切换之前的程序执行位置，每个线程都需要有自己独立的程序计数器，并且不能互相被干扰，否则就会影响到程序的正常执行次序。因此，可以这么说，程序计数器是每个线程所私有的。

　　在JVM规范中规定，如果线程执行的是非native方法，则程序计数器中保存的是当前需要执行的指令的地址；如果线程执行的是native方法，则程序计数器中的值是undefined。

　　由于程序计数器中存储的数据所占空间的大小不会随程序的执行而发生改变，因此，对于程序计数器是不会发生内存溢出现象(OutOfMemory)的。

- **Java栈**

Java栈也称作虚拟机栈（Java Vitual Machine Stack），也就是我们常常所说的栈，跟C语言的数据段中的栈类似。事实上，Java栈是Java方法执行的内存模型。为什么这么说呢？下面就来解释一下其中的原因。

　　Java栈中存放的是一个个的栈帧，每个栈帧对应一个被调用的方法，在栈帧中包括局部变量表(Local Variables)、操作数栈(Operand Stack)、指向当前方法所属的类的运行时常量池（运行时常量池的概念在方法区部分会谈到）的引用(Reference to runtime constant pool)、方法返回地址(Return Address)和一些额外的附加信息。当线程执行一个方法时，就会随之创建一个对应的栈帧，并将建立的栈帧压栈。当方法执行完毕之后，便会将栈帧出栈。因此可知，线程当前执行的方法所对应的栈帧必定位于Java栈的顶部。讲到这里，大家就应该会明白为什么 在 使用 递归方法的时候容易导致栈内存溢出的现象了以及为什么栈区的空间不用程序员去管理了（当然在Java中，程序员基本不用关系到内存分配和释放的事情，因为Java有自己的垃圾回收机制），这部分空间的分配和释放都是由系统自动实施的。对于所有的程序设计语言来说，栈这部分空间对程序员来说是不透明的。下图表示了一个Java栈的模型：

![img](https://images0.cnblogs.com/i/288799/201405/291429030562182.jpg)

　　局部变量表，顾名思义，想必不用解释大家应该明白它的作用了吧。就是用来存储方法中的局部变量（包括在方法中声明的非静态变量以及函数形参）。对于基本数据类型的变量，则直接存储它的值，对于引用类型的变量，则存的是指向对象的引用。局部变量表的大小在编译器就可以确定其大小了，因此在程序执行期间局部变量表的大小是不会改变的。

　　操作数栈，想必学过数据结构中的栈的朋友想必对表达式求值问题不会陌生，栈最典型的一个应用就是用来对表达式求值。想想一个线程执行方法的过程中，实际上就是不断执行语句的过程，而归根到底就是进行计算的过程。因此可以这么说，程序中的所有计算过程都是在借助于操作数栈来完成的。

　　指向运行时常量池的引用，因为在方法执行的过程中有可能需要用到类中的常量，所以必须要有一个引用指向运行时常量。

　　方法返回地址，当一个方法执行完毕之后，要返回之前调用它的地方，因此在栈帧中必须保存一个方法返回地址。

　　由于每个线程正在执行的方法可能不同，因此每个线程都会有一个自己的Java栈，互不干扰。

- **本地方法栈**

本地方法栈与Java栈的作用和原理非常相似。区别只不过是Java栈是为执行Java方法服务的，而本地方法栈则是为执行本地方法（Native Method）服务的。在JVM规范中，并没有对本地方发展的具体实现方法以及数据结构作强制规定，虚拟机可以自由实现它。在HotSopt虚拟机中直接就把本地方法栈和Java栈合二为一。

- **堆**

在C语言中，堆这部分空间是唯一一个程序员可以管理的内存区域。程序员可以通过malloc函数和free函数在堆上申请和释放空间。那么在Java中是怎么样的呢？

　　Java中的堆是用来存储对象本身的以及数组（当然，数组引用是存放在Java栈中的）。只不过和C语言中的不同，在Java中，程序员基本不用去关心空间释放的问题，Java的垃圾回收机制会自动进行处理。因此这部分空间也是Java垃圾收集器管理的主要区域。另外，堆是被所有线程共享的，在JVM中只有一个堆。

- **方法区**

方法区在JVM中也是一个非常重要的区域，它与堆一样，是被线程共享的区域。在方法区中，存储了每个类的信息（包括类的名称、方法信息、字段信息）、静态变量、常量以及编译器编译后的代码等。

　　在Class文件中除了类的字段、方法、接口等描述信息外，还有一项信息是常量池，用来存储编译期间生成的字面量和符号引用。

　　在方法区中有一个非常重要的部分就是运行时常量池，它是每一个类或接口的常量池的运行时表示形式，在类和接口被加载到JVM后，对应的运行时常量池就被创建出来。当然并非Class文件常量池中的内容才能进入运行时常量池，在运行期间也可将新的常量放入运行时常量池中，比如String的intern方法。

　　在JVM规范中，没有强制要求方法区必须实现垃圾回收。很多人习惯将方法区称为“永久代”，是因为HotSpot虚拟机以永久代来实现方法区，从而JVM的垃圾收集器可以像管理堆区一样管理这部分区域，从而不需要专门为这部分设计垃圾回收机制。不过自从JDK7之后，Hotspot虚拟机便将运行时常量池从永久代移除了。

## 过滤器 拦截器

### 过滤器Filter

**定义**

对Servlet容器调用Servlet的过程进行拦截,基于函数回调实现

**常见使用场景**

> - 统一设置编码
> - 过滤敏感字符
> - 登录校验
> - URL级别的访问权限控制
> - 数据压缩

**使用方式**

这了展示的是SpringBoot整合过滤器的方式,

**使用配置类的方式(bean注入)**

> - 编写filter类并实现Filter接口,
> - 实现Filter接口中的init,doFilter方法,与destory方法
> - doFilter方法中使用chain.doFilter(request, response);来放行servlet
> - 编写Filter配置类,注解加上@ConFiguration注解
> - 编写方法,返回值类型为FilterRegistrationBean ,
> - 方法内创建FilterRegistrationBean对象,构造器内传入过滤器类的对象,直接new一个就可以,或者让Spring容器自动注入后再使用
> - 调用FilterRegistrationBean 对象的addUrlPatterns方法类设定过滤的请求路径,

过滤器类

```java
@Slf4j//方便使用下面的log.info()方法输出日志
public class LoginCheckFilter implements Filter {

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;
        log.info("过滤器{}正在生效",LoginCheckFilter.class.getName());
        log.info("过滤器{}拦截到了请求{}",LoginCheckFilter.class.getName(),request.getRequestURI());
        filterChain.doFilter(request,response);
        log.info("请求{}已经被放行",request.getRequestURI());
    }

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("过滤器{}被初始化",LoginCheckFilter.class.getName());
        Filter.super.init(filterConfig);
    }

    @Override
    public void destroy() {
        log.info("过滤器{}被销毁",LoginCheckFilter.class.getName());
        Filter.super.destroy();
    }
}
```

配置类

```java
@Configuration
public class FilterConfig {
    @Bean
    public FilterRegistrationBean<LoginCheckFilter> filterRegistrationBean() {
        FilterRegistrationBean<LoginCheckFilter> loginCheckFilter = new FilterRegistrationBean<>();
        loginCheckFilter.setFilter(new LoginCheckFilter());//注册自定义过滤器
        loginCheckFilter.setName("LoginCheckFilter");//过滤器名称
        loginCheckFilter.addUrlPatterns("/*");//过滤所有路径
        loginCheckFilter.setOrder(1);//优先级，最顶级按照数据从小到大来执行过滤器
        return loginCheckFilter;
    }
}
```

**使用注解@**

创建过滤器类,方法与上面创建过滤器类的步骤一样
过滤器类上面添加注解 @WebFilter(urlPatterns = {“拦截路径”}, filterName = “过滤器名称”)
在启动类上添加@ServeltComponentScan注解来扫描servlet组件（包括：@WebFilter和@WebListener）
注意,使用注解方式无法控制多个过滤器的顺序,()默认执行顺序是按照类名字母的排序,

使用@Order注解控制Bean的执行顺序时,必须配合注解一起使用,才会起作用

但是@Component与@WebFilter注解都会把过滤器类加载一遍,并且@component会把@WebFilter加载的Bean覆盖掉,从而使得@WebFilter配置的拦截路径与过滤器名称失效,

所以如果指定了优先级则配置的拦截路径失效,默认为任意路径,如果不使用@Order注解则无法控制多个过滤器的执行顺序,

**总结**

> - Filter是依赖于Servlet的,需要导入Servlet的依赖
> - 过滤器会在容器启动时被初始化,并且只会初始化一次
> - doFilter()方法在目标请求被拦截前执行,放行调用filterChain.doFilter(servletRequest,servletResponse);
> - 体变量名根据方法体而变
> - destroy()方法在容器销毁时执行,只执行一次
> - Filter可以拦截所有请求,包括静态资源,
> - 过滤器基于函数回调实现,

**扩展**

> - 过滤器,监听器和Servlet是JavaWeb三大组件之一,它们的在SpringBoot中的使用方式都差不多区别如下
> - 使用注解方式使用时都需要在启动类加上@ServletComponentScan注解
> - 三个注解为@WebSrvlet,@WebFilter和@Weblistener
> - Servlet类继承的是HttpServlet,Filter实现的是Filter类,Listener实现ServletContextListener类
> - 使用Bean注入方式使用时,Filter组件方法返回类型是FilterRegistrationBean,Listener组件方法返回类型
> - ServletListenerRefistrationBean,Servlet组件返回类型为ServletRefistrationBean

### 拦截器Interceptor

**定义**

类似于Servlet中的过滤器,主要用于拦截用户请求并做相应的处理,基于[java反射机制](https://so.csdn.net/so/search?q=java反射机制&spm=1001.2101.3001.7020)(动态代理)实现

**常见使用场景**

> - 日志记录
> - 权限校验
> - 登录校验
> - 性能检测[检测方法的执行时间]

其实拦截器和过滤器很像，有些使用场景。无论选用谁都能实现。需要注意的使他们彼此的使用范围，触发机制。

**使用方式**

> - 编写拦截器类并实现HandlerInterceptor 接口
> - 实现HandlerInterceptor 接口的preHandle,postHandle和afterHandle方法
> - 编写配置类,注意加上@configuration注解,并继承WebMvcConfigurer接口

**总结**

> - 拦截器依赖于SpringMvc的,需要导入Mvc的依赖
> - preHandle() 在目标请求完成之前执行。有返回值Boolean类型，true：表示放行
> - postHandle() 在目标请求之完成后执行。
> - afterCompletion() 在整个请求完成之后【modelAndView已被渲染执行】。
> - 拦截器只能拦截action请求,不包括静态资源(有待验证)
> - 基于java反射机制实现

### 过滤器与拦截器的区别

> - 触发顺序不一样,
>
> ```
> 1. 过滤器的触发顺序在拦截器前,也就是先执行过滤器,在执行拦截器, 过滤前 - 拦截前 - Action处理 - 拦截后 - 过滤后。
> 2. 过滤器之间的执行顺序嵌套的,
> 3. 但是拦截器执行顺序也是嵌套执行的但是,因为有三个方法,会先执行pre方法,再一起执行post方法,最后一起执行after方法
> 4. 实现方式不同,过滤器基于函数回调实现,拦截器基于动态代理(反射)实现
> ```
>
> - 拦截器是基于java的反射机制的，而过滤器是基于函数回调。
> - 拦截器不依赖与servlet容器，过滤器依赖与servlet容器。
> - 拦截器只能对action请求起作用，而过滤器则可以对几乎所有的请求起作用。
> - 拦截器可以访问action上下文、值栈里的对象，而过滤器不能访问。
> - 在action的生命周期中，拦截器可以多次被调用，而过滤器只能在容器初始化时被调用一次
> - 拦截器可以获取IOC容器中的各个bean，而过滤器就不行，这点很重要，在拦截器里注入一个service，可以调用业务逻辑。

**[aop](https://so.csdn.net/so/search?q=aop&spm=1001.2101.3001.7020)与过滤器，拦截器的区别**

> - 过滤器，拦截器拦截的是URL。AOP拦截的是类的元数据(包、类、方法名、参数等)。
> - 过滤器并没有定义业务用于执行逻辑前、后等，仅仅是请求到达就执行。
> - 拦截器有三个方法，相对于过滤器更加细致，有被拦截逻辑执行前、后等。
> - AOP针对具体的代码，能够实现更加复杂的业务逻辑。
> - 三者功能类似，但各有优势，从过滤器 -> 拦截器 -> 切面，拦截规则越来越细致。
> - 执行顺序依次是过滤器、拦截器、切面。

### HashCode

**Hash表**

hash是一个函数，该函数中的实现就是一种算法，就是通过一系列的算法来得到一个hash值，这个时候，我们就需要知道另一个东西，hash表，通过hash算法得到的hash值就在这张hash表中，也就是说，hash表就是所有的hash值组成的，有很多种hash函数，也就代表着有很多种算法得到hash值，

**HashCode**

hashcode就是通过hash函数得来的，通俗的说，就是通过某一种算法得到的，hashcode就是在hash表中有对应的位置。

hashCode比较的是哈希码，哈希码是由特定的哈希算法的出。

一个对象肯定有物理地址，也有人把hashcode说成是代表对象的地址，这里肯定会让读者形成误区，对象的物理地址跟这个hashcode地址不一样，hashcode代表对象的地址说的是对象在hash表中的位置，物理地址说的对象存放在内存中的地址 。 通过对象的内部地址(也就是物理地址)转换成一个整数，然后该整数通过hash函数的算法就得到了hashcode 。

举个例子，hash表中有 hashcode为1、hashcode为2、(...)3、4、5、6、7、8这样八个位置，有一个对象A，A的物理地址转换为一个整数17(这是假如)，就通过直接取余算法，17%8=1，那么A的hashcode就为1，且A就在hash表中1的位置。

**equals方法和hashcode的关系？**

>  1、如果两个对象equals相等，那么这两个对象的HashCode一定也相同
>
>  2、如果两个对象的HashCode相同，不代表两个对象就相同，只能说明这两个对象在散列存储结构中，
>
> 存放于同一个位置。hashCode()只表示对象的哈希码，哈希码相同的对象不一定相等，反之，没有重写equals方法的前提下，两个对象相等，则hashCode一定相同

**为什么equals方法重写的话，建议也一起重写hashcode方法**

### Cookie、session和token的区别

由于http是无状态的会话，所以我们需要一个东西来记录。目前我们用到的主要有三种：session，cookie 和 token。

- session：

> 在服务器端记录，每一个会话会产生一个sessionId。当用户打开某个web应用时，便与web服务器产生一次session。服务器使用 sessionId 把用户的信息临时保存在了服务器上，用户离开网站后session会被销毁。这样服务器就会根据每个人sessionId的不同，区别开谁是谁了，从而返回给用户不同的请求结果。
>
> 缺点：
>
> 如果使用单个服务器的话，用户过多的话，会造成服务器开销太大。如果我们系统采用分布式的话，我们登录时，响应我们的那台机器会记录我们登录信息，万一下一个请求，响应我们的不是原来那台机器的话，它并没有存储我们之前会话信息，就会认为我们并没有登录。session粘连或者session复制都不是特别好的方案。
>
> 那既然服务端存储这些 SessionId 这么麻烦，人类又想出一招，那就是把这些SessionId 都存储在客户端。这个时候，cookie运用而生

- cookie

> cookie是服务端保存在客户端的临时的少量的数据。cookie由服务器生成，发送给浏览器，浏览器把cookie以kv形式保存到某个目录下的文本文件内，下一次请求同一网站时会把该cookie发送给服务器。由于cookie是存在客户端上的，所以浏览器加入了一些限制确保cookie不会被恶意使用，同时不会占据太多磁盘空间，所以每个域的cookie数量是有限的。
>
> 但是，cookie 这种方式很容易被恶意攻击者入侵，那么又怎么验证客户端发给我的session id 的确是我生成的呢？ 如果不去验证，服务器都不知道他们是不是合法登录的用户， 那些不怀好意的家伙们就可以伪造session id , 为所欲为了。
>
> 这就需要我们用一种加密的方法或者可以说暗号，来验证这个id是否由我自己的服务器之前生成而非恶意攻击者篡改的。

- token

> token的定义：Token是服务端生成的一串字符串，当作客户端进行请求的一个令牌，当第一次登录后，服务器生成一个Token并将此Token返回给客户端，以后客户端只需带上这个Token前来请求数据即可，无需再次带上用户名和密码。

### 反射 

  反射就是把java类中的各种成分映射成一个个的Java对象。
        例如：一个类有：成员变量、方法、构造方法、包等等信息，利用反射技术可以对一个类进行解剖，把一个个组成部分映射成一个个对象。（其实：一个类中这些成员方法、构造方法、在加入类中都有一个类来描述）
        加载的时候：Class对象的由来是将 .class 文件读入内存，并为之创建一个Class对象。

> Java的反射（reflection）机制是指在程序的运行状态中，可以构造任意一个类的对象，可以了解任意一个对象所属的类，可以了解任意一个类的成员变量和方法，可以调用任意一个对象的属性和方法。这种动态获取程序信息以及动态调用对象的功能称为Java语言的反射机制。反射被视为动态语言的关键。
>   更简单点的说就是Java程序在运行时，通过非new的方式创建一个类的反射对象，再对类进行相关操作,例如切面日志

#### 获取类对应的字节码的对象

**①** 调用某个类的对象的getClass()方法，即：对象.getClass()；

```java
Person p = new Person();
Class clazz = p.getClass();
```

**②** 调用类的class属性类获取该类对应的Class对象，即：类名.class

```java
Class clazz = Person.class;
```

**③** 使用Class类中的forName()静态方法（最安全，性能最好）即：Class.forName(“类的全路径”)

```java
Class clazz = Class.forName("类的全路径");
```

#### 常用方法

 当我们获得了想要操作的类的Class对象后，可以通过Class类中的方法获取和查看该类中的方法和属性。

```java
//获取包名、类名
clazz.getPackage().getName()//包名
clazz.getSimpleName()//类名
clazz.getName()//完整类名
 
//获取成员变量定义信息
getFields()//获取所有公开的成员变量,包括继承变量
getDeclaredFields()//获取本类定义的成员变量,包括私有,但不包括继承的变量
getField(变量名)
getDeclaredField(变量名)
 
//获取构造方法定义信息
getConstructor(参数类型列表)//获取公开的构造方法
getConstructors()//获取所有的公开的构造方法
getDeclaredConstructors()//获取所有的构造方法,包括私有
getDeclaredConstructor(int.class,String.class)
 
//获取方法定义信息
getMethods()//获取所有可见的方法,包括继承的方法
getMethod(方法名,参数类型列表)
getDeclaredMethods()//获取本类定义的的方法,包括私有,不包括继承的方法
getDeclaredMethod(方法名,int.class,String.class)
 
//反射新建实例
clazz.newInstance();//执行无参构造创建对象
clazz.newInstance(222,"韦小宝");//执行有参构造创建对象
clazz.getConstructor(int.class,String.class)//获取构造方法
 
//反射调用成员变量
clazz.getDeclaredField(变量名);//获取变量
clazz.setAccessible(true);//使私有成员允许访问
f.set(实例,值);//为指定实例的变量赋值,静态变量,第一参数给null
f.get(实例);//访问指定实例变量的值,静态变量,第一参数给null
 
//反射调用成员方法
Method m = Clazz.getDeclaredMethod(方法名,参数类型列表);
m.setAccessible(true);//使私有方法允许被调用
m.invoke(实例,参数数据);//让指定实例来执行该方法
```



--序列化

--IOC

