[TOC]

# Bean life cycle

## flow chart

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200512152716262.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NDU3MDc4,size_16,color_FFFFFF,t_70)

测试代码：

- 测试bean

```java
@Data
public class BeanLifeCycle implements InitializingBean, DisposableBean, BeanFactoryAware, BeanNameAware {

    private String name;

    public BeanLifeCycle() {
        System.out.println("BeanTestLifeCycle no parameters constructor");
    }

    public BeanLifeCycle(String name) {
        this.name = name;
        System.out.println("BeanTestLifeCycle parameters constructor");
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("execution BeanFactoryAware.setBeanFactory()");
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("execution BeanNameAware.setBeanName()");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("execution InitializingBean.afterPropertiesSet()");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("execution DisposableBean.destroy()");
    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("execution @PostConstruct()");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("execution @PreDestroy()");
    }

    public void initMethod() {
        System.out.println("execution @Bean.initMethod()");
    }

    public void destroyMethod() {
        System.out.println("execution @Bean.destroyMethod()");
    }

}
```

- 测试BeanPostProcessor

```java
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof BeanLifeCycle) {
            System.out.println("execute BeanLifeCycle BeanPostProcessor.postProcessBeforeInitialization()");
        }
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof BeanLifeCycle) {
            System.out.println("execute BeanLifeCycle BeanPostProcessor.postProcessAfterInitialization()");
        }
        return bean;
    }
}
```

- 配置Bean

```java
@Configuration
public class BeanConfig {

    @Bean(initMethod = "initMethod", destroyMethod = "destroyMethod")
    public BeanLifeCycle getBeanTestLifeCycle() {
        return new BeanLifeCycle();
    }
}
```

- 运行程序，查看打印结果

```java
BeanTestLifeCycle no parameters constructor
execution BeanNameAware.setBeanName()
execution BeanFactoryAware.setBeanFactory()
execution @PostConstruct()
execution InitializingBean.afterPropertiesSet()
execution @Bean.initMethod()
2020-05-12 14:37:12.815  INFO 15640 --- [           main] o.f.c.internal.license.VersionPrinter    : Flyway Community Edition 5.2.1 by Boxfuse
2020-05-12 14:37:12.842  INFO 15640 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
2020-05-12 14:37:13.133  INFO 15640 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
2020-05-12 14:37:13.139  INFO 15640 --- [           main] o.f.c.internal.database.DatabaseFactory  : Database: jdbc:mysql://172.31.9.185:3306/test (MySQL 5.6)
2020-05-12 14:37:13.237  INFO 15640 --- [           main] o.f.core.internal.command.DbValidate     : Successfully validated 1 migration (execution time 00:00.019s)
2020-05-12 14:37:13.251  INFO 15640 --- [           main] o.f.core.internal.command.DbMigrate      : Current version of schema `test`: 1
2020-05-12 14:37:13.253  INFO 15640 --- [           main] o.f.core.internal.command.DbMigrate      : Schema `test` is up to date. No migration necessary.
2020-05-12 14:37:13.444  INFO 15640 --- [           main] o.s.b.a.e.web.EndpointLinksResolver      : Exposing 2 endpoint(s) beneath base path '/actuator'
2020-05-12 14:37:13.556  INFO 15640 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2020-05-12 14:37:13.563  INFO 15640 --- [           main] com.boot.demo.DemoApplication            : Started DemoApplication in 7.369 seconds (JVM running for 7.908)
execution @PreDestroy()
execution DisposableBean.destroy()
execution @Bean.destroyMethod()
```

