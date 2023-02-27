[TOC]

# Spring Boot 自动装配原理

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0yXe43hcrdeQIo8XfEqgOlDw7ZSsj7Q6lW6H3A9wL1ib04G7OcsIGWQYA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

> - Spring Boot 通过 @EnableAutoConfiguration 注解开启自动配置，@EnableAutoConfiguration 注解可以帮助 SpringBoot 应用将所有符合条件的 @Configuration 配置都加载到 IOC 容器，就跟“八爪鱼”一样。
>
>   
>
> - 借助 Spring 框架工具类 SpringFactoriesLoader 的支持，可以智能的加载 META-INF/spring.factories 配置文件中注册的各种 XxxAutoConfiguration 类。
>
> 
>
> - 当某个 XxxAutoConfiguration 类满足其注解 @Conditional 指定的生效条件时，实例化该 XxxAutoConfiguration 类中定义的 Bean 注入 Spring 容器，就可以完成依赖框架的自动装配。

## Spring Boot 源码剖析

Spring Boot应用启动时会调用 SpringApplication.run(String... args) 方法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0y3hicfmFIwicBgKRIMAl9ZV37rhbGl0wrgDm1CUe8Ziax49rbPDbOKPyKw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)



run 方法内部会调用 refreshContext(context) 方法来完成自动装配的操作。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0yZnn1AJOCmTp74P2SFbgyDKvzge06kQYLHt7yl8yWAI3R3mv3GAWldA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

跟剥洋葱一样，通过 debug 的方式一层层往里跟，最终会调用ConfigurationClassParser 类来完成所有配置类的解析，其所有的解析逻辑在 parser.parse(candidates) 方法中。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0y0kJDtBZWbcggyeWwgeF02Uh0Wgia04EMkmN02VSicgESCwPsUjwbpcLg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

调用重载的 parse 方法来完成解析。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0yGus7PyWNINAjTsIzibDibKLe9t28LkOQyhW7e5UliaoWmHK8E0mJUdyAA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接着调用 processConfigurationClass 方法来完成配置类解析处理。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0yWOVNcR3Ll6Au2G1IdV0o8icmW3icIMczI3iaQMbOEgbPibOeyxYXDYyxuA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接着调用 doProcessConfigurationClass 方法，此方法是支持注解配置的核心逻辑处理，方法内部会调用 processImports(configClass, sourceClass, getImports(sourceClass), filter, true) 方法进行递归解析，获取导入的配置类（@Import）。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0y4DPFFvnFbPbSga6xS8icewTmp7cibPTJkEj02ibTWpJPvOgy0kZyPUu0g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

接着看看获取配置类的相关逻辑，此时会看到所有的 bean 都以导入的方式被加载进去。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0yPtdojIQrAzNgmwshSqibl3PXxNIja6wSmt3Z6GYdVIib09psXvAyRYibA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

继续 Debug，会调用 ConfigurationClassParser 中的 parse 方法中的最后一行,继续跟进 this.deferredImportSelectorHandler.process() 方法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0yjXD504F94V0pqtjO3BS7hv9ykzrFR6a6mUJeDUQSicgkwceLqZibgkIg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

继续 Debug，会调用 DeferredImportSelectorHandler 的 process 方法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0yRL2fZdEp4zjodI0uzviaicomPZRib2BEbnicX2J5O68q4c1tcGGk1X2CYQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

继续 Debug，会调用 DeferredImportSelectorGroupingHandler 的 processGroupImports 方法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0yv2DmNXsXLuP01ozWACmHaqQ7tFTe0zPfBLbVoduhyRWiagd4ibNV3IFA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

processGroupImports() 内部，会调用 grouping.getImports() 方法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0ytAme1L6NXZwnzic0Q5kficpdeGyT70sysicWLO83QE8l61tOpdQpbQEmA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

grouping.getImports()方法内部会调用 this.group.process方法。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0yuJawOTp9uImhbCfNc1cSvxfzbZaEicKNyzxEhL4HXibolohDwwIgwe8g/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

