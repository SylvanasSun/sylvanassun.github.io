---
layout:     post
title:      "MyBatis学习笔记-01"
subtitle:   "MyBatis入门"
date:       2016-06-10
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - Java
    - MyBatis
tags:
    - Java
    - MyBatis
---



![MyBatis](http://ww3.sinaimg.cn/mw690/63503acbjw1f4pf96s0tcj209302caa2.jpg)

### 1.什么是MyBatis?

 - MyBatis 本是apache的一个开源项目iBatis, 2010年这个项目由apache software foundation 迁移到了google code，并且改名为MyBatis 。2013年11月迁移到Github。
 - iBATIS一词来源于“internet”和“abatis”的组合，是一个基于Java的持久层框架。iBATIS提供的持久层框架包括SQL Maps和Data Access Objects（DAO）
 - MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。
 - MyBatis 可以对配置和原生Map使用简单的 XML 或注解，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

### 2.MyBatis框架原理

#### MyBatis的功能架构分为三层

1. API接口层：提供给外部使用的接口API，开发人员通过这些本地API来操纵数据库。接口层一接收到调用请求就会调用数据处理层来完成具体的数据处理。  
2. 数据处理层：负责具体的SQL查找、SQL解析、SQL执行和执行结果映射处理等。它主要的目的是根据调用的请求完成一次数据库操作。  
3. 基础支撑层：负责最基础的功能支撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东西，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。
![](http://i4.buimg.com/47c91060a0871659.png)

#### MyBatis框架架构流程
![](http://i4.buimg.com/60494497248097cf.png)

1. MyBatis应用程序通过读取XML配置文件,构造出SqlSessionFactory(SQL会话工厂)。
2. SqlSessionFactory再根据配置,构造一个SqlSession(SQL会话)。MyBatis通过SqlSession完成数据库操作。
3. SqlSession本身不能直接操作数据库,它是通过底层Executor接口操作数据库的。Executor有2个实现类,默认使用缓存执行器。
4. Executor将处理的SQL信息封装到一个底层对象 MappedStatement 中。该对象包含了SQL语句、输入参数信息、输出结果集信息。
5. 输入参数和输出结果集的映射类型为java的简单类型、HashMap集合类型、POJO对象类型。

### 3.MyBatis的优点与缺点

优点

- 学习成本低、简单易学。
- 灵活性较高,可以直接对SQL进行性能优化。
- SQL与业务逻辑代码低耦合,提高了维护性。
- 可以编写动态SQL语句。


缺点

- SQL语句依赖数据库,即数据库发生变更时,需要重新写SQL语句, 移植性较差。
- SQL语句繁多,工作量大。
- 二级缓存机制不佳。
	
### 4.MyBatis入门

我们使用MyBatis完成一个根据id查询用户的需求。

**创建一个JavaBean**

```
public class User {

     private int id ;
     private String username ;// 姓名
     private String sex ;// 性别
     private Date birthday ;// 生日
     private String address ;// 地址
```

**为User类配置一个映射文件**

```java
<? xml version ="1.0" encoding= "UTF-8" ?>
<! DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
< mapper namespace ="com.sun.test.mapper.UserMapper" >
     <!-- 根据id查询User
          在命名空间 com.sun.test.mapper.UserMapper中定义了一个id为findById的映射语句。
          当我们调用时可以使用 namespace.id 来进行调用。
          即：com.sun.test.mapper.UserMapper.findById 
     -->
     <select id= "findById" parameterType ="int" resultType ="com.sun.test.entity.User" >
         select * from user where id = #{id}
     </select >
</ mapper>
```

- 命名空间(Namespace): 必须且非常重要,最好使用对应mapper接口的全限定名(比如:com.sun.test.mapper.UserMapper )。
- parameterType: 输入参数的类型,可以使用类型的全限定名或者别名。
- resultType: 返回参数的类型,如果是集合类型,则应使用集合内可以包含的类型,而不能是集合本身,可以使用类型的全限定名或者别名。
	
**创建一个全局配置文件,并配置数据源、事务管理器、映射文件等基本配置。**	

```
<? xml version ="1.0" encoding= "UTF-8" ?>
<! DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-config.dtd" >
< configuration>
     <environments default= "development">
          < environment id= "development" >
              <!-- 配置事务管理器 -->
              < transactionManager type ="JDBC" />
              <!-- 配置数据源 -->
              < dataSource type ="POOLED" >
                  < property name ="driver" value ="com.mysql.jdbc.Driver" />
                  < property name ="url" value ="jdbc:mysql:///mybatis" />
                  < property name ="username" value ="root" />
                  < property name ="password" value ="root" />
              </ dataSource>
          </ environment>
     </environments >
     <!-- 配置映射文件 -->
     <mappers >
          <!-- 使用package可以查找到所有 mapper接口与 mapper映射文件但需要同名并且放在同一包下 -->
          < package name ="com.sun.test.mapper" />
     </mappers >
</ configuration>
```

**测试**


```java
 @Test
     public void test01() throws IOException {
          // 获得配置文件的资源流
         InputStream inputStream = Resources.getResourceAsStream( "sqlMapConfig.xml");
          // 加载配置文件,构造SqlSessionFactory
         SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream );
          // 使用SqlSessionFactory构造一个SqlSession
         SqlSession sqlSession = sqlSessionFactory .openSession();
          /**
          * 执行statement
          * 参数1：statement的id 建议使用 namespace.statementId
          * 参数2：输入参数的值,它的类型要和映射文件中的parameterType类型一致
          */
         User user = sqlSession .selectOne("com.sun.test.mapper.UserMapper.findById" , 1);
          // 输出结果
         System. out .println(user );
          // 关闭SqlSession
         sqlSession .close();
    }
```


- Resources：MyBatis中的一个工具类,可以从classpath或其他位置加载资源文件。
- SqlSessionFactoryBuilder： 这个类可以被实例化、使用和丢弃,一旦创建了 SqlSessionFactory,就不再需要它了。因此 SqlSessionFactoryBuilder 实例的最佳范围是方法范围（也就是局部方法变量）。
- SqlSessionFactory：一旦被创建就应该在应用的运行期间一直存在,所以它应该是单例的,且在应用运行期间不要多次创建。
- SqlSession：用于操作数据库的对象,每个线程都应该有一个SqlSession实例,它是多例的,并且它是线程不安全的,不能放在全局变量上。

### 5.自增主键

如果你的数据库支持自动生成主键的字段,那么你可以设置 useGeneratedKeys="true",再把keyProperty设置到目标属性上即可。

```
<insert id="insertUser" useGeneratedKeys="true" parameterType ="com.sun.test.entity.User" keyProperty="id">
     insert into user(username,birthday,sex,address)
     values(#{username},#{birthday},#{sex},#{address})
</insert>
```

如果不支持自动生成主键则可以使用另外一种方法生成主键

```
< insert id= "insertUser" parameterType ="com.sun.test.entity.User" >
          <!-- 返回自增主键
             keyProperty:生成主键的属性
             resultType:生成主键的类型
             order:查询主键SQL的执行顺序,相对于insert,如果设置为 BEFORE，那么它会首先选择主键，设置 keyProperty 然后执行插入语句。
             如果设置为 AFTER，那么会先执行插入语句。
             LAST_INSERT_ID():MySQL的函数,获取最后插入的主键
         -->
          < selectKey keyProperty ="id" resultType ="int" order= "AFTER">
             select LAST_INSERT_ID()
          </ selectKey>
         insert into user(username,birthday,sex,address)
        values(#{username},#{birthday},#{sex},#{address})
     </insert >
```

#### #{} 与 ${}的区别

1. #{}可以表示为占位符?,#{id}里面的id表示输入参数的参数名称,如果是简单类型,则名称可以任意填写。
2. ${}表示字符串替换,${value},value表示输入参数的参数名称,如果是简单类型,则名称必须为value。
3. ${}是不安全的,会导致潜在的sql注入攻击,但如果要动态传入order by的列名则必须使用${} 即：order by ${columnName}。

### 6.使用Mapper构建Dao

使用Mapper构建Dao则只需要开发接口即可。

#### 规范

1. mapper接口的全限定名称要和mapper映射文件的namespace一致。
2. mapper接口的方法名称要和mapper映射文件的MappedStatement的id一致。
3. mapper接口的方法参数类型与返回值类型要和mapper映射文件的MappedStatement的parameterType与resultType一致。

例:

![mapper](http://i4.buimg.com/fb28faf508066198.png)

![mapper.xml](http://i4.buimg.com/0382b4a78a6080a2.png)

![test](http://i4.buimg.com/134368652f63916c.png)



