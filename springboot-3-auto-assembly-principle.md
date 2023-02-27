[TOC]

# Springboot启动综述

## Spring Boot项目结构

以2.2.6.RELEASE为例：

![Spring Boot整体项目结构](https://img-blog.csdnimg.cn/4a887ba491584b23b25be8ebcb2c1023.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix55yL6Zuy55qE6Zuy,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

由图中可以看出，2.2.x.RELEASE版本由两个子模块构成：

spring-boot-project：Spring Boot核心项目代码，包含核心、工具、安全、文档、starters等项目
spring-boot-tests：Spring Boot部署及集成的测试

## [spring-boot](https://so.csdn.net/so/search?q=spring-boot&spm=1001.2101.3001.7020)-project项目结构

![项目结构详情](https://img-blog.csdnimg.cn/640cc927a5d44a6da44f8ae859145ded.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix55yL6Zuy55qE6Zuy,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

重点模块解析：

> spring-boot：Spring Boot核心代码，也是入口类SpringApplication类所在项目。
> spring-boot-autoconfigure：Spring Boot自动配置核心功能，默认集成了多种常见框架的自动配置类等。
> spring-boot-dependencies：依赖和插件的版本信息。
> spring-boot-devtools：开发者工具，提供热部署、实时加载、禁用缓存等提升开发效率的功能。
> spring-boot-starters：Spring Boot以预定义的方式集成了其他应用的starter集合。
> spring-boot-test：测试功能相关代码。

## Spring Boot自动配置运行原理

### 自动配置运行原理简介

Spring Boot自动配置运行原理如下图所示：

![自动配置运行原理](https://img-blog.csdnimg.cn/c6fc359b4d6641769081aad085f7f9f0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix55yL6Zuy55qE6Zuy,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

可以用一句话来描述整个过程：Spring Boot通过@EnableAutoConfiguration注解开启自动配置，加载spring.factories中注册的各种AutoConfiguration类，当某个AutoConfiguration类满足其注解@Conditional指定的生效条件（Starters提供的依赖、配置或Spring容器中是否存在某个Bean等）时，实例化该AutoConfiguration类中定义的Bean，并注入Spring容器，就可以完成依赖框架的自动配置。
简单介绍一下图中所示的组件：

> @EnableAutoConfiguration：该注解有组合注解@SpringBootApplication引入，完成自动配置开启，扫描各个jar包下的spring.factories文件，并加载文件中注册的AutoConfiguration类等。
> spring.factories：配置文件，位与jar包的META-INF目录下，按照指定格式注册了自动配置的AutoConfiguration类。spring.factories也可以包含其他类型待注册的类。该配置文件不仅存在于Spring Boot项目中，也可以存在于自定义的自动配置（或Starter）项目中。
> AutoConfiguration类：自动配置类，代表了Spring Boot中一类以XXAutoConfiguration命名的自动配置类。其中定义了三方组件集成Spring所需初始化的Bean和条件。
> @Conditional：条件注解及其衍生注解，在AutoConfiguration类上使用，当满足该条件注解时才会实例化AutoConfiguration类。
> Starters：三方组件的依赖及配置，Spring Boot已经预置的组件。Spring Boot默认的Starters项目往往只包含一个pom依赖的项目。如果是自定义的starter，该项目还需包含spring.factories文件、AutoConfiguration类和其他配置类。

### 自动配置源码解析之@EnableAutoConfiguration

> @EnableAutoConfiguration是开启自动配置的注解，它是由@SpringBootApplication引入的。

### 入口类和@[SpringBootApplication注解](https://so.csdn.net/so/search?q=SpringBootApplication注解&spm=1001.2101.3001.7020)

Spring Boot项目会生成一个 xxxApplication的入口类，如下所示：

```java
@SpringBootApplication
public class SpringBootDemoApplication {
	public static void main(String[] args) {
		SpringApplication.run(SpringBootDemoApplication.class, args);
	}
}
```


该类的main方法用于启动Spring Boot项目的入口。唯一的注解@SpringBootApplication，用于开启自动配置，准确地说是通过该注解内的@EnableAutoApplication开启了自动配置。

@SpringBootApplication源码如下：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {

	// 排除指定自动配置类
	@AliasFor(annotation = EnableAutoConfiguration.class)
	Class<?>[] exclude() default {};

	// 排除指定自动配置类名
	@AliasFor(annotation = EnableAutoConfiguration.class)
	String[] excludeName() default {};

	// 指定扫描的基础包，激活注解组件的初始化
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackages")
	String[] scanBasePackages() default {};

	// 指定扫描的类，用于初始化
	@AliasFor(annotation = ComponentScan.class, attribute = "basePackageClasses")
	Class<?>[] scanBasePackageClasses() default {};

	// 指定是否代理@Bean方法以强制执行bean的生命周期行为
	@AliasFor(annotation = Configuration.class)
	boolean proxyBeanMethods() default true;

}

```

> 该注解组合了@SpringBootConfiguration、@EnableAutoConfiguration和@ComponentScan。@SpringBootConfiguration继承自@Configuration，用于标注当前类是配置类，将当前类中声明的一个或多个以@Bean注解标记的方法的实例装配到Spring容器中。@ComponentScan用于根据定义的扫描路径，把符合扫描规则的类装配到Spring容器中。
>
> 该注解提供了一下成员属性：
> exclude：根据类（Class）排除指定的自动配置，该成员属性覆盖了@SpringBootApplication中组合的@EnableAutoConfiguration中定义的exclude成员属性。
> excludeName：根据类名排除指定的自动配置，覆盖了@EnableAutoConfiguration中的excludeName的成员属性。
> scanBasePackages：指定扫描的基础package，用于激活@Component等注解类的初始化。
> scanBasePackageClasses：扫描指定的类，用于组件的初始化。
> proxyBeanMethods：指定是否代理@Bean方法以强制执行bean的生命周期行为。此功能需要通过运行时生成CGLIB子类来实现方法拦截。
>
> 该注解中还大量使用@AliasFor注解，该注解用于桥接到其他注解，该注解的属性指定了所桥接的注解类。这些属性在其他注解中已经定义过了，之所以使用@AliasFor注解并重新在@SpringBootApplication中定义，更多是为了减少用户使用多注解带来的麻烦。

### @EnableAutoConfiguration注解解析

未使用Spring Boot的情况下，Bean的生命周期由Spring来管理，然而Spring无法自动配置@Configuration注解的类，Spring Boot的核心功能之一就是根据约定自动管理该注解标注的类。@EnableAutoConfiguration的主要功能是启动Spring应用程序上下文时进行自动配置，它会尝试猜测并配置项目可能需要的Bean。自动配置通常是基于项目classpath中引入的类和已定义的Bean来实现的。

@EnableAutoConfiguration注解源码如下：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {

  	// 用于覆盖配置开启或关闭自动配置的功能
	String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

	// 根据类（Class）排除指定的自动配置
	Class<?>[] exclude() default {};

	// 根据类名排除指定的自动配置
	String[] excludeName() default {};

}
```

@EnableAutoConfiguration会猜测需要使用的Bean，但如果在项目中并不需要它预置初始化的Bean，可通过该注解的exclude或excludeName参数进行有针对性的排除。比如当不需要数据库自动配置时，可通过以下方式排除：

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class SpringBootDemoApplication {
}
```

### AutoConfigurationImportSelector解析

@EnableAutoConfiguration的关键功能是通过@Import注解导入的ImportSelector接口来完成的，@Import(AutoConfigurationImportSelector.class)是自动配置功能的核心实现。

### @Import注解解析

@Import注解主要提供导入配置类的功能，通过@Import可以引入@Configuration注解的类，也可以导入实现了ImportSelector接口或ImportBeanDefinitionRegistrar的类。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {
    Class<?>[] value();
}
```

### ImportSelector接口解析

@Import的功能基本都需要借助ImportSelector接口来实现，ImportSelector接口决定可引入哪些@Configuration。ImportSelector是Spring中的接口，源码如下：

```java
public interface ImportSelector {
    String[] selectImports(AnnotationMetadata var1);

    @Nullable
    default Predicate<String> getExclusionFilter() {
        return null;
    }
}
```

mportSelector接口只提供了一个参数为AnnotationMetadata的方法，返回的结果为一个字符串数组。其中参数AnnotationMetadata内包含了被@Import注解的类的注解信息。在selectImports方法内可根据具体实现决定返回哪些配置类的全限定名，将结果以字符串数组的形式返回。

AutoConfigurationImportSelector部分源码如下：

```java
public class AutoConfigurationImportSelector implements DeferredImportSelector, BeanClassLoaderAware,
		ResourceLoaderAware, BeanFactoryAware, EnvironmentAware, Ordered {
    }
```

AutoConfigurationImportSelector并没有直接实现了ImportSelector接口，而是实现了它的子接口DeferredImportSelector，DeferredImportSelector会在所有的@Configuration类加载完成之后再加载返回的配置类，而ImportSelector是在加载完@Configuration类之前先去加载返回的配置类。
AutoConfigurationImportSelector还实现了BeanClassLoaderAware、ResourceLoaderAware、BeanFactoryAware、EnvironmentAware，Spring会保证在调用ImportSelector之前会先调用Aware接口的方法。

### AutoConfigurationImportSelector功能概述

AutoConfigurationImportSelector的核心功能和流程图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/cea23ecac5c24173bef0269a1d6655d5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix55yL6Zuy55qE6Zuy,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

当AutoConfigurationImportSelector被@Import注解引入之后，它的selectImports方法会被调用并执行其实现的[自动装配](https://so.csdn.net/so/search?q=自动装配&spm=1001.2101.3001.7020)逻辑。读者朋友需注意，selectImports方法几乎涵盖了组件自动装配的所有处理逻辑。源码如下：

```java
	/**
   	 * AnnotationMetadata包含了被@Import注解的类的注解信息
   	 * 返回值为配置类的全限定名字符串数组
   	 */
	public String[] selectImports(AnnotationMetadata annotationMetadata) {
    	// 检查自动配置功能是否开启，默认为开启
		if (!isEnabled(annotationMetadata)) {
			return NO_IMPORTS;
		}
    	// 加载自动配置的元信息，配置文件为类路径中META-INF目录下的spring-autoconfigure-metadata.properties文件
		AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader
				.loadMetadata(this.beanClassLoader);
    	// 封装将被引入的自动配置信息
		AutoConfigurationEntry autoConfigurationEntry = getAutoConfigurationEntry(autoConfigurationMetadata,
				annotationMetadata);
    	// 返回符合条件的配置类的全限定名数组
		return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
	}

```

### AutoConfigurationImportSelector代码分析详解

#### isEnabled方法

该方法用来检查自动配置功能是否开启，默认为开启状态

```java
/**
 * 判断是否开启自动配置功能
 */
protected boolean isEnabled(AnnotationMetadata metadata) {
		/**
		 * 如果当前类为AutoConfigurationImportSelector, 程序会从环境中获取key为EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY的配置,
		 * 该常量的值为spring.boot.enableautoconfiguration. 如果获取不到该属性的配置, 返回默认值true, 也就是默认会使用自动配置
		 */
		if (getClass() == AutoConfigurationImportSelector.class) {
			return getEnvironment().getProperty(EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY, Boolean.class, true);
		}
		// 如果当前类为其他类, 直接返回ture
		return true;
	}
```

从代码中可以看出，默认是会使用自动配置的。如果想覆盖或重置EnableAutoConfiguration.ENABLED_OVERRIDE_PROPERTY的配置, 可在application.properties或application.yml中针对参数spring.boot.enableautoconfiguration进行配置，如在application.properties配置关闭自动配置, 则设置

> spring.boot.enableautoconfiguration=false。

#### loadMetadata方法

该方法会加载自动配置的元数据信息，使用的是AutoConfigurationMetadataLoader，从类名可以看出，是自动配置元数据加载器。

```java
// 自动配置元数据加载
final class AutoConfigurationMetadataLoader {

	// 默认加载元数据的路径
	protected static final String PATH = "META-INF/spring-autoconfigure-metadata.properties";

	private AutoConfigurationMetadataLoader() {
	}

	// 加载元数据, 默认加载PATH(即类路径下META-INF/spring-autoconfigure-metadata.properties)中的配置
	static AutoConfigurationMetadata loadMetadata(ClassLoader classLoader) {
		return loadMetadata(classLoader, PATH);
	}

	static AutoConfigurationMetadata loadMetadata(ClassLoader classLoader, String path) {
		try {
			// 获取数据并且存储于Enumeration中
			Enumeration<URL> urls = (classLoader != null) ? classLoader.getResources(path)
					: ClassLoader.getSystemResources(path);
			Properties properties = new Properties();
			while (urls.hasMoreElements()) {
				// 遍历Enumeration中的URL, 加载其中的属性, 存储到Properties
				properties.putAll(PropertiesLoaderUtils.loadProperties(new UrlResource(urls.nextElement())));
			}
			return loadMetadata(properties);
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load @ConditionalOnClass location [" + path + "]", ex);
		}
	}
  
  	// 创建AutoConfigurationMetadata的实现类PropertiesAutoConfigurationMetadata
	static AutoConfigurationMetadata loadMetadata(Properties properties) {
		return new PropertiesAutoConfigurationMetadata(properties);
	}

	// AutoConfigurationMetadata的内部实现类
	private static class PropertiesAutoConfigurationMetadata implements AutoConfigurationMetadata {
		...
  }
}
```

在上述代码中，AutoConfigurationMetadataLoader调用loadMetadata方法，获取默认变量PATH指定的文件，然后加载并存储在Enumeration数据机构中。随后，从文件中获取其中配置的数据存储于Properties内，最终调用在该类内部实现的AutoConfigurationMetadata的子类的构造方法。

spring-autoconfigure-metadata.properties文件内的配置格式如下:
`自动配置类的全限定名.注解名称=值(如果文件内有多个值, 就用英文逗号隔开)`

加载元数据是为了后续的过滤自动配置使用，Spring Boot使用一个Annotation的处理器来收集自动加载的条件，这些条件可以在元数据文件进行配置。Spring Boot会将收集好的@Configuration进行一次性过滤, 进而剔除不满足条件的配置类。官方文档显示，使用这种配置方式可以有效缩短Spring Boot的启动时间，减少@Configuration类的数量，从而减少初始化Bean的耗时。由于这块笔者还未接触到这块优化，对这个解释暂不清楚。

#### getAutoConfigurationEntry方法

该方法用于封装将要引入的自动配置信息，也是自动配置的核心。

```java
	/**
	 * 封装将要引入的自动配置信息
	 */
	protected AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata,
			AnnotationMetadata annotationMetadata) {
    	// 再次检查自动配置功能是否开启
		if (!isEnabled(annotationMetadata)) {
			return EMPTY_ENTRY;
		}
    	// 获取自动装配元数据信息的属性
		AnnotationAttributes attributes = getAttributes(annotationMetadata);
    	// 通过SpringFactoriesLoader类提供的方法加载类路径中META-INF目录下的spring.factories文件中针对EnableAutoConfiguration的注册配置类
		List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes);
    	// 对获得的注册配置类集合进行去重处理，防止多个项目引入同样的配置类
		configurations = removeDuplicates(configurations);
    	// 获取注解中被exclude或excludeName锁排除的类的集合
		Set<String> exclusions = getExclusions(annotationMetadata, attributes);
    	// 检查被排除类是否可实例化，是否被自动注册配置所使用，不符合条件则抛出异常
		checkExcludedClasses(configurations, exclusions);
    	// 从自动配置类集合中去除被排除的类
		configurations.removeAll(exclusions);
    	// 检查配置类的注解是否符合spring.factories文件中AutoConfigurationImportFilter指定的注解检查条件
		configurations = filter(configurations, autoConfigurationMetadata);
    	// 将筛选完成的配置类和排查的配置类构建为事件类，并传入监听器。监听器的配置在spring.factories文件中，通过AutoConfigurationImportListener指定
    	// 并将事件类进行广播
		fireAutoConfigurationImportEvents(configurations, exclusions);
		return new AutoConfigurationEntry(configurations, exclusions);
	}
```

##### getAttributes：获取自动装配元数据信息的属性

该方法会获取注解@EnableAutoConfiguration上的属性，用于后续排除部分组件。

```java
protected AnnotationAttributes getAttributes(AnnotationMetadata metadata) {
  		// 获取注解@EnableAutoConfiguration的类名
		String name = getAnnotationClass().getName();
    	// 获取注解@EnableAutoConfiguration上的属性
		AnnotationAttributes attributes = AnnotationAttributes.fromMap(metadata.getAnnotationAttributes(name, true));
		Assert.notNull(attributes, () -> "No auto-configuration attributes found. Is " + metadata.getClassName()
				+ " annotated with " + ClassUtils.getShortName(name) + "?");
		return attributes;
	}

	protected Class<?> getAnnotationClass() {
		return EnableAutoConfiguration.class;
	}
```

##### getCandidateConfigurations：加载自动配置组件

自动配置组件在类路径中META-INF目录下的spring.factories文件中进行注册，getCandidateConfigurations方法通过Spring Core提供的SpringFactoriesLoader类可以读取spring.factories文件中注册的类。

```java
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
  		// loadFactoryNames会读取META-INF/spring.factories中的配置
  		// 由于第一个参数为EnableAutoConfiguration.class，则只会读取配置文件中针对自动配置的注册类
		List<String> configurations = SpringFactoriesLoader.loadFactoryNames(getSpringFactoriesLoaderFactoryClass(),
				getBeanClassLoader());
		Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you "
				+ "are using a custom packaging, make sure that file is correct.");
		return configurations;
	}

	protected Class<?> getSpringFactoriesLoaderFactoryClass() {
		return EnableAutoConfiguration.class;
	}
```

SpringFactoriesLoader中loadFactoryNames方法部分代码：

```java
public final class SpringFactoriesLoader {

	// 该类加载文件的路径，可能存在多个
	public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";

	...

	// 加载所有的META-INF/spring.factories文件，封装成Map，并根据指定类名进行筛选，获取特定的列表
	public static List<String> loadFactoryNames(Class<?> factoryType, @Nullable ClassLoader classLoader) {
		String factoryTypeName = factoryType.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryTypeName, Collections.emptyList());
	}

  	// 加载所有的META-INF/spring.factories文件，封装成Map，Key为接口的全类名，Value为对应配置值的List集合
	private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		...
	}
```

spring.factories文件中EnableAutoConfiguration配置的部分内容如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/dca838320cae4c98b2da2eb685ea8325.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix55yL6Zuy55qE6Zuy,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

由于程序默认加载的是ClassLoader下面的所有META-INF/spring.factories文件中的配置，所以难免在不同的jar包中出现重复的配置，则需要进行去重操作。

##### removeDuplicates：自动配置组件去重

去重很简单，通过Set集合去重。

```java
protected final <T> List<T> removeDuplicates(List<T> list) {
		return new ArrayList<>(new LinkedHashSet<>(list));
}
```

##### getExclusions：获取需要排除的组件

前面介绍过可以通过配置@EnableAutoConfiguration的注解属性exclude或excludeName进行排除指定组件。

```java
protected Set<String> getExclusions(AnnotationMetadata metadata, AnnotationAttributes attributes) {
		Set<String> excluded = new LinkedHashSet<>();
		// 获取@EableAutoConfiguration注解中配置的exclude属性值
		excluded.addAll(asList(attributes, "exclude"));
		// 获取注解中配置的excludeName属性值
		excluded.addAll(Arrays.asList(attributes.getStringArray("excludeName")));
		// 获取配置文件中key为spring.autoconfigure.exclude的配置值
		excluded.addAll(getExcludeAutoConfigurationsProperty());
		return excluded;
}

private List<String> getExcludeAutoConfigurationsProperty() {
		if (getEnvironment() instanceof ConfigurableEnvironment) {
			Binder binder = Binder.get(getEnvironment());
			// PROPERTY_NAME_AUTOCONFIGURE_EXCLUDE的值为spring.autoconfigure.exclude
			return binder.bind(PROPERTY_NAME_AUTOCONFIGURE_EXCLUDE, String[].class).map(Arrays::asList)
					.orElse(Collections.emptyList());
		}
		String[] excludes = getEnvironment().getProperty(PROPERTY_NAME_AUTOCONFIGURE_EXCLUDE, String[].class);
		return (excludes != null) ? Arrays.asList(excludes) : Collections.emptyList();
	}
```

##### checkExcludedClasses：对排除的组件检查

```java
	private void checkExcludedClasses(List<String> configurations, Set<String> exclusions) {
  		// 有问题的排除类集合
		List<String> invalidExcludes = new ArrayList<>(exclusions.size());
  		// 遍历并判断是否存在对应的配置类
		for (String exclusion : exclusions) {
			if (ClassUtils.isPresent(exclusion, getClass().getClassLoader()) && !configurations.contains(exclusion)) {
				invalidExcludes.add(exclusion);
			}
		}
  		// 如果不为空， 则进行处理
		if (!invalidExcludes.isEmpty()) {
			handleInvalidExcludes(invalidExcludes);
		}
	}
	// 抛出异常
	protected void handleInvalidExcludes(List<String> invalidExcludes) {
		StringBuilder message = new StringBuilder();
		for (String exclude : invalidExcludes) {
			message.append("\t- ").append(exclude).append(String.format("%n"));
		}
		throw new IllegalStateException(String.format(
				"The following classes could not be excluded because they are not auto-configuration classes:%n%s",
				message));
	}
```

checkExcludedClasses方法用来确保被排除的类存在于当前ClassLoader中，并且包含在spring.factories注册的集合中。如果不满足这些条件，调用handleInvalidExcludes方法抛出异常。
如果被排除的类符合条件，调用configurations.removeAll(exclusions)方法从自动配置集合中移除被排除类的集合，完成初步的自动配置组件排除。

##### filter：过滤自动配置组件

```java
   /**
	* 对自动配置类进行再次过滤
    * configurations：经过初次过滤之后的自动配置组件列表
    * autoConfigurationMetadata：元数据文件META-INF/spring-autoconfigure-metadata.properties中配置的实体类
	*/
	private List<String> filter(List<String> configurations, AutoConfigurationMetadata autoConfigurationMetadata) {
		long startTime = System.nanoTime();
    	// 候选自动配置组件
		String[] candidates = StringUtils.toStringArray(configurations);
		boolean[] skip = new boolean[candidates.length];
		boolean skipped = false;
		// 遍历过滤器集合
		for (AutoConfigurationImportFilter filter : getAutoConfigurationImportFilters()) {
      		// 判断是否为Aware的相关子类
			invokeAwareMethods(filter);
      		// 调用match判断是否匹配，参数1为待过滤的自动配置类数组，参数2为自动配置的元数据信息。
      		// 返回的结果为匹配过滤后的结果布尔数组，数组的大小与待过滤的自动配置类数组一致，如果需要排除，则设置对应的值为false
			boolean[] match = filter.match(candidates, autoConfigurationMetadata);
			for (int i = 0; i < match.length; i++) {
        		// 如果不匹配，则该索引位置置为空并设置跳过
				if (!match[i]) {
					skip[i] = true;
					candidates[i] = null;
					skipped = true;
				}
			}
		}
		if (!skipped) {
			return configurations;
		}
		List<String> result = new ArrayList<>(candidates.length);
    	// 将不跳过的自动配置组件生成新的集合返回
		for (int i = 0; i < candidates.length; i++) {
			if (!skip[i]) {
				result.add(candidates[i]);
			}
		}
		if (logger.isTraceEnabled()) {
			int numberFiltered = configurations.size() - result.size();
			logger.trace("Filtered " + numberFiltered + " auto configuration class in "
					+ TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startTime) + " ms");
		}
		return new ArrayList<>(result);
	}

	// 获取Filter列表
	protected List<AutoConfigurationImportFilter> getAutoConfigurationImportFilters() {
    // 跟前面类似，通过SpringFactoriesLoader的loadFactories方法加载文件META-INF/spring.factories中配置的key为AutoConfigurationImportFilter的过滤器
		return SpringFactoriesLoader.loadFactories(AutoConfigurationImportFilter.class, this.beanClassLoader);
	}
```

META-INF/spring.factories文件中相关过滤器如下：

```java
# Auto Configuration Import Filters
org.springframework.boot.autoconfigure.AutoConfigurationImportFilter=\
org.springframework.boot.autoconfigure.condition.OnBeanCondition,\
org.springframework.boot.autoconfigure.condition.OnClassCondition,\
org.springframework.boot.autoconfigure.condition.OnWebApplicationCondition
```

简述该filter方法的过程为：对自动配置组件列表再次过滤，过滤条件为该列表中自动配置类的注解得包含在OnBeanCondition、OnClassCondition、OnWebApplicationCondition中指定的注解，依次包含@ConditionalOnBean、@ConditionalOnClass和@ConditionalOnWebApplication。

##### match方法代码：

接口的match方法主要由其抽象子类FilteringSpringBootCondition实现，但是在实现时又定义了新的抽象方法getOutcomes。

```java
public boolean[] match(String[] autoConfigurationClasses, AutoConfigurationMetadata autoConfigurationMetadata) {
		ConditionEvaluationReport report = ConditionEvaluationReport.find(this.beanFactory);
    	// 进行匹配筛选，返回匹配结果
		ConditionOutcome[] outcomes = getOutcomes(autoConfigurationClasses, autoConfigurationMetadata);
    	// 将匹配结果转换成布尔数组
		boolean[] match = new boolean[outcomes.length];
		for (int i = 0; i < outcomes.length; i++) {
			match[i] = (outcomes[i] == null || outcomes[i].isMatch());
			if (!match[i] && outcomes[i] != null) {
				logOutcome(autoConfigurationClasses[i], outcomes[i]);
				if (report != null) {
					report.recordConditionEvaluation(autoConfigurationClasses[i], this, outcomes[i]);
				}
			}
		}
		return match;
	}
	// 过滤器核心功能，该方法由子类实现
	protected abstract ConditionOutcome[] getOutcomes(String[] autoConfigurationClasses,
			AutoConfigurationMetadata autoConfigurationMetadata);
```

##### OnClassCondition中的getOutcomes方法实现

```java
protected final ConditionOutcome[] getOutcomes(String[] autoConfigurationClasses,
			AutoConfigurationMetadata autoConfigurationMetadata) {
		// 如果有多个处理器，采用后台线程处理
		if (Runtime.getRuntime().availableProcessors() > 1) {
			return resolveOutcomesThreaded(autoConfigurationClasses, autoConfigurationMetadata);
		}
		else {
			OutcomesResolver outcomesResolver = new StandardOutcomesResolver(autoConfigurationClasses, 0,
					autoConfigurationClasses.length, autoConfigurationMetadata, getBeanClassLoader());
			return outcomesResolver.resolveOutcomes();
		}
	}
```

整个过滤过程如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/62ff536d273b423bb15a51c79b908cfd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix55yL6Zuy55qE6Zuy,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

##### fireAutoConfigurationImportEvents：事件注册并广播

在完成以上步骤的过滤、筛选后，获得了需要进行自动配置的类集合。在将该集合返回之前，需要对相关时间进行封装和广播。

```java
private void fireAutoConfigurationImportEvents(List<String> configurations, Set<String> exclusions) {
    	// 通过SpringFactoriesLoader提供的loadFactories方法将spring.factories中配置的接口AutoConfigurationImportListener的实现类加载出来
		List<AutoConfigurationImportListener> listeners = getAutoConfigurationImportListeners();
		if (!listeners.isEmpty()) {
      		// 将筛选出来的自动配置类集合和被排除的自动配置类集合封装成AutoConfigurationImportEvent事件对象
			AutoConfigurationImportEvent event = new AutoConfigurationImportEvent(this, configurations, exclusions);
			for (AutoConfigurationImportListener listener : listeners) {
				invokeAwareMethods(listener);
        		// 将这些事件对象通过监听器提供的onAutoConfigurationImportEvent方法对事件进行广播
				listener.onAutoConfigurationImportEvent(event);
			}
		}
	}

	protected List<AutoConfigurationImportListener> getAutoConfigurationImportListeners() {
		return SpringFactoriesLoader.loadFactories(AutoConfigurationImportListener.class, this.beanClassLoader);
	}
```

## SpringApplication实例化

### SpringApplication实例化简介

常见启动入口类示例：

```java
@SpringBootApplication(exclude = DataSourceAutoConfiguration.class)
public class SpringBootDemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootDemoApplication.class, args);
	}

}
```

启动入口类中，主要通过SpringApplication的静态方法-run方法进行SpringApplication类的实例化操作，然后再使用[实例化对象](https://so.csdn.net/so/search?q=实例化对象&spm=1001.2101.3001.7020)调用另一个run方法来完成整个项目的初始化和启动。

SpringApplication源码中run方法：

```java
// 参数primarySource为加载的主要资源类，通常就是Spring Boot的入口类
// 参数args为传递给应用程序的参数信息
public static ConfigurableApplicationContext run(Class<?> primarySource, String... args) {
		return run(new Class<?>[] { primarySource }, args);
	}

	public static ConfigurableApplicationContext run(Class<?>[] primarySources, String[] args) {
		return new SpringApplication(primarySources).run(args);
	}
```

### SpringApplication实例化流程

核心流程如图所示：

![实例化流程](https://img-blog.csdnimg.cn/d186a5a3188f4849afca5cf2c366ef11.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix55yL6Zuy55qE6Zuy,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

从图中可以看出，在SpringApplication对象实例化的过程中，主要做了3件事：参数赋值给成员变量、应用类型及方法推断、ApplicationContext相关内容加载及实例化。源码如下：

```java
	public SpringApplication(Class<?>... primarySources) {
		this(null, primarySources);
	}

	@SuppressWarnings({ "unchecked", "rawtypes" })
	public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
    	// 赋值成员变量resourceLoader
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
    	// 赋值成员变量primarySources
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    	// 推断Web应用类型
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
    	// 加载并初始化ApplicationContextInitializer及相关实现类
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
    	// 加载并初始化ApplicationListener及相关实现类
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
    	// 推断main方法Class类
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

SpringApplication的核心构造方法有两个参数，第一个为ResourceLoader，ResourceLoader为资源加载的接口，默认采用的是DefaultResourceLoader。第二个为Class<?>… primarySources，默认传入Spring Boot入口类。作为项目引导类，此参数传入的类需要满足一个条件，就是被注解@EnableAutoConfiguration或其组合注解标注。

#### 推断Web应用类型

赋值完成员变量后，接下来是推断Web应用类型，源码如下：

```java
public enum WebApplicationType {

	// 非Web应用
	NONE,

	// 基于SERVLET的Web类型
	SERVLET,

	// 基于REACTIVE的Web类型
	REACTIVE;

	private static final String[] SERVLET_INDICATOR_CLASSES = { "javax.servlet.Servlet",
			"org.springframework.web.context.ConfigurableWebApplicationContext" };

	private static final String WEBMVC_INDICATOR_CLASS = "org.springframework.web.servlet.DispatcherServlet";

	private static final String WEBFLUX_INDICATOR_CLASS = "org.springframework.web.reactive.DispatcherHandler";

	private static final String JERSEY_INDICATOR_CLASS = "org.glassfish.jersey.servlet.ServletContainer";

	private static final String SERVLET_APPLICATION_CONTEXT_CLASS = "org.springframework.web.context.WebApplicationContext";

	private static final String REACTIVE_APPLICATION_CONTEXT_CLASS = "org.springframework.boot.web.reactive.context.ReactiveWebApplicationContext";

  	// 基于classpath中类是否存在来进行类型推断，即判断指定的类是否存在于classpath下，根据判断结果进行组合推断应用类型
	static WebApplicationType deduceFromClasspath() {
    	// isPresent的核心机制是通过反射创建指定的类，根据在创建过程中是否抛出异常来判断该类是否存在
		if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null) && !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
				&& !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
      	// 当DispatcherHandler存在，并且DispatcherServlet和ServletContainer都不存在，则Web类型为REACTIVE
			return WebApplicationType.REACTIVE;
		}
		for (String className : SERVLET_INDICATOR_CLASSES) {
      	// 当Servlet或ConfigurableWebApplicationContext不存在时，则为非Web应用
			if (!ClassUtils.isPresent(className, null)) {
				return WebApplicationType.NONE;
			}
		}
    	// 走到这里说明，Servlet和ConfigurableWebApplicationContext都存在
		return WebApplicationType.SERVLET;
	}
}
```

#### ApplicationContextInitializer加载

Spring Boot使用的容器为ConfigurableApplicationContext，ApplicationContextInitializer是Spring IOC容器提供的一个接口，它是一个回调接口，主要用于用户在ConfigurableApplicationContext类型（或其子类型）的ApplicationContext做refresh方法调用刷新之前，对ConfigurableApplicationContext实例做进一步的设置或处理。

ApplicationContextInitializer接口源码为：

```java
public interface ApplicationContextInitializer<C extends ConfigurableApplicationContext> {
    void initialize(C var1);
}
```

ApplicationContextInitializer接口的initialize方法主要是为了初始化指定的应用上下文。而对应的上下文由参数传入，参数为ConfigurableApplicationContext的子类。

ApplicationContextInitializer的加载分为两个步骤：获取相关实例和设置实例。对应的方法分别为getSpringFactoriesInstances、setInitializers，getSpringFactoriesInstances源码如下：
```java
	private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
		return getSpringFactoriesInstances(type, new Class<?>[] {});
	}

	/**
	 * 用来获取spring.factories配置文件中的相关类, 并进行实例化操作
	 */
	private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
		// 获取类加载器
		ClassLoader classLoader = getClassLoader();
		// 加载META-INF/spring.factories中对应的配置, 并将结果存储于Set中, 方便去重
		Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		// 创建实例
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
		// 排序
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}

	/**
	 * 实例化注册类
	 */
	@SuppressWarnings("unchecked")
	private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes,
			ClassLoader classLoader, Object[] args, Set<String> names) {
		List<T> instances = new ArrayList<>(names.size());
		// 遍历加载到的类名(全限定类名)
		for (String name : names) {
			try {
				// 使用全路径类名通过反射获取class对象
				Class<?> instanceClass = ClassUtils.forName(name, classLoader);
				Assert.isAssignable(type, instanceClass);
				// 获取有参构造器
				Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
				// 执行构造函数获取实例
				T instance = (T) BeanUtils.instantiateClass(constructor, args);
				// 将实例加入到返回结果中
				instances.add(instance);
			}
			catch (Throwable ex) {
				throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, ex);
			}
		}
		return instances;
	}
