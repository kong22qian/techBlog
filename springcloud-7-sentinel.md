[TOC]

# SpringCloud Sentinel

## Seninel简介

![img](https://img-blog.csdnimg.cn/4626f6fd2e854d4b9502a9288f76077f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57yW56CB6K-t6ICFIERyYWdvbiBXdQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

![img](https://img-blog.csdnimg.cn/bb5628f6c6de4d82aa09a6f5049e8148.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57yW56CB6K-t6ICFIERyYWdvbiBXdQ==,size_18,color_FFFFFF,t_70,g_se,x_16)

## [Sentinel](https://so.csdn.net/so/search?q=Sentinel&spm=1001.2101.3001.7020)和Hystrix的区别

![img](https://img-blog.csdnimg.cn/839aae589d304e29b19f14d3c6c2f13a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA57yW56CB6K-t6ICFIERyYWdvbiBXdQ==,size_20,color_FFFFFF,t_70,g_se,x_16)

## sentinel可视化界面安装

![img](https://img-blog.csdnimg.cn/ccc95d2126b542dfa969f2bd95898cbe.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

 下载对应版本的sentinel的jar包，通过终端命令：

```bash
java -jar jar包名
```

启动

![img](https://img-blog.csdnimg.cn/65aeb045a9d04f88ae98fab589d6a69b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

访问对应路径：控制台如下：

![img](https://img-blog.csdnimg.cn/b116ab9d408f47beaffda3b0bc5a5b32.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

## 在[springcloudalibaba](https://so.csdn.net/so/search?q=springcloudalibaba&spm=1001.2101.3001.7020)中整合sentinel

添加依赖

```XML
        <!--sentinel启动器-->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
```

配置yml

```yml
server:
  port: 8002
 
spring:
  application:
    name: WXL-DEV-SERVICE-2
  cloud:
    sentinel:
      transport:
        dashboard: 127.0.0.1:8080
```

启动服务，再访问服务后，观察控制台：因为访问接口以后才会注册到sentinel当中。

![img](https://img-blog.csdnimg.cn/d53b098128e149989e08c3090614bbd3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

## 流控规则

- **实时监控，可用于查看接口访问情况**

![img](https://img-blog.csdnimg.cn/7171bda6e370487ea98a0c8daf5ffafc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

- **簇点链路，可以对对应的资源流控降级**

![img](https://img-blog.csdnimg.cn/55949e96ce564ae7bd9c3bfff9a8e3bc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

- **QPS流控**

![img](https://img-blog.csdnimg.cn/a7ed57bd0fe74a469e79e51fc769f868.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

可以看到当每秒超过2次时被流控：

![img](https://img-blog.csdnimg.cn/ba8a648182b64c519b61e81341f57af5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

 流控文字可自定义：

```java
    @GetMapping("/world")
    @SentinelResource(value = "helloWorld",blockHandlerClass = TestController.class,blockHandler = "helloBlock")
    public String helloWorld() {
        return "Hello world";
    }
 
    public static String helloBlock(BlockException e){
        return "你已被流控";
    }
```

value将该方法定义为sentinel的资源，blockHandlerClass指明流控处理的类，blockHandler是流控时调用的方法。

这里需要注意处理异常的方法必须是静态方法添加static, 并需要添加sentinel的异常参数BlockException。

![img](https://img-blog.csdnimg.cn/9bd607564d54484faf8472cad766b223.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

- **统一异常控制处理**

上面通过注解实现流控灵活性更高，对于需要统一管理的流控处理，我们可以通过统一异常处理来实现。可以自定义处理不同类型的限流。

只需实现对应接口即可，案例代码如下：

```java
package com.dragonwu.exception;
 
import com.alibaba.csp.sentinel.adapter.spring.webmvc.callback.BlockExceptionHandler;
import com.alibaba.csp.sentinel.slots.block.BlockException;
import com.alibaba.csp.sentinel.slots.block.authority.AuthorityException;
import com.alibaba.csp.sentinel.slots.block.degrade.DegradeException;
import com.alibaba.csp.sentinel.slots.block.flow.FlowException;
import com.alibaba.csp.sentinel.slots.block.flow.param.ParamFlowException;
import com.alibaba.csp.sentinel.slots.system.SystemBlockException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.http.MediaType;
import org.springframework.stereotype.Component;
 
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.util.HashMap;
import java.util.Map;
 
@Component
public class MyBlockExceptionHandler implements BlockExceptionHandler {
 
    @Override
    public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, BlockException e) throws Exception {
        System.out.println("BlockExceptioonHandler ++++++++++++++++++++++++++"+e.getRule());
 
        Map<Integer,String> hashMap=new HashMap<>();
 
        if(e instanceof FlowException){
            hashMap.put(100,"接口限流了");
        }else if(e instanceof DegradeException){
            hashMap.put(101,"服务降级了");
        }else if(e instanceof ParamFlowException){
            hashMap.put(102,"热点参数限流了");
        }else if(e instanceof SystemBlockException){
            hashMap.put(103,"触发系统保护规则了");
        }else if(e instanceof AuthorityException){
            hashMap.put(104,"授权规则不通过");
        }
 
        //返回json数据
        httpServletResponse.setStatus(500);
        httpServletResponse.setCharacterEncoding("utf-8");
        httpServletResponse.setContentType(MediaType.APPLICATION_JSON_VALUE);
        new ObjectMapper().writeValue(httpServletResponse.getWriter(),hashMap);
    }
}
```

- **线程流控**

![img](https://img-blog.csdnimg.cn/070dfb4b77a443f5a37fef6507b78887.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

- **关联限流**

![img](https://img-blog.csdnimg.cn/36ad6a0706fc45ad998cf565f8be61d2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

这里的意思是如果/hello/add接口一秒钟之内访问超过2次，则/hello/query会被限流。

- **熔断降级**

![img](https://img-blog.csdnimg.cn/b0d3387f7d08472e8762f6e109c8a030.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

 也要设置熔断时长，熔断时长过完之后会进入半开状态，即若下一次请求为慢请求则再次熔断，直到第一次请求不是慢请求才会恢复正常状态。

## OpenFeign整合Sentinel

导入依赖：

```xml
        <!--OpenFeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```

调用者开发整合配置：

```yml
feign:
  sentinel:
    enabled: true #开启openFeign对sentinel的整合
```

添加openFeign调用接口

```java
package com.wxl.feign;
 
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
 
@FeignClient(name = "WXL-DEV-SERVICE-2", path = "/hello",fallback = ServiceFailFeign.class)
public interface Service1HelloInterface {
    @GetMapping("/world")
    String helloWorld();
}
```

并且这里添加参数fallback值为失败时回调的实现类。

实现类如下:

```java
package com.wxl.feign;
 
import org.springframework.stereotype.Component;
 
@Component
public class ServiceFailFeign implements Service1HelloInterface{
    public String helloWorld() {
        return "降级了！！！";
    }
}
```

当接口请求失败时便会调用失败类里的该方法。

这里我们为了使用效果，在服务生产者的接口里故意写入报错代码：

```java
    @GetMapping("/world")
    public String helloWorld() {
        int i=1/0;
        return "Hello world";
    }
```

请求该消费者服务接口：

![img](https://img-blog.csdnimg.cn/4bee6d4b6395449686797428ff29c600.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)

调用了失败回调方法！ 

## 规则持久化

引入依赖

```XML
        <!--sentinel持久化存储-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
```

为nacos添加配置

![img](https://img-blog.csdnimg.cn/2f144a45ff4448ca89ac9299e1ce2470.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARHJhZ29uICAgV3U=,size_20,color_FFFFFF,t_70,g_se,x_16)