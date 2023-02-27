[TOC]

# Java8(JDK1.8)新特性

## Java8(JDK1.8)新特性

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
>
> 8、其他特性

## java8（JDK1.8）新特性简介

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

## java8（JDK1.8）新特性详细介绍

### Lambda表达式

#### Lambda表达式是什么？

Lambda是一个匿名函数，我们可以将Lambda表达式理解为一段可以传递的代码（将代码像数据一样传递）。使用它可以写出简洁、灵活的代码。作为一种更紧凑的代码风格，使java语言表达能力得到提升。

#### 从匿名类到Lambda转换

```java
package com.chen.test.JAVA8Features;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Demo01 {
    private static Logger log = LoggerFactory.getLogger(Demo01.class);

    public static void main(String[] args) {
        Runnable t1 =new Runnable(){
            @Override
            public void run(){
                log.info("我是没有使用Lambda表达式：不简洁");
            }
        };
        
        Runnable t2 = () -> log.info("我是使用Lambda表达式：简洁、灵活");
        
        t1.run();
        t2.run();
        
    }
}
```

Run result

```java
19:43:39.303 [main] INFO com.chen.test.JAVA8Features.Demo01 - 我是没有使用Lambda表达式：不简洁、代码多
19:43:39.303 [main] INFO com.chen.test.JAVA8Features.Demo01 - 我是使用Lambda表达式：简洁、灵活
​
Process finished with exit code 0
​
```

#### Lambda表达式语法

Lambda表达式在java语言中引入了一种新的语法元素和操作。这种操作符号为“->”,Lambda操作符或箭头操作符，它将Lambda表达式分割为两部分。 左边：指Lambda表达式的所有参数 右边：指Lambda体，即表示Lambda表达式需要执行的功能。

六种语法格式：

1、语法格式一：无参数、无返回值，只需要一个Lambda体

```java
package com.chen.test.JAVA8Features;
​
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
​
public class Demo02 {
    private static Logger log = LoggerFactory.getLogger(Demo02.class);
​
    public static void main(String[] args) {
        Runnable t1 = ()-> log.info("Lambda表达式：简洁、灵活，优雅永不过时");
        t1.run();
    }
}
```

run result

```
22:22:39.125 [main] INFO com.chen.test.JAVA8Features.Demo02 - Lambda表达式：简洁、灵活，优雅永不过时
​
Process finished with exit code 0
```

2、语法格式二：lambda有一个参数、无返回值

```java
package com.chen.test.JAVA8Features;
​
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
​
import java.util.function.Consumer;
​
public class Demo03 {
    private static Logger log = LoggerFactory.getLogger(Demo03.class);
    public static void main(String[] args) {
        Consumer<String> consumer = new Consumer<String>() {
            @Override
            public void accept(String s) {
                log.info(s);
            }
        };
        consumer.accept("爱与被爱的区别");
​
        Consumer<String> consumer1 = (s) -> log.info(s);
        consumer1.accept("接受爱不一定爱对方，爱一定付出真心爱");
    }
}
​
```

run result

```java
23:03:08.992 [main] INFO com.chen.test.JAVA8Features.Demo03 - 爱与被爱的区别
23:03:09.142 [main] INFO com.chen.test.JAVA8Features.Demo03 - 接受爱不一定爱对方，爱一定付出真心爱
​
Process finished with exit code 0
```

3、语法格式三：Lambda只有一个参数时，可以省略（）

```java
package com.chen.test.JAVA8Features;
​
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
​
import java.util.function.Consumer;
​
public class Demo04 {
    private static Logger log = LoggerFactory.getLogger(Demo04.class);
    public static void main(String[] args) {
        Consumer<String> consumer = s -> log.info(s);
        consumer.accept("Lambda只有一个参数时，可以省略（）");
    }
}
```

run result

```java
23:08:27.295 [main] INFO com.chen.test.JAVA8Features.Demo04 - Lambda只有一个参数时，可以省略（）
​
Process finished with exit code 0
```

4、语法格式四：Lambda有两个参数时，并且有返回值