```

getSpringFactoriesInstances方法依然是通过SpringFactoriesLoader类中的loadFactoryNames方法来获得META-INF/spring.factories文件中注册的对应配置，获取到这些配置类的全限定类名之后，通过反射生成实例。ApplicationContextInitializer的相关配置如下：

```
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
org.springframework.boot.context.ContextIdApplicationContextInitializer,\
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
org.springframework.boot.rsocket.context.RSocketPortInfoApplicationContextInitializer,\
org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer
```

在完成配置类集合和实例化操作之后，调用setInitializers方法将实例化的集合添加到SpringApplication的成员变量initializers中，源码如下：

```java
public void setInitializers(Collection<? extends ApplicationContextInitializer<?>> initializers) {
		// 新建一个List，并将其复制给SpringApplication的成员变量initializers
		this.initializers = new ArrayList<>(initializers);
	}
```

#### [ApplicationListener](https://so.csdn.net/so/search?q=ApplicationListener&spm=1001.2101.3001.7020)加载

完成了ApplicationContextInitializer的加载之后，便会进行ApplicationListener的加载。它的常见应用场景为：当容器初始化完成之后，需要处理一些如数据的加载、初始化缓存、特定任务的注册等操作。在此阶段，更多的是用于ApplicationContext管理Bean过程的场景。

Spring事件传播机制是基于观察者模式实现的。比如，在ApplicationContext管理Bean生命周期的过程中，会将一些改变定义为事件（ApplicationEvent），ApplicationContext通过ApplicationListener监听ApplicationEvent，当事件被发布之后，ApplicationListener用来对事件做出具体的操作。

ApplicationListener的这个配置和加载流程和ApplicationContextInitializer完全一致，也是通过SpringFactoriesLoader的loadFactoryNames方法获得META-INF/spring.factories中对应的配置，然后再进行实例化，这里不再赘述。

看一下ApplicationListener使用，ApplicationListener源码如下：

```java
@FunctionalInterface
public interface ApplicationListener<E extends ApplicationEvent> extends EventListener {
  	// onApplicationEvent方法用于处理应用程序事件，参数event为ApplicationEvent的子类，是具体接收到的事件
    void onApplicationEvent(E event);
}
```

从源码中可以看出，ApplicationListener接口和ApplicationEvent类配合使用，如果容器中存在ApplicationListener的Bean，当ApplicationContext调用publishEvent方法时，发布对应的事件，对应的Bean会被触发，即onApplicationEvent方法会被执行。

举个例子，当Application被初始化或刷新时，会触发ContextRefreshedEvent事件，可以实现一个ApplicationListener来监听该事件：

```java
// 对该类进行Bean的实例化
@Component
public class ListenerDemo implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {
        // 打印容器里面初始化了多少个Bean
        System.out.println("监听器获得容器中初始化Bean的数量: " + contextRefreshedEvent.getApplicationContext().getBeanDefinitionCount());
    }
}
```

#### 推断入口类

SpringApplication实例化的最后一步就是推断入口类，通过deduceMainApplicationClass进行推断，源码如下：

```java
	private Class<?> deduceMainApplicationClass() {
		try {
			// 获取栈元素数组
			StackTraceElement[] stackTrace = new RuntimeException().getStackTrace();
			// 遍历栈元素数组
			for (StackTraceElement stackTraceElement : stackTrace) {
				// 匹配第一个main方法, 并返回
				if ("main".equals(stackTraceElement.getMethodName())) {
					return Class.forName(stackTraceElement.getClassName());
				}
			}
		}
		catch (ClassNotFoundException ex) {
			// 如果发生异常，忽略改一次，并继续执行
		}
		return null;
	}
