---
title: Spring Bean生命周期和作用域以及实现方式
date: 2021-05-03 14:30:24
tags:  Spring
categories: Java
---

在`applicationContext.xml`中配置完`bean`之后，`Bean`的声明周期状态有哪些。生命周期的各个阶段可以做什么。在`applicationContext.xml`配置`bean`的作用域有哪些。其中各个作用域代表的是什么。适用于什么情况。这篇文章做一个记录。<!--more-->

## 生命周期

### 初始化

可以直接查看图片，图片来自[Spring Bean Life Cycle](http://www.wideskills.com/spring/spring-bean-lifecycle)

![2017011490488Image_spring_bean.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017011490488Image_spring_bean.png)

从上图看出，`Bean`初始化完成包括9个步骤。其中一些步骤包括接口的实现，其中包括`BeanNameAware`接口，`BeanFactoryAware`接口。`ApplicationContextAware`接口。`BeanPostProcessor`接口，`InitializingBean`接口。那么这些接口在整个生命周期阶段都起到什么作用？后面我们一一介绍。

### 实例化前

当`Bean`全部属性设置完毕后，往往需要执行一些特定的行为，`Spring`提供了两种方式来实现此功能:

- 使用`init-mothod`方法
- 实现`initializingBean`接口

#### 指定初始化方法

如下:

```Java
package com.model;

public class InitBean {
	public static final String NAME = "mark";
	public static final int AGE = 20;
	
	public InitBean() {
		// TODO Auto-generated constructor stub
		System.out.println("执行构造方法");
	}
	
	public String name;
	public int age ;
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	
	public void init(){
		System.out.println("调用init方法进行成员变量的初始化");
		this.name = NAME;
		this.age = AGE;
		System.out.println("初始化完成");
	}
}
```

编写加载器

```Java
package com.model;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.service.UserServiceImpl;

public class Main {

	public static void main(String[] args) {		
		ApplicationContext context = new ClassPathXmlApplicationContext("initbean.xml");
		InitBean bean = (InitBean) context.getBean("init");
	}

}

```

配置`Bean`

注意`init-method`参数

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <bean id="init" class="com.model.InitBean" init-method="init"/>
</beans>
```

执行结果

![2017011451358Screen Shot 2017-01-14 at 12.15.10 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017011451358Screen Shot 2017-01-14 at 12.15.10 PM.png)

#### 实现`InitializingBean`接口

实现`InitializingBean`接口会实现`afterPropertiesSet`方法，这个方法会自动调用。但是这个方式是侵入性的。一般情况下，不建议使用。

实现`afterPropertiesSet`方法

```Java
package com.model;

import org.springframework.beans.factory.InitializingBean;

public class InitBean implements InitializingBean {
	public static final String NAME = "mark";
	public static final int AGE = 20;
	
	public InitBean() {
		// TODO Auto-generated constructor stub
		System.out.println("执行构造方法");
	}
	
	public String name;
	public int age ;
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	
	public void init(){
		System.out.println("调用init方法进行成员变量的初始化");
		this.name = NAME;
		this.age = AGE;
		System.out.println("初始化完成");
	}
	@Override
	public void afterPropertiesSet() throws Exception {
		// TODO Auto-generated method stub
		System.out.println("调用init方法进行成员变量的初始化");
		this.name = NAME;
		this.age = AGE;
		System.out.println("初始化完成");
	}
}
```

配置`xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <!-- <bean id="init" class="com.model.InitBean" init-method="init"/> -->
      <bean id="init" class="com.model.InitBean" init-method="init"/>
</beans>
```

结果:

![2017011434120Screen Shot 2017-01-14 at 12.24.30 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017011434120Screen Shot 2017-01-14 at 12.24.30 PM.png)

### 销毁

![2017011413039Image_spring_destory.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017011413039Image_spring_destory.png)

同样，上图中表示来`Bean`销毁时候的过程。包括`DisposableBean`接口。

#### 使用`destroy-method`方法

```Java
package com.model;

import org.springframework.beans.factory.InitializingBean;

public class InitBean implements InitializingBean {
	public static final String NAME = "mark";
	public static final int AGE = 20;
	
	public InitBean() {
		// TODO Auto-generated constructor stub
		System.out.println("执行构造方法");
	}
	
	public String name;
	public int age ;
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	
	public void init(){
		System.out.println("调用init方法进行成员变量的初始化");
		this.name = NAME;
		this.age = AGE;
		System.out.println("初始化完成");
	}
	@Override
	public void afterPropertiesSet() throws Exception {
		// TODO Auto-generated method stub
		System.out.println("调用init方法进行成员变量的初始化");
		this.name = NAME;
		this.age = AGE;
		System.out.println("初始化完成");
	}
	
	public void close(){
		System.out.println("bean被销毁");
	}
}
```

配置`Bean`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <!-- <bean id="init" class="com.model.InitBean" init-method="init"/> -->
      <bean id="init" class="com.model.InitBean" destroy-method="close"/>
</beans>
```

配置加载器

```Java
package com.model;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.service.UserServiceImpl;

public class Main {

	public static void main(String[] args) {		
		AbstractApplicationContext context = new ClassPathXmlApplicationContext("initbean.xml");
		context.registerShutdownHook();
		InitBean bean = (InitBean) context.getBean("init");
	}

}

```

结果:

![2017011478398Screen Shot 2017-01-14 at 1.02.37 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017011478398Screen Shot 2017-01-14 at 1.02.37 PM.png)

#### 实现`DisposableBean`接口

实现`DisposableBean`接口

```Java
package com.model;

import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;

public class InitBean implements InitializingBean,DisposableBean {
	public static final String NAME = "mark";
	public static final int AGE = 20;
	
	public InitBean() {
		// TODO Auto-generated constructor stub
		System.out.println("执行构造方法");
	}
	
	public String name;
	public int age ;
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	
	public void init(){
		System.out.println("调用init方法进行成员变量的初始化");
		this.name = NAME;
		this.age = AGE;
		System.out.println("初始化完成");
	}
	@Override
	public void afterPropertiesSet() throws Exception {
		// TODO Auto-generated method stub
		System.out.println("调用init方法进行成员变量的初始化");
		this.name = NAME;
		this.age = AGE;
		System.out.println("初始化完成");
	}
	
	public void close(){
		System.out.println("bean被销毁");
	}
	@Override
	public void destroy() throws Exception {
		// TODO Auto-generated method stub
		System.out.println("bean被销毁完成");
	}
}
```

配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <!-- <bean id="init" class="com.model.InitBean" init-method="init"/> -->
     <!--  <bean id="init" class="com.model.InitBean" destroy-method="close"/> -->
      <bean id="init" class="com.model.InitBean"/>
</beans>
```

![2017011439862Screen Shot 2017-01-14 at 1.08.37 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017011439862Screen Shot 2017-01-14 at 1.08.37 PM.png)

## `Spring Bean`的作用域

| 作用域            | 描述                                       |
| -------------- | ---------------------------------------- |
| singleton      | 该作用域将 bean 的定义的限制在每一个 Spring IoC 容器中的一个单一实例(默认)。 |
| prototype      | 该作用域将单一 bean 的定义限制在任意数量的对象实例。            |
| request        | 该作用域将 bean 的定义限制为 HTTP 请求。只在 web-aware Spring ApplicationContext 的上下文中有效。 |
| session        | 该作用域将 bean 的定义限制为 HTTP 会话。 只在web-aware Spring ApplicationContext的上下文中有效。 |
| global-session | 该作用域将 bean 的定义限制为全局 HTTP 会话。只在 web-aware Spring ApplicationContext 的上下文中有效。 |

### 配置示例

```java
<bean id="..." class="..." scope="singleton">
</bean>
```

## 使用方法注入协调作用域不同的`Bean`

正常情况下，如果singleton作用域依赖singleton作用域。即每次获取到的都是一样的对象。同理，prototype作用域依赖prototype作用域，每次获取到的都是新的对象。但是，如果singleton依赖prototype作用域，那么每次获取到的singleton中的prototype都是第一次创建的prototype。如何协调这种关系。保证每次获取到的都是正确的呢。

对于这种情况，`Spring`提供了`lookup`方法用来解决这种问题。

首先我们定义一个原型:

```Java
package com.model;

public class MyHelper {

	public void doSomethingHelpful() {
		
	}
}
```

然后通过接口注入:

```Java
package com.model;

public interface DemoBean {

	MyHelper getHelper();
	void somePeration();
}
```

配置一个单例：

```Java
package com.model;

/**
 * 测试类
 * @author kevin
 *
 */
public abstract class AbstractLookupDemo implements DemoBean {
	
	public abstract MyHelper getMyHelper();
	
	@Override
	public MyHelper getHelper() {
		// TODO Auto-generated method stub
		return getMyHelper();
	}
	
	@Override
	public void somePeration() {
		// TODO Auto-generated method stub
		
	}

}
```

配置文件:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <bean id="helper" class="com.model.MyHelper" scope="prototype"/>
      <bean id="standardLookupBean" class="com.model.StandardLookupDemo">
      		<property name="myHelper" ref="helper"></property>
      </bean>
      <bean id = "abstractLookupBean" class="com.model.AbstractLookupDemo">
      		<lookup-method name="getMyHelper" bean="helper"></lookup-method>
      </bean>
</beans>
```

加载配置文件：

```Java
package com.model;

import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.util.StopWatch;

public class Main {

	public static void main(String[] args) {
			
		AbstractApplicationContext context = new ClassPathXmlApplicationContext("lookBean.xml");
		context.registerShutdownHook();
		System.out.println("传递standardLookupBean");
		test(context, "standardLookupBean");
		System.out.println("传递AbstractLookupDemo");
		test(context, "abstractLookupBean");
	}
	
	public static void test(AbstractApplicationContext context,String beanName) {
		DemoBean bean = (DemoBean) context.getBean(beanName);
		MyHelper helper1 = bean.getHelper();
		MyHelper helper2 = bean.getHelper();
		System.out.println("测试"+beanName);
		System.out.println("两个helper是否相同?"+(helper1==helper2));
		StopWatch stopWatch = new StopWatch();
		stopWatch.start("lookupDemo");
		for (int i = 0; i < 10000; i++) {
			MyHelper helper = bean.getHelper();
			helper.doSomethingHelpful();
		}
		stopWatch.stop();
		System.out.println("获取10000次花费了"+stopWatch.getTotalTimeMillis()+"毫秒");
	}

}

```

结果:

![2017011421897Screen Shot 2017-01-14 at 4.54.47 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017011421897Screen Shot 2017-01-14 at 4.54.47 PM.png)

从上面的结果图看出，以前的方式生成的对象每次都是相同的。通过lookup方式注入每次是不同的。可以解决这种问题。但是有没有更简单的方式，感觉这种方式优点麻烦。

## 让`Bean`感知`Spring`容器

实现`BeanNameAware`,自定设置`id`值。

实现`BeanFactoryAware`,`ApplicationContextAware` 感知`Spring`容器。获取`Spring`容器。

## `Spring`国际化支持

### 配置配置文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
      <bean id="messageSource" class="org.springframework.context.support.ResourceBundleMessageSource">
      		<property name="basenames">
      			<list>
      				<value>message</value>
      			</list>
      		</property>
      </bean>
</beans>
```

### 新建中文配置文件

`message_zh_CN.properties`:

```xml
hello=welcome,{0}
now=now is : {0}
```



### 新建英文配置文件

`message_en_US.properties`:

```xml
hello=\u4F60\u597D,{0}
now=\u73B0\u5728\u7684\u65F6\u95F4\u662F : {0}
```



### 加载配置文件

```Java
package com.model;

import java.util.Date;
import java.util.Locale;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.util.StopWatch;

public class Main {

	public static void main(String[] args) {
		ApplicationContext context = new ClassPathXmlApplicationContext("globalization.xml");
		String[] a = {"读者"};
		String hello = context.getMessage("hello",a, Locale.CHINA);
		Object[] b = {new Date()};
		String now = context.getMessage("now",b, Locale.CHINA);
		System.out.println(hello);
		System.out.println(now);
		hello = context.getMessage("hello",a, Locale.US);
		now = context.getMessage("now",b, Locale.US);
		System.out.println(hello);
		System.out.println(now);
		}

}
```

### 结果

![2017011436101Screen Shot 2017-01-14 at 5.36.53 PM.png](http://7xk0q3.com1.z0.glb.clouddn.com/2017011436101Screen Shot 2017-01-14 at 5.36.53 PM.png)

好了，我要做通信项目去了。没时间解释了。