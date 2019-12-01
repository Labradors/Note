# Spring-boot学习


![](http://7xk0q3.com1.z0.glb.clouddn.com/changcheng.png)

前段时间刚刚把iOS基础学完，感觉没太大意思。现在开始玩微服务。在Java圈内，大家都知道，国外的Spring Cloud和国内的Dubbo是两大框架，不过Dubbo社区活跃度没有Spring Cloud这么高。而且严格来讲，Dubbo只能算是Spring Cloud服务管理的一部分。具体的情况等学完再来评论。当然，微服务还是选择Spring Cloud啊，Spring在Java Web的影响已经无需言语了。Spring boot只能算是微服务的一个小部分，慢慢来。与正常开发的SSM这些比较而言，Spring boot做了封装，简化了很多。并且配置文件也没那么一大串了，每次写项目copy配置文件都烦了。<!--more-->

## 创建项目

官方已经介绍的很好了。我们直接来建立项目。

### 加入maven依赖

```xml
<!-- Inherit defaults from Spring Boot -->
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
  </parent>

  <!-- Add typical dependencies for a web application -->
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
  </dependencies>

  <!-- Package as an executable jar -->
  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
```



### 创建Application

```java
@SpringBootApplication
public class Application {
  public static void main(String[] args){
    SpringApplication.run(Application.class,args);
  }
}
```



### 创建接口

```java
@Controller
@RequestMapping("/user/")
public class UserController {

  @RequestMapping(value = "info.do",method = RequestMethod.GET)
  @ResponseBody
  public User getUser(){
    User user = new User();
    user.setId(1);
    user.setUserName("哈哈哈");
    user.setPhone("15708443946");
    return user;
  }
}
```



### 试试看

用命令试试看: `mvn spring-boot:run  `

![20170626149840642970019.png](http://7xq3zt.com1.z0.glb.clouddn.com/20170626149840642970019.png)

有没有很爽，简直爽的不要不要的，再也没有一大堆的配置。干干净净。强迫症患者刚需。





## Mybatis支持

集成Mybatis简直累死我了，网上一大堆那种把xml和mapper接口写一起，看起来很不爽，毕竟这样展现不出mybatis的舒爽啊。想分离开，看了好久才配置正确。我们来配配

### 添加依赖

```xml
<dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>1.3.0</version>
    </dependency>

    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.21</version>
    </dependency>
```



### 配置application.properties

创建一个配置文件,加入mysql和mybatis的配置

```properties
# mybatis
# 配置model
mybatis.type-aliases-package=tech.jiangtao.boot.pojo
# 配置mapper xml文件目录
mybatis.mapper-locations=classpath:mappers/*.xml

# mysql
spring.datasource.driverClassName = com.mysql.jdbc.Driver
spring.datasource.url = jdbc:mysql://127.0.0.1:3306/mmall?useUnicode=true&characterEncoding=utf-8
spring.datasource.username = root
spring.datasource.password = root
```

### 加入扫描器扫描mapper

在启动类Application加入`@MapperScan("tech.jiangtao.boot.dao")`

### 开发Controller

```java
/**
   * 登录接口
   * @param username
   * @param password
   * @param session
   * @return
   */
  @RequestMapping(value = "login.do",method = RequestMethod.POST)
  @ResponseBody
  public ServerResponse<User> login(String username,String password,HttpSession session){
    ServerResponse<User> response = iUserService.login(username,password);
    // todo 将用户信息放入session中
    if (response.isSuccess()) {
      session.setAttribute(Const.CURRENT_USER,response.getData());
    }
    return response;
  }
```

### 开发Service

```java
@Override public ServerResponse<User> login(String username, String password) {
    int resultCount = userMapper.checkUserName(username);
    if (resultCount == 0) {
      return ServerResponse.createByErrorCodeMsg(400, "用户名不存在");
    }
    //todo 密码登录md5
    String md5Password = MD5Util.MD5EncodeUtf8(password);
    User user = userMapper.selectLogin(username, md5Password);
    if (user == null) {
      return ServerResponse.createByErrorCodeMsg(401, "密码错误");
    }
    // todo 在这儿就把置空，不要将密码发给移动端
    user.setPassword(StringUtils.EMPTY);
    return ServerResponse.createByMsgAndDataSuccess("登录成功", user);
  }
```

mapper和xml文件就不贴了，跟平常的ssm开发一模一样。最后看看返回的数据

### 结果

![20170626149841379962547.png](http://7xq3zt.com1.z0.glb.clouddn.com/20170626149841379962547.png)



是不是想抽根烟庆祝一下。

## Redis支持

## 总结

用了Spring boot就再也不想用那一堆堆配置的玩意了。简直神烦。实在太好用。下一节做一些源码的研究，然后直接进入Spring Cloud玩玩，看看真正的微服务怎么玩。刚刚接入大赏功能，求大赏。