```

该方法先创建一个运行时异常，然后获得栈数组，遍历栈数组，判断类的方法中是否包含main方法。一个被匹配到的类会通过Class.forName方法创建对象，并将其返回，最后在上层方法中将对象赋值给SpringApplication的成员变量mainApplicationClass。

## 运行流程

### SpringApplication run方法简介

在上一篇已经介绍了，当SpringApplication对象被创建之后，通过调用其run方法进行Spring Boot的启动和运行，正式开启SpringApplication的生命周期。

SpringApplication调用的run方法的核心操作如下图所示：

![run方法流程](https://img-blog.csdnimg.cn/eabb16f5c1fa4555907f54cbcdaacfa0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix55yL6Zuy55qE6Zuy,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

对照流程图，来整体看一下run方法的源码：

```java
public ConfigurableApplicationContext run(String... args) {
		// 创建StopWatch对象, 用于统计run方法启动时长
		StopWatch stopWatch = new StopWatch();
		// 启动统计
		stopWatch.start();
  		// 应用上下文对象，Spring Boot使用的应用上下文是ConfigurableApplicationContext
		ConfigurableApplicationContext context = null;
  		// 异常报告器
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		// 配置headless属性
		configureHeadlessProperty();
		/**
		 * 获得SpringApplicationRunListener数组, 并将其封装在变量listeners中
		 * SpringApplicationRunListener接口是监听SpringApplication中run方法各个执行阶段的, 提供了一系列的方法, 用户可以通过回调这些方法, 在启动各个阶段时加入指定的逻辑处理
		 */
		SpringApplicationRunListeners listeners = getRunListeners(args);
		// 启动监听器, 里面的操作为遍历SpringApplicationRunListener数组每个元素, 并执行
		listeners.starting();
		try {
			// 初始化ApplicationArguments对象, ApplicationArguments是用于提供访问运行SpringApplication时的参数
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			// 初始化ConfigurableEnvironment, 加载属性配置, 包括所有的配置属性(如: application.properties中和外部的属性配置)
			// ConfigurableEnvironment接口的主要作用是提供当前运行环境的公开接口, 比如配置文件profiles各类系统属性和变量的设置, 添加, 读取, 合并
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			// 对环境中忽略信息配置项spring.beaninfo.ignore的值进行判断, 如果为true, 跳过BeanInfo类的扫描
			configureIgnoreBeanInfo(environment);
			// 打印Bannner
			Banner printedBanner = printBanner(environment);
			// 创建容器
			context = createApplicationContext();
			// 获取异常报告器
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			// 准备容器, 将组件对象之间进行关联
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
			// 刷新容器
			refreshContext(context);
			// 初始化操作之后执行, 默认实现为空
			afterRefresh(context, applicationArguments);
			// 停止时长统计
			stopWatch.stop();
			// 打印启动日志
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			// 通知监听器: 容器启动完成
			listeners.started(context);
			// 调用ApplicationRunner和CommandLineRunner的运行方法, 用来给用户实现一些在容器启动时的自定义操作
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			// 异常处理
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			// 通知监听器: 容器正在运行
			listeners.running(context);
		}
		catch (Throwable ex) {
			// 异常处理
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```

### SpringApplicationRunListener[监听器](https://so.csdn.net/so/search?q=监听器&spm=1001.2101.3001.7020)

#### SpringApplicationRunListeners容器

从上述源码中可以看到，除了计时统计的功能，第一步就是监听器SpringApplicationRunListeners的获取和使用。SpringApplicationRunListeners是一个SpringApplicationRunListener的容器，它将SpringApplicationRunListener的集合以构造方法传入，并赋值给其成员变量listeners，然后提供了针对listeners成员变量的各种遍历操作方法。源码如下：

```java
class SpringApplicationRunListeners {

	private final Log log;

	private final List<SpringApplicationRunListener> listeners;

	SpringApplicationRunListeners(Log log, Collection<? extends SpringApplicationRunListener> listeners) {
		this.log = log;
		this.listeners = new ArrayList<>(listeners);
	}

	void starting() {
		for (SpringApplicationRunListener listener : this.listeners) {
			listener.starting();
		}
	}

	void environmentPrepared(ConfigurableEnvironment environment) {
		for (SpringApplicationRunListener listener : this.listeners) {
			listener.environmentPrepared(environment);
		}
	}

	void contextPrepared(ConfigurableApplicationContext context) {
		for (SpringApplicationRunListener listener : this.listeners) {
			listener.contextPrepared(context);
		}
	}

	void contextLoaded(ConfigurableApplicationContext context) {
		for (SpringApplicationRunListener listener : this.listeners) {
			listener.contextLoaded(context);
		}
	}

	void started(ConfigurableApplicationContext context) {
		for (SpringApplicationRunListener listener : this.listeners) {
			listener.started(context);
		}
	}

	void running(ConfigurableApplicationContext context) {
		for (SpringApplicationRunListener listener : this.listeners) {
			listener.running(context);
		}
	}

	void failed(ConfigurableApplicationContext context, Throwable exception) {
		for (SpringApplicationRunListener listener : this.listeners) {
			callFailedListener(listener, context, exception);
		}
	}
}
```

SpringApplicationRunListeners的构建很简单，run方法中使用的是getRunListeners私有方法，源码如下：

```java
	private SpringApplicationRunListeners getRunListeners(String[] args) {
		// 构造Class数组
		Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
		// 调用SpringApplicationRunListeners的构造方法
		return new SpringApplicationRunListeners(logger,
				getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args));
	}

	/**
	 * 用来获取spring.factories配置文件中的注册类, 并进行实例化操作
	 */
	private <T> Collection<T> getSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes, Object... args) {
		// 获取类加载器
		ClassLoader classLoader = getClassLoader();
		// 加载META-INF/spring.factories中对应的配置, 并将结果存储于Set中, 方便去重
		Set<String> names = new LinkedHashSet<>(SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		// 创建实例
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes, classLoader, args, names);
		// 排序
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}

	/**
	 * 实例化注册类
	 */
	@SuppressWarnings("unchecked")
	private <T> List<T> createSpringFactoriesInstances(Class<T> type, Class<?>[] parameterTypes,
			ClassLoader classLoader, Object[] args, Set<String> names) {
		List<T> instances = new ArrayList<>(names.size());
		// 遍历加载到的类名(全限定类名)
		for (String name : names) {
			try {
				// 使用全路径类名通过反射获取class对象
				Class<?> instanceClass = ClassUtils.forName(name, classLoader);
				Assert.isAssignable(type, instanceClass);
				// 获取有参构造器
				Constructor<?> constructor = instanceClass.getDeclaredConstructor(parameterTypes);
				// 执行构造函数获取实例
				T instance = (T) BeanUtils.instantiateClass(constructor, args);
				// 将实例加入到返回结果中
				instances.add(instance);
			}
			catch (Throwable ex) {
				throw new IllegalArgumentException("Cannot instantiate " + type + " : " + name, ex);
			}
		}
		return instances;
	}