```java
package com.chen.test.JAVA8Features;
​
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
​
import java.util.Comparator;
​
​
public class Demo05 {
    private static Logger log = LoggerFactory.getLogger(Demo05.class);
​
    public static void main(String[] args) {
        CompareOldMethod(12,10);
        findMaxValue(12,10);
        findMinValue(12,10);
    }
//    没有使用Lambda表达式比较大小
    public static void CompareOldMethod(int num1,int num2){
        Comparator<Integer> comparator = new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                log.info("o1:{}",o1);
                log.info("o2:{}",o2);
                return o1 < o2 ? o2 : o1;
            }
        };
        log.info("OldFindMaxValue:{}",comparator.compare(num1,num2));
    }
​
//    使用lambda表达式
    public static void findMaxValue(int num1,int num2){
        Comparator<Integer> comparatorMax = (o1, o2) ->{
​
            log.info("o1:{}",o1);
            log.info("o2:{}",o2);
            return (o1<o2)? o2 :(o1);
        };
​
        log.info("findMaxValue:{}",(comparatorMax.compare(num1,num2)));
​
    }
    public static void findMinValue(int num1,int num2){
        Comparator<Integer> comparatorMin =  (o1, o2) -> {
            log.info("o1:{}",o1);
            log.info("o2:{}",o2);
            return (o1 < o2) ? o1 : o2;
        };
        log.info("FindMinValue:{}",comparatorMin.compare(num1,num2));
    }
}
​
```

run result

```java
00:17:10.206 [main] INFO com.chen.test.JAVA8Features.Demo05 - o1:12
00:17:10.206 [main] INFO com.chen.test.JAVA8Features.Demo05 - o2:10
00:17:10.206 [main] INFO com.chen.test.JAVA8Features.Demo05 - OldFindMaxValue:12
00:17:10.315 [main] INFO com.chen.test.JAVA8Features.Demo05 - o1:12
00:17:10.315 [main] INFO com.chen.test.JAVA8Features.Demo05 - o2:10
00:17:10.315 [main] INFO com.chen.test.JAVA8Features.Demo05 - findMaxValue:12
00:17:10.315 [main] INFO com.chen.test.JAVA8Features.Demo05 - o1:12
00:17:10.315 [main] INFO com.chen.test.JAVA8Features.Demo05 - o2:10
00:17:10.315 [main] INFO com.chen.test.JAVA8Features.Demo05 - FindMinValue:10
​
Process finished with exit code 0
​
```

5、语法格式五：当Lambda体只有一条语句的时候，return和{}可以省略掉

```java
package com.chen.test.JAVA8Features;
​
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
​
import java.util.Comparator;
​
​
public class Demo05 {
    private static Logger log = LoggerFactory.getLogger(Demo05.class);
​
    public static void main(String[] args) {
        findMaxValue(12,10);
        findMinValue(12,10);
    }
​
//    使用lambda表达式
    public static void findMaxValue(int num1,int num2){
        Comparator<Integer> comparatorMax = (o1, o2) ->{
​
            log.info("o1:{}",o1);
            log.info("o2:{}",o2);
            return (o1<o2)? o2 :(o1);
        };
​
        log.info("findMaxValue:{}",(comparatorMax.compare(num1,num2)));
​
    }
    public static void findMinValue(int num1,int num2){
        Comparator<Integer> comparatorMin =  (o1, o2) -> (o1 < o2) ? o1 : o2;
​
        log.info("FindMinValue:{}",comparatorMin.compare(num1,num2));
    }
}
​
```

run result

```java
00:22:31.059 [main] INFO com.chen.test.JAVA8Features.Demo05 - o1:12
00:22:31.075 [main] INFO com.chen.test.JAVA8Features.Demo05 - o2:10
00:22:31.075 [main] INFO com.chen.test.JAVA8Features.Demo05 - findMaxValue:12
00:22:31.075 [main] INFO com.chen.test.JAVA8Features.Demo05 - FindMinValue:10
​
Process finished with exit code 0
```

6、语法格式六：类型推断：数据类型可以省略，因为编译器可以推断得出，成为“类型推断”

```java
package com.chen.test.JAVA8Features;
​
import com.mysql.cj.callback.MysqlCallbackHandler;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
​
import java.util.ArrayList;
import java.util.Comparator;
import java.util.List;
import java.util.function.Consumer;
​
​
public class Demo07 {
    private static Logger log = LoggerFactory.getLogger(Demo07.class);
​
    public static void main(String[] args) {
        dateType();
    }
​
    public static void dateType(){
        Consumer<String> consumer = (String s) -> log.info(s);
        consumer.accept("Hello World !");
​
        Consumer<String> consumer1 = (s) -> log.info(s);
        consumer1.accept("Hello don't date type !");
    }
}
```

