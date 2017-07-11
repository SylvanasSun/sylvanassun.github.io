---
layout:     post
title:      "Spring Boot 部署测试与应用监控"
subtitle:   "Spring Monitor"
date:       2016-08-05 18:00
author:     "Sylvanas Sun"
header-img: "img/post-bg-universe.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 后端开发
    - Spring
    - Java
---


### Spring Boot部署


----------


#### Jar包形式


----------


&nbsp;&nbsp;如果在创建Spring Boot项目的时候选择的是jar包形式,则只需要使用maven将项目打成jar包即可.

```
mvn pakage
```

&nbsp;&nbsp;Spring Boot项目在打成jar后可以直接运行.

#### War包形式


----------


&nbsp;&nbsp;如果在创建Spring Boot项目的时候选择的是war包形式,则打包的方式与jar一样.

```
mvn pakage
```

&nbsp;&nbsp;之后将生成的war文件放在任意Servlet容器上运行即可.

&nbsp;&nbsp;如果在创建Spring Boot项目的时候选择的是jar,在部署时又想以war包形式部署,则需要以下配置.

 1. 修改pom.xml文件的打包方式
    
 ![](http://ww4.sinaimg.cn/mw690/63503acbjw1f6nkz6y4ntj20bm038mxv.jpg)

 2. 添加以下依赖覆盖默认内嵌的Tomcat.
 
 ![](http://ww3.sinaimg.cn/mw690/63503acbjw1f6nkz77e95j20fy03w758.jpg)
 
 3. 创建ServletInitializer.

 ![](http://ww4.sinaimg.cn/mw690/63503acbjw1f6nkz7gwh7j20o104z75l.jpg)
 
#### 注册为Linux服务


----------


&nbsp;&nbsp;将软件注册为服务,可以通过命令开启、关闭、开机自启动等功能.

&nbsp;&nbsp;如想注册为Linux服务,则先需要修改spring-boot-maven-plugin的配置:

```
    <build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<configuration>
					<executable>true</executable>
				</configuration>
			</plugin>
		</plugins>
	</build>
```

&nbsp;&nbsp;主流的Linux大都使用init.d或systemd来注册服务.

 - init.d方式
 
   1. sudo ln -s /var/apps/spring-boot-demo.jar /etc/init.d/sdemo,其中sdemo就是服务名.

   2. 常用操作命令
      ```
      service sdemo start 启动服务.
      service sdemo stop 停止服务.
      service sdemo status 服务状态.
      chkconfig sdemo on 开机启动.
      ```

   3. 项目日志存放于/var/log/sdemo.log中.

 - systemd方式

   1. 在/etc/systemd/system/目录下新建文件sdemo.service.并写入以下内容:

      ```
      [Unit]
      Description=spring-boot-demo
      After=syslog.target
      
      [Service]
      ExecStart= /usr/bin/java -jar /var/apps/spring-boot-demo.jar
      
      [Install]
      WantedBy=multi-user.target
      
      #其中Description和ExecStart是可更改的.ExecStart是指定java运行需要运行的jar包.
      ```
    
   2. 常用操作命令
      ```
      systemctl start sdemo 启动服务.
      systemctl stop sdemo 停止服务.
      systemctl status sdemo 服务状态.
      systemctl enable sdemo 开机启动.
      journalctl -u sdemo 查看项目日志.
      ```
      
### 热部署


----------


&nbsp;&nbsp;热部署即就是在应用正在运行的时候升级软件，却不需要重新启动应用.

&nbsp;&nbsp;在Spring Boot项目中添加**spring-boot-devtools**依赖即可实现页面和类的热部署.

#### 模板引擎热部署


----------


&nbsp;&nbsp;Spring Boot默认模板引擎开启缓存,如果我们修改了页面后再刷新页面将得不到修改后的内容,我们可以在application.properties中关闭模板引擎的缓存.

```
#Thymeleaf
spring.thymeleaf.cache=false
#FreeMarker
spring.freemarker.cache=false
#Groovy
spring.groovy.template.cache=false
#Velocity
spring.velocity.cache=false
```

### 基于Docker部署


----------


&nbsp;&nbsp;目前主流的PaaS平台都支持发布Docker镜像.而Docker镜像是使用Dockerfile文件来编译镜像的.

#### Dockerfile


----------


 1. FROM指令

    FROM指令指明了当前镜像继承的基镜像.编译当前镜像时会自动下载基镜像.
    ```
    FROM java:8
    ```
    
 2. MAINTAINER指令

    MAINTAINER指令指明了当前镜像的作者
    ```
    MAINTAINER sun
    ```
    
 3. RUN指令

    RUN指令可以在当前镜像上执行Linux命令并形成一个新的层.RUN指令是编译时(build)的动作.
    ```
    RUN /bin/bash -c "echo helloworld"
    ```

 4. CMD指令

    CMD指令指明镜像启动时的默认行为.一个Dockerfile里只能有一个CMD指令.CMD指令设定的命令可以在运行镜像时使用参数覆盖.它是运行时(run)的动作.
    ```
    CMD echo "hello"
    可被 docker run -d image_name echo "world" 覆盖.
    ```
    
 5. EXPOSE指令

    EXPOSE指令指明了镜像运行时的容器必须监听指定的端口.
    ```
    EXPOSE 8080
    ```
    
 6. EVN指令

    EVN指令用于设置环境变量.
    ```
    ENV myName=sun
    ```
    
 7. ADD指令

    ADD指令用于从当前工作目录复制文件到镜像目录.
    ```
    ADD hello.sh /dir/
    ```
    
 8. ENTRYPOINT指令

    ENTRYPOINT指令可以让容器像一个可执行程序一样运行,这样镜像运行时可以像软件一样接收参数执行,ENTRYPOINT是运行时(run)的动作.
    ```
    ENTRYPOINT ["/bin/echo"]
    我们可以镜像传递参数:
    docker run -d image_name "hello world"
    ```
    
#### Example


----------


 1. 首先将打好包的demo上传到Linux服务器.
 2. 在项目同级目录下新建一个Dockerfile文件,如下:
    ```
    FROM java:8
    
    MAINTAINER=sun
    
    ADD spring-boot-demo.jar app.jar
    
    EXPOSE 8080
    
    ENTRYPOINT ["java","-jar","/app.jar"]
    ```
 3. 编译镜像
    ```
    在/var/apps/sdemo目录下执行以下命令
    docker build -t sun/sdemo .
    使用sun/sdemo作为镜像名称,sun为前缀.  
    .是用来指明Dockerfile路径的,这里因为在当前目录下所以使用“.”.
    ```
    
 4. 运行镜像
    ```
    docker run -d -p 8080:8080 --name sdemo sun/sdemo
    ```

### Spring Boot集成测试


----------


&nbsp;&nbsp;Spring Boot的测试与Spring MVC类似,它提供了一个`@SpringApplicationConfiguration`替代`@ContextConfiguration`来指定配置类.

#### Example


----------


 1. Entity
 
    ```java
    @Entity
    public class Person {

        @Id
        @GeneratedValue
        private Long id;
        private String name;
        ...
    }
    ```
    
 2. Dao
 
    ```java
    public interface PersonRepositroy extends JpaRepository<Person,Long> {
    
    }
    ```
    
 3. Controller
 
    ```java
    @RestController
    @RequestMapping("/person")
    public class PersonController {

        @Autowired
        PersonRepositroy personRepositroy;

        @RequestMapping(method = RequestMethod.GET,produces = {MediaType.APPLICATION_JSON_VALUE})
        public List<Person> findAll() {
            return personRepositroy.findAll();
        }

    }
    ```
    
 4. Test
 
    ```java
    @RunWith(SpringJUnit4ClassRunner.class)
    //@ContextConfiguration(classes = {SpringBootTestApplication.class})
    @SpringApplicationConfiguration(classes = SpringBootTestApplication.class)
    @WebAppConfiguration
    @Transactional
    public class PersonApplicationTest {

        @Autowired
        PersonRepositroy personRepositroy;

        MockMvc mockMvc;

        @Autowired
        WebApplicationContext webApplicationContext;

        String expectedJson;

         /**
           * 初始化
           */
         @Before
         public void setUp() throws JsonProcessingException {
             Person p1 = new Person("sun");
             Person p2 = new Person("sylvanas");
             personRepositroy.save(p1);
             personRepositroy.save(p2);
             // 获得期待返回的JSON字符串.
             expectedJson = Obj2Json(personRepositroy.findAll());
             // 初始化MockMvc.
             mockMvc = MockMvcBuilders.webAppContextSetup(webApplicationContext).build();
         }

         protected String Obj2Json(Object obj) throws JsonProcessingException {
             ObjectMapper objectMapper = new ObjectMapper();
             return objectMapper.writeValueAsString(obj);
         }

         @Test
         public void testPersonController() throws Exception {
             String url = "/person";
             // 对/person发送请求,获得请求的执行结果.
             MvcResult result = mockMvc.perform(MockMvcRequestBuilders.get(url)
                     .accept(MediaType.APPLICATION_JSON_VALUE))
                     .andReturn();
             // 获得执行结果的状态
             int status = result.getResponse().getStatus();
             // 获得执行结果的内容
             String content = result.getResponse().getContentAsString();
             // 断言
             Assert.assertEquals("错误:正确的返回值为200", 200, status);
             Assert.assertEquals("错误:返回值和预期返回值不一致", expectedJson, content);
         }

    }
    ```

### Spring Boot应用监控


----------


&nbsp;&nbsp;Spring Boot提供了运行时的应用监控和管理的功能.我们可以通过HTTP、JMX、SSH来进行应用的监控.

#### HTTP


----------


&nbsp;&nbsp;使用HTTP实现对应用的监控需要添加以下依赖:

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

&nbsp;&nbsp;**端点:**

| Endpoint    | Description                                   |
| ----------- | --------------------------------------------- |
| actuator    | 所有EndPoint的列表,需要加入spring HATEOAS支持 |
| autoconfig  | 当前应用的所有自动配置                        |
| beans       | 当前应用的所有Bean信息                        |
| configprops | 当前应用中所有的配置信息                      |
| dump        | 显示当前应用线程状态信息                      |
| env         | 显示当前应用的环境信息                        |
| health      | 显示当前应用的健康情况                        |
| info        | 显示当前应用信息                              |
| metrics     | 显示当前应用的各项指标信息                    |
| mappings    | 显示所有的@RequestMapping映射的路径           |
| shutdown    | 关闭当前应用(默认此项是关闭的)                |
| trace       | 默认最新的http请求                            |

&nbsp;&nbsp;使用http访问应用监控只需要在url中加上/端点名即可.

  - 例如: http://localhost:8080/actuator

&nbsp;&nbsp;端点的开启关闭可以在application.properties中配置.

&nbsp;&nbsp;如果想自定义端点,则需要一个继承AbstractEndpoint的实现类,并注册成Bean即可.

&nbsp;&nbsp;如果想自定义HealthIndicator,则需要一个实现HealthIndicator接口的类,并注册成Bean即可.

### JMX


----------


&nbsp;&nbsp;可以使用Java内置的jconsole来实现JMX监控.

### SSH


----------


&nbsp;&nbsp;Spring Boot借助CraSH (http://www.crashub.org), 实现通过SSH或者TELNET监控和管理应用.我们只需要引入依赖spring-boot-starter-remote-shell即可.

&nbsp;&nbsp;Spring Boot在spring-boot-starter-remote-shell的commands包下定制了命令,它使用的是Groovy语言来编写的.Groovy语言是由Spring主导的运行于JVM的动态语言.

#### 自定义登录用户


----------


&nbsp;&nbsp;Spring Boot支持在application.properties中自定义用户的账号密码.
```
shell.auth.simple.user.name=sun
shell.auth.simple.user.password=sun
```

### end

> 资料参考于 JavaEE开发的颠覆者: Spring Boot实战