```

getSpringFactoriesInstances方法在前两篇也介绍了很多次了，通过SpringFactoriesLoader的loadFactoryNames方法加载META-INF/spring.factories中对应的配置。SpringApplicationRunListener的配置只有一个——EventPublishingRunListener。

```
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```

#### SpringApplicationRunListener解析

接口SpringApplicationRunListener是SpringApplication的run方法监听器，提供了一系列的方法，用户可以通过回调这些方法，在启动的各个流程加入指定的逻辑处理。源码如下

```java
public interface SpringApplicationRunListener {

	// 当run方法第一次被执行时, 会被立即调用, 可用于非常早期的初始化工作
	default void starting() {
	}

	// 当environment准备完成, 在ApplicationContext容器创建之前, 该方法被调用
	default void environmentPrepared(ConfigurableEnvironment environment) {
	}

	// 当ApplicationContext构建完成, 资源还未被加载时, 该方法被调用
	default void contextPrepared(ConfigurableApplicationContext context) {
	}

	// 当ApplicationContext资源加载完成, 未被刷新之前, 该方法被调用
	default void contextLoaded(ConfigurableApplicationContext context) {
	}

	// 当ApplicationContext刷新并启动之后, CommandLineRunner和ApplicationRunner未被调用之前, 该方法被调用
	default void started(ConfigurableApplicationContext context) {
	}

