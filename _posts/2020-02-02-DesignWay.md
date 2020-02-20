---
title: 设计模式学习
date: 2020-02-01 
categories:
- 设计模式
---

### springBean的生命周期

## 创建型模式

- 简单工厂模式
- 工厂方法模式
- 抽象工厂模式
- 建造者模式
- 原型模式
- 单例模式

### 工厂模式：

#### 简单工厂模式：

​	通过抽象一个用来生产的接口，不同的类做不同的实现。工厂方法中提供创造方法，根据传入的参数决定创建那个产品。

```java
//生产图形的接口
public interface Shape {
    void draw();
}
//生产圆

public class Circle implements Shape {
    public Circle() {
        System.out.println("Circle");
    }
    @Override
    public void draw() {
        System.out.println("Draw Circle");
    }
}


//生产四边形

public class Rectangle implements Shape {
    public Rectangle() {
        System.out.println("Rectangle");
    }
    @Override
    public void draw() {
        System.out.println("Draw Rectangle");
    }
}
//创建工厂
public class ShapeFactory {
    // 使用 getShape 方法获取形状类型的对象
    public static Shape getShape(String shapeType) {
        //可通过反射创建避免违反开闭原则
        if (shapeType == null) {
            return null;
        }
        if (shapeType.equalsIgnoreCase("CIRCLE")) {
            return new Circle();
        } else if (shapeType.equalsIgnoreCase("RECTANGLE")) {
            return new Rectangle();
        } else if (shapeType.equalsIgnoreCase("SQUARE")) {
            return new Square();
        }
        return null;
    }
}


```

​	以上的工厂模式违反了开闭原则，但是可以通过反射改进达到传入全路径名动态创建。

#### 工厂方法模式：

​	不同的产品用不同的工厂去生产，这样就可以达到很好的去扩展。

```java
//工厂抽象类，不同工厂去做实现
public interface Factory {
	public Shape getShape();
}
//产圆工厂
public class CircleFactory implements Factory {

    @Override
    public Shape getShape() {
        // TODO Auto-generated method stub
        return new Circle();
    }

}


//产四边形工厂
public class RectangleFactory implements Factory{

    @Override
    public Shape getShape() {
        // TODO Auto-generated method stub
        return new Rectangle();
    }

}


```
#### 抽象工厂模式

​	用来生产一个产品家族，比如说生产枪要生产对应的子弹。把工厂在做一次抽象一个产品家族的生产需要多个工厂。

```java
//枪
public interface Gun {
    public void shooting();
}
//子弹
public interface Bullet {
    public void load();
}
//生产AK
public class AK implements Gun{

    @Override
    public void shooting() {
        System.out.println("shooting with AK");

    }

}
//生产M41
public class M4A1 implements Gun {

    @Override
    public void shooting() {
        System.out.println("shooting with M4A1");

    }

}

//生产AK子弹
public class AK_Bullet implements Bullet {

    @Override
    public void load() {
        System.out.println("Load bullets with AK");
    }

}
//生产M4子弹
public class M4A1
_Bullet implements Bullet {

    @Override
    public void load() {
        System.out.println("Load bullets with M4A1");
    }

}
//工厂接口
public interface Factory {
    public Gun produceGun();
    public Bullet produceBullet();
}
//生产AK和AK子弹的工厂
public class AK_Factory implements Factory{

    @Override
    public Gun produceGun() {
        return new AK();
    }

    @Override
    public Bullet produceBullet() {
        return new AK_Bullet();
    }

}
//生产M41和M41子弹的工厂
public class M4A1_Factory implements Factory{

    @Override
    public Gun produceGun() {
        return new M4A1();
    }

    @Override
    public Bullet produceBullet() {
        return new M4A1_Bullet();
    }

}

```

### 原型模式

​	**使用原型实例指定将要创建的对象类型，通过复制这个实例创建新的对象。**-------------方便创建新对象。

