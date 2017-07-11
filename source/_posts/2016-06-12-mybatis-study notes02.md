---
layout:     post
title:      "MyBatis学习笔记-02"
subtitle:   "MyBatis入门-03"
date:       2016-06-12
author:     "Sylvanas Sun"
header-img: "img/post-bg-rwd.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 后端开发
    - Java
    - MyBatis
---



![MyBatis](http://ww3.sinaimg.cn/mw690/63503acbjw1f4pf96s0tcj209302caa2.jpg)


### 关联映射

#### 1. 业务关系分析

在进行多表关联查询之前,需要对表结构的关联关系与业务关系进行分析与理解。
下图以用户,订单,明细,商品表为例分析:
![数据模型分析](http://ww2.sinaimg.cn/mw690/63503acbjw1f4tl89vexqj20fd0ba74p.jpg)

- 用户表与订单表
    - 一个用户可以创建多个订单 (一对多)
    - 一个订单只能由一个用户创建(一对一)
- 订单表与明细表
    - 一个订单可以含有多个订单明细(一对多)
    - 一个订单明细只能对应一个订单(一对一)
- 明细表与商品表
    - 一个订单明细只能对应一个商品(一对一)
    - 一个商品可以被包含在多个订单明细中(一对多)
- 订单表与商品表 (间接关系)
    - 一个订单可以拥有多个订单明细,而一个订单明细对应一个商品,即一个订单可以拥有多个商品。
        - order -> orderdetail -> items 一对多
    - 一个商品可以被包含在多个订单明细中,而一个订单明细对应一个订单,即一个商品可以被包含在多个订单中。
        - item -> orderdetail -> order 一对多
    - 订单与商品是多对多关系。
- 用户表与商品表 (间接关系)
    - 一个用户可以创建多个订单,一个订单可以对应多个订单明细,一个订单明细对应一个商品,即一个用户可以购买多个商品。
        - user -> order -> orderdetail -> item 一对多
    - 一个商品可以被包含在多个订单明细中,一个订单明细对应一个订单,一个订单对应一个用户,即一个商品可以对应多个用户。
        - item -> orderdetail -> order -> user 一对多
    - 用户与商品是多对多关系。

#### 2. 一对一关联查询

> 在多表查询时,单表的JavaBean不能满足结果集的映射,所以可以使用继承的方法来扩展JavaBean。

``` java
package com.sun.test.entity;

// Orders的扩展类
public class OrdersExtends extends Orders {

	private User user;

	public User getUser() {
		return user;
	}

	public void setUser(User user) {
		this.user = user;
	}

}
```

上面的做法与在Orders类中添加一个User类型的属性作用相同。

> 映射文件配置

``` stylus
<mapper namespace="com.sun.test.mapper.OrdersMapper">

	<!-- selectOrdersAndUserRetMap -->
	<!-- resultMap可以使用extends字段继承一个type一致的resultMap  -->
	<resultMap type="ordersExtends" id="selectOrdersAndUserRetMap">
		<!-- 主键 -->
		<id column="id" property="id" />
		<!-- 其他字段 -->
		<result column="user_id" property="userId"/>
		<result column="number" property="number"/>
		<result column="createtime" property="createtime"/>
		<result column="note" property="note"/>  
		<!-- association:一对一映射
			 	property:关联对象在JavaBean中封装的属性名
			 	javaType:关联对象的Java类型
		 -->
		<association property="user" javaType="com.sun.test.entity.User">
			<id column="user_id" property="id"/>
			<result column="username" property="username"/>
			<result column="address" property="address"/>			
		</association> 
	</resultMap>
	
	<!-- 查询订单与用户信息 -->
	<select id="selectOrdersAndUser" resultMap="selectOrdersAndUserRetMap">
		SELECT
			orders.id,
			orders.user_id,
			orders.number,
			orders.createtime,
			orders.note,
			`user`.username,
			`user`.address
		FROM
			orders,
			`user`
		WHERE orders.user_id = `user`.id
	</select>
	
</mapper>
```

#### 3. 一对多关联查询

> 一对多查询与一对一查询类似,我们要查询商品以及订单明细则需要在商品类中扩展一个List集合。

``` java
public class Items {
    private Integer id;

    private String name;

    private Float price;

    private String pic;

    private Date createtime;

    private String detail;
    
    // 一对多  一个商品对应多个订单明细
    private List<Orderdetail> orderdetails;
```

> 映射文件配置

``` stylus
<mapper namespace="com.sun.test.mapper.ItemsMapper">

	<!-- (一对多)查询商品以及对应的订单明细 -->
	<select id="selectItemsAndOrderdetail" resultMap="selectItemsAndOrderdetailRetMap">
		SELECT
			items.id,
			items.`name`,
			items.price,
			orderdetail.id orderdetail_id,
			orderdetail.items_num
		FROM
			items,
			orderdetail
		WHERE
			items.id = orderdetail.items_id;
	</select>

	<!-- selectItemsAndOrderdetailRetMap -->
	<resultMap type="items" id="selectItemsAndOrderdetailRetMap">
		<id column="id" property="id"/>
		<result column="name" property="name"/>
		<result column="price" property="price"/>
		<!-- 一对多关联
			 collection:一对多映射
			 	ofType:这个集合参数的类型
		 -->
		 <collection property="orderdetails" ofType="orderdetail">
		 	<id column="orderdetail_id" property="id"/>
		 	<result column="items_num" property="itemsNum"/>
		 </collection>
	</resultMap>

</mapper>
```

#### 4. 多对多关联查询

> 查询商品以及对应的用户信息。
>   1. 在商品类中添加一个订单明细集合(一个商品对应多个订单明细)
>   2. 在订单明细类中添加一个订单属性(一个订单明细对应一个订单)
>   3. 在订单类中添加一个用户属性(一个订单对应一个用户)

``` java
    // 一对多  一个商品对应多个订单明细
    private List<Orderdetail> orderdetails;
    
    // 一对一 一个订单明细对应一个订单
    private Orders orders;
    
    //一对一 一个订单对应一个用户
    private User user;
```

> 映射文件配置

``` stylus
	<!-- 查询商品以及对应的用户 -->
	<select id="selectItemsAndUser" resultMap="selectItemsAndUserRetMap">
		SELECT
			items.id,
			items.`name`,
			items.price,
			`user`.id user_id,
			`user`.username,
			`user`.address,
			orderdetail.id orderdetail_id,
			orders.id orders_id
		FROM
			items,
			orderdetail,
			orders,
			`user`
		WHERE
			items.id = orderdetail.items_id
		AND orderdetail.orders_id = orders.id
		AND orders.user_id = `user`.id
	</select>
	
	<!-- selectItemsAndUserRetMap -->
	<resultMap type="items" id="selectItemsAndUserRetMap">
		<id column="id" property="id" />
		<result column="name" property="name"/>
		<result column="price" property="price"/>
		<!-- 一个商品对应多个订单明细  -->
		<collection property="orderdetails" ofType="orderdetail">
			<id column="orderdetail_id" property="id"/>
			<!-- 一个订单明细对应一个订单 -->
			<association property="orders" javaType="orders">
				<id column="orders_id" property="id"/>
				<!-- 一个订单对应一个用户 -->
				<association property="user" javaType="user">
					<id column="user_id" property="id"/>
					<result column="username" property="username"/>
					<result column="address" property="address"/>
				</association>
			</association>
		</collection>
	</resultMap>
```

### 延迟加载

resultMap中的association和collection具有延迟加载的功能。


> 延迟加载: 先加载主信息,在需要的时候加载关联信息,延迟加载即叫按需加载,也叫懒加载。

#### 1. 设置延迟加载
> `MyBatis` 默认是关闭延迟加载的,需要在配置文件中的<settings>标签中手动开启。

| settings              | description                      | 验证值组   | 默认值 |
| --------------------- | -------------------------------- | ---------- | ------ |
| lazyLoadingEnabled    | 全局性设置懒加载。               | true|false | true   |
| aggressiveLazyLoading | 积极性的懒加载,false则是按需加载 | true|false | true   |

``` stylus
	<settings>
		<!-- 开启延迟加载 ,默认值true-->
		<setting name="lazyLoadingEnabled" value="true"/>
		<!-- 设置积极的懒加载,默认值true -->
		<setting name="aggressiveLazyLoading" value="false"/>
	</settings>
```

#### 2. 在association中使用延迟加载

``` stylus
<!-- 查询订单 并延迟加载用户信息 -->
	<select id="selectOrdersAndLazyLoadingUser" resultMap="selectOrdersAndLazyLoadingUserRetMap">
		SELECT * FROM orders
	</select>
	
	<!-- selectOrdersAndLazyLoadingUserRetMap -->
	<resultMap type="orders" id="selectOrdersAndLazyLoadingUserRetMap">
		<id column="id" property="id"/>
		<result column="user_id" property="userId"/>
		<result column="number" property="number"/>
		<result column="createtime" property="createtime"/>
		<result column="note" property="note"/>
		<!-- 一对一关联并延迟加载用户信息
			select:指定延迟加载执行的statementId
			(如果这个statement不在当前namespace中那么则需要使用namespace.statementid)
			column:需要关联查询的列，如果需要传入多个则需要使用 {user_id=id,...}的格式
		 -->
		<association property="user" column="user_id" 
			select="com.sun.test.mapper.UserMapper.findById"></association>
	</resultMap>
```

### 查询缓存

#### 1. 一级缓存

> 一级缓存是SqlSession级别的缓存。在操作数据库时需要构造 sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。

![](http://ww3.sinaimg.cn/mw690/63503acbjw1f4u0a7gs9tj20d709ot8z.jpg)
> 当第一次查询对象1时,将先会去一级缓存中查找是否有对象1的数据信息,如果没有则从数据库中查询到数据信息并构造一个key存储到一级缓存(HashMap)中。
当第二次查询对象1的时候，则可以凭借key在一级缓存中找到数据信息,而不用再去访问数据库。
如果SqlSession进行了事务提交(commit),则会刷新(清空)缓存,目的是为了让缓存存储最新的数据,防止脏读。
`MyBatis`默认开启一级缓存。

#### 2. 二级缓存

> 二级缓存是**namespace**(mapper)级别的缓存，多个SqlSession去操作同一个**namespace**中的mapper的sql语句，多个SqlSession可以共用二级缓存，二级缓存是跨SqlSession的。
二级缓存存储数据不一定是在内存中,所以需要给缓存的对象实现序列化接口。
二级缓存需要手动设置开启。

##### 开启二级缓存

``` stylus
    在settings中设置
	<!-- 开启二级缓存 -->
	<setting name="cacheEnabled" value="true"/>
	
	在mapper映射文件中设置
	<!-- 使用第三方(ehcache)二级缓存
	     flashInterval:设置定时刷新的间隔时间,单位是毫秒。
	     注:使用第三方缓存框架需要先导入jar包和整合包
	-->
    <cache type="org.mybatis.caches.ehcache.EhcacheCache" flashInterval="60000" />
```

### MyBatis与Spring整合
> 由`Spring`维护管理数据源、事务、SqlSessionFactory、mapper。
`MyBatis`的配置文件只需要配置settings、别名等即可。

``` stylus
Spring中的配置
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
	<!-- mybatis的配置文件路径 -->
	<property name="configLocation" value="sqlMapConfig.xml"></property>
	<!-- 注入数据源 -->
	<property name="dataSource" ref="dataSource"></property>
</bean>

批量生成Mapper(生成的Mapper名字默认为首字母)
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
	<!-- 指定需要扫描的mapper配置的包名 -->
	<property name="basePackage" value="com.sun.test.mapper"></property>
	<!-- 注入SqlSessionFactory -->
	<property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"></property>
</bean>

```