	// 当所有准备工作就绪, run方法执行完成之前, 该方法被调用
	default void running(ConfigurableApplicationContext context) {
	}

	// 当应用程序出现错误时, 该方法被调用
	default void failed(ConfigurableApplicationContext context, Throwable exception) {
	}
}
```

从源码中可以看出，接口SpringApplicationRunListener为run方法提供了各个运行阶段的监听事件处理功能，执行时机如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/de245c307ef849a497f7f1ae7dac88d9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix55yL6Zuy55qE6Zuy,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

#### 实现类EventPublishingRunListener解析

从META-INF/spring.factories配置文件中可以看出，SpringApplicationRunListener只有一个默认实现类EventPublishingRunListener，EventPublishingRunListener使用内置的SimpleApplicationEventMulticaster来广播在应用上下文刷新之前触发的事件。

```java
public class EventPublishingRunListener implements SpringApplicationRunListener, Ordered {

	private final SpringApplication application;

	private final String[] args;

	// 事件广播器
	private final SimpleApplicationEventMulticaster initialMulticaster;

	public EventPublishingRunListener(SpringApplication application, String[] args) {
		this.application = application;
		this.args = args;
		// 创建并设置广播器
		this.initialMulticaster = new SimpleApplicationEventMulticaster();
		// 遍历ApplicationListener，并关联SimpleApplicationEventMulticaster广播器
		for (ApplicationListener<?> listener : application.getListeners()) {
			this.initialMulticaster.addApplicationListener(listener);
		}
	}
  
