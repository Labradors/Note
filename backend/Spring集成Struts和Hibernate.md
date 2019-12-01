# Spring集成Struts和Hibernate


`Spring`,`Struts`,`Hiberbate`基础已经学习完成。想自己把这三个框架集成一下，然后再写一个后台管理网站练练手。`Spring`的作用是依赖注入，而`Struts`是显示层的东西，这两个框架集成后是什么样子。一边学习，一边记录。上车。<!--more-->

## Spring集成所需jar包

首先，`Spring`集成`Struts`，那么`applicationContext.xml`和`struts.xml`,`web.xml`肯定是不能少的。前面两个是`Spring`和`Struts`的配置文件，后面一个是整个web的全局配置文件。在每个配置文件中应该怎么配置，怎么相互关联呢。其实就是将`Struts`中指定的`Action` 类为`Spring`注入的类。

三大框架集成开发并不难，难的地方在于各个包的依赖要搞清楚，版本之间的差异也是一点。下面列出`Spring`集成`Struts`所依赖的包:

### 依赖包

此处所有依赖为`Struts2.0`和`Spring3.0`。版本有点老，我用最新版的始终集成不正确。等搞好了再升级版本。

| Number |                 Package                  | Platform  |                 Function                 |
| :----: | :--------------------------------------: | :-------: | :--------------------------------------: |
|   1    |      `commons-fileupload-1.2.2.jar`      |  common   |                  文件上传功能                  |
|   2    |          `commons-io-2.0.1.jar`          |  common   |                                          |
|   3    |          `commons-lang-2.5.jar`          |  common   |                                          |
|   4    |       `commons-logging-1.1.1.jar`        |  common   |                    日志                    |
|   5    |         `freemarker-2.3.16.jar`          |  Struts   |                   模版引擎                   |
|   6    |        `javassist-3.11.0.GA.jar`         |  common   |                   动态编程                   |
|   7    |             `ognl-3.0.1.jar`             |  common   |             表达式语言，提供属性，方法调用              |
|   8    | `org.springframework.asm-3.1.1.RELEASE.jar` |  spring   | Spring独立的asm程序,Spring2.5.6的时候需要asmJar 包3.0.6开始提供他自己独立的asmJar。暂时我自己也不懂这事干嘛的。 |
|   9    | `org.springframework.beans-3.1.1.RELEASE.jar` |  spring   |               Spring IOC实现               |
|   10   | `org.springframework.context-3.1.1.RELEASE.jar` |  spring   | Spring提供在基础IoC功能上的扩展服务，此外还提供许多企业级服务的支持，如邮件服务、任务调度、JNDI定位、EJB集成、远程访问、缓存以及各种视图层框架的封装等 |
|        | `org.springframework.context.support-3.1.1.RELEASE.jar` |  spring   |       Spring-context的扩展支持,用于MVC方面        |
|   12   | `org.springframework.core-3.1.1.RELEASE.jar` |  spring   |               Spring 核心工具包               |
|   13   | `org.springframework.expression-3.1.1.RELEASE.jar` |  spring   |               Spring表达式语言                |
|   14   | `org.springframework.web-3.1.1.RELEASE.jar` |  spring   |              Spring Web工具包               |
|   15   | `org.springframework.web.servlet-3.1.1.RELEASE.jar` |  spring   |             基于servlet的MVC实现              |
|   16   |        `struts2-core-2.2.3.1.jar`        |  struts   |                Struts核心库                 |
|   17   |         `xwork-core-2.2.3.1.jar`         |  struts   |                 xwork核心库                 |
|   18   |   `struts2-spring-plugin-2.2.3.1.jar`    |  struts   |            Spring与Struts相互集成             |
|   19   |            `antlr-2.7.2.jar`             |  common   |                 语言语法分析器                  |
|   20   |          `aopalliance-1.0.jar`           |  common   |                 面向切面编程接口                 |
|   21   |            `commons-dbcp.jar`            |  common   |                DBCP数据库连接池                |
|   22   |            `commons-pool.jar`            |  common   |                DBCP数据库连接池                |
|   23   |            `dom4j-1.6.1.jar`             | hibernate |                 灵活的xml框架                 |
|   24   | `hibernate-jpa-2.0-api-1.0.1.Final.jar`  | hibernate |                  注解使用类                   |
|   25   |             `hibernate3.jar`             | hibernate |                  数据库核心包                  |
|   26   |              `jta-1.1.jar`               | hibernate |                 分布式事务处理                  |
|   27   |  `mysql-connector-java-5.1.18-bin.jar`   | hibernate |                 jdbc连接器                  |
|   28   | `org.springframework.jdbc-3.1.1.RELEASE.jar` | hibernate |              spring与jdbc集成               |
|   29   | `org.springframework.orm-3.1.1.RELEASE.jar` | hibernate |                  数据库集成                   |
|   30   | `org.springframework.transaction-3.1.1.RELEASE.jar` | hibernate |                   事务集成                   |
|   31   |          `slf4j-api-1.6.1.jar`           |  common   |                   日志系统                   |



## 集成

### model层

- 新建`User`model，如下:


