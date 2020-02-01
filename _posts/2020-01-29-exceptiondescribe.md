---
title: 异常处理的一些应用场景
date: 2020-01-29 23:29:08
categories:
- exception
---

一些项目中对于异常的使用场景
---


#### 对于前后端分离的项目

##### 前端通过fetch调用后端接口异常处理流程说明

在接触前后端分离的项目中发现，一个前端项目也需要自己的整个运行流程，所以对于前端项目自己的异常处理也不可或缺。

例如：对于正真的服务器异常（服务器连接不上，出现500之类的错误等）在调用API（通常在前端调用api时这个api是对fetch的进一步封装）后的catch就要捕获到所有的异常包括自定义异常和正真的异常。之后在catch中将异常交给全局异常处理去根据异常种类做相应的处理。

所以要在api这层捕获到异常那么在fetch层就要对业务异常做相应的抛出，因为业务异常通常返回的是自定义的JSON数据，对于fetch来说还是200是请求成功的。所以在fetch的then中就要根据返回的自定义状态码去抛出异常好让API层去捕获到这个异常从而做出响应。这就需要后端同一规范返回信息的格式了也就是后端返回的

```react
fetch(url,opts).then(
	res=>{
		//根据状态码去封装成正真的异常供API层捕获
		if(res.status === 402){
			return Promise.reject(new MyError(res.status,res.msg));
		}
	}
	);
//自定义的异常同样需要继承Error类。
	
```

API层面：

```react
function myAPI(url){
	fetch(url,opt);
}
//调用API
myAPI("localhost:8080/index").then(
	//处理正常请求
	).catch({
	//处理被捕捉到的异常包括业务异常，业务异常在fatch层被包装成了异常
	//此处应该调用全局异常处理方法去处理异常。
	})
```

#### 后端异常处理及流程说明

springboot中的全局异常处理：

```java
import com.shop.DTO.Result;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;
import javax.servlet.http.HttpServletRequest;
@ControllerAdvice
public class MyExceptionAdvice {
    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public Result defaultException(HttpServletRequest request, Exception e){
        e.printStackTrace();
        return  new Result("这是返回地messge");
    }
    //可以添加其他自定义异常的处理
}

```

**如果不予处理，正常异常的返回通常是response的响应码是500等程序通过response.sendError去设置response的响应异常码，业务异常我们通常都是返回正常的json数据然后去判断业务异常的**：

```
    //这是不予处理返回地异常信息，如果自己return那么它的respons的状态是200
    "timestamp": "2020-01-29T12:14:30.119+0000",
    "status": 500,
    "error": "Internal Server Error",
    "message": "我的异常",
    "path": "/foo/bar"
```

**想要在前后端分离的项目中前端能统一处理异常，就要保证自定义的业务异常返回的数据格式和正常异常处理返回的数据格式一致，这样才能在前端统一根据状态码去处理**所以需要一个工具类去包装自定义异常类的信息让它和spring自己处理返回的数据格式一致，这个包装过程应该在全局异常处理方法中去进行然后return以此达到正常返回JSON数据的目的。此时业务异常返回地应该是响应的状态码为200的Json数据，然后前端根据这个收到的JSON数据再去封装成前端能够处理的异常种类由前端去处理。

#### 对于前后端不分离的项目

1.对于正常抛出的异常正常使用httpState中的各种状态。

2.对于业务异常自己定义好state枚举类后，还是自己抛出异常然后交给全局异常处理类去处理。

3.对于一些常见的异常比如404、403等可以采用相当于接口调用然后跳转到指定页面。

```java

import org.springframework.boot.web.server.ErrorPage;
import org.springframework.boot.web.server.ErrorPageRegistrar;
import org.springframework.boot.web.server.ErrorPageRegistry;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpStatus;
@Configuration
public class MyErrorPage implements ErrorPageRegistrar {

    //可以拦截所有的错误页面
    //选择参数是错误页码的构造方式取构造errorpage通过选择相应的枚举类就可以指定那类错误交给什么去处理
    @Override
    public void registerErrorPages(ErrorPageRegistry registry) {
        //权限不足的错误页跳转
        ErrorPage errorPage = new ErrorPage(HttpStatus.FORBIDDEN,"/403");//相当于接口调用
        registry.addErrorPages(errorPage);
    }
}
```

4.其他的业务异常就正常在全局异常处理中处理返回相应的数据即可在前端页面ajax接受数据后按业务逻辑处理。

5.**选择参数是错误页码的构造参数去构造errorpage通过选择相应的枚举类就可以指定那类错误交给什么去处理，测试结果，这里是通过返回的响应码去捕获的所以如果全局拦截做了处理成了return 200 那么这里如果注册了拦截200的页面就会在这里拦截。**

这样注册根据错误码拦截页面也可以：

```java
import org.springframework.boot.web.server.ConfigurableWebServerFactory;
import org.springframework.boot.web.server.ErrorPage;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpStatus;
@Configuration
public class ErrorCustomlizer {

    /**
     * 拦截网络请求错误！都跳转到首页
     *
     * @return
     */
    @Bean
    public WebServerFactoryCustomizer<ConfigurableWebServerFactory> webServerFactoryCustomizer() {
        return (container -> {
            ErrorPage error401Page = new ErrorPage(HttpStatus.UNAUTHORIZED, "/index.html");
            ErrorPage error404Page = new ErrorPage(HttpStatus.NOT_FOUND, "/index.html");
            ErrorPage error500Page = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/index.html");
            container.addErrorPages(error401Page, error404Page, error500Page);
        });

    }
}

```

