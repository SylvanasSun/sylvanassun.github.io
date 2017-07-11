---
layout:     post
title:      "MyBatis-XML配置与映射文件"
subtitle:   "MyBatis入门-02"
date:       2016-06-11
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


## 全局XML配置文件

`MyBatis`配置文件的结构如下：

 - configuration 配置
     - properties 属性
     - settings 参数设置
     - typeAliases 类型别名
     - typeHandlers 类型处理器
     - objectFactory 对象工厂
     - plugins 插件
     - environments 环境集合
        - environment 环境
            - transactionManager 事务管理器
            - dataSource 数据源
        - databaseIdProvider 数据库厂商标识
        - mappers 映射器

### 常用标签配置

#### 1. **properties**

    properties标签可以读取properties配置文件,并引入到 MyBatis 的配置文件中。
    可以通过子标签定义key/value。
    
例如：

``` stylus
<properties resource="com/sun/test/dataSource.properties">
    <property name="username" value="dev_user"/>
    <property name="password" value="F2Fa3!33TYyg"/>
</properties>
```

 其中的属性就可以在整个配置文件中使用key来替换需要动态配置的属性值。
    
例如：

``` stylus
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
``` 

properties的加载顺序

 - 最先加载properties中的子标签所定义的属性。
 - 然后会加载resource标签所引入的properties配置文件中的属性,并覆盖已加载的同名属性。
 - 最后加载作为方法参数(parameterType中的key/value)传递的属性，并覆盖已加载的同名属性。

总结：
 - 通过方法参数传递的属性具有最高优先级。
 - resource引用的配置文件中的属性次之。
 - 最低优先级的是 properties 子标签中指定的属性。

#### 2. **settings**

    settings会影响MyBatis的运行行为。
    
一个配置完整的 settings 的示例如下：

``` stylus
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

#### 3. **typeAliases**

类型别名,可以对JavaBean设置一个别名,用来减少完全限定名的冗余。
例如：

```stylus
<!-- 类型别名 -->
	<typeAliases>
		<!-- 设置单个别名 -->
		<typeAlias type="com.sun.test.entity.User" alias="user"/>
		<!-- package指定一个包名,会在包下搜索需要的JavaBean
			   如果没有注解,则默认设置为JavaBean的首字母小写的非限定类名作为它的别名。
			   如果有@Alias注解,则使用其注解值作为它的别名。
		 -->
		 <package name="com.sun.test.entity"/>
	</typeAliases>
```

`MyBatis`默认支持的类型别名如下,它们都是大小写不敏感的，需要注意的是由基本类型名称重复导致的特殊处理。

| 别名       | 映射的类型 |
| ---------- | ---------- |
| _byte      | byte       |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |

#### 4. **Mappers**
映射器(Mappers)用于引入Mapper映射文件。

使用相对路径的方式引入

``` stylus
<!-- 相对路径 -->
<mappers>
  <mapper resource="com/sun/test/mapper/UserMapper.xml"/>
</mappers>
```

使用绝对路径的方式引入

``` stylus
<!-- 绝对路径s -->
<mappers>
  <mapper url="file:///D:\sun\mappers\UserMapper.xml"/>
</mappers>
```

使用Mapper接口的全限定类名(需要mapper接口与mapper映射文件名称相同,并且在同一个包下)

``` stylus
<!-- Mapper接口的全限定类名 -->
<mappers>
  <mapper class="com.sun.test.mapper.UserMapper"/>
</mappers>
```

扫描指定包下的所有映射文件(需要mapper接口与mapper映射文件名称相同,并且在同一个包下)

``` stylus
<!-- 扫描指定包下的所有映射文件 -->
<mappers>
  <package name="com.sun.test.mapper"/>
</mappers>
```


----------


## Mapper映射文件
 - `cache` 缓存配置
 - `cache-ref` 其他namespace缓存配置的引用
 - `resultType` 结果集映射的类型
 - `resultMap` 结果集映射Map
 - `parameterType` 输入映射类型
 - `sql` 可被引用的复用sql语句块
 - `insert` 映射插入语句
 - `update` 映射更新语句
 - `select` 映射查询语句
 - `delete` 映射删除语句

### resultType
注意事项:

 1. 如果使用resultType进行结果集映射,则需要查询出的列名与映射的JavaBean属性名称一致。
 2. SQL语句中列名如果有别名,则列名为别名的名称。
 3. 如果查询的列名和JavaBean所映射的属性名全不一致,则映射的JavaBean对象为null。
 4. 如果查询的列名和JavaBean所映射的属性名少数不一致,则映射的JavaBean对象不为null,但只有名称一致的属性有值。

例如：

``` stylus
    <!-- 根据id查询User -->
	<select id="findById" parameterType="int" resultType="com.sun.test.entity.User">
		select * from user where id = #{id}
	</select>
```

### resultMap
注意事项:

 1. 使用resultMap进行结果集映射,不需要列名与映射的JavaBean属性名称一致。
 2. 使用resultMap进行结果集映射,需要先定义一个resultMap。

例如:

``` stylus
	<!-- 定义一个resultMap -->
	<resultMap type="com.sun.test.entity.User" id="userResultMap">
		<!-- 主键 -->
		<id column="id" property="id"/>
		<!-- 其他字段 -->
		<result column="username" property="username"/>
	</resultMap>
	<select id="findByIdWithRetMap" parameterType="int" resultMap="userResultMap">
		select id,username from user where id = #{id}
	</select>
```

### 动态SQL

#### 1. SQL片段

sql片段可以存储动态的sql语句,提高复用性。

``` stylus
<!-- SQL片段,可以存储动态的SQL语句 -->
	<sql id="sql">
		<!-- 判断用户名不为空 -->
		<if test="username != null and username != ''">
			and username = #{usernmae}
		</if>
		<!-- 判断性别不为空 -->
		<if test="sex != null and sex != ''">
			and sex = #{sex}
		</if>
	</sql>
```

#### 2. if标签

``` stylus
<select id="findByIdOrLikeName" resultType="com.sun.test.entity.User">
  select * from user where id = #{id}
  <!-- 如果if返回为true 则会加上这条语句 -->
  <if test="username != null and username != ''">
    and username like '%${username}%'
  </if>
</select>
```

#### 3. where标签

``` stylus
<select id="findAllOrLikeName" resultType="com.sun.test.entity.User">
  select * from user
  <!-- where标签只有在if为true的情况下才会添加where语句
       并且会将第一条语句的and去掉。
  -->
  <where>
    <if test="username != null and username != ''">
        and username like '%${username}%'
     </if>
  </where>
</select>
```

#### 4. foreach标签
foreach标签可以对一个集合进行遍历,通常是在构建一个in条件语句的时候使用。

``` stylus
	<select id="selectUserIn" resultType="com.sun.test.entity.User"  parameterType="list">
		select * from user
		<where>
			<if test="list != null and list.size > 0">
				<!-- 
					collection：集合参数的名称。
					item：遍历集合时的值
					open：遍历开始时需要拼接的sql语句
					close：遍历结束时需要拼接的sql语句
					separator：遍历过程中需要拼接的字符串
				 -->
				<foreach collection="list" item="id" open="and id in(" close=")" separator=",">
					#{id}
				</foreach>
			</if>
		</where>
	</select>
```

#### 5. bind标签
bind标签可以从OGNL表达式中创建一个变量并将其绑定到上下文。

``` stylus
<select id="selectUserLikeName" resultType="com.sun.test.entity.User">
  <bind name="username" value="'%' + user.getUsername() + '%'" />
  select * from user
  where username like #{username}
</select>
```
