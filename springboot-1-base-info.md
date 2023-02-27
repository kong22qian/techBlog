[TOC]

# Springboot 重点

## SpringBoot一个请求的处理全过程

平时只是在用SpringBoot框架，但并没有详细研究过请求执行的一个具体过程，所以本文主要来梳理一下SpringBoot请求处理的全过程。

### 请求处理流程图

**容器包含关系图**

![请添加图片描述](https://img-blog.csdnimg.cn/36d6caf6cf404cdb9de5b0a3f4a2632e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aqR5Liq5bCP6JyX54mb,size_15,color_FFFFFF,t_70,g_se,x_16)

**请求简要流程图**

![请添加图片描述](https://img-blog.csdnimg.cn/8bab755e933a40e7b04d3c4650e06e1f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aqR5Liq5bCP6JyX54mb,size_20,color_FFFFFF,t_70,g_se,x_16)

**请求详细流程图**

![请添加图片描述](https://img-blog.csdnimg.cn/5275e27a5688444db764b48b4ad5d999.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6aqR5Liq5bCP6JyX54mb,size_20,color_FFFFFF,t_70,g_se,x_16)

## 请求处理流程详解

### 请求处理主要流程

> 过滤器链chain.doFilter之前的执行（过滤器是在Server容器中的，如Tomcat）；
> 拦截器链preHandle方法执行；
> 路径映射、参数绑定（参数解析、参数转换、参数校验）；
> Controller的具体方法执行；
> 返回值处理（含信息转换）；
> 拦截器链postHandle方法执行；
> 异常解析器处理异常（@ControllerAdvice、自定义异常解析器都在这里执行）；
> 视图解析渲染；
> 拦截器链afterCompletion方法执行；
> 过滤器链chain.doFilter之后的执行。

### 请求处理详细流程

1. Tomcat线程接受到请求，经过一系列调用后，调用到ApplicationFilterChain的doFilter方法。doFilter方法调用ApplicationFilterChain的internalDoFilter方法，依次执行过滤器链的每个Filter的doFilter。

2. 过滤器链的所有doFIlter执行完毕， 控制权交回ApplicationFilterChain， 经过一系列调用后，调用到DispatcherServlet的doDispatch方法。

​      ==**doDispatch方法的主要流程：**==

​              2.1 DispatcherServlet的getHandler方法：得到处理执行器链（包含处理器和拦截器链）。

​              2.2 DispatcherServlet的getHandlerAdapter方法：得到处理器适配器。

​              2.3 HandlerExecutionChain的applyPreHandle方法：执行执行器链中的所有拦截器方法preHandle。

​              2.4 AbstractHandlerMethodAdapter的handle方法：该方法主要包含路径映射、参数bangd (参数解析、参数转换、参数校验)、调用具体控制器方法、返回值处理（含信息转换）等操作。

​              ==**handle方法的主要流程:**==

​                        2.4.1 调用RequestMappingHandlerAdapter的handleInternal方法，handleInternal方法又调用RequestMappingHandlerAdapter的invokeHandlerMethod方法。

​                         ==**invokeHandlerMethod方法的主要流程：**==

​                                   2.4.1.1 调用RequestMappingHandlerAdapter的createInvocableHandlerMethod方法：注册参数解析器、返回值处理器、信息转化器等到ServletInvocableHandlerMethod 对象实例中。

​                                    2.4.1.2 调用ServletInvocableHandlerMethod的invokeAndHandle方法。

​                                    ==**invokeAndHandle方法的主要流程：**==

​                                              2.4.1.2.1 调用InvocableHandlerMethod的invokeForRequest方法，invokeForRequest方法又调用InvocableHandlerMethod的doInvoke方法。

​                                              ==**doInvoke方法的主要流程：**==

​                                                     2.4.1.2.1.1 调用InvocableHandlerMethod的getMethodArgumentValues方法：路径映射、参数绑定(参数解析、参数转换、参数校验)。
​                                                     2.4.1.2.1.2  调用Method的invoke方法，内部调用DelegatingMethodAccessorImpl的invoke方法，内部调用InvocableHandlerMethod的doInvoke方法，内部调用NativeMethodAccessorImpl的invoke方法，内部调用NativeMethodAccessorImpl的invoke0方法，内部调用具体Controller的具体方法，得到响应结果。
​                                               2.4.1.2.2 调用HandlerMethodReturnValueHandlerComposite的handleReturnValue方法：返回值处理（含信息转换）。

​              2.5 调用HandlerExecutionChain的applyPostHandle方法：执行执行器链中的所有拦截方法postHandle。

​              2.6 调用DispatcherServlet的processDispatchResult方法。

​                    ==**processDispatchResult方法的主要流程：**==

​                      2.6.1 调用DispatcherServlet的processHandlerException方法：异常处理（获取合适的异常解析器处理异常信息，@ControllerAdvice全局异常处理和自定义异常解析器都是在这一步执行的）。
​                       2.6.2 调用DispatcherServlet的render方法：视图解析渲染。
​                       2.6.3 调用HandlerExecutionChain的triggerAfterCompletion方法：执行执行器链中的所有拦截方法afterCompletion。

3. 控制权交回ApplicationFilterChain ， 继续执行过滤器链的所有doFIlter之后的代码。

```
https://blog.csdn.net/JokerLJG/article/details/123247460
```

## Spring的 IOC和[AOP](https://so.csdn.net/so/search?q=AOP&spm=1001.2101.3001.7020)机制 ？

（1）我们是在使用 Spring框架的过程中，其实就是为了使用 IOC（控制反转）、依赖注入（DI与IOC一样）和AOP（面向切面编程），这两个也是 Spring 的灵魂。

（2）主要用到的设计模式有工厂模式和代理模式

==**IOC就是典型的工厂模式，AOP就是典型的代理模式的体现。**==

      代理模式是常用的Java设计模式，他的特征是代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。

（3）Spring的 IoC容器是 Spring的核心，Spring AOP是 Spring框架的重要组成部分。

### **IoC：控制反转:**

      在传统的程序设计中，当调用者需要被调用者的协助时，通常由调用者来创建被调用者的实例。但在 Spring里创建被调用者的工作不再由调用者来完成，因此控制反转（IoC）；创建被调用者实例的工作通常由 Spring容器来完成，然后注入调用者，因此也被称为依赖注入（DI），依赖注入和控制反转是同一个概念。
    
      面向切面编程（AOP）是以另一个角度来考虑程序结构，通过分析程序结构的关注点来完善面向对象编程（OOP）。OOP将应用程序分解成各个层次的对象，而AOP将程序分解成多个切面（切面就是要添加的非核心功能）。Spring AOP 只实现了方法级别的连接点，在J2EE应用中，AOP拦截到方法级别的操作就已经足够。在 Spring中，未来使 IoC方便地使用健壮、灵活的企业服务，需要利用 Spring AOP实现为IoC和企业服务之间建立联系。
    
      IOC控制反转也叫依赖注入。利用了工厂模式将对象交给容器管理，你只需要在 Spring 总配置文件中配置相应的bean，以及设置相关的属性，让 Spring容器来生成类的实例对象以及管理对象。在 Spring容器启动的时候，Spring会把你在配置文件中配置的 bean都初始化好（这里就会涉及到复杂的 bean 创建的生命周期），然后在你需要调用的时候，就把它已经初始化好的那些 bean 分配给你需要调用这些 bean的类（假设这个类名是A），分配的方法就是调用 A 的 setter方法来注入，而不需要你在A里面 new 这些 bean了。
### **AOP：面向切面编程（Aspect-Oriented Programming）:**

      AOP可以说是对OOP的补充和完善。OOP引入封装、继承和多态性等概念来建立一种对象层次结构，用以模拟公共行为的一个集合。当我们需要为分散的对象引入公共行为的时候，OOP则显得无能为力。也就是说，OOP允许你定义从上到下的关系，但并不适合定义从左到右的关系。例如日志功能、事务管理、权限认证、异常处理等等吧。日志代码往往水平地散布在所有对象层次中，而与它所散布到的对象的核心功能毫无关系。在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。将程序中的交叉业务逻辑（比如安全，日志，事务等），封装成一个切面，然后注入到目标对象（具体业务逻辑）中去。
    
      实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。
简单点解释，比方说你想在你的 service 层所有类中都加上一个打印"你好"的功能,这时就可以用 AOP 思想来做。你先写个类及其方法，方法实现打印"你好"，然后将这个类都注入到每一个需要实现打印的类中即可实现。

-------------------------------------------------------------------------------------------------------------

==**问：控制反转和依赖注入式同一个概念吗？**==

控制反转和依赖注入是对同一件事情的不同描述，从某个方面讲，就是它们描述的角度不同。

>  依赖注入是从应用程序的角度在描述，可以把依赖注入描述完整点：应用程序依赖容器创建并注入它所需要的外部资源；
>
>   而控制反转是从容器的角度在描述，描述完整点：容器控制应用程序，由容器反向的向应用程序注入应用程序所需要的外部资源。

==**问：bean 和 Java对象的区别：**==

> ① bean 是经历过完整的 bean 生命周期生成的放在单例池（singletonObjects)中的对象（大部分bean 是这样的）是有 Spring 容器创建出来的；
>
> ② bean 是一个Java对象，而Java对象并不一定是 bean（这一点并没有官方做支撑，个人理解）。
>
> ③ bean 创建好之后它的属性就是赋完值的，也就是 bean 是属性不是默认值的一个对象，而 new 出来的对象的属性是默认值。

-------------------------------------------------------------------------------------------------------------------------------------------------

## Spring中 `@Autowired` 和 `@Resource` 注解的区别？

@Resource和@Autowired都是做bean的注入时使用，其实@Resource并不是 Spring 的注解，它的包是javax.annotation.Resource，需要导入，但是Spring支持该注解的注入。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200804133924707.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMDEyNzky,size_16,color_FFFFFF,t_70)

**（1）共同点**

   两者都可以写在**字段和setter方法**上。两者如果都写在字段上，那么就不需要再写setter方法。

**（2）不同点**

（1）@Autowired
     @Autowired为Spring提供的注解，需要导入包org.springframework.beans.factory.annotation.Autowired；只按照 byType（类型）注入。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200912195056801.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMDEyNzky,size_16,color_FFFFFF,t_70#pic_center)

 @Autowired注解是按照类型（byType）装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它的 `required` 属性为 false。如果我们想使用按照名称（byName）来装配，可以结合@Qualifier注解一起使用。如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020091220285336.png#pic_center)

（2）@Resource
      @Resource默认按照 byName 自动注入，由J2EE提供，需要导入包javax.annotation.Resource。

      @Resource有两个重要的属性：name和 type，而Spring将@Resource注解的name属性解析为bean的名字，而type属性则解析为bean的类型。所以，如果使用name属性，则使用byName的自动注入策略，而使用type属性时则使用byType自动注入策略。如果既不制定name也不制定type属性，这时将通过反射机制使用byName自动注入策略。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200912203323943.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMDEyNzky,size_16,color_FFFFFF,t_70#pic_center)

注：最好是将@Resource放在 `setter` 方法上，因为这样更符合面向对象的思想，通过set、get去操作属性，而不是直接去操作属性。

@Resource装配顺序：

> ①如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常。
>
> ②如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常。
>
> ③如果指定了type，则从上下文中找到类似匹配的唯一bean进行装配，找不到或是找到多个，都会抛出异常。
>
> ④如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配。

@Resource的作用相当于@Autowired，只不过@Autowired按照byType自动注入。

## 依赖注入的方式有几种，各是什么?

- 构造器注入

      将被依赖对象通过构造函数的参数注入给依赖对象，并且在初始化对象的时候注入。
      
      优点：对象初始化完成后便可获得可使用的对象。
      缺点：
          ① 当需要注入的对象很多时，构造器参数列表将会很长；
          ② 不够灵活。若有多种注入方式，每种方式只需注入指定几个依赖，那么就需要提供多个重载的构造函数，麻烦。

- setter方法注入

      IoC Service Provider 通过调用成员变量提供的setter函数将被依赖对象注入给依赖类。
      
      优点：灵活。可以选择性地注入需要的对象。
      缺点：依赖对象初始化完成后由于尚未注入被依赖对象，因此还不能使用。

- 接口注入

      依赖类必须要实现指定的接口，然后实现该接口中的一个函数，该函数就是用于依赖注入。该函数的参数就是要注入的对象。
      
      优点：接口注入中，接口的名字、函数的名字都不重要，只要保证函数的参数是要注入的对象类型即可。
      缺点：侵入性太强，不建议使用。

## 讲一下什么是 Spring ？

 Spring是一个轻量级的IoC和AOP容器框架。是为Java应用程序提供基础性服务的一套框架，目的是用于简化企业应用程序的开发，它使得开发者只需要关心业务需求。

**常见的配置方式有三种：**
基于XML的配置、基于注解的配置、基于Java的配置（接口）。

那么怎么理解Spring的轻量级？

      我觉得首先我们会发现 Spring 是一个体积非常小的，核心包才 1 M左右；另外最重要的是 Spring 在启动时是不需要启动它的全部服务的。

主要由以下几个模块组成：

> Spring Core：核心类库，提供IOC服务；
> Spring Context：提供框架式的Bean访问方式，以及企业级功能（JNDI、定时任务等）；
> Spring AOP：AOP服务；
> Spring DAO：对JDBC的抽象，简化了数据访问异常的处理；
> Spring ORM：对现有的ORM框架的支持；
> Spring Web：提供了基本的面向Web的综合特性，例如多方文件上传；
> Spring MVC：提供面向Web应用的Model-View-Controller实现。

## Spring的AOP理解：

前置知识：

```
https://blog.csdn.net/qq_43012792/article/details/107729104
```

1、Spring的事务支持是由AOP（面向切面编程）这一思想做支撑，然而面向切面编程底层的落地实现是Spring工厂的动态代理，而Spring底层又对JDK或cgLib这两种实现动态代理的方式做了封装，Spring可以利用AOP思想实现事务、日志、性能监控。

代理的作用或好处：
作用：为原始对象增加额外功能；
好处：解耦合、便于原始对象的维护。

2、什么是代理：代理就相当于租房中介，他为房东提供发放广告和提供看房的额外功能。

3、什么是代理类：代理类就是为原始类添加额外功能的类，这个类是由JVM通过动态字节码技术生成的。

4、什么是动态字节码技术：是利用类加载器动态的在JVM中，由原始类和额外功能的字节码自动生成代理类的字节码，从而使用类加载器在堆空间中创建Class对象。

5、代理实现的三要素：①提供原始对象(为其增加额外功能)；②额外功能；③要求代理类要与原始类拥有相同的方法（可以是实现其接口，也可以继承）

6、JDK提供动态代理：是利用Proxy类中的一个静态方法newProxyInstance创建代理类对象

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210716135858859.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMDEyNzky,size_16,color_FFFFFF,t_70)

