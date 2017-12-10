title: 新年第一发-Idea和MyEclipse添加Hibernate支持
date: 2016-01-10 17:42:45
updated: 2016-01-13 10:50:45
tags: [Hibernate]
categories: 后端

---
![](http://7xk0q3.com1.z0.glb.clouddn.com/idea-test.png)
&nbsp;&nbsp;&nbsp;&nbsp;前言: 前面练习或者所做项目都是直接使用MVC模式下Dao层直接使用sql语句对数据库进行操作，这种方式简单但是效率低下并且更新和维护困难，空闲下学习使用[ORM](https://zh.wikipedia.org/zh-cn/%E5%AF%B9%E8%B1%A1%E5%85%B3%E7%B3%BB%E6%98%A0%E5%B0%84)的方式来操作数据库。Hibernate就是一种Java EE中比较完善的框架，  利用xml或者注释的方式作为数据库层与上层之间的桥梁。
<!--more-->
## Hibernate原理
&nbsp;&nbsp;&nbsp;&nbsp;在JDBC编程中,我们是使用sql语句直接对数据库进行增删查该的操作，如果实体类改变，我们就需要改与之相关的所有sql语句，[ORM](https://zh.wikipedia.org/zh-cn/%E5%AF%B9%E8%B1%A1%E5%85%B3%E7%B3%BB%E6%98%A0%E5%B0%84)的出现解决了这种难题，Hibernate的作用就是替换MVC模式下的Dao层，利用POJO与实体类的映射自动生成相应的SQL语句。当实体类发生改变时，我们只需要更改实体类的配置即可。简单，方便，易维护。
## Idea中配置Hibernate
&nbsp;&nbsp;&nbsp;&nbsp;在这里使用Idea配置Hibernate，因为在Idea下比在MyEclipse下更加方便，而且Idea拥有更强大的代码提示功能。ok,开始:
1. File->New Project->Java Enterprise->配置Java->Java EE版本->Application Server。勾选Web Application,Hibernate,create hibernate configure and main class,import database schema,选择下载Hibernate库,点击next.如图:![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE-new-hibernate-project.png)

2. 配置项目名->点击next,如图:![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE-hibernate-project-name.png)
3. 弹出一个import database schema->choose data source->主机->数据库->用户名->密码->测试一下->成功点击apply->ok,如图:![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BEhibernate-database.png)
4. 点击package->选择实体类的包。
5. 刷新database schema mapping->选择需要映射的表->ok,最后如图：![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BEimpot-database-schema.png)

## MyEclipse中配置Hibernate支持
1. File->New->Web Project
2. 选择菜单Window->open perspictive->MyEclipse Database explore->选择MyEclise Derby->右键->新建MySql Connection->如下图，填写内容跟idea一样。-》finish![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BE-MyEclipse-connection-database.png)
3. 右键点击新建的连接->open connection
4. 右键项目->MyEclipse->Project Facets->install Hibernate facet->选择Hibernate版本->next->finish,如图：![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BEsession-factory.png)
5. 进入database explore->右键表->Hebernate reverse engineering->选择那种方式映射，那个项目逆向工程->finish,如图：![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BEmyeclipse-database-engineering.png)
6. 配置cfg.xml,添加:
```xml
 <property name="myeclipse.connection.profile">Kevin</property>
	<property name="show_sql">true</property>
	<property name="hbm2ddl.auto">create</property>
	<property name="current_session_context_class">thread</property>
	<property name="javax.persistence.validation.mode">none</property>
```

7. 新建servlet

```java
package org.jiangtao.servlet;

import java.io.IOException;
import java.io.PrintWriter;
import java.sql.Time;
import java.util.List;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.hibernate.Session;
import org.hibernate.Transaction;
import org.jiangtao.bean.Person;
import org.jiangtao.sessionfactory.HibernateSessionFactory;

//@WebServlet(urlPatterns="/queryObject")
public class QueryObjectServlet extends HttpServlet {

	public void doGet(HttpServletRequest request, HttpServletResponse response)
			throws ServletException, IOException {

		response.setContentType("text/html");
		PrintWriter out = response.getWriter();
		Session session = HibernateSessionFactory.getSessionFactory()
				.openSession();
		Transaction aTransaction = session.beginTransaction();
		Person person = new Person(1, "你好", 21, "1", "1234", new Time(121212));
		session.persist(person);
		List<Person> persons = session.createQuery(" from Person ").list();
		StringBuffer buffer = new StringBuffer();
		buffer.append("所有的用户:");
		for (Person person2 : persons) {
			buffer.append(person2.toString());
		}
		System.out.println(buffer);
		aTransaction.commit();
		session.close();
		out.flush();
		out.close();
	}
}
```
查询结果如下:

![](http://7xk0q3.com1.z0.glb.clouddn.com/QQ%E6%88%AA%E5%9B%BEquery-result.png)
January 13, 2016 10:53 AM