在原型模式结构中定义了一个抽象原型类，所有的Java类都继承自java.lang.Object，而Object类提供一个clone()方法，可以将一个Java对象复制一份。因此在Java中可以直接使用Object提供的clone()方法来实现对象的克隆，Java语言中的原型模式实现很简单。

能够实现克隆的Java类必须实现一个标识接口Cloneable，表示这个Java类支持复制。如果一个类没有实现这个接口但是调用了clone()方法，Java编译器将抛出一个CloneNotSupportedException异常。

##### 案例：

​	1.复制粘贴

​	2.spring中对Bean实例的创建，达到每次创建的Bean不对原来的Bean有影响。

##### clone方法

clone在第一步是和new相似的， 都是分配内存，调用clone方法时，分配的内存和源对象（即调用clone方法的对象）相同，然后再使用原对象中对应的各个域，填充新对象的域（**new是调用构造函数去填充**）， 填充完成之后，clone方法返回，一个新的相同的对象被创建，同样可以把这个新对象的引用发布到外部 。

**浅拷贝：**成员变量是一个实例对象时，新的对象拷贝的是实例对象的引用。（引用复制）

**深拷贝：**成员变量是一个实例对象时，新的对象是新复制对象的引用。（对象复制）

**clone方法执行的是浅拷贝，如果想要实现深拷贝，可以通过覆盖Object中的clone方法的方式。**

## 结构型模式

- 适配器模式

- 桥接模式

- 组合模式

- 装饰模式

- 外观模式

- 享元模式

- 代理模式

### 适配器模式

​	Adapter 模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。通常用来后期的维护和扩展。

```java
//客户端的请求处理方法
public interface Target {
	public void request();
}
//已存在的接口
public class Adaptee {
 /*
 * 原本存在的方法
 */
	public void specificRequest(){
		//业务代码
	}
}
//适配器将他们结合工作
public class Adapter implements Target{
 /*
 * 持有需要被适配的接口对象
 */
	private Adaptee adaptee;
/*
 * 构造方法，传入需要被适配的对象
 * @param adaptee 需要被适配的对象
 */
	public Adapter(Adaptee adaptee){
		this.adaptee = adaptee;
	}
	@Override
	public void request() {
		// TODO Auto-generated method stub
		adaptee.specificRequest();
	}  
}
//客户端
/*
 * 使用适配器的客户端
 */
public class Client {
 	public static void main(String[] args){
	 //创建需要被适配的对象
 	Adaptee adaptee = new Adaptee();
 	//创建客户端需要调用的接口对象
 	Target target = new Adapter(adaptee);
 	//请求处理
	 target.request();
	}	
}


```

使用情况：

1. 当你想使用一个已经存在的类，而它的接口不符合你的需求；
2. 你想创建一个可以复用的类，该类可以与其他不相关的类或不可预见的类协同工作；
3. 你想使用一些已经存在的子类，但是不可能对每一个都进行子类化以匹配它们的接口，对象适配器可以适配它的父亲接口。

### 代理模式

​	用户不与真正提供服务的用户接触，用户通过代理类去做默写事，事实上是代理类与真实对象的交互，交互结束后把结果返回给用户，以用户的角度来看是代理类提供了服务。

应用：

#### 延迟加载

延迟加载的核心思想是：如果当前并没有使用这个组件，则不需要真正地初始化它，使用一个代理对象替代它的原有的位置，只要在真正需要的时候才对它进行加载。比如数据库连接不做查询没必要初始化某些类，可以直接返回一个代理类去表示连接成功了，但是一些不用的类并没有加载。

#### 动态代理

​	利用反射机制在运行时创建代理类。

​	**jdk动态代理：**代理类有一个invocationHandler的实现去对真实对象进行增强，只需要在构造函数中传入真实	对象即可。而生成代理类需要三个参数，真是对象的类加载器，真实对象的接口，处理真实对象的		invocationhandler。

