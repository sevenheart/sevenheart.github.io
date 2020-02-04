---
title: springbean的生命周期个人理解
date: 2020-02-01 
categories:
- Spring
---

### springBean的生命周期

![](../../../sources/img/spring/bean的生命周期1.png)

1. Spring启动，查找并加载需要被Spring管理的bean，进行Bean的实例化（Bean的实例化需要通过**BeanDefinition**对象它是通过Spring扫描得到的）
2. Bean实例化后对将Bean的引入和值注入到Bean的属性中
3. 如果Bean实现了BeanNameAware接口的话，Spring将Bean的Id传递给setBeanName()方法
4. 如果Bean实现了BeanFactoryAware接口的话，Spring将调用setBeanFactory()方法，将BeanFactory容器实例传入
5. 如果Bean实现了ApplicationContextAware接口的话，Spring将调用Bean的setApplicationContext()方法，将bean所在应用上下文引用传入进来。
6. 如果Bean实现了BeanPostProcessor接口，Spring就将调用他们的postProcessBeforeInitialization()方法。
7. 如果Bean 实现了InitializingBean接口，Spring将调用他们的afterPropertiesSet()方法。类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
8. 如果Bean 实现了BeanPostProcessor接口，Spring就将调用他们的postProcessAfterInitialization()方法。
9. 此时，Bean已经准备就绪，可以被应用程序使用了。他们将一直驻留在应用上下文中，直到应用上下文被销毁。
10. 如果bean实现了DisposableBean接口，Spring将调用它的destory()接口方法，同样，如果bean使用了destory-method 声明销毁方法，该方法也会被调用。

#### 个人理解补充的生命周期

1. 在spring实例化之前也就是查找需要被spring管理的bean的时候首先要通过**解析xml文件或者通过注解扫描**拿到所有需要被spring管理的bean的信息。

2. 拿到这些信息后通过一个实现ImportBeanDefinitionRegistrar接口中的RegisterBeanDefinition的方法的类去包装拿到的信息把他们包装成BeanDefinition对象（包装包含添加要生成对象需要的构造参数等）

   **举例：**一个我们new出来的对象要想交给Spring管理，就需要一个类去实现FactoryBean的两个方法去生成相应的Bean，而且这个类也要被作为Bean去交给Spring才行。

   ```xml
   <Bean id = "bean" class="..FactoryBean">
   	<properties name = "类中的成员变量"  value = "我们想要注入的class"></properties>
   <Bean/>
       
       //但是这样写还是先把这个类作为Bean交给spring管理，这样就要加入一个配置一个这样的Bean,或者通过@component等对象去做。
   ```

   **所以要想动态的去生成Bean就不能让这个类初始化时就交个Spring去管理**

   通过注解扫描的原理，它是@Import了一个实现了ImportBeanDefinitionRegistrar对象的类来完成扫描的。所以我们自己也写一个这样的注解，这样的类然后在类中动态的完成我们需要的类的初始化后再交给Spring就可以了。

   ```java
   
   import org.springframework.beans.factory.support.AbstractBeanDefinition;
   import org.springframework.beans.factory.support.BeanDefinitionBuilder;
   import org.springframework.beans.factory.support.BeanDefinitionRegistry;
   import org.springframework.context.annotation.ImportBeanDefinitionRegistrar;
   import org.springframework.core.type.AnnotationMetadata;
   public class BeanDefintionDemo implements ImportBeanDefinitionRegistrar {
       public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
   
           //luxuFactoryBean中有一个Class类型的成员变量用来动态的生成交给Spring管理的Bean，通过构造参数传入这个值。
           //我们通过反射找到我们要生产的Bean的名字然后循环放入这个BeanDefinition中的构造函数的参数中，
           // 这样就会在实例化FactoryBean的时候动态的生成我们想要spring管理的Bean
   
          //得到我们自定义的可以生成Bean的Beanfactory的BeanDefinition
           BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition(luxuFactoryBean.class);
           //把这个BeanDefinition强转一下
           AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();
           //在luxuFactoryBean的BeanDefinition中加入我们要通过构造函数传入的用来动态生成Spring管理的Bean的对象。
           beanDefinition.getConstructorArgumentValues().addGenericArgumentValue("要传入类的全路径名");
           //这时侯Spring就可以根据BeanDefinition生成我们的luxuFactoryBean的实例了，然后在通过它的构造函数生成相应的交给Spring管理的Bean。
           beanDefinitionRegistry.registerBeanDefinition("通过自定义factoryBean生成Bean的名字",beanDefinition);
       }
   }
   
   ```

   

3. 当spirng拥有BeanDefinition后就会去一步步的执行bean的生命周期了。

   

![](../../../sources/img/spring/bean的生命周期2.png)

#### spring实例化的方式：

1.通过FactoryBean去创建需要的Bean然后交给spring管理

2.根据BeanDefinition去创建想要的Bean。

