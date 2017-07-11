---
layout:     post
title:      "Spring Boot消息通信"
subtitle:   "Spring Message"
date:       2016-08-03 18:00
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


### SSL安全套接层


----------


#### 概述


----------


&nbsp;&nbsp;SSL(Secure Sockets Layer)是专门用于网络通信安全及数据完整性的安全协议,它会在网络传输层对网络连接进行加密.

&nbsp;&nbsp;SSL协议是位于TCP/IP协议与其他各种应用层协议之间的.

&nbsp;&nbsp;SSL的体系结构中包含两个协议子层:

 1. SSL记录协议(SSL Record Protocol),它建立在可靠的传输协议(TCP)之上,为高层协议提供数据封装、压缩、加密等基本功能的支持.SSL纪录协议针对HTTP协议进行了特别的设计，使得超文本的传输协议HTTP能够在SSL运行.

 2. SSL握手协议(SSL Handshake Protocol),它建立在SSL记录协议之上.SSL握手协议层包括SSL握手协议（SSL HandShake Protocol）、SSL密码参数修改协议（SSL Change Cipher Spec Protocol）、应用数据协议（Application Data Protocol）和SSL告警协议（SSL Alert Protocol）.它主要用于在实际数据传输开始前,通信双方进行身份认证、协商加密算法、交换加密密钥等.

 &nbsp;&nbsp;在B/S应用中,是通过**HTTPS**实现SSL的,**HTTPS**即是在HTTP下加入SSL层,它的安全基础是SSL.
 
#### 工作流程


----------


 1. 客户端向服务器发送一个开始信息“Hello”以便开始一个新的会话连接.
 2. 服务器根据客户的信息确定是否需要生成新的主密钥，如需要则服务器在响应客户的“Hello”信息时将包含生成主密钥所需的信息.
 3. 客户根据收到的服务器响应信息，产生一个主密钥，并用服务器的公开密钥加密后传给服务器.
 4. 服务器回复该主密钥，并返回给客户一个用主密钥认证的信息，以此让客户认证服务器.

 
#### Spring Boot中配置SSL 


----------


&nbsp;&nbsp;在配置SSL之前需要生成一个证书,它可以是自签名的,也可以是从SSL证书授权中心获得的.

##### 生成SSL证书

&nbsp;&nbsp;可以在控制台中输入以下指令,生成自签名的SSL证书:

```
keytool -genkey -alias tomcat
```

&nbsp;&nbsp;之后会在当前目录生成一个.keystore文件,它就是SSL证书文件.

##### 配置到项目中

 1. 将.keystore文件复制到src/main/resources/static下.
 2. 在application.properties中如下配置:
    ```
    server.port = 8090
    server.ssl.key-store = .keystore
    server.ssl.key-store-password = 123456
    server.ssl.keyStoreType = JKS
    server.ssl.keyAlias : tomcat
    ```

#### http自动转向https


----------


&nbsp;&nbsp;如果想要实现输入http自动转向到https,则需要配置**TomcatEmbeddedServletContainerFactory**,并添加Tomcat的connector来实现.

```java
package cn.sun.sylvanas.spring_boot_security;

import org.apache.catalina.Context;
import org.apache.catalina.connector.Connector;
import org.apache.tomcat.util.descriptor.web.SecurityCollection;
import org.apache.tomcat.util.descriptor.web.SecurityConstraint;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.embedded.EmbeddedServletContainerFactory;
import org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class SpringBootSecurityApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringBootSecurityApplication.class, args);
	}

	@Bean
	public EmbeddedServletContainerFactory servletContainerFactory() {
		TomcatEmbeddedServletContainerFactory tomcat = new TomcatEmbeddedServletContainerFactory(){
			@Override
			protected void postProcessContext(Context context) {
				SecurityConstraint securityConstraint = new SecurityConstraint();
				securityConstraint.setUserConstraint("CONFIDENTIAL");
				SecurityCollection securityCollection = new SecurityCollection();
				securityCollection.addPattern("/");
				securityConstraint.addCollection(securityCollection);
				context.addConstraint(securityConstraint);
			}
		};
		tomcat.addAdditionalTomcatConnectors(httpConnector());
		return tomcat;
	}

	@Bean
	public Connector httpConnector() {
		Connector connector = new Connector("org.apache.coyote.http11.Http11NioProtocol");
		connector.setScheme("http");
		connector.setPort(8080);
		connector.setSecure(false);
		connector.setRedirectPort(8090);
		return connector;
	}

}
```