```java
package com.action;

import java.io.Serializable;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
@javax.persistence.Table(name="user")
public class User implements Serializable{

	private static final long serialVersionUID = 1L;
	@Id
	@GeneratedValue
	@Column(name="id")
	public int id;
	@Column(name="name")
	public String name;
	@Column(name="password")
	public String password;
	
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}
	@Override
	public String toString() {
		return "User [name=" + name + ", password=" + password + "]";
	}
}
```
### dao层

- 新建dao接口:

```Java
package com.dao.impl;

import java.util.List;

import com.action.User;
import com.action.UserAction;

public interface UserDao {

	public void save(User action);
	
	public User getUser(int id);
	
	public void update(User action);
	
	public void delete(User userAction);
	
	public List<User> findByName(String name);
}
```
- 实现dao接口

```Java
package com.dao.impl;

import java.util.List;

import org.hibernate.SessionFactory;
import org.springframework.orm.hibernate3.HibernateTemplate;

import com.action.User;
import com.action.UserAction;

public class UserDaoImpl implements UserDao {
	
	private SessionFactory sessionFactory;
	private HibernateTemplate mHibernateTemplate;
	
	public SessionFactory getSessionFactory() {
		return sessionFactory;
	}

	public void setSessionFactory(SessionFactory sessionFactory) {
		this.sessionFactory = sessionFactory;
	}

	public HibernateTemplate getHbernateTemplate() {
		if (mHibernateTemplate==null) {
			mHibernateTemplate = new HibernateTemplate(sessionFactory);
		}
		return mHibernateTemplate;
	}
	
	public void save(User action) {
		// TODO Auto-generated method stub
		getHbernateTemplate().save(action);
	}

	public User getUser(int id) {
		// TODO Auto-generated method stub
		User userAction = getHbernateTemplate().get(User.class, new Integer(id));
		return userAction;
	}

	public void update(User action) {
		// TODO Auto-generated method stub
		getHbernateTemplate().update(action);
	}

	public void delete(User userAction) {
		// TODO Auto-generated method stub
		getHbernateTemplate().delete(userAction);
	}

	@SuppressWarnings("unchecked")
	public List<User> findByName(String name) {
		// TODO Auto-generated method stub
		String queryString = "from User u where u.name like ?";
		return getHbernateTemplate().find(queryString);
	}

	
}
```

### view层

-  显示以及action

```Java
/**
 * 
 */
package com.action;

import javax.servlet.http.HttpServletResponse;

import org.apache.struts2.ServletActionContext;

import com.dao.impl.UserDaoImpl;
import com.opensymphony.xwork2.ActionSupport;

/**
 * @author kevin
 *
 */
public class UserAction extends ActionSupport {
		public String name;
		public String password;
		private UserDaoImpl userDao;
		
		public String getName() {
			return name;
		}
		
		public void setUserDao(UserDaoImpl userDao) {
			this.userDao = userDao;
		}
		
		public UserDaoImpl getUserDao() {
			return userDao;
		}

		public void setName(String name) {
			this.name = name;
		}
		
		
		public String getPassword() {
			return password;
		}

		public void setPassword(String password) {
			this.password = password;
		}

		@Override
		public String execute() throws Exception {
			// 不能直接new 得从applicationContext中获取
			HttpServletResponse response = ServletActionContext.getResponse();
			response.setContentType("text/xml;charset=UTF-8");
			User user = new User();
			user.name = name;
			user.password = password;
			userDao.save(user);
			response.getWriter().write(user.toString());
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
<s:textfield name="name" label="username"></s:textfield>
<s:textfield name="password" label="password"></s:textfield>
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
${password} 
</body>
</html>
```

### 配置文件

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
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
		http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
		http://www.springframework.org/schema/aop
		http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
		
	<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource"> 
		<property name="driverClassName">
			<value>com.mysql.jdbc.Driver</value>
		</property>
		<property name="url">
			<value>jdbc:mysql://localhost/spring</value>
		</property>
		<property name="username">
			<value>root</value>
		</property>
		<property name="password">
			<value>123456</value>
		</property>
	</bean>
	<bean id="sessionFactory" class="org.springframework.orm.hibernate3.annotation.AnnotationSessionFactoryBean">
		<property name="dataSource">
			<ref local="dataSource"/>
		</property>
		<property name="annotatedClasses">
			<list>
				<value>com.action.User</value>
			</list>
		</property>
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
				<prop key="show_sql">true</prop>
			</props>
		</property>
	</bean>
	<bean id="userDao" class="com.dao.impl.UserDaoImpl">
		<property name="sessionFactory">
			<ref local="sessionFactory"/>
		</property>
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
## 结果显示

- 输入页面

![2017012052053Screen Shot 2017-01-20 at 4.43.13 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017012052053Screen Shot 2017-01-20 at 4.43.13 PM.png)

- 结果页面

![2017012041414Screen Shot 2017-01-20 at 4.44.46 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017012041414Screen Shot 2017-01-20 at 4.44.46 PM.png)

- 数据库

![2017012082110Screen Shot 2017-01-20 at 4.45.07 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017012082110Screen Shot 2017-01-20 at 4.45.07 PM.png)

最后看起来，还是不难的嘛。其实UserDao可以抽象出来，只需要单次注入，等以后再完善。