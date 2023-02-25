[TOC]

# RabbitMq base info

## 什么是[消息队列](https://so.csdn.net/so/search?q=消息队列&spm=1001.2101.3001.7020)：

**消息队列的优点：**

> （1）解耦：将系统按照不同的业务功能拆分出来，消息生产者只管把消息发布到 MQ 中而不用管谁来取，消息消费者只管从 MQ 中取消息而不管是谁发布的。消息生产者和消费者都不知道对方的存在；
>
> （2）异步：主流程只需要完成业务的核心功能；对于业务非核心功能，将消息放入到消息队列之中进行异步处理，减少请求的等待，提高系统的总体性能；
>
> （3）削峰/限流：将所有请求都写到消息队列中，消费服务器按照自身能够处理的请求数从队列中拿到请求，防止请求并发过高将系统搞崩溃；

**消息队列的缺点：**

> （1）系统的可用性降低：系统引用的外部依赖越多，越容易挂掉，如果MQ 服务器挂掉，那么可能会导致整套系统崩溃。这时就要考虑如何保证消息队列的高可用了
>
> （2）系统复杂度提高：加入消息队列之后，需要保证消息没有重复消费、如何处理消息丢失的情况、如何保证消息传递的有序性等问题；
>
> （3）数据一致性问题：A 系统处理完了直接返回成功了，使用者都以为你这个请求就成功了；但是问题是，要是 BCD 三个系统那里，BD 两个系统写库成功了，结果 C 系统写库失败了，就会导致数据不一致了

**Kafka、ActiveMQ、RabbitMQ、RocketMQ 消息队列的选型：**

