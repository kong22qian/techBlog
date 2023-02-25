[TOC]

# @Import Annotation  

## @Import注解提供了三种用法

* @Import一个普通类，spring会将该类加载到spring容器中

* @Import一个类，该类实现了ImportBeanDefinitionRegistrar接口，在重写的registerBeanDefinitions方法里面，能拿到BeanDefinitionRegistry bd的注册器，能手工往beanDefinitionMap中注册 beanDefinition

* @Import一个类， 该类实现了ImportSelector 重写selectImports方法该方法返回了String[]数组的对象，数组里面的类都会注入到spring容器当中

## 测试

> ==场景一:  import普通类==

1. 自定义一个类 没有任何注解 

   ```java
   public class MyClass {	
   	public void test() {
   		System.out.println("test方法");
   	}
   }
   ```

2. 写一个importConfig类 import这个myClass类

   ```java
   @Import(MyClass.class)
   public class ImportConfig {
   }
   ```

3. 通过AnnotationConfigApplicationContext初始化spring容器，调用test方法 看输出，MyClass类被加载进了 spring容器当中

   ![img](https://img-blog.csdnimg.cn/aee98405add545bfac5cdda6318b510b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lit5bm05Y2x5py655qE6ICB55S35Lq6,size_20,color_FFFFFF,t_70,g_se,x_16)



> ==场景二：实现ImportBeanDefinitionRegistrar==

1.  创建一个普通类MyClassRegistry

   ```java
   public class MyClassRegistry {	
   	public void test() {
   		System.out.println("MyClassRegistry test方法");
   	}
   }
   ```

2. 创建MyImportRegistry 实现ImportBeanDefinitionRegistrar接口 注册我们定义的普通类MyClassRegistry

   ```java
   public class MyImportRegistry implements ImportBeanDefinitionRegistrar{
   	@Override
   	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
   		RootBeanDefinition bd = new RootBeanDefinition();
   		bd.setBeanClass(MyClassRegistry.class);
   		registry.registerBeanDefinition("myClassRegistry", bd);
   	}
   }
   ```

3. import该类  

   ```java
   @Import(MyImportRegistry.class)
   public class ImportConfig {
   }
   ```

4. 执行main方法 查看输出

   ![img](https://img-blog.csdnimg.cn/f1f654a234544847b28f25aff31a5538.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lit5bm05Y2x5py655qE6ICB55S35Lq6,size_20,color_FFFFFF,t_70,g_se,x_16)

> ==场景三: 实现ImportSelector==

1. 创建一个普通类MyClassImport 

   ```java
   public class MyClassImport {	
   	public void test() {
   		System.out.println("MyClassImport test方法");
   	}
   }
   ```

2. 创建MyImportSelector实现ImportSelector接口 注册我们定义的普通类MyClassImport

   ```java
   public class MyImportSelector implements ImportSelector { 
   	@Override
   	public String[] selectImports(AnnotationMetadata importingClassMetadata) {
   		return new String[] {MyClassImport.class.getName()};
   	} 
   }
   ```

3. import该类

   ```java
   @Import(MyImportSelector.class)
   public class ImportConfig {
   	
   }
   ```

4. 执行main方法 查看输出

   ![img](https://img-blog.csdnimg.cn/3839e967fd2942c5bbf40fb9ce17ae45.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5Lit5bm05Y2x5py655qE6ICB55S35Lq6,size_20,color_FFFFFF,t_70,g_se,x_16)

## 总结
==场景二应用于spring-mybatis当中 扫描dao信息 生成代理类信息==  
==场景三应用于springboot的自动装配当中 加载自动装配需要的类信息==  

https://blog.csdn.net/weixin_45453628/article/details/124234317