  	// 当run方法第一次被执行时，该方法被调用
  	@Override
	public void starting() {
		this.initialMulticaster.multicastEvent(new ApplicationStartingEvent(this.application, this.args));
	}

  	// 当environment准备完成，在ApplicationContext容器创建之前，该方法被调用
	@Override
	public void environmentPrepared(ConfigurableEnvironment environment) {
		this.initialMulticaster
				.multicastEvent(new ApplicationEnvironmentPreparedEvent(this.application, this.args, environment));
	}

  	// 当ApplicationContext容器构建完成，资源还未被加载时，该方法被调用
	@Override
	public void contextPrepared(ConfigurableApplicationContext context) {
		this.initialMulticaster
				.multicastEvent(new ApplicationContextInitializedEvent(this.application, this.args, context));
	}

  	// 当ApplicationContext容器资源加载完成，未被刷新之前，该方法被调用
	@Override
	public void contextLoaded(ConfigurableApplicationContext context) {
		// 遍历application中所有的监听器
		for (ApplicationListener<?> listener : this.application.getListeners()) {
			// 如果为ApplicationContextAware, 则将上下文信息设置到该监听器内
			if (listener instanceof ApplicationContextAware) {
				((ApplicationContextAware) listener).setApplicationContext(context);
			}
			// 将application中的监听器全部添加到上下文中
			context.addApplicationListener(listener);
		}
		// 广播事件ApplicationPreparedEvent
		this.initialMulticaster.multicastEvent(new ApplicationPreparedEvent(this.application, this.args, context));
	}

  	// 当ApplicationContext容器刷新并启动后，CommandLineRunner和ApplicationRunner未被调用之前，该方法被调用
	@Override
	public void started(ConfigurableApplicationContext context) {
		context.publishEvent(new ApplicationStartedEvent(this.application, this.args, context));
	}

  	// 当所有准备工作就绪，run方法执行完成之前，该方法被调用
	@Override
	public void running(ConfigurableApplicationContext context) {
		context.publishEvent(new ApplicationReadyEvent(this.application, this.args, context));
	}

  	// 当应用程序出现错误时, 该方法被调用
	@Override
	public void failed(ConfigurableApplicationContext context, Throwable exception) {
		ApplicationFailedEvent event = new ApplicationFailedEvent(this.application, this.args, context, exception);
		if (context != null && context.isActive()) {
			context.publishEvent(event);
		}
		else {
			if (context instanceof AbstractApplicationContext) {
				for (ApplicationListener<?> listener : ((AbstractApplicationContext) context)
						.getApplicationListeners()) {
					this.initialMulticaster.addApplicationListener(listener);
				}
			}
			this.initialMulticaster.setErrorHandler(new LoggingErrorHandler());
			this.initialMulticaster.multicastEvent(event);
		}
	}
}
```

Spring Boot完成基本的初始化之后，会遍历SpringApplication的所有ApplicationListener实例，并将它们与SimpleApplicationEventMulticaster进行关联，方便SimpleApplicationEventMulticaster后续将事件传递给所有的监听器。

EventPublishingRunListener针对不同的事件的处理流程基本相同，可以概括为以下几个步骤：

当run方法运行到某个阶段时，调用EventPublishingRunListener的某个方法。
EventPublishingRunListener的某个方法将application参数和args参数封装到对应的事件中，这里的事件均为SpringApplicationEvent的实现类。
通过成员变量initialMulticaster的multicastEvent方法对事件进行广播，或者通过该方法参数ConfigurableApplicationContext的publishEvent来对事件进行发布。
对应的ApplicationListener被触发，执行响应的业务逻辑。
从上述步骤中可以看出，有些方法时通过成员变量initialMulticaster的multicastEvent方法对事件进行广播，有些方法是通过参数ConfigurableApplicationContext的publishEvent来对事件进行发布。从这些方法中可以找到contextLoaded这个方法，它是两种不同事件广播形式的分水岭。contextLoaded方法在发布事件之前做了两件事：一，遍历application的所有监听器实现类，如果该实现类还实现了ApplicationContextAware接口，则将上下文信息设置到该监听器内；二、将application中的监听器实现类全部添加到上下文中。最后调用事件广播。在contextLoaded方法执行之前，上下文还没有初始化完成，所以无法通过它的publishEvent方法进行事件发布.

### 初始化ApplicationArguments

在run方法源码中，监听器启动之后，紧接着便是执行ApplicationArguments对象的初始化，ApplicationArguments是用于提供访问运行SpringApplication时的参数。

```java
ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);

public DefaultApplicationArguments(String... args) {
		Assert.notNull(args, "Args must not be null");
		this.source = new Source(args);
		this.args = args;
}
```

在DefaultApplicationArguments将参数args封装为Source对象，Source对象是基于Spring框架的SimpleCommandLinePropertySource来实现的。

### 初始化ConfigurableEnvironment

完成ApplicationArguments参数的准备之后，便开始通过prepareEnvironment方法对ConfigurableEnvironment对象进行初始化操作。ConfigurableEnvironment接口的主要作用是提供当前运行环境的公开接口，比如配置文件profiles各类系统属性和变量的设置、添加、读取、合并等功能。
```java
	private ConfigurableEnvironment prepareEnvironment(SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments) {
		// 获取或创建环境
		ConfigurableEnvironment environment = getOrCreateEnvironment();
		// 配置环境, 主要包括PropertySource和activeProfiles的配置
		configureEnvironment(environment, applicationArguments.getSourceArgs());
		// 将ConfigurationPropertySources附加到指定环境中的第一位, 并动态跟踪环境的添加或删除
		ConfigurationPropertySources.attach(environment);
		// 通知监听器listener环境准备完成
		listeners.environmentPrepared(environment);
		// 将环境绑定到SpringApplication, 将环境绑定到name为spring.main的目标上
		bindToSpringApplication(environment);
		// 判断是否定制的环境, 如果不是定制的则将环境转换到StandardEnvironment
		if (!this.isCustomEnvironment) {
			environment = new EnvironmentConverter(getClassLoader()).convertEnvironmentIfNecessary(environment,
					deduceEnvironmentClass());
		}
		// 再次将ConfigurationPropertySources附加到指定环境中的第一位, 并动态跟踪环境的添加或删除
		ConfigurationPropertySources.attach(environment);
		return environment;
	}
```

#### 获取或创建环境

该方法比较简单，如果environment存在，则直接返回。如果environment不存在，则根据SpringApplication实例化时推断的webApplicationType来进行区分创建环境。

```java
	private ConfigurableEnvironment getOrCreateEnvironment() {
		// 如果environment存在, 则直接返回
		if (this.environment != null) {
			return this.environment;
		}
		// 根据前面推断的webApplicationType, 创建不同的环境实现
		switch (this.webApplicationType) {
		case SERVLET:
			return new StandardServletEnvironment();
		case REACTIVE:
			return new StandardReactiveWebEnvironment();
		default:
			return new StandardEnvironment();
		}
	}