&nbsp;&nbsp;此时访问http://localhost:8080会自动转向到https://localhost:8090.


### WebSocket


----------


#### 概述


----------


![](http://ww2.sinaimg.cn/mw690/63503acbjw1f6mihqw3jpj20sg0dttc2.jpg)
&nbsp;&nbsp;WebSocket protocol 是HTML5一种新的协议.它实现了浏览器与服务器全双工异步通信(full-duplex).即浏览器可以向服务端发送消息,服务端也可以向浏览器发送消息.WebSocket需要浏览器的支持,如IE 10+、Chrome 13+、Firefox 6+.

&nbsp;&nbsp;WebSocket是通过一个socket来实现双工异步通信功能的.直接使用WebSocket协议开发程序会很繁琐,所以一般使用它的子协议**STOMP**,**STOMP**是一个更高级别的协议,它使用基于帧(frame)的格式来定义消息,具有一个类似`@RequestMapping`的`@MessageMapping`.

#### Spring Boot支持


----------


&nbsp;&nbsp;Spring Boot对内嵌的Tomcat(7、8)、Jetty9、Undertow提供了WebSocket支持.依赖为spring-boot-starter-websocket.

&nbsp;&nbsp;配置WebSocket需要在配置类上使用`@EnableWebSocketMessageBroker`注解开启支持,并且继承**AbstractWebSocketMessageBrokerConfigurer**类,重写其方法进行配置.

#### Example


----------


##### 广播

&nbsp;&nbsp;广播即服务端有消息时,会将消息发送给所有连接了当前endpoint的浏览器.

 1. 配置

 ![](http://ww1.sinaimg.cn/mw690/63503acbjw1f6mihr9qtsj20om0ddadp.jpg)
 
 2. Controller
   ```java
   /**
     * @MessageMapping注解映射/hello这个地址,类似于@RequestMapping
     * 当服务端有消息时,会对订阅了@SendTo中的路径的浏览器发送消息.
     */
    @MessageMapping("/hello")
    @SendTo("/topic/getHello")
    public String hello(String name) throws Exception {
        Thread.sleep(3000);
        return "Hello " + name + "!";
    }
   ```
   
 3. js
  ```javascript
  <html xmlns:th="http://www.thymeleaf.org">

<head>
    <meta content="text/html;charset=UTF-8"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
    <meta name="viewport" content="width=device-width,initial-scale=1"/>
    <title>Spring Boot+WebSocket+广播式</title>
</head>
<body onload="disconnect()">
<noscript><h2 style="color: #ff0000">貌似你的浏览器不支持websocket</h2></noscript>
<div>
    <div>
        <button id="connect" onclick="connect();">连接</button>
        <button id="disconnect" onclick="disconnect();">断开连接</button>
    </div>
    <div id="conversationDiv">
        <label>输入你的名字</label><input type="text" id="name"/>
        <button id="sendName" onclick="sendName();">发送</button>
        <p id="response"></p>
    </div>
</div>
<!-- 导入jQuery -->
<script th:src="@{jquery-3.1.0.min.js}"/>
<script th:src="@{sockjs-1.0.0.min.js}"/>
<script th:src="@{stomp.min.js}"/>
<script type="text/javascript">
    var stompClient = null;

    function setConnected(connected) {
        document.getElementById('connect').disabled = connected;
        document.getElementById('disconnect').disabled = !connected;
        document.getElementById('conversationDiv').style.visibility = connected ? 'visible' : 'hidden';
        $('#response').html();
    }

    function connect() {
        var socket = new SockJS('/endpointSun');
        stompClient = Stomp.over(socket);
        stompClient.connect({}, function (frame) {
            setConnected(true);
            console.log('Connected: ' + frame);
            // 订阅/topic/getHello
            stompClient.subscribe('/topic/getHello',function(response){
                showResponse(JSON.parse(response.body).responseMessage);
            });
        });
    }

    function disconnect() {
        if(stompClient != null){
            stompClient.disconnect();
        }
        setConnected(false);
        console.log("Disconnected");
    }

    function sendName() {
        var name = $('#name').val();
        // 发送到/hello
        stompClient.send("/hello",{},JSON.stringify({'name':name}));
    }

    function showResponse(message){
        var response = $("#response");
        response.html(message);
    }

</script>
</body>

</html>
  ```
##### P2P

&nbsp;&nbsp;P2P即点对点,它不同于广播式,P2P需要指定消息由谁接收,且只能由一个人接收.

 1. 配置与广播式相同.

 2. Controller
   ```java
   /**
     * 通过SimpMessagingTemplate向浏览器发送消息.
     */
    @Autowired
    private SimpMessagingTemplate messagingTemplate;

    /**
     * Spring MVC可以直接在参数中注入principal对象,它包含当前用户的信息.
     */
    @MessageMapping("/chat")
    public void handleChat(Principal principal, String msg) {
        /**
         * 通过SimpMessagingTemplate.convertAndSendToUser向用户发送消息,
         * 第一个参数是接收者的用户,第二参数是浏览器订阅的地址,第三个参数是要发送的消息.
         */
        if (principal.getName().equals("sun")) {
            messagingTemplate.convertAndSendToUser("sylvanas",
                    "/queue/notifications", principal.getName() + "-send:" + msg);
        } else {
            messagingTemplate.convertAndSendToUser("sun",
                    "/queue/notifications", principal.getName() + "-send:" + msg);
        }
    }
   ```

 3. js
  ```javascript
  <!DOCTYPE html>
<!--suppress ALL -->
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">

<head>
    <meta content="text/html;charset=UTF-8"/>
    <meta http-equiv="X-UA-Compatible" content="IE=edge"/>
    <meta name="viewport" content="width=device-width,initial-scale=1"/>
    <script th:src="@{jquery-3.1.0.min.js}"/>
    <script th:src="@{stomp.min.js}"/>
    <script th:src="@{sockjs-1.0.0.min.js}"/>
</head>
<body>
<p>
    聊天室
</p>

<form id="chatForm">
    <textarea rows="4" cols="60" name="text"></textarea>
    <input type="submit"/>
</form>

<script th:inline="javascript">
    $('#chatForm').submit(function(e){
        e.preventDefault();
        var text = $('#chatForm').find('textarea[name="text"]').val();
        sendSpittle(text);
    });

    var socket = new SockJS("/endpointChat");
    var stomp = Stomp.over(socket);
    stomp.connect('guest','guest',function(frame){
        // 与messagingTemplate.convertAndSendToUser中定义的订阅地址保持一致
        // 但前面要多出一个/user,这个/user是必须的,只有使用了/user才会发送消息到指定的用户.
        stomp.subscribe("/user/queue/notifications",handleNotification);
    });

    function handleNotification(message){
        $("#output").append("<b>Received: " + message.body + "</b><br/>");
    }

    function sendSpittle(text){
        stomp.send("/chat",{},text);
    }

    $('#stop').click(function(){socket.close()});

</script>

<div id="output"></div>

</body>

</html> 
  ```

### 异步消息队列


----------


&nbsp;&nbsp;异步消息队列主要用于各系统之间的通信与解耦,异步消息即为发送者无需关心消息接收者的处理和返回.

&nbsp;&nbsp;异步消息的主要概念为消息代理(message broker)和目的地(destination).消息是由消息代理负责接管并传递到指定目的地的.

&nbsp;&nbsp;目的地主要有2种:
 
 1. queue:用于P2P(point-to-point)的消息通信.
 2. topic:用于发布/订阅(publish/subscribe)的消息通信.

#### Spring Boot支持


----------


&nbsp;&nbsp;Spring对JMS和AMQP的支持来自于spring-jms、spring-rabbit.它们分别需要ConnectionFactory的实现来连接消息代理.并且提供了JmsTemplate和RabbitTemplate.

 - JMS(Java Message Service)Java消息服务,它是基于JVM消息代理的规范.
 - AMQP(Advanced Message Queuing Protocol)高级消息队列协议,它不仅支持JVM,还支持跨语言和平台.AMQP的主要实现有RabbitMQ.

&nbsp;&nbsp;Spring Boot对JMS支持的实有ActiveMQ,HornetQ,Artemis.以ActiveMQ为例:
 - Spring Boot自动配置了ActiveMQConnectionFactory与JmsTemplate.
 - 通过`spring.activemq`为前缀的属性来配置ActiveMQ相关的属性.
 - Spring Boot还自动开启了`@EnableJms`,即使用注解式消息监听的支持.

&nbsp;&nbsp;Spring Boot对AMQP的实现RabbitMQ自动配置了如下内容:
 - 自动配置了ConnectionFactory和RabbitTemplate.
 - 通过`spring.rabbitmq`为前缀的属性来配置RabbitMQ相关的属性.
 - 自动开启了`@EnableRabbit`,即使用注解式消息监听的支持.

#### JMS Example


----------


&nbsp;&nbsp;添加依赖

```
        <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jms</artifactId>
		</dependency>
		<dependency>
			<groupId>org.apache.activemq</groupId>
			<artifactId>activemq-client</artifactId>
		</dependency>

		<!-- 嵌入ActiveMQ -->
		<dependency>
			<groupId>org.apache.activemq</groupId>
			<artifactId>activemq-broker</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
```

&nbsp;&nbsp;在application.properties中配置activemq的地址.

```
spring.activemq.broker-url=tcp://192.168.145.152:61616
```

&nbsp;&nbsp;定义JMS发送的消息需要实现MessageCreator接口.
```java
public class Msg implements MessageCreator {

    /**
     * 重写createMessage方法.
     */
    @Override
    public Message createMessage(Session session) throws JMSException {
        return session.createTextMessage("Hello World");
    }

}
```

&nbsp;&nbsp;发送消息,定义目的地.
```java
/**
 * Spring Boot提供了一个CommandLineRunner接口,用于程序启动后执行的代码
 */
@SuppressWarnings("SpringJavaAutowiringInspection")
@SpringBootApplication
public class SpringBootActivemqApplication implements CommandLineRunner {
	

	public static void main(String[] args) {
		SpringApplication.run(SpringBootActivemqApplication.class, args);
	}

	@Autowired
	private JmsTemplate jmsTemplate;

	@Override
	public void run(String... strings) throws Exception {
		// 向目的地my-destination发送消息
		jmsTemplate.send("my-destination",new Msg());
	}
}
```

&nbsp;&nbsp;接收消息

```java
@Component
public class Receiver {

    /**
     * @JmsListener是Spring 4.1提供的一个新特性,用来简化JMS开发.
     * destination指定要监听的目的地.
     */
    @JmsListener(destination = "my-destination")
    public void receiveMessage(String message) {
        System.out.println("接收到: <" + message + ">");
    }

}
```

#### AMQP Example


----------


&nbsp;&nbsp;Spring Boot默认Rabbit主机位localhost,端口号为5672.

&nbsp;&nbsp;添加以下依赖:
```
        <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-amqp</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
```

&nbsp;&nbsp;在application.properties中配置RabbitMQ的地址.
```
spring.rabbitmq.host=192.168.145.152
spring.rabbitmq.port=5672
```

&nbsp;&nbsp;发送消息,定义目的地.

```java
@SuppressWarnings("SpringJavaAutowiringInspection")
@SpringBootApplication
public class SpringBootRabbitmqApplication implements CommandLineRunner {
	

	public static void main(String[] args) {
		SpringApplication.run(SpringBootRabbitmqApplication.class, args);
	}

	@Autowired
	private RabbitTemplate rabbitTemplate;

	/**
	 * 定义目的地
     */
	@Bean
	public Queue myQueue() {
		return new Queue("my-queue");
	}


	/**
	 * 使用rabbitTemplate的convertAndSend方法向队列my-queue发送消息.
     */
	@Override
	public void run(String... strings) throws Exception {
		rabbitTemplate.convertAndSend("my-queue","Hello RabbitMQ!");
	}
}
```

&nbsp;&nbsp;接收消息

```java
@Component
public class Receiver {

    @RabbitListener(queues = "my-queue")
    public void receiveMessage(String message) {
        System.out.println("Received <" + message + ">");
    }

}
```

### end

> 资料参考于 JavaEE开发的颠覆者: Spring Boot实战
