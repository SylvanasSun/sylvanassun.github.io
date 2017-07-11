---
layout:     post
title:      "Spring数据访问方案-Spring Data"
subtitle:   "Spring Data"
date:       2016-07-31 18:00
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

&nbsp;&nbsp;Spring Data是Spring用来解决数据访问的解决方案,它包含了大量关系型数据库以及NoSQL数据库的数据持久层访问解决方案.
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
#### Spring Data Commons

&nbsp;&nbsp;Spring Data提供了统一的API来对各种数据存储技术进行数据访问操作,这是通过Spring Data Commons来实现的,它是所有Spring Data子项目的依赖.

#### Spring Data Repository

&nbsp;&nbsp;Spring Data Repository是数据访问的统一标准,它是抽象的,不同的数据访问技术有不同的Repository,它的顶级接口为Repository接口.

![](http://ww2.sinaimg.cn/mw690/63503acbjw1f6j3zxk8f5j20i104imxu.jpg)

**&nbsp;&nbsp;Repository用一个实体类型与ID类型作为泛型.**

&nbsp;&nbsp;Repository的子接口CrudRepository定义了CRUD操作的相关内容:

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6j45n3hnxj20q00hs0vq.jpg)

&nbsp;&nbsp;CrudRepository的子接口PagingAndSortingRepository定义了分页与排序的相关内容:

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6j45ne4tnj20ti07aabj.jpg)

### Spring Data JPA

#### JPA规范


----------

&nbsp;&nbsp;JPA是Java Persistence API的缩写,它是一个基于O/R(Object-Relational Mapping)映射的标准规范.例如Hibernate就是JPA规范的实现.

#### 数据访问层


----------


 1. 在Spring Data JPA中定义数据访问层首先需要继承JpaRepository接口.
 2. 之后可以通过@EnableJpaRepository注解开启Spring Data JPA的支持,@EnableJpaRepository接收的value参数用于扫描数据访问层所在包下的接口定义.

#### 查询方法


----------


&nbsp;&nbsp;Spring Data JPA支持通过定义在Repository接口中的方法名来定义查询,方法名是根据实体类的属性名来确定的.

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6j94ghv86j20ma0bjtaw.jpg)

&nbsp;&nbsp;其中findBy关键字可以用find、read、readBy、query、queryBy、get、getBy替代.

&nbsp;&nbsp;Spring Data JPA可以使用top和first关键字查询指定数量的数据.