![img](https://img-blog.csdnimg.cn/20210323233207364.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NDUyMzM3MDA=,size_16,color_FFFFFF,t_70)

每种MQ没有绝对的好坏，主要依据使用场景，扬长避短，利用其优势，规避其劣势。

- 中小型软件公司，技术实力较为一般，建议选RabbitMQ：一方面，erlang语言天生具备高并发的特性，而且管理界面用起来十分方便。代码是开源的，而且社区十分活跃，可以解决开发过程中遇到的bug，这点对于中小型公司来说十分重要。

> 不考虑 rocketmq 的原因是，rocketmq是阿里出品，如果阿里放弃维护rocketmq，中小型公司一般抽不出人来进行rocketmq的定制化开发，因此不推荐。
> 不考虑 kafka 的原因是：中小型软件公司不如互联网公司，数据量没那么大，选消息中间件应首选功能比较完备的，所以kafka排除

- 大型软件公司：根据具体使用场景在[rocketMq](https://so.csdn.net/so/search?q=rocketMq&spm=1001.2101.3001.7020)和kafka之间二选一。

一方面，大型软件公司，具备足够的资金搭建分布式环境，也具备足够大的数据量。针对rocketMQ，大型软件公司有能力对rocketMQ进行定制化开发。至于[kafka](https://so.csdn.net/so/search?q=kafka&spm=1001.2101.3001.7020)，如果是大数据领域的实时计算、日志采集功能，肯定是首选kafka了。

## RabbitMQ的构造：

RabbitMQ 是 AMQP 协议的一个开源实现，所以其内部实际上也是 AMQP 中的基本概念：

![RabbitMQ 内部结构](https://img-blog.csdnimg.cn/img_convert/1eb56a0d98c1f4ee460c9245b9ea754c.png)

> （1）生产者Publisher：生产消息，就是投递消息的一方。消息一般包含两个部分：消息体（payload）和标签（Label）
> （2）消费者Consumer：消费消息，也就是接收消息的一方。消费者连接到RabbitMQ服务器，并订阅到队列上。消费消息时只消费消息体，丢弃标签。
> （3）Broker服务节点：表示消息队列服务器实体。一般情况下一个Broker可以看做一个RabbitMQ服务器。
> （4）Queue：消息队列，用来存放消息。一个消息可投入一个或多个队列，多个消费者可以订阅同一队列，这时队列中的消息会被平摊（轮询）给多个消费者进行处理。
> （5）Exchange：交换器，接受生产者发送的消息，根据路由键将消息路由到绑定的队列上。
> （6）Routing Key： 路由关键字，用于指定这个消息的路由规则，需要与交换器类型和绑定键(Binding Key)联合使用才能最终生效。
> （7）Binding：绑定，通过绑定将交换器和队列关联起来，一般会指定一个BindingKey，通过BindingKey，交换器就知道将消息路由给哪个队列了。
> （8）Connection ：网络连接，比如一个TCP连接，用于连接到具体broker
> （9）Channel： 信道，AMQP 命令都是在信道中进行的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接，一个TCP连接可以用多个信道。客户端可以建立多个channel，每个channel表示一个会话任务。
> （10）Message：消息，由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。
> （11）Virtual host：虚拟主机，用于逻辑隔离，表示一批独立的交换器、消息队列和相关对象。一个Virtual host可以有若干个Exchange和Queue，同一个Virtual host不能有同名的Exchange或Queue。最重要的是，其拥有独立的权限系统，可以做到 vhost 范围的用户控制。当然，从 RabbitMQ 的全局角度，vhost 可以作为不同权限隔离的手段

## Exchange交换器的类型：

Exchange分发消息时根据类型的不同分发策略有区别，目前共四种类型：direct、fanout、topic、headers

（1）direct：消息中的路由键（RoutingKey）如果和 Bingding 中的 bindingKey 完全匹配，交换器就将消息发到对应的队列中。是基于完全匹配、单播的模式。

（2）fanout：把所有发送到fanout交换器的消息路由到所有绑定该交换器的队列中，fanout 类型转发消息是最快的。

（3）topic：通过模式匹配的方式对消息进行路由，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。

> 匹配规则：
>
> - ① RoutingKey 和 BindingKey 为一个 点号 '.' 分隔的字符串。 比如: java.xiaoka.show
> - ② BindingKey可使用 * 和 # 用于做模糊匹配：*匹配一个单词，#匹配多个或者0个单词

（4）headers：不依赖于路由键进行匹配，是根据发送消息内容中的headers属性进行匹配，除此之外 headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到了

## 生产者消息的过程：

> （1）Producer 先连接到 Broker，建立连接 Connection，开启一个信道 channel
> （2）Producer 声明一个交换器并设置好相关属性
> （3）Producer 声明一个队列并设置好相关属性
> （4）Producer 通过绑定键将交换器和队列绑定起来
> （5）Producer 发送消息到 Broker，其中包含路由键、交换器等信息
> （6）交换器根据接收到的路由键查找匹配的队列
> （7）如果找到，将消息存入对应的队列，如果没有找到，会根据生产者的配置丢弃或者退回给生产者。
> （8）关闭信道

## 消费者接收消息过程：

> （1）Producer 先连接到 Broker，建立连接 Connection，开启一个信道 channel
> （2）向 Broker 请求消费相应队列中消息，可能会设置响应的回调函数。
> （3）等待 Broker 回应并投递相应队列中的消息，接收消息。
> （4）消费者确认收到的消息，ack。
> （5）RabbitMQ从队列中删除已经确定的消息。
> （6）关闭信道

## [RabbitMQ](https://so.csdn.net/so/search?q=RabbitMQ&spm=1001.2101.3001.7020)工作模式

RabbitMQ主要有五种工作模式，分别是：

> 1、简单模式（Hello World）
> 2、工作队列模式（Work Queue）
> 3、发布/订阅模式(publish/subscribe)
> 4、路由模式（routing）
> 5、主题模式（Topic）

先附上工具类`代码，主要用于方便创建连接：

```java
public class RabbitUtils {
    private static ConnectionFactory connectionFactory = new ConnectionFactory();
    static {
        connectionFactory.setHost("81.71.140.7");
        connectionFactory.setPort(5672);//5672是RabbitMQ的默认端口号
        connectionFactory.setUsername("bq123");
        connectionFactory.setPassword("bq123");
        connectionFactory.setVirtualHost("/zm");
    }
    public static Connection getConnection(){
        Connection conn = null;
        try {
            conn = connectionFactory.newConnection();
            return conn;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}
```

### ***简单模式（一对一）***

![在这里插入图片描述](https://img-blog.csdnimg.cn/5b5e620a7b4d4f85a1f57de257bc7c72.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2NDk5Mg==,size_16,color_FFFFFF,t_70#pic_center)

消息的消费者(consumer) 监听消息队列,如果队列中有消息,就消费掉,消息被拿走后,自动从队列中删除(隐患：消息可能没有被消费者正确处理,就因为消费者的虚拟连接通道自动返回确认信号，导致消息从队列中删除了,造成消息的丢失。所以建议这里可以设置成手动的ack,即确认信号我们手动确认。但如果设置成手动ack，处理完后要及时发送ack消息给队列，否则会造成内存溢出)。

流程：
1、建立生产者与服务器的TCP长连接。
2、在TCP连接中再建立独立的轻量级的虚拟连接Channel。
3、通过Channel连接去声明一个队列，如果队列已存在则直接使用存在的队列。
4、生产者通过Channel往队列中发送消息。
5、消费者同生产者一样建立与服务器的连接。
6、消费者声明虚拟连接Channel.
7、消费者通过Channel声明一个队列（和生产者声明的队列同名）
8、消费者从该队列中获取消息，并且返回确认。收到确认后，队列删除已读消息。

生产者/消费者：

```java
public class Producer {

    public static void main(String[] args) throws IOException, TimeoutException {


        //获取TCP长连接
        Connection conn = RabbitUtils.getConnection();
        //创建通信“通道”，相当于TCP中的虚拟连接
        Channel channel = conn.createChannel();

        //创建队列,声明并创建一个队列，如果队列已存在，则使用这个队列
        //第一个参数：队列名称ID
        //第二个参数：是否持久化，false对应不持久化数据，MQ停掉数据就会丢失
        //第三个参数：是否队列私有化，false则代表所有消费者都可以访问，true代表只有第一次拥有它的消费者才能一直使用，其他消费者不让访问
        //第四个：是否自动删除,false代表连接停掉后不自动删除掉这个队列
        //其他额外的参数, null
        channel.queueDeclare(RabbitConstant.QUEUE_HELLOWORLD,false, false, false, null);

        String message = "hello666";
        //四个参数
        //exchange 交换机，暂时用不到，在后面进行发布订阅时才会用到
        //队列名称
        //额外的设置属性
        //最后一个参数是要传递的消息字节数组
        channel.basicPublish("", RabbitConstant.QUEUE_HELLOWORLD, null,message.getBytes());
        channel.close();
        conn.close();
        System.out.println("===发送成功===");

    }
}
```

```java

public class Consumer {

    public static void main(String[] args) throws IOException, TimeoutException {


        //获取TCP长连接
         Connection conn = RabbitUtils.getConnection();
        //创建通信“通道”，相当于TCP中的虚拟连接
        Channel channel = conn.createChannel();

        //创建队列,声明并创建一个队列，如果队列已存在，则使用这个队列
        //第一个参数：队列名称ID
        //第二个参数：是否持久化，false对应不持久化数据，MQ停掉数据就会丢失
        //第三个参数：是否队列私有化，false则代表所有消费者都可以访问，true代表只有第一次拥有它的消费者才能一直使用，其他消费者不让访问
        //第四个：是否自动删除,false代表连接停掉后不自动删除掉这个队列
        //其他额外的参数, null
        channel.queueDeclare(RabbitConstant.QUEUE_HELLOWORLD,false, false, false, null);

        //从MQ服务器中获取数据

        //创建一个消息消费者
        //第一个参数：队列名
        //第二个参数代表是否自动确认收到消息，false代表手动编程来确认消息，这是MQ的推荐做法
        //第三个参数要传入DefaultConsumer的实现类
        channel.basicConsume(RabbitConstant.QUEUE_HELLOWORLD, false, new Reciver(channel));


    }
}


class  Reciver extends DefaultConsumer {

    private Channel channel;
    //重写构造函数,Channel通道对象需要从外层传入，在handleDelivery中要用到
    public Reciver(Channel channel) {
        super(channel);
        this.channel = channel;
    }

    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {

         String message = new String(body);
         System.out.println("消费者接收到的消息："+message);

         System.out.println("消息的TagId："+envelope.getDeliveryTag());
        //false只确认签收当前的消息，设置为true的时候则代表签收该消费者所有未签收的消息
        channel.basicAck(envelope.getDeliveryTag(), false);
    }
}
```

### **工作队列模式（一对多，互相抢夺消息）**

消息产生者将消息放入队列，消费者可以有多个,消费者1,消费者2同时监听同一个队列,消息被消费。C1， C2共同争抢当前的消息队列内容,谁先拿到谁负责消费消息(隐患：高并发情况下,默认会产生某一个消息被多个消费者共同使用,可以设置一个开关(syncronize) 保证一条消息只能被一个消费者使用)。

![在这里插入图片描述](https://img-blog.csdnimg.cn/4dcfb7a0ba924f078e418a1b530b8e0d.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2NDk5Mg==,size_16,color_FFFFFF,t_70#pic_center)

***订票下`单系统（消息生产者）***

```java
public class OrderSystem {

    public static void main(String[] args) throws IOException, TimeoutException {
        Connection connection = RabbitUtils.getConnection();
        Channel channel = connection.createChannel();
        channel.queueDeclare(RabbitConstant.QUEUE_SMS, false, false, false, null);

        for(int i = 1 ; i <= 100 ; i++) {
            SMS sms = new SMS("乘客" + i, "13900000" + i, "您的车票已预订成功");
            String jsonSMS = new Gson().toJson(sms);
            channel.basicPublish("" , RabbitConstant.QUEUE_SMS , null , jsonSMS.getBytes());
        }
        System.out.println("发送数据成功");
        channel.close();
        connection.close();
    }
}
```

***短信发送系统1号（消息消费者）***

```java
public class SMSSender1 {

    public static void main(String[] args) throws IOException {


        Connection connection = RabbitUtils.getConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(RabbitConstant.QUEUE_SMS, false, false, false, null);

        //如果不写basicQos（1），则自动MQ会将所有请求平均发送给所有消费者
        //basicQos,MQ不再对消费者一次发送多个请求，而是消费者处理完一个消息后（确认后），在从队列中获取一个新的
        channel.basicQos(1);//处理完一个取一个

        channel.basicConsume(RabbitConstant.QUEUE_SMS , false , new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String jsonSMS = new String(body);
                System.out.println("SMSSender1-短信发送成功:" + jsonSMS);

                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                channel.basicAck(envelope.getDeliveryTag() , false);
            }
        });
    }
}
```

***短信发送系统2号（消息消费者）***

```java
public class SMSSender2 {

    public static void main(String[] args) throws IOException {


        Connection connection = RabbitUtils.getConnection();
        final Channel channel = connection.createChannel();

        channel.queueDeclare(RabbitConstant.QUEUE_SMS, false, false, false, null);

        //如果不写basicQos（1），则自动MQ会将所有请求平均发送给所有消费者
        //basicQos,MQ不再对消费者一次发送多个请求，而是消费者处理完一个消息后（确认后），在从队列中获取一个新的
        channel.basicQos(1);//处理完一个取一个

        channel.basicConsume(RabbitConstant.QUEUE_SMS , false , new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String jsonSMS = new String(body);
                System.out.println("SMSSender2-短信发送成功:" + jsonSMS);

                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                channel.basicAck(envelope.getDeliveryTag() , false);
            }
        });
    }
}

```

### ***发布 / 订阅模式 (所有消费者共享消息)***

生产者将消息发给broker，由交换机将消息转发到绑定此交换机的每个队列，每个绑定交换机的队列都将接收到消息。

![在这里插入图片描述](https://img-blog.csdnimg.cn/c43d3e8610a240598de266c1c64ede24.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2NDk5Mg==,size_16,color_FFFFFF,t_70#pic_center)

流程：
1、生产者与消费者都与服务器建立好TCP连接，并建立各自的虚拟连接。
2、生产者通过Channel将消息发送至交换机中
3、消费者通过Channel声明一个队列，并且绑定对应的交换机
4、交换机会将消息发给每个与它绑定的队列中，所以队列都能获得全部消息。
5、消费者从队列中读取消息，并且返回确认。收到确认后，相关队列删除已读消息。

生产者

```java
public class WeatherBureau {


    public static void main(String[] args) throws Exception {
        Connection connection = RabbitUtils.getConnection();
        String input = new Scanner(System.in).next();
        Channel channel = connection.createChannel();

        //第一个参数交换机名字   其他参数和之前的一样
        channel.basicPublish(RabbitConstant.EXCHANGE_WEATHER,"" , null , input.getBytes());

        channel.close();
        connection.close();
    }
}
```

消费者1

```java
public class BiaDu {

    public static void main(String[] args) throws IOException {
        //获取TCP长连接
        Connection connection = RabbitUtils.getConnection();
        //获取虚拟连接
        final Channel channel = connection.createChannel();
        //声明队列信息
        channel.queueDeclare(RabbitConstant.QUEUE_BAIDU, false, false, false, null);

        //queueBind用于将队列与交换机绑定
        //参数1：队列名 参数2：交互机名  参数三：路由key（暂时用不到)
        channel.queueBind(RabbitConstant.QUEUE_BAIDU, RabbitConstant.EXCHANGE_WEATHER, "");
        channel.basicQos(1);
        channel.basicConsume(RabbitConstant.QUEUE_BAIDU , false , new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("新浪天气收到气象信息：" + new String(body));
                channel.basicAck(envelope.getDeliveryTag() , false);
            }
        });
    }

}
```

消费者2

```java
public class Sina {

    public static void main(String[] args) throws IOException {
        //获取TCP长连接
        Connection connection = RabbitUtils.getConnection();
        //获取虚拟连接
        final Channel channel = connection.createChannel();
        //声明队列信息
        channel.queueDeclare(RabbitConstant.QUEUE_SINA, false, false, false, null);

        //queueBind用于将队列与交换机绑定
        //参数1：队列名 参数2：交互机名  参数三：路由key（暂时用不到)
        channel.queueBind(RabbitConstant.QUEUE_SINA, RabbitConstant.EXCHANGE_WEATHER, "");
        channel.basicQos(1);
        channel.basicConsume(RabbitConstant.QUEUE_SINA , false , new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("新浪天气收到气象信息：" + new String(body));
                channel.basicAck(envelope.getDeliveryTag() , false);
            }
        });
    }

}
```

### ***路由模式***

![在这里插入图片描述](https://img-blog.csdnimg.cn/5eb14b48689a4140b5e8fdfc3c420abb.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2NDk5Mg==,size_16,color_FFFFFF,t_70#pic_center)

1.生产者发送的消息是以key-value的形式，当消息进入交换机，交换机根据key的不同，将其分配到不同的队列。
2.消费者通过Channel声明一个队列时，需要绑定给队列绑定一个key和一个交换机。
2.业务场景:error 通知;EXCEPTION;错误通知的功能;传统意义的错误通知;客户通知;利用key路由,可以将程序中的错误封装成消息传入到消息队列中,开发者可以自定义消费者,实时接收错误;

生产者

```java
public class WeatherBureau {


    public static void main(String[] args) throws Exception {

        Map area = new LinkedHashMap<String, String>();
        area.put("china.hunan.changsha.20201127", "中国湖南长沙20201127天气数据");
        area.put("china.hubei.wuhan.20201127", "中国湖北武汉20201127天气数据");
        area.put("china.hunan.zhuzhou.20201127", "中国湖南株洲20201128天气数据");
        area.put("us.cal.lsj.20201127", "美国加州洛杉矶20201127天气数据");

        area.put("china.hebei.shijiazhuang.20201128", "中国河北石家庄20201128天气数据");
        area.put("china.hubei.wuhan.20201128", "中国湖北武汉20201128天气数据");
        area.put("china.henan.zhengzhou.20201128", "中国河南郑州20201128天气数据");
        area.put("us.cal.lsj.20201128", "美国加州洛杉矶20201128天气数据");


        Connection connection = RabbitUtils.getConnection();
        Channel channel = connection.createChannel();

        Iterator<Map.Entry<String, String>> itr = area.entrySet().iterator();
        while (itr.hasNext()) {
            Map.Entry<String, String> me = itr.next();
            //第一个参数交换机名字   第二个参数作为 消息的routing key
            channel.basicPublish(RabbitConstant.EXCHANGE_WEATHER_ROUTING,me.getKey() , null , me.getValue().getBytes());

        }

        channel.close();
        connection.close();
    }
}
```

消费者1

```java
public class BiaDu {

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitUtils.getConnection();
        final Channel channel = connection.createChannel();
        channel.queueDeclare(RabbitConstant.QUEUE_BAIDU, false, false, false, null);
        //queueBind用于将队列与交换机绑定
        //参数1：队列名 参数2：交互机名  参数三：路由key
        channel.queueBind(RabbitConstant.QUEUE_BAIDU, RabbitConstant.EXCHANGE_WEATHER_ROUTING, "china.hunan.changsha.20201127");
        channel.queueBind(RabbitConstant.QUEUE_BAIDU, RabbitConstant.EXCHANGE_WEATHER_ROUTING, "china.hebei.shijiazhuang.20201128");
        channel.basicQos(1);
        channel.basicConsume(RabbitConstant.QUEUE_BAIDU , false , new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("百度天气收到气象信息：" + new String(body));
                channel.basicAck(envelope.getDeliveryTag() , false);
            }
        });

    }

}
```

消费者2

```java
public class Sina {

    public static void main(String[] args) throws IOException {
        //获取TCP长连接
        Connection connection = RabbitUtils.getConnection();
        //获取虚拟连接
        final Channel channel = connection.createChannel();
        //声明队列信息
        channel.queueDeclare(RabbitConstant.QUEUE_SINA, false, false, false, null);

        //指定队列与交换机以及routing key之间的关系
        channel.queueBind(RabbitConstant.QUEUE_SINA, RabbitConstant.EXCHANGE_WEATHER_ROUTING, "us.cal.lsj.20201127");
        channel.queueBind(RabbitConstant.QUEUE_SINA, RabbitConstant.EXCHANGE_WEATHER_ROUTING, "china.hubei.wuhan.20201127");
        channel.queueBind(RabbitConstant.QUEUE_SINA, RabbitConstant.EXCHANGE_WEATHER_ROUTING, "us.cal.lsj.20201128");
        channel.queueBind(RabbitConstant.QUEUE_SINA, RabbitConstant.EXCHANGE_WEATHER_ROUTING, "china.henan.zhengzhou.20201012");

        channel.basicQos(1);
        channel.basicConsume(RabbitConstant.QUEUE_SINA , false , new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("新浪天气收到气象信息：" + new String(body));
                channel.basicAck(envelope.getDeliveryTag() , false);
            }
        });
    }

}
```

### ***主题模式（路由模式的一种）***

![在这里插入图片描述](https://img-blog.csdnimg.cn/daa5ed9bc9b44319ae7ca34edfb55169.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2NDk5Mg==,size_16,color_FFFFFF,t_70#pic_center)

![在这里插入图片描述](https://img-blog.csdnimg.cn/0d8b630563ce412db59844ecb48e6ba5.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NTI2NDk5Mg==,size_16,color_FFFFFF,t_70#pic_center)

生产者

```java
public class WeatherBureau {
    
    public static void main(String[] args) throws Exception {

        Map area = new LinkedHashMap<String, String>();
        area.put("china.hunan.changsha.20201127", "中国湖南长沙20201127天气数据");
        area.put("china.hubei.wuhan.20201127", "中国湖北武汉20201127天气数据");
        area.put("china.hunan.zhuzhou.20201127", "中国湖南株洲20201127天气数据");
        area.put("us.cal.lsj.20201127", "美国加州洛杉矶20201127天气数据");

        area.put("china.hebei.shijiazhuang.20201128", "中国河北石家庄20201128天气数据");
        area.put("china.hubei.wuhan.20201128", "中国湖北武汉20201128天气数据");
        area.put("china.henan.zhengzhou.20201128", "中国河南郑州20201128天气数据");
        area.put("us.cal.lsj.20201128", "美国加州洛杉矶20201128天气数据");


        Connection connection = RabbitUtils.getConnection();
        Channel channel = connection.createChannel();

        Iterator<Map.Entry<String, String>> itr = area.entrySet().iterator();
        while (itr.hasNext()) {
            Map.Entry<String, String> me = itr.next();
            //第一个参数交换机名字   第二个参数作为 消息的routing key
            channel.basicPublish(RabbitConstant.EXCHANGE_WEATHER_TOPIC,me.getKey() , null , me.getValue().getBytes());

        }

        channel.close();
        connection.close();
    }
}
```

消费者1

```java
public class BiaDu {

    public static void main(String[] args) throws IOException {
        Connection connection = RabbitUtils.getConnection();
        final Channel channel = connection.createChannel();
        channel.queueDeclare(RabbitConstant.QUEUE_BAIDU, false, false, false, null);
        //queueBind用于将队列与交换机绑定
        //参数1：队列名 参数2：交互机名  参数三：路由key
        channel.queueBind(RabbitConstant.QUEUE_BAIDU, RabbitConstant.EXCHANGE_WEATHER_TOPIC, "*.*.*.20201127");
       // channel.queueBind(RabbitConstant.QUEUE_BAIDU, RabbitConstant.EXCHANGE_WEATHER_ROUTING, "china.hebei.shijiazhuang.20201128");
        channel.basicQos(1);
        channel.basicConsume(RabbitConstant.QUEUE_BAIDU , false , new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("百度天气收到气象信息：" + new String(body));
                channel.basicAck(envelope.getDeliveryTag() , false);
            }
        });

    }

}
```

消费者2

```java
public class Sina {

    public static void main(String[] args) throws IOException {
        //获取TCP长连接
        Connection connection = RabbitUtils.getConnection();
        //获取虚拟连接
        final Channel channel = connection.createChannel();
        //声明队列信息
        channel.queueDeclare(RabbitConstant.QUEUE_SINA, false, false, false, null);

        //指定队列与交换机以及routing key之间的关系
        channel.queueBind(RabbitConstant.QUEUE_SINA, RabbitConstant.EXCHANGE_WEATHER_TOPIC, "us.#");

        channel.basicQos(1);
        channel.basicConsume(RabbitConstant.QUEUE_SINA , false , new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("新浪天气收到气象信息：" + new String(body));
                channel.basicAck(envelope.getDeliveryTag() , false);
            }
        });
    }

}
```

## 如何保证消息不被重复消费？

正常情况下，消费者在消费消息后，会给消息队列发送一个确认，消息队列接收后就知道消息已经被成功消费了，然后就从队列中删除该消息，也就不会将该消息再发送给其他消费者了。不同消息队列发出的确认消息形式不同，RabbitMQ是通过发送一个ACK确认消息。但是因为网络故障，消费者发出的确认并没有传到消息队列，导致消息队列不知道该消息已经被消费，然后就再次消息发送给了其他消费者，从而造成重复消费的情况。

重复消费问题的解决思路是：保证消息的唯一性，即使多次传输，也不让消息的多次消费带来影响，也就是保证消息等幂性；==幂等性指一个操作执行任意多次所产生的影响均与一次执行的影响相同。具体解决方案如下：==

（1）改造业务逻辑，使得在重复消费时也不影响最终的结果。例如对SQL语句： update t1 set money = 150 where id = 1 and money = 100; 做了个前置条件判断，即 money = 100 的情况下才会做更新，更通用的是做个 version 即版本号控制，对比消息中的版本号和数据库中的版本号。

（2）基于数据库的的唯一主键进行约束。消费完消息之后，到数据库中做一个 insert 操作，如果出现重复消费的情况，就会导致主键冲突，避免数据库出现脏数据。

（3）通过记录关键的key，当重复消息过来时，先判断下这个key是否已经被处理过了，如果没处理再进行下一步。

> ① 通过数据库：比如处理订单时，记录订单ID，在消费前，去数据库中进行查询该记录是否存在，如果存在则直接返回。
> ② 使用全局唯一ID，再配合第三组主键做消费记录，比如使用 redis 的 set 结构，生产者发送消息时给消息分配一个全局ID，在每次消费者开始消费前，先去redis中查询有没有消费记录，如果消费过则不进行处理，如果没消费过，则进行处理，消费完之后，就将这个ID以k-v的形式存入redis中(过期时间根据具体情况设置)。

## **如何保证消息不丢失，进行可靠性传输？**

对于消息的可靠性传输，每种MQ都要从三个角度来分析：生产者丢数据、消息队列丢数据、消费者丢数据。以RabbitMQ为例：

### **生产者丢数据：**

RabbitMQ提供事务机制（transaction）和确认机制（confirm）两种模式来确保生产者不丢消息。

（1）事务机制：

> 发送消息前，开启事务（channel.txSelect()），然后发送消息，如果发送过程中出现什么异常，事务就会回滚（channel.txRollback()），如果发送成功则提交事务（channel.txCommit()）
>
>     该方式的缺点是生产者发送消息会同步阻塞等待发送结果是成功还是失败，导致生产者发送消息的吞吐量降下降。

    ```java
        // 开启事务
        channel.txSelect
        try {
            // 发送消息
        } catch(Exception e){
            // 回滚事务
            channel.txRollback;
            //再次重试发送这条消息
            ....
        }      
        //提交事务
        channel.txCommit;
    ```

（2）确认机制：

生产环境常用的是confirm模式。生产者将信道 channel 设置成 confirm 模式，一旦 channel 进入 confirm 模式，所有在该信道上发布的消息都将会被指派一个唯一的ID，一旦消息被投递到所有匹配的队列之后，rabbitMQ就会发送一个确认给生产者（包含消息的唯一ID），这样生产者就知道消息已经正确到达目的队列了。如果rabbitMQ没能处理该消息，也会发送一个Nack消息给你，这时就可以进行重试操作。

        Confirm模式最大的好处在于它是异步的，一旦发布消息，生产者就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者便可以通过回调方法来处理该确认消息。

处理Ack和Nack的代码如下所示：

```java
channel.addConfirmListener(new ConfirmListener() {  
    @Override  
    public void handleNack(long deliveryTag, boolean multiple) throws IOException {  
        System.out.println("nack: deliveryTag = "+deliveryTag+" multiple: "+multiple);  
    }  
    @Override  
    public void handleAck(long deliveryTag, boolean multiple) throws IOException {  
        System.out.println("ack: deliveryTag = "+deliveryTag+" multiple: "+multiple);  
    }  
});
```

### **消息队列丢数据：**

 处理消息队列丢数据的情况，一般是开启持久化磁盘。持久化配置可以和生产者的 confirm 机制配合使用，在消息==持久化磁盘后，再给生产者发送一个Ack信号==。这样的话，如果消息持久化磁盘之前，即使 RabbitMQ 挂掉了，生产者也会因为收不到Ack信号而再次重发消息。

持久化设置如下（必须同时设置以下 2 个配置）：

> （1）创建queue的时候，将queue的持久化标志durable在设置为true，代表是一个持久的队列，这样就可以保证 rabbitmq 持久化 queue 的元数据，但是不会持久化queue里的数据；
> （2）发送消息的时候将 deliveryMode 设置为 2，将消息设置为持久化的，此时 RabbitMQ 就会将消息持久化到磁盘上去。
>         这样设置以后，RabbitMQ 就算挂了，重启后也能恢复数据。在消息还没有持久化到硬盘时，可能服务已经死掉，这种情况可以通过引入镜像队列，但也不能保证消息百分百不丢失（整个集群都挂掉）

### **消费者丢数据：**

  消费者丢数据一般是因为采用了自动确认消息模式。该模式下，虽然消息还在处理中，但是消费中者会自动发送一个确认，通知 RabbitMQ 已经收到消息了，这时 RabbitMQ 就会立即将消息删除。这种情况下，如果消费者出现异常而未能处理消息，那就会丢失该消息。

        解决方案就是采用手动确认消息，设置 autoAck = False，等到消息被真正消费之后，再手动发送一个确认信号，即使中途消息没处理完，但是服务器宕机了，那 RabbitMQ 就收不到发的ack，然后 RabbitMQ 就会将这条消息重新分配给其他的消费者去处理。
    
        但是 RabbitMQ 并没有使用超时机制，RabbitMQ 仅通过与消费者的连接来确认是否需要重新发送消息，也就是说，只要连接不中断，RabbitMQ 会给消费者足够长的时间来处理消息。另外，采用手动确认消息的方式，我们也需要考虑一下几种特殊情况：

如果消费者接收到消息，在确认之前断开了连接或取消订阅，RabbitMQ 会认为消息没有被消费，然后重新分发给下一个订阅的消费者，所以存在消息重复消费的隐患
如果消费者接收到消息却没有确认消息，连接也未断开，则RabbitMQ认为该消费者繁忙，将不会给该消费者分发更多的消息

> 需要注意的点：
>
> 1、消息可靠性增强了，性能就下降了，因为写磁盘比写 RAM 慢的多，两者的吞吐量可能有 10 倍的差距。所以，是否要对消息进行持久化，需要综合考虑业务场景、性能需要，以及可能遇到的问题。若想达到单RabbitMQ服务器 10W 条/秒以上的消息吞吐量，则要么使用其他的方式来确保消息的可靠传输，要么使用非常快速的存储系统以支持全持久化，例如使用 SSD。或者仅对关键消息作持久化处理，且应该保证关键消息的量不会导致性能瓶颈。
>
> 2、当设置 autoAck = False 时，如果忘记手动 ack，那么将会导致大量任务都处于 Unacked 状态，造成队列堆积，直至消费者断开才会重新回到队列。解决方法是及时 ack，确保异常时 ack 或者拒绝消息。
>
> 3、启用消息拒绝或者发送 nack 后导致死循环的问题：如果在消息处理异常时，直接拒绝消息，消息会重新进入队列。这时候如果消息再次被处理时又被拒绝 。这样就会形成死循环。

## 如何保证消息的有序性？

针对保证消息有序性的问题，解决方法就是保证生产者入队的顺序是有序的，出队后的顺序消费则交给消费者去保证。

（1）方法一：拆分queue，使得一个queue只对应一个消费者。由于MQ一般都能保证内部队列是先进先出的，所以把需要保持先后顺序的一组消息使用某种算法都分配到同一个消息队列中。然后只用一个消费者单线程去消费该队列，这样就能保证消费者是按照顺序进行消费的了。但是消费者的吞吐量会出现瓶颈。如果多个消费者同时消费一个队列，还是可能会出现顺序错乱的情况，这就相当于是多线程消费了

![img](https://img-blog.csdnimg.cn/20210323225225442.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NDUyMzM3MDA=,size_16,color_FFFFFF,t_70)

（2）方法二：对于多线程的消费同一个队列的情况，可以使用重试机制：比如有一个微博业务场景的操作，发微博、写评论、删除微博，这三个异步操作。如果一个消费者先执行了写评论的操作，但是这时微博都还没发，写评论一定是失败的，等一段时间。等另一个消费者，先执行发微博的操作后，再执行，就可以成功。

## 如何处理消息堆积情况?

场景题：几千万条数据在MQ里积压了七八个小时。

### **出现该问题的原因：**

消息堆积往往是生产者的生产速度与消费者的消费速度不匹配导致的。有可能就是消费者消费能力弱，渐渐地消息就积压了，也有可能是因为消息消费失败反复复重试造成的，也有可能是消费端出了问题，导致不消费了或者消费极其慢。比如，消费端每次消费之后要写mysql，结果mysql挂了，消费端hang住了不动了，或者消费者本地依赖的一个东西挂了，导致消费者挂了。

所以如果是 bug 则处理 bug；如果是因为本身消费能力较弱，则优化消费逻辑，比如优化前是一条一条消息消费处理的，那么就可以批量处理进行优化。

### **临时扩容，快速处理积压的消息：**

（1）先修复 consumer 的问题，确保其恢复消费速度，然后将现有的 consumer 都停掉；

（2）临时创建原先 N 倍数量的 queue ，然后写一个临时分发数据的消费者程序，将该程序部署上去消费队列中积压的数据，消费之后不做任何耗时处理，直接均匀轮询写入临时建立好的 N 倍数量的 queue 中；

（3）接着，临时征用 N 倍的机器来部署 consumer，每个 consumer 消费一个临时 queue 的数据

（4）等快速消费完积压数据之后，恢复原先部署架构 ，重新用原先的 consumer 机器消费消息。

这种做法相当于临时将 queue 资源和 consumer 资源扩大 N 倍，以正常 N 倍速度消费。

### **恢复队列中丢失的数据：**

如果使用的是 rabbitMQ，并且设置了过期时间，消息在 queue 里积压超过一定的时间会被 rabbitmq 清理掉，导致数据丢失。这种情况下，实际上队列中没有什么消息挤压，而是丢了大量的消息。所以就不能说增加 consumer 消费积压的数据了，这种情况可以采取 “批量重导” 的方案来进行解决。在流量低峰期，写一个程序，手动去查询丢失的那部分数据，然后将消息重新发送到mq里面，把丢失的数据重新补回来。

### **MQ长时间未处理导致MQ写满的情况如何处理：**

如果消息积压在MQ里，并且长时间都没处理掉，导致MQ都快写满了，这种情况肯定是临时扩容方案执行太慢，这种时候只好采用 “丢弃+批量重导” 的方式来解决了。首先，临时写个程序，连接到mq里面消费数据，消费一个丢弃一个，快速消费掉积压的消息，降低MQ的压力，然后在流量低峰期时去手动查询重导丢失的这部分数据。

## 如何保证消息队列的高可用

RabbitMQ 是基于主从（非分布式）做高可用性的，RabbitMQ 有三种模式：单机模式、普通集群模式、镜像集群模式

### 单机模式：

一般没人生产用单机模式

### **普通集群模式：**

 普通集群模式用于提高系统的吞吐量，通过添加节点来线性扩展消息队列的吞吐量。也就是在多台机器上启动多个 RabbitMQ 实例，而队列 queue 的消息只会存放在其中一个 RabbitMQ 实例上，但是每个实例都同步 queue 的元数据（元数据是 queue 的一些配置信息，通过元数据，可以找到 queue 所在实例）。消费的时候，如果连接到了另外的实例，那么该实例就会从数据实际所在的实例上的queue拉取消息过来，就是说让集群中多个节点来服务某个 queue 的读写操作

        但普通集群模式的缺点在于：无高可用性，queue所在的节点宕机了，其他实例就无法从那个实例拉取数据；RabbitMQ 内部也会产生大量的数据传输。


![img](https://img-blog.csdnimg.cn/20210323230117751.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NDUyMzM3MDA=,size_16,color_FFFFFF,t_70)

### **镜像队列集群模式：**

  镜像队列集群是RabbitMQ 真正的高可用模式，集群中一般会包含一个主节点master和若干个从节点slave，如果master由于某种原因失效，那么按照slave加入的时间排序，"资历最老"的slave会被提升为新的master。

        镜像队列下，所有的消息只会向master发送，再由master将命令的执行结果广播给slave，所以master与slave节点的状态是相同的。比如，每次写消息到 queue 时，master会自动将消息同步到各个slave实例的queue；如果消费者与slave建立连接并进行订阅消费，其实质上也是从master上获取消息，只不过看似是从slave上消费而已，比如消费者与slave建立了TCP连接并执行Basic.Get的操作，那么也是由slave将Basic.Get请求发往master，再由master准备好数据返回给slave，最后由slave投递给消费者。
    
       从上面可以看出，队列的元数据和消息会存在于多个实例上，也就是说每个 RabbitMQ 节点都有这个 queue 的完整镜像，任何一个机器宕机了，其它机器节点还包含了这个 queue 的完整数据，其他消费者都可以到其它节点上去消费数据。
（1）缺点：

- 性能开销大，消息需要同步到所有机器上，导致网络带宽压力和消耗很重

- 非分布式，没有扩展性，如果 queue 的数据量大到这个机器上的容量无法容纳了，此时该方案就会出现问题了

（2）如何开启镜像集群模式呢？

        在RabbitMQ 的管理控制台Admin页面下，新增一个镜像集群模式的策略，指定的时候是可以要求数据同步到所有节点的，也可以要求同步到指定数量的节点，再次创建 queue 的时候，应用这个策略，就会自动将数据同步到其他的节点上去了。
![img](https://img-blog.csdnimg.cn/20210323230306476.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2E3NDUyMzM3MDA=,size_16,color_FFFFFF,t_70)

## 其他：

（1）交换器无法根据自身类型和路由键找到符合条件队列时，有哪些处理方式：设置mandatory = true，代表返回消息给生产者；设置mandatory = false，代表直接丢弃

（2）消费者得到消息队列中的数据的方式：push 和 pull

（3）消息基于什么传输：由于 TCP 连接的创建和销毁开销较大，且并发数受系统资源限制，会造成性能瓶颈。所以RabbitMQ 使用信道 channel 的方式来传输数据，信道是建立在真实的 TCP 连接内的虚拟连接，且每条 TCP 连接上的信道数量没有限制。

（4）死信队列DLX：

        DLX也是一个正常的Exchange，和一般的Exchange没有任何区别。能在任何的队列上被指定，实际上就是设置某个队列的属性。当这个队列出现死信（dead message，就是没有任何消费者消费）的时候，RabbitMQ就会自动将这条消息重新发布到Exchange上去，进而被路由到另一个队列。可以监听这个队列中的消息作相应的处理。消息变为死信的几种情况：
> - 消息被拒绝（basic.reject/basic.nack）同时 requeue=false（不重回队列）
> - TTL 过期
> - 队列达到最大长度，无法再添加

（5）延迟队列：存储对应的延迟消息，当消息被发送以后，并不想让消费者立刻拿到消息，而是等待特定时间后，消费者才能拿到这个消息进行消费。在 RabbitMQ 中并不存在延迟队列，但我们可以通过设置消息的过期时间和死信队列来实现延迟队列，消费者监听死信交换器绑定的队列，而不要监听消息发送的队列。

（6）优先级队列：优先级高的队列会先被消费，可以通过 x-max-priority 参数来实现。但是当消费速度大于生产速度且 Broker 没有堆积的情况下，优先级显得没有意义。

（7）RabbitMQ 要求集群中至少有一个磁盘节点，其他节点可以是内存节点，当节点加入或离开集群时，必须要将该变更通知到至少一个磁盘节点。如果只有一个磁盘节点，刚好又是该节点崩溃了，那么集群可以继续路由消息，但不能创建队列、创建交换器、创建绑定、添加用户、更改权限、添加或删除集群节点。也就是说集群中的唯一磁盘节点崩溃的话，集群仍然可以运行，但直到该节点恢复前，无法更改任何东西。