---
title: springbean的生命周期个人理解
date: 2020-02-01 
categories:
- Spring
---

### springBean的生命周期

![](../../../sources/img/spring/Bean的生命周期1.png)

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

   

![](../../../sources/img/spring/Bean的生命周期2.png)

#### spring实例化的方式：

1.通过FactoryBean去创建需要的Bean然后交给spring管理------他是一个spring的Bean可以用来生成其他的Bean。

2.根据BeanDefinition去创建想要的Bean。

```
整个都包含Bean生命周期内:
-> new AnnotationConfigApplicationContext(XXX); 扫描注解获得beanDefinition
-> applicationContext.getBean() 创建Bean实例
```

### 生命周期的顺序：

```text
-> 生成BeanDefinition
-> 调用 BeanFactoryPostProcessor.postProcessBeanFactory
->  bean实例化
    -> 调用 AbstractAutowireCapableBeanFactory.doCreateBean
        -> createBeanInstance
            -> 调用构造方法
-> populateBean属性注入
    -> 调用set方法注入属性
-> BeanPostProcessor.postProcessBeforeInitialization
-> bean初始化initializeBean
    -> afterPropertiesSet
    -> init-method
-> BeanPostProcessor.postProcessAfterInitialization
```

#### 1.Bean实例化

在实例化之前会调用BeanFactoryPostProcessor的方法修改BeanDefinition
调用AbstractAutowireCapableBeanFactory.doCreateBean，createBeanInstance 实例化Bean

#### 2.属性注入

populateBean

#### 3.Bean初始化

在调用Bean初始化之前会调用BeanPostProcessor postProcessBeforeInitialization

- 1.实现InitializingBean
- 2.实现afterPropertiesSet
- 3.配置init-method
  在调用Bean初始化之后会调用BeanPostProcessor postProcessAfterInitialization

### 1.1.BeanFactoryPostProcessor

该接口中有一个方法：
**void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;**
实现BeanFactoryPostProcessor 接口 通过覆盖postProcessBeanFactory方法，可以在bean实例化之前修改**BeanDefinition**

- 在Spring的bean创建之前，修改bean的定义属性。
  Spring允许BeanFactoryPostProcessor在容器实例化任何其他Bean之前读取配置元数据，并可以根据需要进行修改。
  BeanFactoryPostProcessor是在spring容器加载了bean的beanDefinition之后，在bean实例化之前执行的。
- 可以同时配置多个BeanFactoryPostProcessor，并通过order来控制执行顺序
  (eg:可以吧bean的scope从singleton改为prototype)

```text
@Override
public void postProcessBeanFactory(ConfigurableListableBeanFactory arg0)
        throws BeansException {
    System.out
            .println("BeanFactoryPostProcessor调用postProcessBeanFactory方法");
    BeanDefinition bd = arg0.getBeanDefinition("person");
    bd.getPropertyValues().addPropertyValue("phone", "110");

}
```

### 1.1.1.常见的BeanFactoryPostProcessor实现类

- org.springframework.beans.factory.config.PropertyPlaceholderConfigurer
- org.springframework.beans.factory.config.PropertyOverrideConfigurer
- org.springframework.beans.factory.config.CustomEditorConfigurer

### 1.2.BeanPostProcessor

该接口有两个方法：
**Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException;**
**Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException;**
这两个方法，可以在spring容器实例化bean之后，在执行bean的初始化方法前后，添加一些自己的处理逻辑。
BeanPostProcessor是在spring容器加载了beanDefinition并且实例化bean之后执行的。BeanPostProcessor的执行顺序是在BeanFactoryPostProcessor之后。**springAOP就是在这一步将目标对象通过动态代理转换成代理对象然后返回的。**

1.他是在InitializingBean.afterPropertiesSet或者自定义的初始化方法之前去调用的（**InitializingBean和DisposableBean接口**确保Bean生成和销毁前后做点什么）。

2.BeanPostProcessor.postProcessAfterInitialization是在InitializingBean.afterPropertiesSet或者自定义的初始化方法之后去调用的。

### 1.2.1.常见的BeanPostProcessor实现类

- org.springframework.context.annotation.CommonAnnotationBeanPostProcessor：支持@Resource注解的注入

- org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor：支持@Required注解的注入

