---
title: springsecurity个人理解及一些概述
date: 2020-01-07 09:30:09
categories:
- springSecurity
---

SpringSecurity的一些使用概述及个人理解

####  springsecurity

它是基于spring的一套权限管理框架，它实质是由一系列的 拦截器链去做权限校验的。

通常的资源访问方式：

client——>fliter——>fliter——>servlet——>调用资源

springsecurity就是在项目启动前配置好过滤器，并且在过滤器中为指定的资源分配指定的角色，只有访问资源的用户拥有这个角色，那么就能够访问资源，否则返回权限不足的错误。

如果将访问的资源路径加入了security的配置文件中但是没有为它设置相应角色的身份认证，那么该资源不受约束可以随意访问。

配置文件Demo：

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
@Configuration
/*启用拦截*/
@EnableWebSecurity
public class MySecurityConfig extends WebSecurityConfigurerAdapter {
    /**
     * 配置一个给密码加密的方式，为了支持security5必须给密码加密而做的操作。
     * @return
     */
    @Bean
    public PasswordEncoder createPasswordEncode(){
        //这里没有md5加密，security舍弃了它。不安全，
        //这里采用BCryptPasswordEncoder,会自动生产盐比较安全。
        return new BCryptPasswordEncoder();
    }
    /**
     * 配置认证资源以及认证的方式。
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        //设置认证规则，在spring中是通过配置文件中的http标签设置的。
        //对所有请求进行认证
        http.
                requestMatchers()//请求匹配
                .antMatchers("/**")//匹配那些请求
                .and()
                .authorizeRequests()//认证请求
                //拦截的底部有所有资源的拦截，所以自定义登录页面以及异常页面也会拦截，需要放行
                //需要进行认证的资源以及那个角色可以访问这个资源
                .antMatchers("/login").permitAll()//放行登录，不配置也可以不配做默认放行
                .antMatchers("/product/add").hasAnyAuthority("PRODUCT_ADD")
                .antMatchers("/product/update").hasAnyAuthority("PRODUCT_UPDATE")
                .antMatchers("/product/delete").hasAnyAuthority("PRODUCT_DELETE")
                .antMatchers("/product/select").hasAnyAuthority("PRODUCT_SELECT")
              //  .antMatchers("/foo/**").hasAnyAuthority("PRODUCT_ADD")
                //需要进行认证的资源以及那个角色可以访问这个资源
                .and()
                //全部采用表单登录的认证方式，也就是默认的认证方式
                //四的时候默认是.httpBasic方式
                //.httpBasic();//弹框形式
               .formLogin();//默认表单形式，如果前后端分离的项目可以采用，不加他
               // .formLogin().loginPage("/login")自定义登陆页面
            //禁用CSRF跨站访问，不然点击登录也没效果;自己实验不配置也行或许时版本原因。
               // .csrf().disable();
               //配置自定义的登录路径
    }
}

```

登录信息及角色配置：可以通过编码的方式写在业务逻辑中

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetails;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.core.userdetails.UsernameNotFoundException;
import org.springframework.security.crypto.password.PasswordEncoder;
import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
/*获取用户信息包含用户的密码等*/
@Configuration
public class MyUserDetailService  implements UserDetailsService {
    //加密方式，在配置类中有配置。会自动装配
    //login时搞一个方法调用这个验证方法同是验证token
    @Autowired
    private PasswordEncoder passwordEncoder;

    /**
     * 可以通过这个方法传入一个用户名然后返回一个UserDetail可以包含密码等信息。
     * @param
     * @return
     * @throws UsernameNotFoundException
     */
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        //UserDetail是个接口它的父类是user类
        //密码必须是加密的，security5之后密码不加密会报错
        //参数三是一个角色集合，可以采用创建集合或者用AuthorityUtils.commaSeparatedStringToAuthorityList()工具类，
        // 把字符串转成角色集合
        System.out.println("用户名是"+username);
        //对密码进行加密
        //这里的密码就可以从数据库取了。
        String encoderpassword = passwordEncoder.encode("123456");

        System.out.println("加密后的密码是"+encoderpassword);
        //System.out.println("输入的密码和查找的密码是否匹配"+passwordEncoder.matches(encoderpassword,"123456"));
        //调用添加角色的方法，生成角色集合。
        Collection<GrantedAuthority> authoritesList = authorities();

        //添加单一角色：
        //AuthorityUtils.commaSeparatedStringToAuthorityList("Role_User"）
        return new User(username,encoderpassword,authoritesList);
    }
    /**
     * 为当前用户指定一些角色
     * User的构造方法角色是一个角色Collection所以创建一个返回角色集合的方法，
     * AuthorityUtils.commaSeparatedStringToAuthorityList("Role_User")只能创建一个角色不适用。
     */
    private Collection<GrantedAuthority> authorities(){
        //添加两个角色，分别有添加和修改的权限
        List<GrantedAuthority> authorityList  = new ArrayList<GrantedAuthority>();
                authorityList.add(new SimpleGrantedAuthority("PRODUCT_ADD"));
                authorityList.add(new SimpleGrantedAuthority("PRODUCT_UPDATE"));
        return authorityList;
    }
}
```

以上是一个默认登录的配置，下面写一个场景：前后端分离的项目前端登录携带username以及密码，后端对该用户进行token校验以及相关角色的赋予。

参考链接:

[https://blog.csdn.net/feiyangtianyao/article/details/93194577]: 	"springsecurity中的登录验证"



#### 将spring security的登录及角色赋予写在业务逻辑中

前后端分离的项目中如果没有登录那么应该返回json然后前端跳到登录页面，所以后端不能自己跳到登录页面。

**在配置类中不写认证方式就不会出现自动跳转到认证页面的行为。**

##### 表单形式提交登录认证流程:

**springSecurity中的登录流程是在UsernamePasswordAuthenticationFilter中验证**的，验证过程如下：

1. 首先过滤器会调用自身的attemptAuthentication方法，从request中取出authentication, authentication是在**org.springframework.security.web.context.SecurityContextPersistenceFilter**过滤器中通过捕获用户提交的登录表单中的内容生成的一个**org.springframework.security.core.Authentication**接口实例.

2. 拿到authentication对象后，过滤器会调用ProviderManager类的authenticate方法，并传入该对象
3. ProviderManager类的authenticate方法再调用自身的doAuthentication方法，在doAuthentication方法中会调用类中的List<AuthenticationProvider> providers集合中的各个AuthenticationProvider接口实现类中的authenticate(Authentication authentication)方法进行验证，由此可见，真正的验证逻辑是由各个各个AuthenticationProvider接口实现类来完成的,DaoAuthenticationProvider类是默认情况下注入的一个AuthenticationProvider接口实现类
4. AuthenticationProvider接口通过UserDetailsService来获取用户信息

##### 非表单形式提交登录验证流程：限定了一个登录的路径的访问才被拦截

非表单形式提交UsernamePasswordAuthenticationFilter无法过滤到否则可以用上面的方法去做，所以需要重写该方法，然后主动去获取request中的信息并继续交给spring处理验证什么的都可以自己查数据库。

##### 资源访问token验证流程：

在每个资源访问时都去BasicAuthenticationFilter过滤器完成token的验证然后生成Authentication交给springsecurity去管理即可。所以重写BasicAuthenticationFilter去完成自定义的token验证然后生成Authentication放入spring security这样就算是认证成功了。

参考链接：

[https://blog.csdn.net/feiyangtianyao/article/details/93505598]: 	"资源权限访问token"

