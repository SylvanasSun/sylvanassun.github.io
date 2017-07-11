---
layout:     post
title:      "Spring Data 事务&缓存"
subtitle:   "Spring Data Transation&Cache"
date:       2016-08-01 18:00
author:     "Sylvanas Sun"
header-img: "img/post-bg-universe.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 后端开发
    - Spring
    - Java
---


### Spring Transaction


----------


&nbsp;&nbsp;Spring的事务机制是用统一的机制来处理不同数据访问计数的事务处理.它提供了一个**PlatformTransactionManager**接口,不同的数据访问技术使用不同的接口实现.

| 数据访问技术 |接口实现 |
| ---------- | ---------------------------- |
| JDBC       | DataSourceTransactionManager |
| JPA        | JpaTransactionManager        |
| Hibernate  | HibernateTransactionManager  |
| JDO        | JdoTransactionManager        |
| 分布式事务 | JtaTransactionManager        |

#### 声明式事务


----------


&nbsp;&nbsp;Spring使用`@Transactional`注解在方法上表明事务支持.被注解的方法在被调用时,会开启一个新的事务,当方法无异常完成提交后.Spring会提交事务.

&nbsp;&nbsp;`@Transactional`注解也可以用在类上,表明这个类下的所有方法都有事务支持.如果类和方法都使用了`@Transactional`,则使用方法上的注解覆盖类级别注解.

&nbsp;&nbsp;`@Transactional`注解是基于AOP的实现操作.

&nbsp;&nbsp;注意:`@Transactional`注解是由Spring提供的,而不是来自javax.transaction.

&nbsp;&nbsp;Spring提供`@EnableTransactionManagerment`注解在配置类上开启声明式事务,Spring容器会自动扫描带有注解`@Transactional`的类和方法.

#### Spring Data JPA支持


----------


&nbsp;&nbsp;Spring Data JPA对所有方法默认开启了事务支持,查询类事务默认启用readOnly-true属性.

#### Spring Boot支持


----------


&nbsp;&nbsp;Spring Boot会自动配置事务管理器.

 - 当使用JDBC时,Spring Boot会自动配置DataSourceTransactionManager.
 - 当使用JPA时,Spring Boot会自动配置JpaTransactionManager.

&nbsp;&nbsp;在Spring Boot中,不需要显式开启`@EnableTransactionManagement`注解.

### Spring Cache


----------


&nbsp;&nbsp;Spring提供了CacheManager和Cache接口用来统一各种不同的数据缓存技术.

 - CacheManager是各种缓存技术抽象接口.
 - Cache接口包含各种缓存操作.

| CacheManager              | 描述                                       |
| ------------------------- | ------------------------------------------ |
| SimpleCacheManager        | 使用简单的Collection存储缓存,主要用于测试. |
| NoOpCacheManager          | 不会实际存储缓存.                          |
| EhCacheCacheManager       | 使用EhCache缓存技术.                       |
| ConcurrentMapCacheManager | 使用ConcurrentMap存储缓存.                 |
| GuavaCacheManager         | 使用Google Guava的GuavaCache缓存技术.      |
| HazelcastCacheManager     | 使用Hazelcast缓存技术.                     |
| JCacheCacheManager        | 支持JCache(JSR-107)规范的实现作为缓存技术. |
| RedisCacheManager         | 使用Redis作为缓存技术.                     |


**&nbsp;&nbsp;不管使用什么缓存技术,都需要注册一个该实现的CacheManager的Bean.**

#### 声明式缓存


----------


&nbsp;&nbsp;Spring提供了以下注解用来声明式缓存.它与`@Transactional`注解一样是基于AOP操作的实现.

| Annotation  | Description                                                                                         |
| ----------- | --------------------------------------------------------------------------------------------------- |
| @CachePut   | 不管什么情况,都会把方法的返回值存入缓存中.@CachePut的属性与@Cacheable保持一致.                      |
| @Cacheable  | Spring会先查看缓存中是否存有数据,如果有,则直接返回缓存数据,如果没有,则将调用方法的返回值存入缓存中. |
| @Caching    | 可以通过@Caching注解组合多个注解策略在一个方法上.                                                   |
| @CacheEvict | 将一条或多条缓存数据从缓存中删除.                                                                   |

**&nbsp;&nbsp;开启声明式缓存需要在配置类上使用注解`@EnableCaching`**

#### Example


----------


![](http://ww2.sinaimg.cn/mw690/63503acbjw1f6ldwfwuexj20ol0k5afd.jpg)

#### Spring Boot支持


----------


&nbsp;&nbsp;Spring Boot自动配置了CacheManager的各种实现,默认情况下使用的是ConcurrentMapCacheManager.支持以`spring.cache`为前缀的属性来配置缓存相关的配置.

```
spring.cache.type= #缓存技术的类型,可选ehcache,guava,simple,none,generic,hazelcast,infinispan,jcache,redis.
spring.cache.cache-name=#程序启动时创建缓存名称
spring.cache.ehcache.config=#ehcache配置文件的地址.
spring.cache.hazelcast.config=#hazelcast配置文件的地址.
spring.cache.infinispan.config=#infinispan配置文件的地址.
spring.cache.jcache.config=#jcache配置文件的地址.
spring.cache.jcache.provider=#当多个jcache实现在类路径中的时候,指定jcache实现.
spring.cache.guava.spec=# guava specs
```

**&nbsp;&nbsp;使用Spring Boot只需要导入相关缓存技术的依赖,并在配置类使用`@EnableCaching`注解开启缓存支持即可.**

### end

> 资料参考于 JavaEE开发的颠覆者: Spring Boot实战