- org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor：支持@Autowired注解的注入

- [org.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor](https://link.zhihu.com/?target=https%3A//links.jianshu.com/go%3Fto%3Dhttp%3A%2F%2Forg.springframework.orm.jpa.support.PersistenceAnnotationBeanPostProcessor)：支持@PersistenceUnit和@PersistenceContext注解的注入

- org.springframework.context.support.ApplicationContextAwareProcessor：用来为bean注入ApplicationContext等容器对象

### 将自定义对象放入spring容器的方法：

  FactoryBean是一个用来生产自定义bean的Bean，只要容器中可以把这个Bean的实现类加入那么容器就可以获取到自定义的Bean。

public interface FactoryBean<T> {

```java
//返回的对象实例
T getObject() throws Exception;
//Bean的类型
Class<?> getObjectType();
//true是单例，false是非单例  在Spring5.0中此方法利用了JDK1.8的新特性变成了default方法，返回true
boolean isSingleton();
}
```
```java
//FactoryBean接口的实现类
@Component
public class FactoryBeanLearn implements FactoryBean {

    @Override
    public Object getObject() throws Exception {
        //这个Bean是我们自己new的，这里我们就可以控制Bean的创建过程了
        return new FactoryBeanServiceImpl();
    }

    @Override
    public Class<?> getObjectType() {
        return FactoryBeanService.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
//接口
public interface FactoryBeanService {

    /**
     * 测试FactoryBean
     */
    void testFactoryBean();
}
//实现类
public class FactoryBeanServiceImpl implements FactoryBeanService {
    /**
     * 测试FactoryBean
     */
    @Override
    public void testFactoryBean() {
        System.out.println("我是FactoryBean的一个测试类。。。。");
    }
}
//单测
@Test
public void test() {
        ClassPathXmlApplicationContext cac = new ClassPathXmlApplicationContext("classpath:com/zkn/spring/learn/base/applicationContext.xml");
        FactoryBeanService beanService = cac.getBean(FactoryBeanService.class);
        beanService.testFactoryBean();
    }
```

调用getBean(Class requiredType)方法根据类型来获取容器中的bean的时候，对应的例子就是：根据类型com.zkn.spring.learn.service.FactoryBeanService来从Spring容器中获取Bean**(首先明确的一点是在Spring容器中没有FactoryBeanService类型的BeanDefinition。但是却有一个Bean和FactoryBeanService这个类型有一些关系)**。

Spring在根据type去获取Bean的时候，会先获取到beanName。获取beanName的过程是：先循环Spring容器中的所有的beanName，然后根据beanName获取对应的BeanDefinition，如果当前bean是FactoryBean的类型，则会从Spring容器中根据beanName获取对应的Bean实例，接着调用获取到的Bean实例的getObjectType方法获取到Class类型，判断此Class类型和我们传入的Class是否是同一类型。如果是则返回beanName，对应到我们这里就是：根据factoryBeanLearn获取到FactoryBeanLearn实例，调用FactoryBeanLearn的getObjectType方法获取到返回值FactoryBeanService.class。和我们传入的类型一致，所以这里获取的beanName为factoryBeanLearn。换句话说这里我们把factoryBeanLearn这个beanName映射为了：FactoryBeanService类型。即FactoryBeanService类型对应的beanName为factoryBeanLearn这是很重要的一点。在这里我们也看到了FactoryBean中三个方法中的一个所发挥作用的地方。

### 依赖注入过程

当一个bean(UserController对象)被创建时，该bean所拥有的依赖（userService）就要被注入（实例化），依赖注入过程同样是调用BeanFactory，获取所依赖的属性（依赖注入的方式：构造方法，set方式，注解方式等， 获取方法不尽相同），再查找xml配置文件，通过反射实例化对应的依赖，并将实例化后的依赖（userService）封装（反射相关操作）进bean(UserController对象)中，至此，依赖注入完成。

这里补充一个特殊情况：比如Service1引用了Service2，而Service2也引用了Service1。如果现在正在创建Service1，发现它依赖了Service2，在向Service1注入Service2的过程中肯定要先去创建Service2。这时候发现又要创建Service1，当然不能再创建一个Service1了，而是先将未初始化好的Service1引用先注入到Service2中，然后初始化Service2后再回过来注入到Service1中。