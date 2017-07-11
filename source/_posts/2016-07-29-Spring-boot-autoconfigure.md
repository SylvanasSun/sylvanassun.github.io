---
layout:     post
title:      "浅谈Spring Boot自动配置的运作原理"
subtitle:   "Spring Boot AutoConfiguration"
date:       2016-07-29 18:00
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - Java
    - Spring Boot
tags:
    - Spring
    - Spring Boot
    - Java
---



### Spring Boot

&nbsp;&nbsp;Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。它使用“习惯优于配置”的理念可以让你的项目快速运行部署。使用Spring Boot可以不用或者只需要很少的Spring配置。

&nbsp;&nbsp;而Spring Boot核心的功能就是自动配置。它会根据在类路径中的jar、类自动配置Bean,当我们需要配置的Bean没有被Spring Boot提供支持时,也可以自定义自动配置。

### 自动配置的运作原理

**&nbsp;&nbsp;Spring Boot自动配置其实是基于Spring 4.x提供的条件配置(Conditional)实现的。**

&nbsp;&nbsp;有关自动配置的源码在spring-boot-autoconfigure-1.x.x.jar内,如下图:

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6b6vnzlfgj20co0kimyz.jpg)

#### 如何查看当前项目已启动和未启动的自动配置

 1. 在application.propertie中设置debug=true属性.
 2. 在运行jar时添加--debug指令.

&nbsp;&nbsp;当使用以上两种任意一种方法后,启动项目会在控制台输出已启动和未启动的自动配置日志.

#### @SpringBootApplication

&nbsp;&nbsp;生成Spring Boot项目时,会自动生成一个入口类.入口类使用了@SpringBootApplication注解,它是Spring Boot的核心注解,它是一个组合注解,核心功能由@EnableAutoConfiguration注解提供.

![@SpringBootApplication](http://ww1.sinaimg.cn/mw690/63503acbjw1f6b772ib2sj20he0ft77h.jpg)

#### @EnableAutoConfiguration

![@EnableAutoConfiguration](http://ww3.sinaimg.cn/mw690/63503acbjw1f6b772m676j20gq07qdhi.jpg)

 1. @Import注解提供导入配置的功能,它导入了EnableAutoConfigurationImportSelector.
 2. EnableAutoConfigurationImportSelector使用函数SpringFactoriesLoader.loadFactoryNames扫描META-INF/spring.factories文件中声明的jar包.

![](http://ww1.sinaimg.cn/mw690/63503acbjw1f6b7h23prdj213c0kh7g2.jpg)

 3. spring.factories文件在spring-boot-autoconfigure-1.x.x.jar中.

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f6b7lru822j20ei05ggmu.jpg)

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6b7lryzw2j20ut0ntaok.jpg)

 4. spring.factories中声明的类基本上都使用了@Conditional注解.

#### Conditional

&nbsp;&nbsp;Spring Boot在org.springframework.boot.autoconfigure.condition包下定义了以下注解.

| 注解名                          | 作用                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| @ConditionalOnJava              | 基于JVM版本作为判断条件.                                     |
| @ConditionalOnBean              | 当容器中有指定的Bean的条件下.                                |
| @ConditionalOnClass             | 当类路径下游指定的类的条件下.                                |
| @ConditionalOnExpression        | 基于SpEL表达式作为判断条件.                                  |
| @ConditionalOnJndi              | 在JNDI存在的条件下查找指定的位置.                            |
| @ConditionalOnMissingBean       | 当容器中没有指定Bean的情况下.                                |
| @ConditionalOnMissingClass      | 当类路径下没有指定的类的情况下.                              |
| @ConditionalOnNotWebApplication | 当前项目不是web项目的条件下.                                 |
| @ConditionalOnProperty          | 指定的属性是否有指定的值.                                    |
| @ConditionalOnResource          | 类路径是否有指定的值.                                        |
| @ConditionalOnSingleCandidate   | 当指定Bean在容器中只有一个,或者虽然有多个但是指定首选的Bean. |
| @ConditionalOnWebApplication    | 当前项目是web项目的条件下.                                   |

&nbsp;&nbsp;以上这些注解都组合了@Conditional元注解.

### 分析@ConditionalOnNotWebApplication

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f6b8n61hytj20fb04dgmt.jpg)

