[TOC]

# SpringCloud Ribbon 

知识点：

- Ribbon介绍（3点）
- 使用方法
- 负载均衡策略和自定义负载均衡策略
- Ping机制
- Ribbon配置

## 介绍

Spring Cloud Ribbon 是一套基于 Netflix Ribbon 实现的客户端负载均衡和服务调用工具。通过Spring Cloud的封装，可以让我们轻松地将面向服务的REST模版请求自动转换成客户端负载均衡的服务调用。Spring Cloud Ribbon虽然只是一个工具类框架，它不像服务注册中心、配置中心、API网关那样需要独立部署，但是它几乎存在于每一个Spring Cloud构建的微服务和基础设施中。因为微服务间的调用，API网关的请求转发等内容，实际上都是通过Ribbon来实现的。

- Spring Cloud Ribbon 是一套基于 Netflix Ribbon 实现的客户端负载均衡工具
- spring cloud进行二次封装，让我们可以将面向服务的Rest模板（RestTemplate）请求装换成客户端负载均衡的服务调用
- 不需要单独部署

## 客户端负载均衡

在调用者服务中实现负载均衡，来分发请求至被调用者。如图所示

![ribbon客户端负载均衡](https://img-blog.csdnimg.cn/img_convert/1c977d7911558688985c8648038d4786.png)

## 使用方法

### 单独使用

导入Ribbon使用的依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    <version>2.2.9.RELEASE</version>
</dependency>
```

```yml
# 配置被调用服务集合
goods-service.ribbon.listOfServers=\
  http://localhost:9090/goods,http://localhost:9093/goods
  
# httpClient 连接池最大总连接数
ribbon.MaxTotalConnections=200
# httpClient 每个host最大连接数
ribbon.MaxConnectionsPerHost=50
```

#### LoadBalancerClient方式

```java
@Autowired
private LoadBalancerClient loadbalancerClient;

@GetMapping
public String goods(){
    log.info("begin do order");
    ServiceInstance si=loadbalancerClient.choose("goods-service");
    String url=String.format("http://%s:%s",si.getHost(),si.getPort());
    log.info("ribbon-url:{}",url);
    String goodsInfo=restTemplate.getForObject(url,String.class);
    return goodsInfo;
}
```

#### 通过`@LoadBalanced`注解方式

配置`RestTemplate`时，添加`@LoadBalanced`注解

```java
@Configuration
public class RestTemplateConfiguration {

    @LoadBalanced
    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

接口调用，直接通过服务名`goods-service`调用

```java
@GetMapping
public String goods(){
    log.info("begin do order");
    String url="http://goods-service/goods";
    String goodsInfo=restTemplate.getForObject(url,String.class);
    return goodsInfo;
}
```

### 基于Eureka的使用

添加依赖

```xml
<!--Spring Cloud Ribbon 依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    <version>2.2.9.RELEASE</version>
</dependency>
<!--Spring Cloud Eureka 客户端依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

注入RestTemplate,通过`@LoadBalanced`注解开启

```java
@Configuration
public class RestTemplateConfiguration {

    @Bean
    @LoadBalanced //在客户端使用 RestTemplate 请求服务端时，开启负载均衡（Ribbon）
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

配置文件`application.properties`

```yml
server:
  port: 80 #端口号
# eureka配置
eureka:
  client:
    register-with-eureka: false # 本微服务为服务消费者，不需要将自己注册到服务注册中心
    fetch-registry: true  # 本微服务为服务消费者，需要到服务注册中心搜索服务
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/ #服务注册中心集群
```

直接使用`goods-service`，服务提供者的服务名称调用，ribbon会通过eureka获取服务列表，通过负载策略选择服务

```java
@GetMapping
public String goods(){
    log.info("begin do order");
    String url="http://goods-service/goods";
    String goodsInfo=restTemplate.getForObject(url,String.class);
    return goodsInfo;
}
```

## Ribbon自定义负载均衡策略

Ribbon提供的负载均衡策略

![image-20220317100025264](https://img-blog.csdnimg.cn/img_convert/d05d0d710bad316f196fc2d28b7a4677.png)

> 1	RoundRobinRule	按照线性轮询策略，即按照一定的顺序依次选取服务实例
> 2	RandomRule	随机选取一个服务实例
> 3	RetryRule	按照 RoundRobinRule（轮询）的策略来获取服务，如果获取的服务实例为 null 或已经失效，则在指定的时间之内不断地进行重试（重试时获取服务的策略还是 RoundRobinRule 中定义的策略），如果超过指定时间依然没获取到服务实例则返回 null 。
> 4	WeightedResponseTimeRule	WeightedResponseTimeRule 是 RoundRobinRule 的一个子类，它对 RoundRobinRule 的功能进行了扩展。 根据平均响应时间，来计算所有服务实例的权重，响应时间越短的服务实例权重越高，被选中的概率越大。刚启动时，如果统计信息不足，则使用线性轮询策略，等信息足够时，再切换到 WeightedResponseTimeRule。
> 5	BestAvailableRule	继承自 ClientConfigEnabledRoundRobinRule。先过滤点故障或失效的服务实例，然后再选择并发量最小的服务实例。
> 6	AvailabilityFilteringRule	先过滤掉故障或失效的服务实例，然后再选择并发量较小的服务实例。
> 7	ZoneAvoidanceRule	默认的负载均衡策略，综合判断服务在不同区域（zone）的性能和服务（server）的可用性，来选择服务实例。在没有区域的环境下，该策略与轮询（RoundRobinRule）策略类似。

如上图所示，Ribbon已经提供很多负载均衡策略，如果我们想定义自己的负载均衡策略可以通过如下方式，实现

### 继承`AbstractLoadBalancerRule`,实现`choose`方法

```java
package com.gupaoedu.mall.gpmallportal;

import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.ILoadBalancer;
import com.netflix.loadbalancer.Server;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

import java.util.List;

public class DefinieIpHashRule extends AbstractLoadBalancerRule {
    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
        //....
    }

    public Server choose(ILoadBalancer lb, Object key) {
        if(lb==null){
            return null;
        }
        Server server=null;
        while(server==null){
            //表示启动的服务列表.(默认情况下单纯只用Ribbon时，不会对目标服务做心跳检测）
            List<Server> upList=lb.getReachableServers();
            List<Server> allList=lb.getAllServers();
            int serverCount=upList.size();
            if(serverCount==0){
                return null;
            }
            int index=ipAddressHash(serverCount);
            server=upList.get(index);
        }
        return server;
    }
    private int ipAddressHash(int serverCount){
        ServletRequestAttributes requestAttributes= (ServletRequestAttributes)RequestContextHolder.getRequestAttributes();
        String remoteAddr=requestAttributes.getRequest().getRemoteAddr();
        int code=Math.abs(remoteAddr.hashCode());
        return code%serverCount;
    }

    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(),key);
    }
}
```

配置文件,配置服务名称是`goods-service`的使用自定义负载均衡

```yml
goods-service.ribbon.NFLoadBalancerRuleClassName=com.xxxx.xxx.DefinieIpHashRule
```

## Ribbon的Ping机制

在Ribbon负载均衡器中，提供了ping机制，每隔一段时间，就会去ping服务器，由 com.netflix.loadbalancer.IPing 接口去实现。

单独使用ribbon，不会激活ping机制，默认采用DummyPing（在RibbonClientConfiguration中实例化），isAlive()方法直接返回true。

Ribbon和Eureka集成，默认采用NIWSDiscoveryPing（在EurekaRibbonClientConfiguration中实例化的），只有服务器列表的实例状态为up的时候 才会为Alive。

> IPing中默认内置了一些实现方法如下
>
> 1. PingUrl: 使用httpClient对目标服务逐个实现Ping操作。
> 2. DummyPing： 默认认为对方服务是正常的，直接返回true。
> 3. NoOpPing：永远返回true。

#### 实现`com.netflix.loadbalancer.IPing`接口

```java
public class HealthChecker implements IPing {

    @Override
    public boolean isAlive(Server server) {
        String url="http://"+server.getId()+"/healthCheck";
        boolean isAlive=true;
        HttpClient httpClient=new DefaultHttpClient();
        HttpUriRequest request=new HttpGet(url);
        try {
            HttpResponse response = httpClient.execute(request);
            isAlive = response.getStatusLine().getStatusCode() == 200;
        }catch (Exception e){
            isAlive=false;
        }finally {
            request.abort();
        }
        return isAlive;
    }
}
```

配置文件

```java
goods-service.ribbon.NFLoadBalancerPingClassName=com.xxxxx.xxx.HealthChecker
```