```

#### [配置环境](https://so.csdn.net/so/search?q=配置环境&spm=1001.2101.3001.7020)

在获得环境变量对象之后，开始对环境变量和参数进行相应的设置，主要包括转换服务的设置、PropertySources的设置和activeProfiles的设置。

```java
	protected void configureEnvironment(ConfigurableEnvironment environment, String[] args) {
		// 如果为true则获取并设置转换服务
		if (this.addConversionService) {
			ConversionService conversionService = ApplicationConversionService.getSharedInstance();
			environment.setConversionService((ConfigurableConversionService) conversionService);
		}
		// 配置PropertySources
		configurePropertySources(environment, args);
		// 配置Profiles
		configureProfiles(environment, args);
	}
```

首先判断addConversionService变量是否为true，也就是判断是否需要添加转换服务，如果需要，则获取转换服务实例，并对环境设置转换服务。随后进行PropertySources和Profiles的配置。

```java
	// 配置PropertySources
	protected void configurePropertySources(ConfigurableEnvironment environment, String[] args) {
    	// 获取环境中的属性资源信息
		MutablePropertySources sources = environment.getPropertySources();
    	// 如果默认属性配置存在则将其放置于属性资源的最后位置
		if (this.defaultProperties != null && !this.defaultProperties.isEmpty()) {
			sources.addLast(new MapPropertySource("defaultProperties", this.defaultProperties));
		}
    	// 如果命令行属性存在
		if (this.addCommandLineProperties && args.length > 0) {
      		// 常量值为：commandLineArgs
			String name = CommandLinePropertySource.COMMAND_LINE_PROPERTY_SOURCE_NAME;
      		// 如果默认属性资源中不包含该命令，则将命令行属性放置在第一位，如果包含，则通过CommandLinePropertySource进行处理
			if (sources.contains(name)) {
				PropertySource<?> source = sources.get(name);
				CompositePropertySource composite = new CompositePropertySource(name);
				composite.addPropertySource(
						new SimpleCommandLinePropertySource("springApplicationCommandLineArgs", args));
				composite.addPropertySource(source);
				sources.replace(name, composite);
			}
			else {
        		// 放置在第一位
				sources.addFirst(new SimpleCommandLinePropertySource(args));
			}
		}
	}
```

主要是对参数的优先级进行处理，首先，如果存在默认属性配置，则将默认属性配置放置在最后，也就是优先级最低。对于命令参数，如果命令参数已经存在于属性配置中，则使用CompositePropertySource类进行相同name的参数处理；如果命令参数并不存在于属性配置中，则直接将其设置为优先级最高。

```java
protected void configureProfiles(ConfigurableEnvironment environment, String[] args) {
    	// 如果存在额外的Profiles，则将其放置在第一位，随后再获得其他的Profiles
		Set<String> profiles = new LinkedHashSet<>(this.additionalProfiles);
		profiles.addAll(Arrays.asList(environment.getActiveProfiles()));
		environment.setActiveProfiles(StringUtils.toStringArray(profiles));
	}
```

### 忽略信息配置

在上述完成ConfigurableEnvironment的初始化之后，程序又对环境中的忽略信息配置项（参数**spring.beaninfo.ignore**）的值进行判断，进而设置为系统参数中的忽略项。

```java
private void configureIgnoreBeanInfo(ConfigurableEnvironment environment) {
    	// 如果系统参数中spring.beaninfo.ignore为null
		if (System.getProperty(CachedIntrospectionResults.IGNORE_BEANINFO_PROPERTY_NAME) == null) {
      		// 获取环境中spring.beaninfo.ignore的数量
			Boolean ignore = environment.getProperty("spring.beaninfo.ignore", Boolean.class, Boolean.TRUE);
      		// 设置对应的系统参数
			System.setProperty(CachedIntrospectionResults.IGNORE_BEANINFO_PROPERTY_NAME, ignore.toString());
		}
	}
```

spring.beaninfo.ignore的配置用来决定是否跳过BeanInfo类的扫描，如果设置为true，则跳过。

### 打印Banner

完成环境的基本处理之后，下面就是控制台Banner的打印了。

```java
	private Banner printBanner(ConfigurableEnvironment environment) {
    	// 如果处于关闭状态，则返回null
		if (this.bannerMode == Banner.Mode.OFF) {
			return null;
		}
    	// 如果resourceLoader不存在则创建一个默认的ResourceLoader
		ResourceLoader resourceLoader = (this.resourceLoader != null) ? this.resourceLoader
				: new DefaultResourceLoader(getClassLoader());
    	// 创建SpringApplicationBannerPrinter
		SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(resourceLoader, this.banner);
    	// 打印到日志中
		if (this.bannerMode == Mode.LOG) {
			return bannerPrinter.print(environment, this.mainApplicationClass, logger);
		}
    	// 打印到控制台
		return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
	}
```

### 创建应用上下文

根据推断的Web应用类型，创建对应的上下文。

```java
	public static final String DEFAULT_CONTEXT_CLASS = "org.springframework.context."
			+ "annotation.AnnotationConfigApplicationContext";

	public static final String DEFAULT_SERVLET_WEB_CONTEXT_CLASS = "org.springframework.boot."
			+ "web.servlet.context.AnnotationConfigServletWebServerApplicationContext";

	public static final String DEFAULT_REACTIVE_WEB_CONTEXT_CLASS = "org.springframework."
			+ "boot.web.reactive.context.AnnotationConfigReactiveWebServerApplicationContext";	

	protected ConfigurableApplicationContext createApplicationContext() {
		// 获取容器中的类变量
		Class<?> contextClass = this.applicationContextClass;
		// 如果为null, 则根据Web应用类型按照默认类进行创建
		if (contextClass == null) {
			try {
				switch (this.webApplicationType) {
				case SERVLET:
					contextClass = Class.forName(DEFAULT_SERVLET_WEB_CONTEXT_CLASS);
					break;
				case REACTIVE:
					contextClass = Class.forName(DEFAULT_REACTIVE_WEB_CONTEXT_CLASS);
					break;
				default:
					contextClass = Class.forName(DEFAULT_CONTEXT_CLASS);
				}
			}
			catch (ClassNotFoundException ex) {
				throw new IllegalStateException(
						"Unable create a default ApplicationContext, please specify an ApplicationContextClass", ex);
			}
		}
    	// 如果存在对应的Class配置，则通过Spring提供的BeanUtils来进行实例化
		return (ConfigurableApplicationContext) BeanUtils.instantiateClass(contextClass);
	}
```

### 准备应用上下文

完成了应用上下文的创建，SpringApplication通过prepareContext方法来进行应用上下文的准备，核心功能和流程如下图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/bac009707c5247009696f28441de3fdb.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA54ix55yL6Zuy55qE6Zuy,size_20,color_FFFFFF,t_70,g_se,x_16#pic_center)

结合流程图，看一下prepareContext方法的源码：

```java
private void prepareContext(ConfigurableApplicationContext context, ConfigurableEnvironment environment,
			SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
		// --------------------- 应用上下文准备阶段 ---------------------
		// 设置上下文的配置环境
		context.setEnvironment(environment);
		// 应用上下文后置处理
		postProcessApplicationContext(context);
		// 在上下文刷新之前, ApplicationContextInitializer初始化上下文
		applyInitializers(context);
		// 通知监听器上下文准备完成
		listeners.contextPrepared(context);

		// --------------------- 应用上下文加载阶段 ---------------------
		// 打印日志, 启动Profile
		if (this.logStartupInfo) {
			logStartupInfo(context.getParent() == null);
			logStartupProfileInfo(context);
		}
		// 获取ConfigurableListableBeanFactory
		ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
		// 将beanFactory注册为单例对象
		beanFactory.registerSingleton("springApplicationArguments", applicationArguments);
		if (printedBanner != null) {
			// 注册打印日志对象
			beanFactory.registerSingleton("springBootBanner", printedBanner);
		}
		if (beanFactory instanceof DefaultListableBeanFactory) {
			// 设置是否允许覆盖注册
			((DefaultListableBeanFactory) beanFactory)
					.setAllowBeanDefinitionOverriding(this.allowBeanDefinitionOverriding);
		}
		if (this.lazyInitialization) {
			context.addBeanFactoryPostProcessor(new LazyInitializationBeanFactoryPostProcessor());
		}
		// 获取全部配置源, 其中包含primarySources(被@SpringBootApplication注解或@EnableAutoConfiguration注解注释的类)和sources
		Set<Object> sources = getAllSources();
		Assert.notEmpty(sources, "Sources must not be empty");
		// 将配置源中的Bean加载到上下文中
		load(context, sources.toArray(new Object[0]));
		// 通知监听器上下文加载完成
		listeners.contextLoaded(context);
	}
```

从代码中可以看出，准备应用上下文分为两个阶段，准备阶段、加载阶段，下面依次来看下这两个阶段。

### 应用上下文准备阶段

在上下文准备阶段，主要有3步操作：对应用上下文Context设置environment、应用上下文后置处理、ApplicationContextInitializer初始化Context。

#### 对应用上下文Context设置environment

```java
	public void setEnvironment(ConfigurableEnvironment environment) {
    	// 设置Context的environment
		super.setEnvironment(environment);
    	// 设置Context的reader属性的conditionEvaluator属性
		this.reader.setEnvironment(environment);
    	// 设置Context的scanner属性的environment属性
		this.scanner.setEnvironment(environment);
	}
