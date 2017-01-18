---
title: Spring集成Struts和Hibernate
date: 2017-01-17 22:56:00
tags: Spring
categories: Java EE
---

`Spring`,`Struts`,`Hiberbate`基础已经学习完成。想自己把这三个框架集成一下，然后再写一个后台管理网站练练手。`Spring`的作用是依赖注入，而`Struts`是显示层的东西，这两个框架集成后是什么样子。一边学习，一边记录。上车。<!--more-->

## Spring集成Struts

首先，`Spring`集成`Struts`，那么`applicationContext.xml`和`struts.xml`,`web.xml`肯定是不能少的。前面两个是`Spring`和`Struts`的配置文件，后面一个是整个web的全局配置文件。在每个配置文件中应该怎么配置，怎么相互关联呢。其实就是将`Struts`中指定的`Action` 类为`Spring`注入的类。

三大框架集成开发并不难，难的地方在于各个包的依赖要搞清楚，版本之间的差异也是一点。下面列出`Spring`集成`Struts`所依赖的包:

### 依赖包

此处所有依赖为`Struts2.0`和`Spring3.0`。版本有点老，我用最新版的始终集成不正确。等搞好了再升级版本。

| Number |                 Package                  | Platform |                 Function                 |
| :----: | :--------------------------------------: | :------: | :--------------------------------------: |
|   1    |      `commons-fileupload-1.2.2.jar`      |  common  |                  文件上传功能                  |
|   2    |          `commons-io-2.0.1.jar`          |  common  |                                          |
|   3    |          `commons-lang-2.5.jar`          |  common  |                                          |
|   4    |       `commons-logging-1.1.1.jar`        |  common  |                    日志                    |
|   5    |         `freemarker-2.3.16.jar`          |  Struts  |                   模版引擎                   |
|   6    |        `javassist-3.11.0.GA.jar`         |  common  |                   动态编程                   |
|   7    |             `ognl-3.0.1.jar`             |  common  |             表达式语言，提供属性，方法调用              |
|   8    | `org.springframework.asm-3.1.1.RELEASE.jar` |          | Spring独立的asm程序,Spring2.5.6的时候需要asmJar 包3.0.6开始提供他自己独立的asmJar。暂时我自己也不懂这事干嘛的。 |
|   9    | `org.springframework.beans-3.1.1.RELEASE.jar` |          |               Spring IOC实现               |
|   10   | `org.springframework.context-3.1.1.RELEASE.jar` |          | Spring提供在基础IoC功能上的扩展服务，此外还提供许多企业级服务的支持，如邮件服务、任务调度、JNDI定位、EJB集成、远程访问、缓存以及各种视图层框架的封装等 |
|        | `org.springframework.context.support-3.1.1.RELEASE.jar` |          |       Spring-context的扩展支持,用于MVC方面        |
|   12   | `org.springframework.core-3.1.1.RELEASE.jar` |          |               Spring 核心工具包               |
|   13   | `org.springframework.expression-3.1.1.RELEASE.jar` |          |               Spring表达式语言                |
|   14   | `org.springframework.web-3.1.1.RELEASE.jar` |          |              Spring Web工具包               |
|   15   | `org.springframework.web.servlet-3.1.1.RELEASE.jar` |          |             基于servlet的MVC实现              |
|   16   |        `struts2-core-2.2.3.1.jar`        |          |                Struts核心库                 |
|   17   |         `xwork-core-2.2.3.1.jar`         |          |                 xwork核心库                 |
|   18   |   `struts2-spring-plugin-2.2.3.1.jar`    |          |            Spring与Struts相互集成             |



### 集成

- 添加全局web配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
  <display-name>SpringSS</display-name>
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>
  
  <filter>
	<filter-name>struts2</filter-name>
	<filter-class> org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter </filter-class>
  </filter>

  <listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <filter-mapping>
	<filter-name>struts2</filter-name>
	<url-pattern>/*</url-pattern>	
  </filter-mapping>
</web-app>
```
- Spring配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN 2.0//EN" "http://www.springframework.org/dtd/spring-beans-2.0.dtd">
<beans>
	<!--在Spring容器中注册Bean-->
<bean id="userAction" class="com.action.UserAction" >
<!--设值注入-->
    	<property name="name" value="哈哈哈" />
    </bean>
</beans>
```

- Struts配置文件

```xml
<!DOCTYPE struts PUBLIC
"-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
"http://struts.apache.org/dtds/struts-2.0.dtd">

<struts>
<constant name="struts.objectFactory" value="spring" />
    <package name="default" extends="struts-default">
        <action name="user" class="userAction">
            <result name="success">/user.jsp</result>
        </action>
    </package>
</struts>
```
### Action以及各个jsp 文件

- Action

```Java
/**
 * 
 */
package com.action;

import com.opensymphony.xwork2.ActionSupport;

/**
 * @author kevin
 *
 */
public class UserAction extends ActionSupport {
		public String name;

		public String getName() {
			return name;
		}

		public void setName(String name) {
			this.name = name;
		}
		
		@Override
		public String execute() throws Exception {
		// TODO Auto-generated method stub
		return "success";
		}
}
```

- 第一个页面
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@taglib prefix="s" uri="/struts-tags"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1>测试</h1>
<s:form action="user">
<s:submit></s:submit>
</s:form>
</body>
</html>
```

- 第二个页面

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1>妈的智障</h1>
${name} 
</body>
</html>
```