## 函数式接口

### 什么是函数式接口？

函数式接口：只包含一个抽象方法的接口，称为函数式接口，并且可以使用lambda表达式来创建该接口的对象，可以在任意函数式接口上使用@FunctionalInterface注解，来检测它是否是符合函数式接口。同时javac也会包含一条声明，说明这个接口是否符合函数式接口。

### 自定义函数式接口

```java
package com.chen.test.JAVA8Features;
​
@FunctionalInterface
public interface FunctionDemo1 {
    public void fun();
}
```

### 泛型函数式接口

```java
package com.chen.test.JAVA8Features;
​
@FunctionalInterface
public interface FunctionGeneric<T> {
    public void fun(T t);
​
}
```

### java内置函数式接口

==**(Function、Consumer、Supplier、Predicate) java.util.function**==

**Function (函数型接口)**

函数型接口：有输入参数，也有返回值。

```java
 * @param <T> the type of the input to the function
 * @param <R> the type of the result of the function
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Function<T, R> {
​
    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);
```

其中T表示输入参数，R为返回值

代码展示：

```java
    public void functionTest(){
//        Function function = new Function<String,String>(){
//            @Override
//            public String apply(String s) {
//                return s;
//            }
//        };
//        log.info("函数型接口 :{}",function.apply("没有使用Lambda表达式"));
​
        Function function = s -> s;
        log.info("函数型接口：{}",function.apply("Function Demo"));
    }
```

**Consumer（消费型接口）**

消费型接口：有入参，没有会有返回值

```java
* @param <T> the type of the input to the operation
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Consumer<T> {
​
    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);
```

代码展示：

```java
 public void consumerTest(){
//        非Lambda表达式
//        Consumer<String> consumer = new Consumer<String>() {
//            @Override
//            public void accept(String s) {
//                log.info(s);
//            }
//        };
//        consumer.accept("消费型函数：没有使用Lambda表达式");
​
//        使用Lambda表达式
        Consumer<String> consumer = s -> log.info(s);
        consumer.accept("消费型函数：Consumer Demo");
​
    }
```

**Supplier（供给型接口）**

供给型接口：没有输入参数，有返回值

```java
 *
 * @param <T> the type of results supplied by this supplier
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Supplier<T> {
​
    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

代码展示：

```java
  public void supplierTest(){
//        非Lambda表达式
//        Supplier supplier = new Supplier<String>(){
//            @Override
//            public String get() {
//                return "供给型接口：没有使用Lambda表达式";
//            }
//        };
//        log.info(String.valueOf(supplier.get()));
​
        Supplier supplier =  () -> "供给型接口：Supplier Demo";
        log.info(String.valueOf(supplier.get()));
    }
```

**Predicate（断定型接口）**

断言型接口：既有输入参数也有返回值，返回类型是boolean类型

```java
* @param <T> the type of the input to the predicate
 *
 * @since 1.8
 */
@FunctionalInterface
public interface Predicate<T> {
​
    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);
```

展示代码：

```java
public void predicateTest() {
//        Predicate<String> predicate = new Predicate<String>() {
//            @Override
//            public boolean test(String s) {
//                return s.equals("Predicate Demo");
//            }
//        };
//        log.info("断言型接口：{}",predicate.test("没有使用Lambda表达式"));
        Predicate<String> predicate = s -> s.equals("Predicate Demo");
        log.info("断言型接口：{}",predicate.test("Predicate Demo"));
    }
