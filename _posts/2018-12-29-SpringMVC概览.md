---
title: SpringMVC简单分析
categories:
 - Spring全家桶
tags:
 - SpringMVC
typora-root-url: ..\sources
---

### SpringMVC概览

##### 一、请求流程：

![](/img/springmvc执行流程.png)

##### 二、请求处理过程概述及内部执行流程

![](/img/springmvc处理流程.png)

![](/img/springmvc内部执行流程.png)

##### 三、源码流程概览

![](/img/DisparchServlet初始化.png)

![](/img/springmvc中各组件的做法.png)

**解析：**

servlet的配置文件在WEB-INFO指定位置就是为了tomcat可以找到并且创建servlet。

dispatcherServlet类下有一个配置文件里边配置了所有的处理器、适配器视图解析器等。该文件在DispatcherServlet的静态代码块中随着类加载而被解析加载。

# Normal block

```
alert('Hello World!');
```

    print 'helloworld'

# Highlight block

```javascript
alert( 'Hello, world!' );
```

```python
print 'helloworld'
```

```ruby
def foo
  puts 'foo'
end
```

{% highlight ruby %}
def foo
  puts 'foo'
end
{% endhighlight %}

{% highlight ruby linenos %}
def foo
  puts 'foo'
end
{% endhighlight %}