```java
    // 1、创建原始对象
        UserServiceImpl userService = new UserServiceImpl();
        // 2、提供额外功能
        InvocationHandler handler = new InvocationHandler() {
             /** @param proxy 代理实例，一般不会用到
             *   @param method 原始方法
             *   @param args 原始方法的参数
             *   @return 原始对象返回值   */
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) {
                System.out.println("在原始方法执行之前执行的逻辑 -- 额外功能"); // 2.1 这其实就是Spring提供的前置通知-MethodBeforeAdvice
                Object invoke = null;
                try {
                    invoke = method.invoke(userService, args); // 调用原始方法，执行原始方法逻辑
                } catch (Exception e) {
                    System.out.println("在原始方法执行报错后执行的逻辑 -- 额外功能"); // 2.2 这其实就是Spring提供的异常通知-ThrowsAdvice
                }
                System.out.println("在原始方法执行之后执行的逻辑 -- 额外功能"); // 2.3 这其实就是Spring提供的后置通知-AfterReturningAdvice
                return invoke; //
            }
        };
        // 创建代理对象
        UserServiceImpl userServiceProxy = (UserServiceImpl)Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(),
                                                                                   userServiceImpl.getClass().getInterfaces(),
                                                                                   handler);
        userServiceProxy.save(new User()); // 使用代理对象调用原始类中的方法，在执行此方法时会按照代理对象的逻辑执行额外功能

```