```

#### 应用上下文后置处理

设置beanNameGenerator、resourceLoader、classLoader、conversionService，在此阶段，beanNameGenerator和resourceLoader都为null，实际上只做了最后一步的设置转换服务。

```java
	protected void postProcessApplicationContext(ConfigurableApplicationContext context) {
		if (this.beanNameGenerator != null) {
			// 如果beanNameGenerator不为null, 则将当前的beanNameGenerator按照默认名字进行注册
			context.getBeanFactory().registerSingleton(AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR,
					this.beanNameGenerator);
		}
		// 如果resourceLoader不为null, 则根据上下文的类型分别进行ResourceLoader和ClassLoader的设置
		if (this.resourceLoader != null) {
			if (context instanceof GenericApplicationContext) {
				((GenericApplicationContext) context).setResourceLoader(this.resourceLoader);
			}
			if (context instanceof DefaultResourceLoader) {
				((DefaultResourceLoader) context).setClassLoader(this.resourceLoader.getClassLoader());
			}
		}
		// 如果为true则获取并设置转换服务
		if (this.addConversionService) {
			context.getBeanFactory().setConversionService(ApplicationConversionService.getSharedInstance());
		}
	}
```

#### 对应用上下文初始化

```java
	protected void applyInitializers(ConfigurableApplicationContext context) {
		// 获取ApplicationContextInitializer集合并遍历
		for (ApplicationContextInitializer initializer : getInitializers()) {
			// 解析当前遍历的initializer实现ApplicationContextInitializer的泛型类型
			Class<?> requiredType = GenericTypeResolver.resolveTypeArgument(initializer.getClass(),
					ApplicationContextInitializer.class);
			// 判断泛型是否和ConfigurableApplicationContext匹配
			Assert.isInstanceOf(requiredType, context, "Unable to call initializer.");
			// 初始化上下文
			initializer.initialize(context);
		}
	}
```

### 应用上下文加载阶段

在上下文加载阶段，主要有5步操作：打印日志和Profile的设置、设置是否允许覆盖注册、获取全部配置源、将配置源加载到上下文、通知监听器Context加载完成。

获取ConfigurableListableBeanFactory并注册单例对象，注册的单例对象包含：ApplicationArguments和Banner。当BeanFactory为DefaultListableBeanFactory时，进入设置是否允许覆盖注册的处理逻辑。当ApplicationArguments类单例对象注册之后，也就意味着我们在使用Spring应用上下文的过程中可以通过依赖注入来使用该对象。

#### 获取全部配置源

```java
public Set<Object> getAllSources() {
		Set<Object> allSources = new LinkedHashSet<>();
    	// 如果primarySources不为空，则将其添加到Set集合中
    	// primarySources为被@SpringBootApplication注解或@EnableAutoConfiguration注解修饰的类
		if (!CollectionUtils.isEmpty(this.primarySources)) {
			allSources.addAll(this.primarySources);
		}
    	// 如果sources不为空，则将其添加到Set集合中
		if (!CollectionUtils.isEmpty(this.sources)) {
			allSources.addAll(this.sources);
		}
		return Collections.unmodifiableSet(allSources);
	}
```

#### 加载配置源

在获取到所有配置源之后，通过load方法将配置源信息加载到上下文中。

```java
	protected void load(ApplicationContext context, Object[] sources) {
    	// 日志打印
		if (logger.isDebugEnabled()) {
			logger.debug("Loading source " + StringUtils.arrayToCommaDelimitedString(sources));
		}
    	// 创建BeanDefinitionLoader，通过BeanDefinitionLoader来完成配置资源的加载操作
		BeanDefinitionLoader loader = createBeanDefinitionLoader(getBeanDefinitionRegistry(context), sources);
		if (this.beanNameGenerator != null) {
			loader.setBeanNameGenerator(this.beanNameGenerator);
		}
		if (this.resourceLoader != null) {
			loader.setResourceLoader(this.resourceLoader);
		}
		if (this.environment != null) {
			loader.setEnvironment(this.environment);
		}
    	// 调用BeanDefinitionLoader的load方法
		loader.load();
	}

	// BeanDefinitionLoader构造方法
	// 可以看出BeanDefinitionLoader支持基于AnnotatedBeanDefinitionReader、XmlBeanDefinitionReader、GroovyBeanDefinitionReader等多种类型的加载操作。
	BeanDefinitionLoader(BeanDefinitionRegistry registry, Object... sources) {
		Assert.notNull(registry, "Registry must not be null");
		Assert.notEmpty(sources, "Sources must not be empty");
		this.sources = sources;
		this.annotatedReader = new AnnotatedBeanDefinitionReader(registry);
		this.xmlReader = new XmlBeanDefinitionReader(registry);
		if (isGroovyPresent()) {
			this.groovyReader = new GroovyBeanDefinitionReader(registry);
		}
		this.scanner = new ClassPathBeanDefinitionScanner(registry);
		this.scanner.addExcludeFilter(new ClassExcludeFilter(sources));
	}

	// BeanDefinitionLoader中的load方法，加载支持的范围包括：Class、Resource、Package、CharSequence
	private int load(Object source) {
		Assert.notNull(source, "Source must not be null");
		if (source instanceof Class<?>) {
			return load((Class<?>) source);
		}
		if (source instanceof Resource) {
			return load((Resource) source);
		}
		if (source instanceof Package) {
			return load((Package) source);
		}
		if (source instanceof CharSequence) {
			return load((CharSequence) source);
		}
		throw new IllegalArgumentException("Invalid source type " + source.getClass());
	}
```

### 刷新应用上下文

应用上下文准备完成之后，便开始对应用上下文进行刷新。

```java
	private void refreshContext(ConfigurableApplicationContext context) {
		// 调用refresh方法
		refresh(context);
		if (this.registerShutdownHook) {
			try {
				// 注册shutdownHook线程, 实现销毁时的回调
				context.registerShutdownHook();
			}
			catch (AccessControlException ex) {
				// Not allowed in some environments.
			}
		}
	}

	protected void refresh(ApplicationContext applicationContext) {
		Assert.isInstanceOf(AbstractApplicationContext.class, applicationContext);
		// 调用spring中的refresh方法, 在refresh中实现对@EnableAutoConfiguration注解的自动配置
		((AbstractApplicationContext) applicationContext).refresh();
	}
```

最后是通过Spring中的AbstractApplicationContext类的refresh进行刷新，这就属于Spring的内容了，可以简单看下。

```java
public void refresh() throws BeansException, IllegalStateException {
  	// 整个通过同步处理
		synchronized (this.startupShutdownMonitor) {
			// 准备刷新工作
			prepareRefresh();

			// 通知子类刷新并获取BeanFactory, 这个获取到的是DefaultListableBeanFactory
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// 为当前上下文Context准备bean工厂
			prepareBeanFactory(beanFactory);

			try {
				// 设置BeanFactory的后置处理器
				postProcessBeanFactory(beanFactory);

				// 调用BeanFactory的后置处理器, 这些后置处理器是在Bean定义中向容器注册的
				invokeBeanFactoryPostProcessors(beanFactory);

				// 注册Bean的后置处理器, 在Bean创建过程中调用
				registerBeanPostProcessors(beanFactory);

				// 对上下文的消息源进行初始化
				initMessageSource();

				// 初始化上下文中的事件机制
				initApplicationEventMulticaster();

				// 初始化其他的特殊Bean
				onRefresh();

				// 检查监听Bean并且将这些Bean向容器注册
				registerListeners();

				// 实例化所有非懒加载的单例Bean
				finishBeanFactoryInitialization(beanFactory);

				// 发布容器事件, 结束刷新过程
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				destroyBeans();

				cancelRefresh(ex);

				throw ex;
			}

			finally {
				resetCommonCaches();
			}
		}
	}
```

### 调用[ApplicationRunner](https://so.csdn.net/so/search?q=ApplicationRunner&spm=1001.2101.3001.7020)和CommandLineRunner

应用上下文刷新完成之后，调用ApplicationRunner和CommandLineRunner的运行方法, 用来给用户实现一些在容器启动时的自定义操作。

```java
	private void callRunners(ApplicationContext context, ApplicationArguments args) {
		List<Object> runners = new ArrayList<>();
    	// 从应用上下文context中获得类型为ApplicationRunner的Bean，并将其加入到集合中
		runners.addAll(context.getBeansOfType(ApplicationRunner.class).values());
    	// 从应用上下文context中获得类型为CommandLineRunner的Bean，并将其加入到集合中
		runners.addAll(context.getBeansOfType(CommandLineRunner.class).values());
    	// 排序
		AnnotationAwareOrderComparator.sort(runners);
		for (Object runner : new LinkedHashSet<>(runners)) {
			if (runner instanceof ApplicationRunner) {
        		// 调用callRunner方法
				callRunner((ApplicationRunner) runner, args);
			}
			if (runner instanceof CommandLineRunner) {
				callRunner((CommandLineRunner) runner, args);
			}
		}
	}

	private void callRunner(ApplicationRunner runner, ApplicationArguments args) {
		try {
      		// 执行其run方法 
			(runner).run(args);
		}
		catch (Exception ex) {
			throw new IllegalStateException("Failed to execute ApplicationRunner", ex);
		}
	}
```