```

==**java内置四种大函数式接口，可以使用Lambda表达式**==

```java
package com.chen.test.JAVA8Features;
​
import com.google.common.base.Function;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
​
import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.Supplier;
​
public class FunDemo01 {
    private static Logger log = LoggerFactory.getLogger(FunDemo01.class);
    public static void main(String[] args) {
        FunDemo01 demo01 = new FunDemo01();
        demo01.functionTest();
        demo01.consumerTest();
        demo01.supplierTest();
        demo01.predicateTest();
​
    }
    public void functionTest(){
//        非Lambda表达式
//        Function function = new Function<String,String>(){
//            @Override
//            public String apply(String s) {
//                return s;
//            }
//        };
//        log.info("函数型接口 :{}",function.apply("没有使用Lambda表达式"));
​
        Function function = s -> s;
        log.info("函数型接口：{}",function.apply("Function Demo"));
    }
​
    public void consumerTest(){
//        非Lambda表达式
//        Consumer<String> consumer = new Consumer<String>() {
//            @Override
//            public void accept(String s) {
//                log.info(s);
//            }
//        };
//        consumer.accept("消费型函数：没有使用Lambda表达式");
​
//        使用Lambda表达式
        Consumer<String> consumer = s -> log.info(s);
        consumer.accept("消费型函数：Consumer Demo");
​
    }
    public void supplierTest(){
//        非Lambda表达式
//        Supplier supplier = new Supplier<String>(){
//            @Override
//            public String get() {
//                return "供给型接口：没有使用Lambda表达式";
//            }
//        };
//        log.info(String.valueOf(supplier.get()));
​
        Supplier supplier =  () -> "供给型接口：Supplier Demo";
        log.info(String.valueOf(supplier.get()));
    }
    public void predicateTest() {
//        Predicate<String> predicate = new Predicate<String>() {
//            @Override
//            public boolean test(String s) {
//                return s.equals("Predicate Demo");
//            }
//        };
//        log.info("断言型接口：{}",predicate.test("没有使用Lambda表达式"));
        Predicate<String> predicate = s -> s.equals("Predicate Demo");
        log.info("断言型接口：{}",predicate.test("Predicate Demo"));
    }
​
}
```

## 方法引用和构造器引用

1、方法引用

当要传递给Lambda体的操作已经有实现方法，可以直接使用方法引用(实现抽象方法的列表，必须要和方法引用的方法参数列表一致)

方法引用：使用操作符“：：”将方法名和（类或者对象）分割开来。

有下列三种情况：

> 对象：：实例方法
>
> 类：：实例方法
>
> 类：：静态方法

代码展示：

```java
package com.chen.test.JAVA8Features;
​
public class MethodRefDemo {
    public static void main(String[] args) {
        FunctionGeneric<String> strName = s -> System.out.println(s);
        strName.fun("Lambda表达式没有使用方法引用");
        
        //方法引用
        FunctionGeneric<String> strName2 = System.out::println;
        strName2.fun("使用方法引用");
​
​
    }
}
​
```

2、构造器引用

本质上：构造器引用和方法引用相识，只是使用了一个new方法

使用说明：函数式接口参数列表和构造器参数列表要一致，该接口返回值类型也是构造器返回值类型

格式：ClassName :: new

代码展示：

```java
package com.chen.test.JAVA8Features;
​
import java.util.function.Function;
​
public class MethodRefDemo {
    public static void main(String[] args) {
​
        //构造器引用
        Function<String, Integer> fun1 = (num) -> new Integer(num);
        Function<String, Integer> fun2 = Integer::new;
​
        //数组引用
        Function<Integer,Integer[]> fun3 = (num) ->new Integer[num];
        Function<Integer,Integer[]> fun4 = Integer[]::new;
    }
}
​
```

## 强大的Stream API

### 什么是Stream？

Java8中两个最为重要特性：==第一个的是Lambda表达式，另一个是Stream API。==

==StreamAPI它位于java.util.stream包中==，StreamAPI帮助我们更好地对数据进行集合操作，它本质就是对数据的操作进行流水线式处理，也可以理解为一个更加高级的迭代器,主要作用是遍历其中每一个元素。简而言之,StreamAP提供了一种高效且易于使用的处理数据方式。

### Stream特点：

> 1、Stream自己不会存储数据。
>
> 2、Stream不会改变源对象。相反，它们会返回一个持有结果的新Stream对象
>
> 3、Stream操作时延迟执行的。这就意味着它们等到有结果时候才会执行。
>
> 和list不同，Stream代表的是任意Java对象的序列，且stream输出的元素可能并没有预先存储在内存中，而是实时计算出来的。它可以“存储”有限个或无限个元素。
>
> 例如：我们想表示一个全体自然数的集合，使用list是不可能写出来的，因为自然数是无线的，不管内存多大也没法放到list中，但是使用Sream就可以。

### Stream操作的三个步骤？

> - 1、创建Stream：一个数据源（例如：set 、list），获取一个流
> - 2、中间操作：一个中间操作连接，对数据源的数据进行处理
> - 3、终止操作:一个终止操作，执行中间操作连，产生结果。

1、创建流

创建流方式有多种：

**第一种：通过集合**

对于Collection接口（List 、Set、Queue等）直接调用Stream()方法可以获取Stream

```java
        List<String> list = new ArrayList<>();
        Stream<String> stringStream = list.stream(); //返回一个顺序流
        Stream<String> parallelStream = list.parallelStream(); //返回一个并行流（可多线程）