```java
@CallerSensitive
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
throws IllegalArgumentException

```

① loader类加载器：利用该类加载器去生成字节码，创建Class对象；
② interfaces接口集合：这就是为了使生成的代理类和原始类的方法一致；
③ InvocationHandler额外功能接口的接口：额外功能的逻辑
利用这三个参数就可以创建代理类对象，cgLib的实现与以上步骤高度吻合，区别就是cgLib是使用继承原始类的方式去让代理类与原始类方法相同的。

**Spring对JDK动态代理的封装:**

Spring就是利用JDK的动态代理来实现AOP思想的，它封装之后提供最重要的接口叫`MethodInterceptor`.

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210716140125904.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQzMDEyNzky,size_16,color_FFFFFF,t_70)

```java
public class proxyTest implements MethodInterceptor {
    /**
     * @param methodInvocation 原始对象
     * @return 原始对象返回值
     * @throws Throwable 异常处理
     */
    @Override
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        System.out.println("之前");
        Object proceed = methodInvocation.proceed(); // 调用原始方法，执行原始方法逻辑
        System.out.println("之后");
        return proceed;
    }
}    

```

你会发现，Spring提供的这个MethodInterceptor 接口的实现方法invoke和代理newProxyInstance方法高度吻合，Spring就是把这个方法给封装了，但是最底层还是完全一样的。
      OOP面向对象，允许开发者定义纵向的关系，但并适用于定义横向的关系，导致了大量代码的重复，而不利于各个模块的重用。

      AOP，一般称为面向切面，作为面向对象的一种补充，用于将那些与业务无关，但却对多个对象产生影响的公共行为和逻辑，抽取并封装为一个可重用的模块，这个模块被命名为"切面"（Aspect），减少系统中的重复代码，降低了模块间的耦合度，同时提高了系统的可维护性。可用于权限认证、日志、事务处理。
    
      AOP实现的关键在于代理模式，AOP代理主要分为静态代理和动态代理。静态代理的代表为AspectJ，动态代理则以Spring AOP为代表。

