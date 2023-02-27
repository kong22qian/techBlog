[TOC]

# Starter make

## springboot [starter](https://so.csdn.net/so/search?q=starter&spm=1001.2101.3001.7020)简介

启动器starter命名:

> \#官方
> [spring-boot](https://so.csdn.net/so/search?q=spring-boot&spm=1001.2101.3001.7020)-starter-jdbc
> spring-boot-starter-web
> spring-boot-starter-freemarker
> \#第三方
> sms-spring-boot-starter
> myLog-spring-boot-starter

什么是SpringBoot starter机制:

> SpringBoot中的starter是一种非常重要的机制(自动化配置)，能够抛弃以前繁杂的配置，将其统一集成进starter，
> 应用者只需要在maven中引入starter依赖，SpringBoot就能自动扫描到要加载的信息并启动相应的默认配置。
> starter让我们摆脱了各种依赖库的处理，需要配置各种信息的困扰。SpringBoot会自动通过classpath路径下的类发现需要的Bean，
> 并注册进IOC容器。SpringBoot提供了针对日常企业应用研发各种场景的spring-boot-starter依赖模块。
> 所有这些依赖模块都遵循着约定成俗的默认配置，并允许我们调整这些配置，即遵循“约定大于配置”的理念。

为什么要自定义starter:

> 在我们的日常开发工作中，经常会有一些独立于业务之外的配置模块，我们经常将其放到一个特定的包下，
> 然后如果另一个工程需要复用这块功能的时候，需要将代码硬拷贝到另一个工程，重新集成一遍，麻烦至极。
> 如果我们将这些可独立于业务代码之外的功能配置模块封装成一个个starter，复用的时候只需要将其在pom中引用依赖即可，
> SpringBoot为我们完成自动装配，简直不要太爽

什么时候需要创建自定义starter:

> 在我们的日常开发工作中，可能会需要开发一个通用模块，以供其它工程复用。SpringBoot就为我们提供这样的功能机制，
> 我们可以把我们的通用模块封装成一个个starter，这样其它工程复用的时候只需要在pom中引用依赖即可，
> 由SpringBoot为我们完成自动装配。
>
> 常见场景：
> 1.通用模块-短信发送模块
> 2.基于AOP技术实现日志切面 
> 3.分布式雪花ID，Long-->string，解决精度问题
> jackson2/fastjson
> 4.微服务项目的数据库连接池配置
> 5.微服务项目的每个模块都要访问redis数据库，每个模块都要配置redisTemplate
> 也可以通过starter解决

## ssm短信启动器制作

### 创建Starter项目

​     ssm-spring-boot-starter

![img](https://img-blog.csdnimg.cn/191d2f3f63bd45dc84d86596d99fb97a.png)

该组件可用于注解（@Data）等 就不用再写setget的方法了

![img](https://img-blog.csdnimg.cn/dd5bdd3239894c569cfa5df962e3c2bf.png)

 创建项目成功之后

### 定义Starter需要的配置类

引入需要的pom依赖:

> <!--表示两个项目之间依赖不传递；不设置optional或者optional是false，表示传递依赖-->
> <!--例如：project1依赖a.jar(optional=true),project2依赖project1,则project2不依赖a.jar-->
> <dependency>
>    <groupId>org.springframework.boot</groupId>
>    <artifactId>spring-boot-configuration-processor</artifactId>
>    <optional>true</optional>
> </dependency>

 在新建的项目中 新建一个properties 的包 里面放

### SmsProperties 

```java

package com.cdl.ssmspringbootstarter.properties;
 
import org.springframework.boot.context.properties.ConfigurationProperties;
 
/**
 * @author cdl
 * @site www.cdl.com
 * @create 2022-11-03 16:57
 *
 *短信发送时，一般要提供短信发送凭证
 * spring:
 *     application:
 *         name: ssm-spring-boot-starter
 *当需要使用自定义sms的模块时，在yml文件中进行配置
 * spboot:
 *     sms:
 *         xxx（accessKeyId）: ssm-spring-boot-starter
 *
 * @ConfigurationProperties 此时是会报错的 需要在starer项目的配置类中解决
 */
@ConfigurationProperties(prefix = "spboot.sms")
public class SmsProperties {
 
    //短信发送接口的账号
    private String accessKeyId;//这个随便定义 但是要与xxx保持一致
    //短信发送接口的凭证
    private String accessKeySecret;
 
    public String getAccessKeyId() {
        return accessKeyId;
    }
 
    public void setAccessKeyId(String accessKeyId) {
        this.accessKeyId = accessKeyId;
    }
 
    public String getAccessKeySecret() {
        return accessKeySecret;
    }
 
    public void setAccessKeySecret(String accessKeySecret) {
        this.accessKeySecret = accessKeySecret;
    }
 
 
}
```

### 编写Starter项目的业务功能

新建一个service的包 放入业务代码

SmsService 

```java
package com.cdl.ssmspringbootstarter.service;
 
/**
 * @authorCDL
 * @site www.cdl.com
 *
 */
public interface SmsService {
    /**
     * 发送短信
     *
     * @param phone        要发送的手机号
     * @param signName     短信签名-在短信控制台中找
     * @param templateCode 短信模板-在短信控制台中找
     * @param data         要发送的内容
     */
    void send(String phone, String signName, String templateCode, String data);
}
```

SmsServiceImpl 

```java
package com.cdl.ssmspringbootstarter.service;
 
/**
 * 接入阿里短信接口的业务类（没有买阿里的服务 只是模拟一下）
 */
public class SmsServiceImpl implements SmsService {
 
    private String accessKeyId;//访问ID、即帐号
    private String accessKeySecret;//访问凭证，即密码
 
    public SmsServiceImpl(String accessKeyId, String accessKeySecret) {
        this.accessKeyId = accessKeyId;
        this.accessKeySecret = accessKeySecret;
    }
 
    @Override
    public void send(String phone, String signName, String templateCode, String data) {
//        调阿里的接口
        System.out.println("接入短信系统，accessKeyId=" + accessKeyId + ",accessKeySecret=" + accessKeySecret);
        System.out.println("短信发送，phone=" + phone + ",signName=" + signName + ",templateCode=" + templateCode + ",data=" + data);
    }
}
```

### 编写自动配置类

> 1. @Configuration：
>    定义一个配置类
> 2. @EnableConfigurationProperties：
>    @EnableConfigurationProperties注解的作用是@ConfigurationProperties注解生效。
>    如果只配置@ConfigurationProperties注解，在IOC容器中是获取不到properties配置文件转化的bean的

新建一个config的包 放配置类

SmsAutoConfig 

```java
package com.cdl.ssmspringbootstarter.config;
 
import com.cdl.ssmspringbootstarter.properties.SmsProperties;
import com.cdl.ssmspringbootstarter.service.SmsService;
import com.cdl.ssmspringbootstarter.service.SmsServiceImpl;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
 
/**
 * @author cdl
 * @site www.cdl.com
 * @create 2022-11-03 17:21
 */
@Configuration
@EnableConfigurationProperties({SmsProperties.class})
public class SmsAutoConfig {
 
    @Autowired
    private SmsProperties smsProperties;
 
    //将短信发送的业务类交给spring容器进行管理
    @Bean
    public SmsService smsService(){
        return new SmsServiceImpl(smsProperties.getAccessKeyId(),
                smsProperties.getAccessKeySecret());
    }
 
}
```

### 编写spring.factories文件加载自动配置类

在resources下新建META-INF文件夹，然后创建spring.factories文件

spring.factories

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.cdl.ssmspringbootstarter.config.SmsAutoConfig
```

> 注1：其中AutoConfig是starter配置文件的类限定名，多个之间逗号分割，还可以\进行转义即相当于去掉后面换行和空格符号  
>
> Auto Configure
>
> ​    org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
> ​    com.baomidou.mybatisplus.autoconfigure.MybatisPlusLanguageDriverAutoConfiguration,\
> ​    com.baomidou.mybatisplus.autoconfigure.MybatisPlusAutoConfiguration 

### 打包安装

> 打包时需要注意一下，SpringBoot项目打包的JAR是可执行JAR，它的类放在BOOT-INF目录下，
> 如果直接作为其他项目的依赖，会找不到类。可以通过修改pom文件来解决，代码如下：
> <plugin>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-maven-plugin</artifactId>
>     <configuration>
>         <classifier>exec</classifier>
>     </configuration>
> </plugin>

![img](https://img-blog.csdnimg.cn/68df9e09debc45358200ef3e7c2ab965.png)

![img](https://img-blog.csdnimg.cn/7db3eb8168e249cea4010a78f8c3d3da.png)

![img](https://img-blog.csdnimg.cn/fbf76e98587c41faaf41d5c7e2e3585d.png)

 右键 按步骤执行

![img](https://img-blog.csdnimg.cn/4ef65a7ed2174df782ae6724e4ceb6f8.png)

出现jar包

![img](https://img-blog.csdnimg.cn/8c0ad1a00dad4969a187f2c61aa32ed2.png)

## sms短信调用启动器starter测试

### 其它项目引用

新建一个项目

spingbootxy

勾选以下两个组件

![img](https://img-blog.csdnimg.cn/a778a04c89c843bd94615a787c328878.png)

 **添加pom依赖**

> #构建项目测试用的spboot项目，勾选组件lombok以及spring-boot-starter-web
> #添加自定义starter依赖如下
>
> <dependency>
>     <groupId>com.cdl</groupId>
>     <artifactId>ssm-spring-boot-starter</artifactId>
>     <version>0.0.1-SNAPSHOT</version>
> </dependency>

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.cdl</groupId>
    <artifactId>springbootxy</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springbootxy</name>
    <description>Demo project for Spring Boot</description>
 
    <properties>
        <java.version>1.8</java.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <spring-boot.version>2.3.7.RELEASE</spring-boot.version>
    </properties>
 
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
 
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
 
    <!--导入短信发送自定义的starter的启动器-->
        <dependency>
            <groupId>com.cdl</groupId>
            <artifactId>ssm-spring-boot-starter</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
 
 
    </dependencies>
 
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
 
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.7.RELEASE</version>
                <configuration>
                    <mainClass>com.cdl.springbootxy.SpringbootxyApplication</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <id>repackage</id>
                        <goals>
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
 
</project>
```

配置

application.yml

```yaml
server:
    port: 8080
spring:
    application:
        name: springbootxy
spboot:
    sms:
        access-key-id: xiaochen
        access-key-secret: 123456
```

调用jar中业务层的方法 写一个controller

SmsController

```java
package com.cdl.springbootxy.controller;
 
import com.cdl.ssmspringbootstarter.service.SmsService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
/**
 * @author cdl
 * @site www.cdl.com
 * @create 2022-11-03 17:58
 */
@RestController
public class SmsController {
 
    @Autowired
    private SmsService smsService;
    @RequestMapping("/sms/send")
    public String sendSms(){
        smsService.send("16607478549","签名","1000","你好自定义短信发送starter");
        return "success";
    }
 
}
```

在启动类运行，若启动类没有问题则是成功的

![img](https://img-blog.csdnimg.cn/0f87f6d4e58c4600ab6aca06df19228f.png)

 前端使用浏览器访问测试的controller 模仿发送

![img](https://img-blog.csdnimg.cn/7f94e1701a174025bfcc6e6ecf6f8e91.png)

 控制台

![img](https://img-blog.csdnimg.cn/2aecc10f101841759d2bdf25dda502a4.png)

 可见能够使用成功 该新项目中没有发送的业务代码