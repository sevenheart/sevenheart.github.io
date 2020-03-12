---
type: Spring
title: Bean的循环依赖
date: 2020-02-23
category: Spring
---

 ###  SpringBean的循环依赖

#### 问题描述：

```
@Component
public class A {
  private B b;
  public void setB(B b) {
    this.b = b;
  }
}
@Component
public class B {
  private A a;
  public void setA(A a) {
    this.a = a;
  }
}
```

如上的情况如果要注入相关的Bean属性就是循环依赖了。

**Bean的创建可以通过application.getBean()创建**

#### 解决方法：

根据Bean的生命周期来看，Bean的创建有多部生成：
第一步：根据BeanDefinition对象经过BeanFactoryPostProcess的实现类去实例化Bean（通过反射）此时Bean未发生属性注入。

第二步：对生成的实例进行属性注入。

第三步：经过BeanPostProcess（后置处理器）中的方法处理生成正真可以使用的Bean

**分析：**

1.A对象首先实例化完成后，开始属性注入。

2.发现B的Bean不存在开始初始化B。

3.B实例化完成以后B就存在了。

4.B开始属性注入，发现A也存在（A是半成品，无属性注入），就把A注入然后返回。

5.这时候A就创建好了。

因为半成品的A、B他们的地址都是一样的所以不在乎是否完整的生成。