（1）AspectJ是静态代理的增强，所谓静态代理，就是AOP框架会在编译阶段生成AOP代理类，因此也称为编译时增强，他会在编译阶段将AspectJ(切面)织入到 Java字节码中，运行的时候就是增强之后的AOP对象。

（2）Spring AOP使用的动态代理，所谓的动态代理就是说AOP框架不会去修改字节码，而是每次运行时在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

**Spring AOP中的动态代理主要有两种方式，JDK动态代理和 cglib 动态代理：**

①JDK动态代理只提供接口的代理，不支持类的代理。核心InvocationHandler接口和Proxy类，InvocationHandler 通过invoke()方法反射来调用目标类中的代码，动态地将横切逻辑和业务编织在一起；接着，Proxy利用InvocationHandler动态创建一个符合某一接口的的实例, 生成目标类的代理对象。

②如果代理类没有实现 InvocationHandler 接口，那么Spring AOP会选择使用CGLIB来动态代理目标类。cglib（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。cglib 是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用 cglib 做动态代理的。

（3）静态代理与动态代理区别在于生成AOP代理对象的时机不同，相对来说AspectJ的静态代理方式具有更好的性能，但是AspectJ需要特定的编译器进行处理，而Spring AOP则无需特定的编译器处理。

## Spring的IOC理解