![](http://ww1.sinaimg.cn/mw690/63503acbjw1f6j9asrv0sj20dq0543yx.jpg)

#### 查询关键字


----------

| 关键字            | 示例                                | 同功能JPQL                            |
| ----------------- | ----------------------------------- | ------------------------------------- |
| And               | findByNameAndAge                    | where x.name = ?1 and x.age = ?2      |
| Or                | findByNameOrAge                     | where x.name = ?1 or x.age = ?2       |
| Is                | findByNameIs                        | where x.name = ?1                     |
| Equals            | findByNameEquals                    | where x.name = ?1                     |
| Between           | findByAgeBetween                    | where x.age between ?1 and ?2         |
| LessThan          | findByAgeLessThan                   | where x.age < ?1                      |
| LessThanEqual     | findByAgeLessThanEqual              | where x.age <= ?1                     |
| GreaterThan       | findByAgeGreaterThan                | where x.age > ?1                      |
| GreaterThanEqual  | findByAgeGreaterThanEqual           | where x.age >= ?1                     |
| After             | findByStartDateAfter                | where x.startDate > ?1                |
| Before            | findByStartDateBefore               | where x.startDate < ?1                |
| IsNull            | findByNameIsNull                    | where x.name is null                  |
| IsNotNull&NotNull | findByName(Is)NotNull               | where x.name not null                 |
| Like              | findByNameLike                      | where x.name like ?1                  |
| NotLike           | findByNameNotLike                   | where x.name not like ?1              |
| StartingWith      | findByNameStartingWith              | where x.name like ?1(参数前面加%)     |
| EndingWith        | findByNameEndingWith                | where x.name like ?1(参数后面加%)     |
| Containing        | findByNameContaining                | where x.name like ?1(参数前后都加%)   |
| OrderBy           | findByNameOrderByAgeDesc            | where x.name = ?1 order by x.age desc |
| Not               | findByNameNot                       | where x.name <> ?1                    |
| In                | findByAgeIn(Collection<Age> age)    | where x.age in ?1                     |
| NotIn             | findByAgeNotIn(Collection<Age> age) | where x.age not in ?1                 |
| True              | findByActiveTrue()                  | where x.active = true                 |
| False             | findByActiveFalse()                 | where x.active = false                |
| IgnoreCase        | findByNameIgnoreCase                | where UPPER(x.name) = UPPER(?1)                                      |

#### @Query&@NamedQuery查询


----------

&nbsp;&nbsp;Spring Data JPA支持使用@Query注解在接口的方法上实现查询.

![](http://ww1.sinaimg.cn/mw690/63503acbjw1f6jatc82fnj20rf06375q.jpg)

&nbsp;&nbsp;Spring Data JPA支持使用@NamedQuery定义查询方法,一个名称映射一条查询语句.

![](http://ww2.sinaimg.cn/mw690/63503acbjw1f6jatcdurzj20kq02uaar.jpg)

![](http://ww2.sinaimg.cn/mw690/63503acbjw1f6jatcnebfj20jn05k0tl.jpg)

#### 更新查询


----------


&nbsp;&nbsp;Spring Data JPA支持使用@Modifying注解和@Query注解组合进行数据更新操作.

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f6jthq9ojpj20do05fgml.jpg)

#### Specification


----------


&nbsp;&nbsp;Spring Data JPA提供了一个Specification接口可以让我们快速地构造基于准则的查询.通过重写Specification接口的toPredicate方法用来构造查询条件.

 1. 数据访问接口需继承JpaSpecificationExecutor接口.
 2. 构造查询条件类
 
 ![](http://ww1.sinaimg.cn/mw690/63503acbjw1f6judons88j20pz0cugoi.jpg)

 3. 调用条件查询

 ![](http://ww2.sinaimg.cn/mw690/63503acbjw1f6judoyj61j20go031q3d.jpg)
 
 
#### 排序&分页


----------


&nbsp;&nbsp;Spring Data JPA提供了Sort类和Page接口及Pageable接口完成排序和分页.

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f6julp4p8mj20ph0c40v9.jpg)

#### 自定义Repository


----------


&nbsp;&nbsp;如果我们想自定义Repository,可以继承Repository的子接口PagingAndSortingRepository.

 - 定义自定义的Repository接口.

 ![](http://ww2.sinaimg.cn/mw690/63503acbjw1f6jv8nwlvej20j006n0tw.jpg)
 
 - 定义自定义的Repository接口实现.

 ![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6jv8o9wlrj20qp0dyn0m.jpg)
 
 - 定义RepositoryFactoryBean

 ![](http://ww2.sinaimg.cn/mw690/63503acbjw1f6jv8oxqiwj20t00nxgsx.jpg)
 
 - 开启自定义支持需要使用@EnableJpaRepositories注解的repositoryFactoryBeanClass指定FactoryBean.

#### Spring Boot支持


----------
   
    
 - Spring Boot使用`spring.datasource`前缀用来配置dataSource.
 
 - Spring Boot自动开启了注解事务支持(@EnableTransactionManagement),并配置了jdbcTemplate.

 - Spring Boot提供了初始化数据的功能,在类路径下的schema.sql文件会自动初始化表结构;在类路径下的data.sql文件会自动插入表数据.

 - Spring Boot为我们自动配置了transactionManager、jpaVendorAdapter、entityManagerFactory等Bean.JpaBaseConfiguration还有一个getPackagesToScan方法用于自动扫描带有注解@Entity的实体类

 - Spring Boot自动配置了OpenEntityManagerInViewInterceptor拦截器,并注册到了Spring MVC的拦截器中.解决了页面访问数据时会话连接已关闭的错误.

 - Spring Boot自动开启了对Spring Data JPA的支持,无需再配置类中显式声明@EnableJpaRepositories.

&nbsp;&nbsp;在Spring Boot下使用Spring Data JPA,只需要添加依赖spring-boot-stater-data-jpa,然后定义DataSource、实体类、数据访问层即可,无需其他配置.

### Spring Data REST


----------


&nbsp;&nbsp;Spring Data REST支持将Spring Data JPA,Spring Data MongoDB,Spring Data Neo4j,Spring Data GernFile,Spring Data Cassandra的Repository自动转换成REST服务.

&nbsp;&nbsp;Spring Data REST的配置是定义在**RepositoryRestMvcConfiguration**配置类中的,我们可以通过继承这个配置类或者使用@Import注解导入此配置类来使用Spring Data REST.

#### Spring Boot支持


----------


&nbsp;&nbsp;Spring Boot已经自动配置了**RepositoryRestMvcConfiguration**,所以只需要引入依赖spring-boot-starter-data-rest,不需要任何其他配置.

 - Spring Boot使用`spring.data.rest`前缀用来配置**RepositoryRestMvcConfiguration**的属性.

 - 如果想在自定义的领域类Repository中将方法暴露为REST资源,则需要使用@RestResource注解.
   ![http://localhost:8080/persons/search/nameStartsWith?name=xx访问此REST资源.](http://ww1.sinaimg.cn/mw690/63503acbjw1f6l1pub41ej20hw01ydga.jpg)
    
 - 如需要分页,则可以使用参数page=?&size=?来实现分页.
   例:http://localhost:8080/persons/?page=1&size=10.

 - 如需要排序,则可以使用参数sort来实现排序.
   例:http://localhost:8080/persons/?sort=age,desc.

 - 自定义根路径需要在application.properties中设置`spring.data.rest.base-path`属性.

 - Spring Data REST的节点路径是默认在实体类之后加s,如果需要自定义节点路径则要在领域类Repository上使用**@RepositoryRestResource**注解的path属性进行设置.

### Spring Data MongoDB


----------


&nbsp;&nbsp;MongoDB是一个基于Document文档的NoSQL数据库,它使用面向对象的思想,每一条记录都是一个文档对象.

&nbsp;&nbsp;Spring Data MongoDB提供了以下的注解用来定义领域类:

| 注解      | 描述                            |
| --------- | ------------------------------- |
| @Document | 映射领域对象与MongoDB的一个文档 |
| @Id       | 映射当前属性为ID                |
| @DbRef    | 当前属性将参考其他文档          |
| @Field    | 为文档的属性定义名称            |
| @Version  | 将当前属性作为版本              |

&nbsp;&nbsp;Spring Data MongoDB还提供了一个MongoTemplate封装了数据访问的方法,我们还需要为MongoClient和MongoDbFactory来配置数据库连接属性.开启MongoDB的Repository需要在配置类上使用注解@EnableMongoRepositories.

![](http://ww4.sinaimg.cn/mw690/63503acbjw1f6l41qx7pbj20qa0am775.jpg)

&nbsp;&nbsp;定义Spring Data MongoDB的Repository只需要继承MongoRepository接口即可.

#### Spring Boot支持


----------


&nbsp;&nbsp;使用Spirng Boot主要配置数据库连接、MongoTemplate.可以使用`spring.data.mongodb`配置MongoDB相关的属性.

 - Spring Boot自动开启了@EnableMongoRepositories注解.
 - Spring Boot提供了一些默认配置:默认MongoDB端口为27017,服务器为localhost,数据库为test.
 - 在Spring Boot下使用MongoDB只需要引入依赖spring-boot-starter-data-mongodb,不需要其他配置.

### Spring Data Redis


----------


&nbsp;&nbsp;Spring Data Redis提供了ConnectionFactory和RedisTemplate.

 - 根据Redis不同的JavaClient,Spring Data Redis提供了不同的ConnectionFactory.

   - JedisConnectionFactory:使用Jedis作为客户端.
   - JredisConnectionFactory:使用Jredis作为客户端.
   - LettuceConnectionFactory:使用Lettuce作为客户端.
   - SrpConnectionFactory:使用Spullara/redis-protoccol作为客户端.

 - 配置ConnectionFactory和RedisTemplate如下:
   ![](http://ww1.sinaimg.cn/mw690/63503acbjw1f6l51jk3k8j20m707iwgh.jpg)

&nbsp;&nbsp;Spring Data Redis提供了RedisTemplate和StringRedisTemplate两个模板对象进行数据操作.StringRedisTemplate只针对键值都是字符类型的数据进行操作.

| 数据操作方法  | 描述                    |
| ------------- | ----------------------- |
| opsForValue() | 操作简单属性的数据      |
| opsForList()  | 操作list数据            |
| opsForSet()   | 操作set数据             |
| opsForZSet()  | 操作ZSet(有序的set)数据 |
| opsForHash()  | 操作hash散列的数据      |

#### Serializer


----------


&nbsp;&nbsp;当我们进行存储操作的时候,键值对都是通过Spring提供的Serializer序列化到数据库的.

 - RedisTemplate默认使用的是JdkSerializationRedisSerizlizer.
 - StringRedisTemplate默认使用的是StringRedisSerializer.

#### Spring Boot支持


----------


&nbsp;&nbsp;Spring Boot默认配置了JedisConnectionFactory、RedisTemplate和StringRedisTemplate.

&nbsp;&nbsp;Spring Boot使用`spring.redis`为前缀在application.properties中配置Redis相关的属性.

![](http://ww2.sinaimg.cn/mw690/63503acbjw1f6l5oypp5fj20j90fxwhx.jpg)

### end

> 资料参考于 JavaEE开发的颠覆者: Spring Boot实战
