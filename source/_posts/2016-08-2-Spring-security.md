---
layout:     post
title:      "Spring Security安全控制"
subtitle:   "Spring Security"
date:       2016-08-02 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - Java
    - Spring Boot
tags:
    - Spring Boot
    - Spring
    - Java
---


### 概述


----------


&nbsp;&nbsp;Spring Security是Spring项目的安全框架,基于IoC和AOP来实现安全的功能.

&nbsp;&nbsp;Spring Security有两个重要概念:

 - 认证(Authentication),认证即确认用户可以访问当前系统.
 - 授权(Authorization),授权即确定用户在当前系统下所拥有的权限.

### 配置


----------


&nbsp;&nbsp;Spring Security使用过滤器来实现所有安全的功能,我们只需要注册一个特殊的**DelegationFilterProxy**过滤器到**WebApplicationInitializer**即可(WebApplicationInitializer是Spring提供用来配置Servlet3.0+配置的接口,从而替代web.xml,实现此接口的类会自动被SpringServletContainerInitializer加载,启动Servlet容器).

&nbsp;&nbsp;也可以让自定义的Initializer类继承**AbstractSecurityWebApplicationInitializer**抽象类,这个抽象类实现了WebApplicationInitializer接口,并通过onStartup方法调用.它已经注册了**DelegationFilterProxy**过滤器.

#### Spring Security配置类


----------


&nbsp;&nbsp;只需要在配置类上使用`@EnableWebSecurity`注解,然后继承**WebSecurityConfigurerAdapter**.可以通过重写configure方法来配置相关的内容.

&nbsp;&nbsp;在Spring Security中,通过重写以下方法放行静态资源:

![](http://ww1.sinaimg.cn/mw690/63503acbjw1f6lgu4c6xxj20h404x0tn.jpg)

#### Authentication


----------


&nbsp;&nbsp;在Spring Security中,通过重写以下方法来完成认证的配置:

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6lft8zsnrj20mu01i3yx.jpg)

&nbsp;&nbsp;认证用户需要用户数据的来源,AuthenticationManagerBuilder提供了内存中和JDBC的两种用户数据来源.

 - 内存中

   ![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6lft99qxaj20mj05fjsf.jpg)

 - JDBC

   ![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6lft9j25zj20mo09b40j.jpg)
   
&nbsp;&nbsp;如果需要自定义用户数据来源,则可以通过实现**UserDetailsService**接口.

![Role实体类](http://ww2.sinaimg.cn/mw690/63503acbjw1f6lg2d5ezej20gn0cadhp.jpg)

![User实体类](http://ww4.sinaimg.cn/mw690/63503acbjw1f6lg2cv17nj20ls0g80wk.jpg)

![自定义用户认证](http://ww3.sinaimg.cn/mw690/63503acbjw1f6lg2dfhjxj20qe0d7tcp.jpg)

![注册自定义的用户认证](http://ww2.sinaimg.cn/mw690/63503acbjw1f6lg2dua5xj20md06s75u.jpg)


#### Authorization


----------


&nbsp;&nbsp;在Spring Security中,通过重写以下方法来完成授权的配置:

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f6lgfcr8g3j20hy01jt8z.jpg)

&nbsp;&nbsp;Spring Security使用2种匹配器用来匹配请求路径:

 - antMatchers:使用Ant风格的路径匹配.
 - regexMatchers:使用正则表达式匹配路径.

&nbsp;&nbsp;Spring Security提供以下方法用于安全处理:

| Method                     | Description                          |
| -------------------------- | ------------------------------------ |
| anyRequest()               | 匹配所有请求路径.                    |
| access(String)             | Spring EL表达式结果为true时可以访问. |
| anonymous()                | 匿名可访问.                          |
| denyAll()                  | 用户不能访问.                        |
| fullyAuthenticated()       | 用户完全认证时可访问.                |
| hasAnyAuthority(String...) | 如果用户有参数,则其中任一权限可访问. |
| hasAnyRole(String...)      | 如果用户有参数,则其中任一角色可访问. |
| hasAuthority(String)       | 如果用户有参数,则其权限可访问.       |
| hasIpAddress(String)       | 如果用户来自参数中的IP则可访问.      |
| hasRole(String)            | 如果用户有参数中的角色可访问.        |
| permitAll()                | 用户可任意访问.                      |
| rememberMe()               | 允许通过remember-me登陆的用户访问.   |
| authenticated()            | 用户登录后可访问                     |

### 自定义登录实现


----------


```java
 @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                .anyRequest().authenticated() // 所有请求需要认证登陆后才能访问.
                .and()
                .formLogin() // 通过formLogin定制登录操作.
                    .loginPage("/login") // 定制登录页面的访问地址.
                    .defaultSuccessUrl("/index") // 登录成功后转向的页面.
                    .failureUrl("/login?error") // 登录失败后转向的页面.
                    .permitAll() 
                .and()
                .rememberMe() // 开启cookie存储用户信息.
                    .tokenValiditySeconds(1209600) // 指定cookie的有效时间,单位为秒.
                    .key("userKey") // 指定cookie中的私钥.
                .and()
                .logout() // 通过logout()定制注销操作.
                    .logoutUrl("/logout") // 指定注销的URL路径.
                    .logoutSuccessUrl("/index") // 指定注销成功后转向的页面.
                .permitAll();    
    }
```

### Spring Boot支持


----------


&nbsp;&nbsp;Spring Boot主要通过SecurityAutoConfiguration和SecurityProperties来完成自动配置.

&nbsp;&nbsp;SecurityAutoConfiguration导入了SpringBootWebSecurityConfiguration中的配置,我们可以获得如下的自动配置:

 1. 自动配置了一个内存中的用户,账号为user,密码在程序启动时print在控制台中.
 2. 放行了`/css/**`,`/js/**`,`/images/**`,`/**/favicon.ico`等静态文件存放的路径.
 3. 自动配置的securityFilterChainRegistration的Bean.
 4. 使用以`security`为前缀的属性配置Security相关的配置.
    ```
    security.user.name=#内存中的默认用户账号,默认为user.
    security.user.password=#默认用户的密码.
    security.user.role=#默认用户的角色.
    security.require-ssl=false #是否需要ssl支持,默认为false.
    security.enable-csrf=false #是否开启“跨站请求伪造”支持,默认false.
    security.ignored= #用逗号隔开需要放行的路径.
    ....
    ```
**当我们需要自定义扩展配置的时候,只需要配置类继承WebSecurityConfigurerAdapter即可,不需要使用`@EnableWebSecurity`注解开启支持.**    

### end

> 资料参考于 JavaEE开发的颠覆者: Spring Boot实战