```

**第二种：通过数组**

把数组变成Stream使用Arrays.stream()方法

```java
        Stream<String> stream1 = Arrays.stream(new String[]{"CBB", "YJJ", "CB", "CJJ"});
​
```

**第三种：Stream.of()静态方法直接手动生成一个Stream**

```java
        Stream<String> stream = Stream.of("A", "B", "C", "D");
```

**第四种：创建无限流**

```java
        //迭代
        //遍历10个奇数
        Stream.iterate(1,t->t+2).limit(10).forEach(System.out::println);
​
        //生成
        //生成10个随机数
        Stream.generate(Math::random).limit(10).forEach(System.out::println);
​
```

**第五种：自己构建**

**第六种：其他等等**

2、中间操作

> 一个流可以后面跟随着0个或者多个中间操作，其目的是打开流，做出某种程度的数据过滤、去重、排序、映射、跳过等。然后返回一个新的流，交给下一个使用，仅仅是调用这个方法，没有真正开始遍历。
>
> map (mapToInt, flatMap 等)、 filter、 distinct、 sorted、 peek、 limit、 skip、 parallel、 sequential、 unordered

3、终止操作：一个终止操作，执行中间操作连，产生结果。

> forEach、 forEachOrdered、 toArray、 reduce、 collect、 min、 max、 count、 anyMatch、 allMatch、 noneMatch、 findFirst、 findAny、 iterator

```java
package com.chen.test.JAVA8Features.Stream;
​
import java.util.*;
import java.util.stream.Collectors;
​
public class StreamDemo01 {
    public static void main(String[] args) {
​
        List<Integer> list = new ArrayList<>();
        for (int i = 0; i < 10; i++) {
            list.add(i);
        }
        //map
        List<Integer> collect = list.stream().map(n -> n * 2).collect(Collectors.toCollection(ArrayList::new));
        collect.forEach(System.out::println);
        //filer 过滤
        List<Integer> list1 = list.stream().filter(n -> n % 2 == 0).collect(Collectors.toList());
        list1.forEach(System.out::println);
        //distinct 去重
        List<Integer> list2 = list.stream().distinct().collect(Collectors.toList());
        list2.forEach(System.out::println);
        //skip 跳过
        List<Integer> list3 = list.stream().skip(3).collect(Collectors.toList());
        list3.forEach(System.out::println);
        //limit 截取
        Set<Integer> set = list.stream().limit(3).collect(Collectors.toSet());
        set.forEach(System.out::println);
        //skip  and limit 组合使用
        List<Integer> list4 = list.stream().skip(3).limit(5).collect(Collectors.toList());
        list4.forEach(System.out::println);
​
    }
}
​
```

## 接口中默认方法和静态方法

**1、默认方法**

java8允许接口中包含具体实现的方法体，该方法是默认方法，它需要使用default关键字修饰

2、**静态方法**

java8中允许接口中定义静态方法，使用static关键字修饰

代码展示：

```java
package com.chen.test.JAVA8Features.DefaultMethod;
​
public interface DefaultMethodDemo {
    
    default Integer addMethod(int a ,int b){
        System.out.println("我是默认方法");
        return a+b;
    }
    static void test(){
        System.out.println("我是静态方法");
    }
}
​
```

## 新时间日期接口

https://blog.csdn.net/lemon_TT/article/details/109145432

## Optional类

optional类是一个容器，代表一个值存在或者不存在，原来使用null表示一个值存不存在，现在使用optional可以更好的表达这个概念，并且可以避免空指针异常。

Optional常用的方法：

> Optional.of(T t) : 创建一个Optional实例；
>
> Optional.empty() : 创建一个空的Optional实例；
>
> Optional.ofNullable(T t) :若t不为空创建一个Optional实例否则创建一个空实例；
>
> isPresent() : 判断是否包含值；
>
> orElse(T t) ：如果调用对象包含值，返回该值，否则返回t；
>
> orElseGet(Supplier s) : 如果调用对象包含值，返回该值，否则返回s获取的值；
>
> map（Function f） ： 如果有值对其处理，并返回处理后的Optional，否则返回Optional.empty();
>
> flatMap(Function mapper) : 与map类似，要求返回值必须是Optional。