---
title: Spring DI的三种实现方式以及优劣
date: 2017-01-13 22:56:00
tags: Spring
categories: Java EE
---

从现在起，`master`分支切换为`Java EE`,80%的时间都花在`Java EE`上面。其余时间再慢慢分配。我已经不再是`Android`的人了。做了10个月`Android`,除了`NDK`比较弱，别的方面都还好。过完年回来再花点时间研究`NDK`。谈起`Spring`,能表达的只能是6666。在`Java EE`领域的能力真的是一家独大。通常情况下，获取一个对象的实例是调用者创建被调用者的实例。而使用`Spring`框架，调用者并不负责创建被调用者的实例。这部分工作由`Spring`框架来完成，并且在对象需要使用的地方，由Spring自动注入。这就是`Spring`的依赖注入和控制反转功能。而具体是如何使用的呢？有哪些使用方式？这些使用方式有哪些优劣之处。这篇文章就是记录此功能的。我也是学习者，不是老司机。不对的地方多多指证。<!--more-->

## 设值注入

设值注入的方式很简单，通过定义setter方法，`Spring`调用无参构造方法，然后调用你所定义的setter函数。将值设置给程序所需要的地方。直接上代码。

定义`model`:

```Java
package com.model;

public class User {

	public String username;
	public String password;
	public String getUsername() {
		return username;
	}
	public void setUsername(String username) {
		this.username = username;
	}
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}
}
```

定义`dao`层接口:

```Java
package com.dao;

import com.model.User;

public interface UserDao {

	public void save(User user);
}
```

实现`dao`,只是一个简单的测试。:

```Java
package com.dao;

import com.model.User;

public class UserDaoImpl implements UserDao {

	@Override
	public void save(User user) {
		System.out.println(user.getUsername()+"   "+user.getPassword()+" write to mysql");	
	}
}
```

定义`service`:

```Java
package com.service;

import com.model.User;

public interface UserService {
	public void addUser(User user);
}
```

定义`service`实现:

```Java
package com.service;

import com.dao.UserDao;
import com.model.User;

public class UserServiceImpl implements UserService{
	
	private UserDao userDao;

	@Override
	public void addUser(User user) {
		userDao.save(user);
	}
	
	public UserDao getUserDao() {
		return userDao;
	}
	
	public void setUserDao(UserDao userDao) {
		this.userDao = userDao;
	}
}
```

定义xml配置信息:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <bean id="u" class="com.dao.UserDaoImpl"/>
       <bean id="userService" class="com.service.UserServiceImpl">
       		<property name="userDao">
       			<ref bean="u"/>
       		</property>
       </bean>
</beans>
```

测试:

```Java
package com.model;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.service.UserServiceImpl;

public class Main {

	public static void main(String[] args) {
//		ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
//		Person person = (Person) context.getBean("chinese");
//		person.speak();
//		person = (Person) context.getBean("american");
//		person.speak();
		
		ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
		UserServiceImpl serviceImpl = (UserServiceImpl) context.getBean("userService");
		User user = new User();
		user.setUsername("Kevin");
		user.setPassword("123456");
		serviceImpl.addUser(user);
	}
}
```

结果图:

![Screen Shot 2017-01-13 at 11.22.34 AM](http://7xk0q3.com1.z0.glb.clouddn.com/Screen%20Shot%202017-01-13%20at%2011.22.34%20AM.png)

## 构造方法注入

只需要在`UserServiceImpl`的实现上加上如下构造方法:

```Java
// 构造方法注入 
	public UserServiceImpl(UserDao userDao) {
		this.userDao = userDao;
	}
```

配置`bean.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <bean id="u" class="com.dao.UserDaoImpl"/>
       <bean id="userService" class="com.service.UserServiceImpl">
       		<!-- <property name="userDao">
       			<ref bean="u"/>
       		</property> -->
       		<constructor-arg>
       			<ref bean="u"/>
       		</constructor-arg>
       </bean>
</beans>
```

运行结果:

![Screen Shot 2017-01-13 at 11.22.34 AM](http://7xk0q3.com1.z0.glb.clouddn.com/Screen%20Shot%202017-01-13%20at%2011.22.34%20AM.png)

## 接口注入

接口注入需要实现特定接口，这种方式是侵入性的。 `Spring`不支持侵入式的操作。`EJB`和Avalon就是使用这种方式。



## 三种实现方式的比较

因为接口注入的方式在`Spring`中已经被遗弃了。所以暂时我们只比较构造方法注入和设值注入:

### 构造方法注入

#### 优点

- 符合Java设计原则
- 减少繁琐的设值。
- 系统层次清晰度明显

#### 缺点

- 错误比较难找
- 需要手动继承
- 很难处理一些特殊关系

### 设值注入

#### 优点

- 对于关系比较复杂或者成员变量比较多的情况。这种情况比较适合。

#### 缺点

容易破坏类的状态和行为。