```java
/**
* 这是invocationHandler类
**/
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
public class SellProxy implements InvocationHandler {
    private Object realObject;
    public SellProxy(Object object){
        this.realObject = object;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("调用真实对象之前");
        method.invoke(realObject,args);
        System.out.println("调用真是对象之后");
        return null;
    }
}
/**
* 这是真实对象的一个接口之一
**/
public interface SellGun {
    //卖枪接口
    void sellGun();
}
/**
* 这是真实对象的实现之一
**/
public class AKGun implements SellGun {
    @Override
    public void sellGun() {
        System.out.println("我卖的是AK");
    }
}
/**
* 这是测试类
**/
public class test {
    public static void main(String[] args) {
        AKGun akGun = new AKGun();
        M41Gun m41Gun = new M41Gun();
        //为他们创建每个代理类需要的关键对象IncocationHandler
        InvocationHandler invocationHandler1 = new SellProxy(akGun);
        InvocationHandler invocationHandler2 = new SellProxy(m41Gun);
        //创建代理对象
        SellGun akDynamicProxy = (SellGun) Proxy.newProxyInstance(AKGun.class.getClassLoader(),AKGun.class.getInterfaces(),invocationHandler1);
        SellGun m41DynamicProxy = (SellGun) Proxy.newProxyInstance(M41Gun.class.getClassLoader(),M41Gun.class.getInterfaces(),invocationHandler2);
        //开始调用
        akDynamicProxy.sellGun();
        m41DynamicProxy.sellGun();

        //如果要卖其他种类的商品只需要添加一个接口去实现它卖的具体内容即可，比如卖可乐
        Kole kole = new Kole();
        InvocationHandler invocationHandler3 = new SellProxy(kole);
        Drink koleDynamicProxy = (Drink) Proxy.newProxyInstance(Kole.class.getClassLoader(),Kole.class.getInterfaces(),invocationHandler3);
        koleDynamicProxy.sellDrink();

    }
}
```



## 行为型模式

- 职责链模式

- 命令模式

- 解释器模式

-  迭代器模式

- 中介者模式

- 备忘录模式

- 观察者模式

- 状态模式

- 策略模式

- 模板方法模式

- 访问者模式

  

### 责任链模式

使多个对象都有机会处理请求，从而避免请求的发送者和接受者之间的耦合关系，将这个对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理他为止。

**举例：**一条过滤器链。   

```java
//责任链上的对象
public abstract class Handler {
    /**
     * 持有后继的责任对象
     */
    protected Handler successor;
    /**
     * 示意处理请求的方法，虽然这个示意方法是没有传入参数的
     * 但实际是可以传入参数的，根据具体需要来选择是否传递参数
     */
    public abstract void handleRequest();
    /**
     * 取值方法
     */
    public Handler getSuccessor() {
        return successor;
    }
    /**
     * 赋值方法，设置后继的责任对象
     */
    public void setSuccessor(Handler successor) {
        this.successor = successor;
    }  
}

//处理流程

public class ConcreteHandler extends Handler {
    /**
     * 处理方法，调用此方法处理请求
     */
    @Override
    public void handleRequest() {
        /**
         * 判断是否有后继的责任对象
         * 如果有，就转发请求给后继的责任对象
         * 如果没有，则处理请求
         */
        if(getSuccessor() != null)
        {            
            System.out.println("放过请求");
            getSuccessor().handleRequest();            
        }else
        {            
            System.out.println("处理请求");
        }
    }
 
//测试方法：
 public static void main(String[] args) {
        //组装责任链
        Handler handler1 = new ConcreteHandler();
        Handler handler2 = new ConcreteHandler();
        handler1.setSuccessor(handler2);
        //提交请求
        handler1.handleRequest();
	}



```





### 迭代器模式

### 观察者模式

### 策略模式

### 模板方法模式

对于大量的重复操作，其中一步不同的操作要用模板方法模式如：jdbctemplete，redistemplete。

