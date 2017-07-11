---
layout:     post
title:      "Spring MVC原理及流程"
subtitle:   "Spring MVC workflow"
date:       2016-06-15
author:     "Sylvanas Sun"
catalog:    true
categories: 
    - 后端
    - Java
    - Spring
tags:
    - Java
    - SpringMVC
---



### Spring MVC介绍

 1. Spring MVC属于SpringFrameWork的后续产品，已经融合在Spring Web Flow里面。
 2. 通过策略接口，Spring MVC是高度可配置的，而且包含多种视图技术，例如 JavaServer Pages（JSP）、Velocity、Tiles、iText和POI。
 3. 使用 Spring 可插入的MVC架构，从而在使用Spring进行WEB开发时，可以选择使用Spring的SpringMVC框架或集成其他MVC开发框架，如Struts1，Struts2等。
 4. Spring MVC是一个完全基于MVC模式(Model、View、Controller)的框架。

### Spring MVC工作原理

![](http://ww1.sinaimg.cn/mw690/63503acbjw1f4v325kscyj20ui0qi776.jpg)

Spring MVC工作流程:

 1. 用户发起请求。
 2. DispatcherServlet接收到请求,并去调用HandlerMapping查找处理器。
 3. HandlerMapping根据请求的URL查找对应的处理器,并返回给前端控制器DispatcherServlet。
 4. DispatcherServlet调用HandlerAdapter执行处理器。
 5. HandlerAdapter先判断处理器的类型进行适配,然后执行处理器。
 6. 处理器进行数据和业务请求的处理,将ModelAndView对象返回给HandlerAdapter。
 7. HandlerAdapter将ModelAndView对象返回到前端控制器DispatcherServlet。
 8. DispatcherServlet调用ViewResolver解析逻辑视图ModelAndView。
 9. ViewResolver通过逻辑视图的名称查找对应的视图对象,并返回给前端控制器。
 10. DispatcherServlet调用View渲染视图到前端。
 11. 响应处理的结果。

Spring MVC主要由以下几个部分组成:

 - 前端控制器(DispatcherServlet):接收请求并响应到前端,并且负责各组件职责的分派。
 - 处理器(Handler):处理数据与业务请求,Handler需要符合适配器的规则。
 - 处理器映射器(HandlerMapping):根据每个请求的URL查找对应的处理器Handler。
 - 处理器适配器(HandlerAdapter):根据类型适配每个处理器Handler并执行。
 - 视图解析器(ViewResolver):根据逻辑视图的名称将逻辑视图解析为视图对象。
 - 视图(View):在Spring MVC中,View是一个接口,通过不同的实现类支持不同的类型。(jsp、freemarker等)

注：

 - DispatcherServlet在创建时会默认从DispatcherServlet.properties中加载springmvc的各个组件配置。
 - 如果在servletname-servlet.xml(Spring MVC配置文件)中配置了各个组件的配置则会优先使用servletname-servlet.xml中的配置。