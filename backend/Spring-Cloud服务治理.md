# Spring-Cloud服务治理

![20170627149857360736999.jpg](http://7xq3zt.com1.z0.glb.clouddn.com/20170627149857360736999.jpg)

前面写了一篇[Spring boot使用案例](http://blog.jiangtao.tech/2017/06/25/Spring%20boot%E5%AD%A6%E4%B9%A0(%E4%B8%80)/)。这一节来入门Spring Cloud Eureka服务治理。Spring Cloud Eureka使用的是Netflix Eureka，然后在其基础之上，对Spring boot做了二次封装。Eureka由两个组件组成：**Eureka**服务器**和Eureka客户端**。Eureka服务器用作服务注册服务器。Eureka客户端是一个java客户端，用来简化与服务器的交互、作为轮询负载均衡器，并提供服务的故障切换支持。并且Eureka客户端注册到Eureka服务注册中心后，会每30s发送一次心跳，如果几分钟步伐送心跳。那就会被服务中心移除。这篇文章主要记录Spring Cloud Eureka的注册中心搭建，服务中心搭建，高可用的注册中心以及服务的发现和消费。<!--more-->



## 客户端搭建

客户端搭建和上一章没什么大的不同，只是做一些配置和加一些依赖.

### maven依赖

```xml
<!-- Inherit defaults from Spring Boot -->
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
  </parent>


<!-- Add typical dependencies for a web application -->
  <dependencies>
    <!--spring mvc-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--spring boot test-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>

    <!--监控状况-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!--服务注册和服务发现-->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <!--spring boot-->
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>1.3.0</version>
    </dependency>
    
    
    
    <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Camden.SR5</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

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

### 配置

- 在Application上加入`@EnableEurekaClient`
- 声明一个application.properties文件，指定当前的版本`spring.profiles.active=dev`,我们这里指定的是开发版本。

![2017062814986587448974.png](http://7xq3zt.com1.z0.glb.clouddn.com/2017062814986587448974.png)

- 分别建立上边三个版本的文件，然后在里面配置如下:

```properties
spring.application.name=Base
server.port=8080

# eureka
 # 指定服务中心地址
eureka.client.service-url.defaultZone=http://127.0.0.1:8083/eureka
 # 指定可以使用ip地址访问
eureka.instance.prefer-ip-address=true
 # 指定eureka当前实例
eureka.instance.instance-id=${spring.application.name}:${spring.application.instance_id:${server.port}}
```

现在如果启动，会找不到我们指定的服务中心地址。我们先把服务中心写完在一起运行。



## 注册中心搭建

### 加入依赖

```xml
<!-- Inherit defaults from Spring Boot -->
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Camden.SR5</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>

    # 服务
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-eureka-server</artifactId>
    </dependency>

    <!--添加认证服务-->
    <!--<dependency>-->
      <!--<groupId>org.springframework.boot</groupId>-->
      <!--<artifactId>spring-boot-starter-security</artifactId>-->
    <!--</dependency>-->
  </dependencies>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
```



### 创建服务中心和配置

- 加入服务中心注解`@EnableEurekaServer`
- 同创建客户端一样添加配置文件

```properties
server.port=8083

# 不将自己作为一个服务去发现和注册
#eureka.client.register-with-eureka=false
#eureka.client.fetch-registry=false


logging.level.com.netflix.eureka=OFF
logging.level.com.netflix.discovery=OFF
# 申明自己的服务中心地址
eureka.client.serviceUrl.defaultZone=http://127.0.0.1:8083/eureka

# spring
spring.application.name=eureka
```

### 运行结果

我们打开服务中心地址：

![20170628149865946966999.png](http://7xq3zt.com1.z0.glb.clouddn.com/20170628149865946966999.png)

发现有一个实例在当前服务中心注册，并且在客户端每隔30s发送心跳到服务端。如下：

![20170628149865955678402.png](http://7xq3zt.com1.z0.glb.clouddn.com/20170628149865955678402.png)

## 高可用注册中心

上面我们讲的是单个服务中心的情况，很显然，这不靠谱。生产环境肯定要多个服务中心才行。其实多个服务中心就是大家相互注册，相互绑定。我们现在有两个服务中心，如下:

- `eureka.client.serviceUrl.defaultZone=http://127.0.0.1:8084/eureka`

- `eureka.client.serviceUrl.defaultZone=http://127.0.0.1:8083/eureka`

- 客户端指定两个服务中心：`eureka.client.service-url.defaultZone=http://127.0.0.1:8083/eureka,http://127.0.0.1:8084/eureka`

  然后分别启动这两个实例,我们打开两个服务中心。

![20170628149866007491197.png](http://7xq3zt.com1.z0.glb.clouddn.com/20170628149866007491197.png)





![20170628149866010867447.png](http://7xq3zt.com1.z0.glb.clouddn.com/20170628149866010867447.png)

服务中心分别注册了对方，并且，都可以检查到客户端的实例。最重要的要作为一个服务的配置。

```pro
# 将自己作为一个服务去发现
eureka.client.register-with-eureka=true
eureka.client.fetch-registry=true
```



## 服务的发现和消费

Spring Cloud 使用Ribbon来进行服务的负载均衡。在这里，只是提一下，顺便用代码实现一下，看一下效果，毕竟当前我也只学习到这。哈哈。废话不多说，本来写到这不太想写了。但还是好好写完。

###  添加消费者

#### 添加依赖 

```xml
<!-- Inherit defaults from Spring Boot -->
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.5.4.RELEASE</version>
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>
  </properties>

  <!-- Add typical dependencies for a web application -->
  <dependencies>
    <!--spring mvc-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--spring boot test-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
    </dependency>

    <!--监控状况-->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!--服务注册和服务发现-->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
  </dependencies>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Camden.SR5</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
      </plugin>
    </plugins>
  </build>
```

#### 配置

```properties
# spring
spring.application.name=consumer
server.port=9090
eureka.client.serviceUrl.defaultZone=http://127.0.0.1:8084/eureka
```



#### 添加主函数

```java
@SpringBootApplication
@EnableEurekaClient
public class App
{
    public static void main( String[] args )
    {
        SpringApplication.run(App.class,args);
    }

    @Bean
    @LoadBalanced
    RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

#### 编写Controller

```java
@RestController
public class ConsumerController {

  @Autowired
  RestTemplate restTemplate;

  @RequestMapping("/product")
  public String product(){
    return restTemplate.getForEntity("http://BASE/manage/product/detail.do?productId=26",String.class).getBody();
  }
}
```



### 启动

启动两个客户端实例，一个注册中心实例，一个消费者实例，像下面这样:

![20170628149866412381583.png](http://7xq3zt.com1.z0.glb.clouddn.com/20170628149866412381583.png)

然后我们访问一下刚才那个链接：

`http://127.0.0.1:9090/product?productId=26`

多访问几次，你就看到命令行这样：



![20170628149866426523105.png](http://7xq3zt.com1.z0.glb.clouddn.com/20170628149866426523105.png)





![20170628149866427758874.png](http://7xq3zt.com1.z0.glb.clouddn.com/20170628149866427758874.png)

这样负载均衡就实现了。我把客户端代码贴一下，免得一脸懵逼。

- Controller

```java
  @RequestMapping(value = "detail.do", method = RequestMethod.GET)
  @ResponseBody
  public ServerResponse getDetail(HttpSession session, Integer productId) {
    //User user = (User) session.getAttribute(Const.CURRENT_USER);
    //if (user == null) {
    //  return ServerResponse.createByErrorMsg("未登录");
    //}
    //if (iUserService.checkAdmin(user).isSuccess()) {
      // 获取产品详情
      return iProductService.getDetail(productId);
    //} else {
    //  return ServerResponse.createByErrorMsg("没有权限");
    //}
  }
```

- Service

```java
/**
   * 获取上商品详情
   * @param productId
   * @return
   */
  @Override public ServerResponse getDetail(Integer productId) {
    if (productId==null){
      return ServerResponse.createByErrorMsg("参数错误");
    }
    Product product = productMapper.selectByPrimaryKey(productId);
    if (product==null){
      return ServerResponse.createByErrorMsg("没有该商品信息");
    }
    ProductDetailVo vo = new ProductDetailVo();
    BeanUtils.copyProperties(product,vo);
    vo.setImageHost(PropertiesUtil.getProperty(Const.IMAGE_HOST));
    Category category = categoryMapper.selectByPrimaryKey(product.getCategoryId());
    if (category==null){
      vo.setParentCategoryId(0);
    }else {
      vo.setParentCategoryId(category.getParentId());
    }
    logger.info("正在获取产品信息....");
    return ServerResponse.createByDataSuccess(vo);
  }
```

完事，睡觉了。