（1）IOC就是控制反转，是指创建对象的控制权的转移，以前创建对象的主动权和时机是由自己把控的，而现在这种权力转移到Spring容器中，并由容器根据配置文件去创建实例和管理各个实例之间的依赖关系，对象与对象之间松散耦合，也利于功能的复用（核心功能代码更加"干净"）。
DI依赖注入，和控制反转是同一个概念的不同角度的描述，即应用程序在运行时依赖IoC容器来动态注入对象需要的外部资源。

（2）最直观的表达就是，IOC让对象的创建不用去new了，可以由 Spring自动生产，使用 Java的反射机制，根据配置文件在运行时动态的去创建对象以及管理对象，并调用对象的方法的。

（3）Spring的IOC有三种注入方式 ：构造器注入、setter方法注入、接口注入。IoC让相互协作的组件保持松散的耦合，而AOP编程允许你把遍布于应用各层的功能分离出来形成可重用的功能组件。

## 解释一下 Spring bean的生命周期

首先说一下Servlet的生命周期：实例化，初始init，接收请求service，销毁destroy；

      对于整个Spring Framework 体系而言，Spring 的 bean 它是由 BeanDefinition 来的，BeanDefinition 是 Spring中一个建模的类。在Spring 容器启动时它会去根据配置文件的要求做一个扫描，把Java类扫描成一个 BeanDefinition，存到 BeanDefinitionMap 中，之后再去对 BeanDefinitionMap 做一个遍历，遍历完了之后就会做一个验证，比如说：是否单例、是否原型、是否懒加载、是否有DeponsOn、是否抽象、是否FactoryBean、是否这个Bean 的名字符合规则等等验证。再之后它会去调用doGetBean 方法判断是否在单例池（singletonObjects）当中已经存在该正在创建的这个 Spring Bean ，有就直接使用。再看是否有没有被提前暴露。假如没有提前暴露就会继续执行一套完整的 Spring Bean 生命周期。

Spring的 bean生命周期，如下：

（1）推断构造方法：
      首先一个 Spring bean 对象是对应一个 Java 类的，而一个Java 类中又有很多个构造方法，Spring 在创建 bean 时先需要推断出一个最佳的构造方法，以此构造方法来进行实例化；

（2）实例化Java对象：
      对于BeanFactory容器，当客户向容器请求一个尚未初始化的 Java 对象时，或初始化这个Java对象的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean方法进行实例化。对于ApplicationContext容器，当容器启动结束后，通过获取 BeanDefinition 对象中的信息，通过Java 反射机制实例化这个Java 对象（原型 bean）。