this.group.process 方法可以说是自动装配的重点了。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0yYhJvxVOJXtPMKqyaTRdBYdVNnoCQh8Gnq9apaoOqgFOl03vg33ufiaw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

getAutoConfigurationEntry 方法详细解释一下

```java
/**
 * Return the {@link AutoConfigurationEntry} based on the {@link AnnotationMetadata}
 * of the importing {@link Configuration @Configuration} class.
 * @param annotationMetadata the annotation metadata of the configuration class
 * @return the auto-configurations that should be imported
 */
protected AutoConfigurationEntry getAutoConfigurationEntry(AnnotationMetadata annotationMetadata) {
   if (!isEnabled(annotationMetadata)) {
      return EMPTY_ENTRY;
   }
   AnnotationAttributes attributes = getAttributes(annotationMetadata);
   // 通过 SpringFactoriesLoader 类提供的方法加载类路径中的 META-INF 目录下的 
   // spring.factories 文件中针对 EnableAutoConfiguration 的注册配置类
   List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
   // 对获得的注册配置类集合进行去重处理，防止多个项目引入同样的配置类
   configurations = removeDuplicates(configurations);
   // 获得注解中被 exclude 或 excludeName 所排除的类的集合
   Set<String> exclusions = getExclusions(annotationMetadata, attributes);
   // 检查被排除类是否可实例化，是否被自动注册配置所使用，不符合条件则抛出异常
   checkExcludedClasses(configurations, exclusions);
   // 从自动配置类集合中去除被排除的类
   configurations.removeAll(exclusions);
   // 检查配置类的注解是否符合 spring.factories 文件中 AutoConfigurationImportFilter 指定的注解检查条件
   configurations = getConfigurationClassFilter().filter(configurations);
   // 将筛选完成的配置类和排查的配置类构建为事件类，并传入监听器。监听器的配置在于 spring.factories 文件中
   // 通过 AutoConfigurationImportListener 指定
   fireAutoConfigurationImportEvents(configurations, exclusions);
   return new AutoConfigurationEntry(configurations, exclusions);
}
```

能够发现getCandidateConfigurations 方法中会通过 SpringFactoriesLoader 类来加载类路径中的 META-INF 目录下的 spring.factories 文件中针对 EnableAutoConfiguration 的注册配置类。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0y9Zw36YnE2O0gSKFTQsvhJgVrmezicJaZArRs34kLsIHSTymVKQMwGxA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

为了调试方便，在源码中的 process 方法上里加入了打印输出。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0youUSdibKAvFgojMVfatK35XGCUIGYrdHuU0buotMq1AeLNWogKfCkuw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

运行后，此时控制台输出如下。

```java
AutoConfigurationImportSelector.AutoConfigurationGroup.process() , entries = {org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.aop.AopAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.sql.init.SqlInitializationAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration=smoketest.simple.SampleSimpleApplication, org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration=smoketest.simple.SampleSimpleApplication}
```

## 例行回顾

本文采取 Debug 的方式跟了一下 Spring Boot 自动装配的源码，旨在感受一下自动装配的实现方式，其实这种自动装配的思想，在开发轮子时或许能够借鉴一下，会对轮子的扩展带来质的改变。

为了方便记忆，把 Spring Boot 自动装配繁琐的流程抽象一下。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWPSsdETLG4Erv9SST05dZ0yY57OBrFawoK51aMl07ZBZfgNBgYs5OOgRrLYLWsicIbNXh5E4uIWvhg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

另外 Spring Boot 自动装配源码 Debug 主线，感兴趣可以自行跟一下源码。

![图片](https://mmbiz.qpic.cn/mmbiz_png/waH0DGXhQWMFFRDdAnh4Vnrcus7yy4W7jadic1ibHmTcia4y5yIgxExicM5P1bm62WCbhHBkBscIBz9SVFxuicicUXyA/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)