&nbsp;&nbsp;@ConditionalOnNotWebApplication使用的条件类是OnWebApplicationCondition.

```java
package org.springframework.boot.autoconfigure.condition;

import org.springframework.boot.autoconfigure.condition.ConditionOutcome;
import org.springframework.boot.autoconfigure.condition.ConditionalOnWebApplication;
import org.springframework.boot.autoconfigure.condition.SpringBootCondition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.annotation.Order;
import org.springframework.core.type.AnnotatedTypeMetadata;
import org.springframework.util.ClassUtils;
import org.springframework.util.ObjectUtils;
import org.springframework.web.context.WebApplicationContext;
import org.springframework.web.context.support.StandardServletEnvironment;

@Order(-2147483628)
class OnWebApplicationCondition extends SpringBootCondition {
    private static final String WEB_CONTEXT_CLASS = "org.springframework.web.context.support.GenericWebApplicationContext";

    OnWebApplicationCondition() {
    }

    public ConditionOutcome getMatchOutcome(ConditionContext context, AnnotatedTypeMetadata metadata) {
        boolean webApplicationRequired = metadata.isAnnotated(ConditionalOnWebApplication.class.getName());
        ConditionOutcome webApplication = this.isWebApplication(context, metadata);
        return webApplicationRequired && !webApplication.isMatch()?ConditionOutcome.noMatch(webApplication.getMessage()):(!webApplicationRequired && webApplication.isMatch()?ConditionOutcome.noMatch(webApplication.getMessage()):ConditionOutcome.match(webApplication.getMessage()));
    }

    private ConditionOutcome isWebApplication(ConditionContext context, AnnotatedTypeMetadata metadata) {
        if(!ClassUtils.isPresent("org.springframework.web.context.support.GenericWebApplicationContext", context.getClassLoader())) {
            return ConditionOutcome.noMatch("web application classes not found");
        } else {
            if(context.getBeanFactory() != null) {
                String[] scopes = context.getBeanFactory().getRegisteredScopeNames();
                if(ObjectUtils.containsElement(scopes, "session")) {
                    return ConditionOutcome.match("found web application \'session\' scope");
                }
            }

            return context.getEnvironment() instanceof StandardServletEnvironment?ConditionOutcome.match("found web application StandardServletEnvironment"):(context.getResourceLoader() instanceof WebApplicationContext?ConditionOutcome.match("found web application WebApplicationContext"):ConditionOutcome.noMatch("not a web application"));
        }
    }
}
```

&nbsp;&nbsp;OnWebApplicationCondition在isWebApplication函数中进行条件判断.

 1. 判断GenericWebApplicationContext是否在类路径中.
 2. 判断容器中是否存在名为session的scope.
 3. 判断当前容器的Environment是否为StandardServletEnvironment.
 4. 判断当前的ResourceLoader是否为WebApplicationContext.
 5. 最后通过ConditionOutcome.isMatch函数返回布尔值确定条件.

### 分析自动配置的实现

&nbsp;&nbsp;以http编码为例,如果在常规项目中则需要在web.xml中配置一个filter.而Spring Boot内置了http编码的自动配置,无需配置filter.

#### properties配置类

![](http://ww2.sinaimg.cn/mw690/63503acbjw1f6b9s692hxj20p30geq7e.jpg)

#### 自动配置Bean

![](http://ww1.sinaimg.cn/mw690/63503acbjw1f6b9q6z1y8j20wc0idwkd.jpg)

&nbsp;&nbsp;@ConditionalOnProperty:当设置spring.http.encoding=enabled的情况下,如果没有设置则默认为true,即符合条件.

&nbsp;&nbsp;characterEncodingFilter()返回OrderedCharacterEncodingFilter这个对象,并根据注入的HttpEncodingProperties配置类设置参数.

&nbsp;&nbsp; @ConditionalOnMissingBean({CharacterEncodingFilter.class}):在容器中没有这个Bean的时候则新建这个Bean.

### end

> 资料参考于 JavaEE开发的颠覆者: Spring Boot实战