（3）进行初始化：
      看是否需要做一个 Definition 的合并；验证这个 Spring 容器是否支持循环依赖（循环依赖：A类中需要注入B类对应的 bean，B类中也需要注入A类对应的 bean），注意的是单例池都是支持循环依赖的。假如支持循环依赖，那么 Spring 就会把 ObjectFactory工厂对象 进行提前暴露（暴露：就是将这个工厂对象放在一个二级缓存这个 map 中，供循环依赖时创建另一个 bean 时使用）。

那么问题来了为什么不直接先暴露一个 Spring Bean 呢？
答案大概是如果提前就暴露 Spring Bean ，那么就不便于我们对 Bean 进行一个扩展，要是先暴露一个 ObjectFactory 这个工厂对象，后期就会做一个很好的扩展了。

（4）设置对象属性（依赖注入）：
实例化后的对象被封装在 BeanWrapper对象中，紧接着，Spring根据BeanDefinition中的信息，以及通过BeanWrapper提供的设置属性的接口完成依赖注入。

（5）处理Aware接口：
接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给 bean，还有进行回调。

①如果这个 bean已经实现了BeanNameAware 接口，会调用它实现的setBeanName(String beanId)方法，此处传递的就是Spring配置文件中Bean的 id 值；

②如果这个Bean已经实现了 BeanFactoryAware 接口，会调用它实现的setBeanFactory() 方法，传递的是Spring工厂自身；

③如果这个Bean已经实现了 ApplicationContextAware 接口，会调用setApplicationContext(ApplicationContext) 方法，传入Spring上下文，说白了就是IOC容器；

④ 如果这个Bean已经实现了 ClassLoaderAware 接口，会调用setClassLoader() 方法.

（6）BeanPostProcessor：后置处理器
      如果想对Bean进行一些自定义的处理，那么可以让Bean实现了BeanPostProcessor接口，那将会调用postProcessBeforeInitialization(Object obj, String s)方法。

（7）InitializingBean 与 init-method：
如果Bean在Spring配置文件中配置了 init-method属性或者Bean实现了InitializingBean接口，并重写了里面的afterPropertiesSet方法，则会自动调用，进行对应的初始化方法。

（8）如果这个Bean实现了BeanPostProcessor接口，将会调用postProcessAfterInitialization(Object obj, String s)方法；由于这个方法是在Bean初始化结束时调用的，所以可以被应用于内存或缓存技术；

以上几个步骤完成后，Bean就已经被正确创建了，之后就会把这个 bean 放到容器中就可以使用这个 Bean 了，调用方法 getBean()。

（9）DisposableBean：
当Bean不再需要时，会经过清理阶段，如果Bean实现了DisposableBean这个接口，会调用其实现的destroy()方法；

（10）destroy-method：
最后，如果这个Bean的Spring配置中配置了destroy-method属性，会自动调用其配置的销毁方法。

##  解释Spring支持的几种bean的作用域

Spring容器中的bean可以分为5个范围：

> （1）singleton：默认，每个容器中只有一个bean的实例，单例的模式由BeanFactory自身来维护。
> （2）prototype：为每一个bean请求提供一个实例。
> （3）request：为每一个网络请求创建一个实例，在请求完成以后，bean会失效并被垃圾回收器回收。
> （4）session：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效。
> （5）global-session：全局作用域，global-session和Portlet应用相关。当你的应用部署在Portlet容器中工作时，它包含很多portlet。如果你想要声明让所有的portlet共用全局的存储变量的话，那么这全局变量需要存储在global-session中。全局作用域与Servlet中的session作用域效果相同。

## Spring基于xml注入bean的几种方式：

> （1）Set方法注入；
> （2）构造器注入：
> 　　①通过index设置参数的位置；②通过type设置参数类型；
> （3）静态工厂注入；
> （4）实例工厂注入。

## Spring框架中都用到了哪些设计模式？

> （1）工厂模式：BeanFactory就是简单工厂模式的体现，用来创建对象的实例；
> （2）单例模式：Bean默认为单例模式；
> （3）代理模式：Spring的AOP功能用到了JDK的动态代理和CGLIB字节码生成技术；
> （4）模板方法：用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate；
> （5）观察者模式：定义的对象间存在一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都会得到通知被制动更新，如 ApplicationContext 事件机制是观察者设计模式的实现，通过 ApplicationEvent 类和 ApplicationListener接口，可以实现ApplicationContext事件处理。