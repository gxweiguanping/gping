---
title: springcloud
date: 2023-02-12 23:33:02
tags: 微服务
categories: 微服务
cover: https://gitee.com/studentgitee/note-picture/raw/master/md0024.png
---
![image-20230210143018927](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210143018927.png)

## 1.微服务介绍

### 1.1系统架构演变

随着互联网的发展，网站应用的规模也在不断的扩大，进而导致系统架构也在不断的进行变化。从互联网早起到现在，系统架构大体经历了下面几个过程: 单体应用架构—>垂直应用架构—>分布式架构—>SOA架构—>微服务架构，当然还有悄然兴起的Service Mesh(服务网格化)。

#### 1.1.1 单体应用架构

互联网早期，一般的网站应用流量较小，只需一个应用，将所有功能代码都部署在一起就可以，这样可以减少开发、部署和维护的成本。
比如说一个电商系统，里面会包含很多用户管理，商品管理，订单管理，物流管理等等很多模块，我们会把它们做成一个web项目，然后部署到一台tomcat服务器上。

![image-20210923145216353](https://gitee.com/studentgitee/note-picture/raw/master/image-20210923145216353.png)

**优点：**
项目架构简单，小型项目的话， 开发成本低
项目部署在一个节点上， 维护方便
**缺点：**
全部功能集成在一个工程中，对于大型项目来讲不易开发和维护
项目模块之间紧密耦合，单点容错率低
无法针对不同模块进行针对性优化和水平扩展

#### 1.1.2 垂直应用架构

随着访问量的逐渐增大，单一应用只能依靠增加节点来应对，但是这时候会发现并不是所有的模块都会有比较大的访问量，还是以上面的电商为例子， 用户访问量的增加可能影响的只是用户和订单模块， 但是对消息模块的影响就比较小. 那么此时我们希望只多增加几个订单模块， 而不增加消息模块. 此时单体应用就做不到了， 垂直应用就应运而生了.

所谓的垂直应用架构，就是将原来的一个应用拆成互不相干的几个应用，以提升效率。比如我们可以将上面电商的单体
应用拆分成:
电商系统(用户管理 商品管理 订单管理)
后台系统(用户管理 订单管理 客户管理)
CMS系统(广告管理 营销管理)
这样拆分完毕之后，一旦用户访问量变大，只需要增加电商系统的节点就可以了，而无需增加后台和CMS的节点。

![image-20210923145409503](https://gitee.com/studentgitee/note-picture/raw/master/image-20210923145409503.png)

**优点：**
系统拆分实现了流量分担，解决了并发问题，而且可以针对不同模块进行优化和水扩展一个系统的问题不会影响到其他系统，提高容错率
**缺点：**
系统之间相互独立， 无法进行相互调用
系统之间相互独立， 会有重复的开发任务

#### 1.1.3 分布式架构

当垂直应用越来越多，重复的业务代码就会越来越多。这时候，我们就思考可不可以将重复的代码抽取出来，做成统一的业务层作为独立的服务，然后由前端控制层调用不同的业务层服务呢？这就产生了新的分布式系统架构。它将把工程拆分成表现层和服务层两个部分，服务层中包含业务逻辑。表现层只需要处理和页面的交互，业务逻辑都是调用服务层的服务来实现。



![image-20210923145435964](https://gitee.com/studentgitee/note-picture/raw/master/image-20210923145435964.png)

**优点：**
抽取公共的功能为服务层，提高代码复用性
**缺点：**
系统间耦合度变高，调用关系错综复杂，难以维护

#### 1.1.4 SOA架构

在分布式架构下，当服务越来越多，容量的评估，小服务资源的浪费等问题逐渐显现，此时需增加一个调度中心对集群进行实时管理。此时，用于资源调度和治理中心(SOA Service OrientedArchitecture)是关键。

![image-20210923145454206](https://gitee.com/studentgitee/note-picture/raw/master/image-20210923145454206.png)

**优点:**
使用治理中心（ESB\dubbo）解决了服务间调用关系的自动调节
**缺点:**
服务间会有依赖关系，一旦某个环节出错会影响较大( 服务雪崩 )
服务关系复杂，运维、测试部署困难

#### 1.1.5 微服务架构

微服务架构在某种程度上是面向服务的架构SOA继续发展的下一步，它更加强调服务的"彻底拆分"。

![image-20210923145512582](https://gitee.com/studentgitee/note-picture/raw/master/image-20210923145512582.png)

**微服务架构与SOA架构的不同：**

1、微服务架构比 SOA架构粒度会更加精细，让专业的人去做专业的事情（专注），目的提高效率，每个服务于服务之间互不影响，微服务架构中，每个服务必须独立部署，微服务架构更加轻巧，轻量级。

2、SOA 架构中可能数据库存储会发生共享，微服务强调独每个服务都是单独数据库，保证每个服务于服务之间互不影响。项目体现特征微服务架构比 SOA 架构更加适合与互联网公司敏捷开发、快速迭代版本，因为粒度非常精细。
**优点：**
服务原子化拆分，独立打包、部署和升级，保证每个微服务清晰的任务划分，利于扩展
微服务之间采用Restful等轻量级http协议相互调用
**缺点：**
分布式系统开发的技术成本高（容错、分布式事务等）
复杂性更高。各个微服务进行分布式独立部署，当进行模块调用的时候，分布式将会变得更加麻烦。

### 1.2微服务架构介绍

#### 1.2.1微服务中文网址

```
http://blog.cuicc.com/blog/2015/07/22/microservices
```

#### 1.2.2 微服务架构的常见问题

- 一旦采用微服务系统架构，就势必会遇到这样几个问题：
- 这么多小服务，如何管理他们？(服务治理 注册中心[服务注册 发现 剔除])    eureka、 nacos
- 这么多小服务，他们之间如何通讯？(restful rpc dubbo feign)    httpclient("url",参数)，  springBootrestTemplate("url",参数) ,feign
- 这么多小服务，一旦出现问题了，应该如何自处理？(容错)     sentinel
- 这么多小服务，客户端怎么访问他们？(网关)     zuul、gateway
- 这么多小服务，一旦出现问题了，应该如何排错? (链路追踪)  sleuth+zipkin
- 对于上面的问题，是任何一个微服务设计者都不能绕过去的，因此大部分的微服务产品都针对每一个问题提供了相应的组件来解决它们。

![image-20210923150451194](https://gitee.com/studentgitee/note-picture/raw/master/image-20210923150451194.png)

#### 1.2.3 常见微服务架构

```
1.dubbo: zookeeper +dubbo + SpringMVC/SpringBoot
配套 通信方式：rpc
注册中心：zookeeper / redis
配置中心：diamond

2.SpringCloud：全家桶+轻松嵌入第三方组件(Netflix)
配套 通信方式：http restful
注册中心：eruka / consul
配置中心：config
断 路 器：hystrix
网关：zuul
分布式追踪系统：sleuth + zipkin

3.SpringCloud Alibaba
```

## 2.springCloud搭建

官网地址：https://spring.io/projects/spring-cloud

包括了诸如下列功能：

- 服务注册发现Eureka
- 统一配置中心 SpringCloudConfig
- 服务网关 Zuul
- 服务降级容错 Hystrix
- 服务调用跟踪

![image-20210926093528808](https://gitee.com/studentgitee/note-picture/raw/master/image-20210926093528808.png)



![v2-25915a395592b4a88a3a35dff89ab969_r](https://gitee.com/studentgitee/note-picture/raw/master/v2-25915a395592b4a88a3a35dff89ab969_r.jpg)



### 2.1版本要求

> Spring Cloud 的版本号并不是我们通常见的数字版本号，而是一些很奇怪的单词。这些单词均为英国伦敦地铁站的站名。同时根据字母表的顺序来对应版本时间顺序，比如：最早 的 Release 版本 Angel，第二个 Release 版本 Brixton（英国地名），然后是 Camden、 Dalston、Edgware、Finchley、Greenwich(格林威治)、Hoxton。

![image-20210925114602935](https://gitee.com/studentgitee/note-picture/raw/master/image-20210925114602935.png)

我用的版本如下：

- spring-boot 2.1.6.RELEASE
- spring-cloud Greenwich.RELEASE

### 2.2创建父工程

1、创建父工程

![image-20210925115847392](https://gitee.com/studentgitee/note-picture/raw/master/image-20210925115847392.png)

2、选择Maven POM

![image-20210925115752951](https://gitee.com/studentgitee/note-picture/raw/master/image-20210925115752951.png)

3、引入依赖

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <packaging>pom</packaging>
    <modules>
        <module>eureka-server</module>
    </modules>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.wgp</groupId>
    <artifactId>springcloud</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springcloud</name>
    <description>Demo project for Spring Boot</description>
    <properties>
        <java.version>1.8</java.version>
        <spring.cloud-version>Greenwich.RELEASE</spring.cloud-version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
        <!-- springcloud版本管理-->
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring.cloud-version}</version>
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

</project>

```

## 3.服务注册中心

### 3.1什么是服务注册中心

> 所谓服务注册中心就是在整个的微服务架构中单独提出一个服务, 这个服务不完成系统的任何的业务功能,仅仅用来完成对整个微服务系统的服务注册和服务发现,以及对服务健康状态的监控和管理功能。

![image-20211008112843035](https://gitee.com/studentgitee/note-picture/raw/master/image-20211008112843035.png)



```
# 1.服务注册中心
一  可以对所有的微服务的信息进行存储，如微服务的名称、IP、 端口等
一  可以在进行服务调用时通过服务发现查询可用的微服务列表及网络地址进行服务调用
一  可以对所有的微服务进行心跳检测，如发现某实例长时间无法访问，就会从服务注册表移除该实例。
```



### 3.2eureka

#### **eureka服务端**

eureka官网地址：

https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.1.RELEASE/reference/html/#service-discovery-eureka-clients

> eureka包含eureka-server和eureka-client



a、创建Maven子工程后引入依赖

```
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud</artifactId>
        <groupId>com.wgp</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>eureka-server</artifactId>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <!--  eureka-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>
</project>
```

b、编写配置文件application.yml

```
server:
  port: 10001
spring:
  application:
    name: eureka-server
eureka:
  client:
    service-url:
      defaultZone: http://localhost:10001/eureka  #eureka访问地址http://localhost:10001
    fetch-registry: false #启动完成后再进行注册
    register-with-eureka: false #eureka当做服务注册中心使用，自身不进行注册

```

c、编写启动类加上注解

**@EnableEurekaServer**和 @SpringBootApplication

```
@SpringBootApplication
@EnableEurekaServer
public class EurekaWgpApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaWgpApplication.class, args);
    }
}
```

d、服务注册中心搞定，启动成功后访问[http://localhost:10001，可以看到注册中心页面：

![image-20210925115641029](https://gitee.com/studentgitee/note-picture/raw/master/image-20210925115641029.png)



#### **eureka客户端**

a、引入依赖

```
<!--  eurekaclient-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

b、编写配置文件application.yml指定eureka

```
server:
  port: 端口号
spring:
  application:
    name: 服务名称
eureka:
  client:
    service-url:
      defaultZone: http://localhost:10001/eureka  #eureka访问地址http://localhost:10001
```

c、编写启动类加上注解

```
@EnableFeignClients和 @SpringBootApplication
```



#### eureka自我保护机制

官方解释：https://github.com/Netflix/eureka/wiki/Understanding-Eureka-Peer-to-Peer-Communication

> Eureka Server 在运行期间会去统计心跳失败比例在 15 分钟之内是否低于 85%，如果低于 85%，Eureka Server 会将这些实例保护起来，让这些实例不会过期，但是在保护期内如果服务刚好这个服务提供者非正常下线了，此时服务消费者就会拿到一个无效的服务实例，此时会调用失败，对于这个问题需要服务消费者端要有一些容错机制，如重试，断路器等。



Eureka Server自动进入自我保护机制，此时会出现以下几种情况：

- Eureka Server不再从注册列表中移除因为长时间没收到心跳而应该过期的服务。
- Eureka Server仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上，保证当前节点依然可用。
- 当网络稳定时，当前Eureka Server新的注册信息会被同步到其它节点中。



1、 注册中心关闭自我保护机制，修改检查失效服务的时间。

```text
eureka:
  server: 
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 3000
```

2、 微服务修改减短服务心跳的时间。

```text
# 默认90秒
lease-expiration-duration-in-seconds: 10
# 默认30秒
lease-renewal-interval-in-seconds: 3
```

3、以上配置建议在生产环境使用。



## 4.服务间的通讯

java 项目中如何实现接口调用？

**1）Httpclient**
HttpClient 是 Apache Jakarta Common 下的子项目，用来提供高效的、最新的、功能丰富的支持 Http 协议的客户端编程工具包，并且

它支持 HTTP 协议最新版本和建议。HttpClient相比传统 JDK 自带的 URLConnection，提升了易用性和灵活性，使客户端发送 HTTP 请求

变得容易，提高了开发的效率。



**2）Okhttp**
一个处理网络请求的开源项目，是安卓端最火的轻量级框架，由 Square 公司贡献，用于替代HttpUrlConnection 和 Apache 

HttpClient。OkHttp 拥有简洁的 API、高效的性能，并支持多种协议（HTTP/2 和 SPDY）。



**3）HttpURLConnection**

HttpURLConnection 是 Java 的标准类，它继承自 URLConnection，可用于向指定网站发送GET 请求、POST 请求。

HttpURLConnection 使用比较复杂，不像 HttpClient 那样容易使用。



**4）RestTemplate    WebClient**

RestTemplate 是 Spring 提供的用于访问 Rest 服务的客户端，RestTemplate 提供了多种便捷访问远程 HTTP 服务的方法，能够大大提

高客户端的编写效率。

### 4.1基于RestTemplate服务调用

**简介**

> RestTemplate 是从 Spring3.0 开始支持的一个 HTTP 请求工具，它提供了常见的REST请求方案的模版，例如 GET 请求、POST 请求、PUT 请求、DELETE 请求以及一些通用的请求执行方法 exchange 以及 execute。RestTemplate 继承自InterceptingHttpAccessor 并且实现了 RestOperations 接口，其中 RestOperations 接口定义了基本的 RESTful 操作，这些操作在 RestTemplate 中都得到了实现。接下来我们就来看看这些操作方法的使用。



**实例**

**1、编写两个服务 user-server 和order-server**

![image-20210925132430566](https://gitee.com/studentgitee/note-picture/raw/master/image-20210925132430566.png)

**2、order-server服务的搭建**

**引入依赖**

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud</artifactId>
        <groupId>com.wgp</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>order-server</artifactId>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <!--  eureka-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
</project>
```

**编写配置文件application.yml**

```
server:
  port: 8002
spring:
  application:
    name: order-server
eureka:
  client:
    service-url:
      defaultZone: http://localhost:10001/eureka  #eureka地址     访问地址http://localhost:10001

```

**编写启动类**

```
@SpringBootApplication
@EnableEurekaClient 
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```

**编写Controller**

```
@RestController
public class OrderController {

    @Value("${server.port}")
    private String port;

    @GetMapping
    public String test(){
        return "order is ok,调用的端口为："+port;
    }

}
```

**3、user-server服务的搭建**

**引入依赖**

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloud</artifactId>
        <groupId>com.wgp</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>user-server</artifactId>
<dependencies>
    <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
    <!--  eureka-->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>

</project>
```

**编写配置文件application.yml**

```
server:
  port: 8001
spring:
  application:
    name: user-server
eureka:
  client:
    service-url:
      defaultZone: http://localhost:10001/eureka  #eureka地址     访问地址http://localhost:10001

```

**编写启动类**

```
@SpringBootApplication
@EnableEurekaClient
public class UserApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class, args);
    }
}
```

**编写Controller**

```
@RestController
public class UserController {

    @GetMapping("/user")
    public String invokeTest(){
        // 使用RestTemplate调用order服务
        RestTemplate restTemplate = new RestTemplate();
        String invoke = restTemplate.getForObject("http://localhost:8002", String.class);
        return "user调用服务成功，返回值为:"+invoke;
    }
}

```

**4、访问发现调用成功**

![image-20210925133234304](https://gitee.com/studentgitee/note-picture/raw/master/image-20210925133234304.png)

**5、存在的问题**

- 调用的地址写死
- 当服务是集群的时候，不能实现负载均衡



### 4.2基于Ribbon的负载均衡

官网地址：https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.1.RELEASE/reference/html/#spring-cloud-ribbon

#### **简介**

Spring Cloud Ribbon是一个基于HTTP和TCP的**客户端**负载均衡工具，它基于Netflix Ribbon实现。通过Spring Cloud的封装，可以让我

们轻松地将面向服务的REST模版请求自动转换成客户端负载均衡的服务调用。



![image-20210924094050229](https://gitee.com/studentgitee/note-picture/raw/master/image-20210924094050229.png)

**服务器端负载均衡**

例如Nginx，通过Nginx进行负载均衡，先发送请求，然后通过负载均衡算法，在多个服务器之间选择一个进行访问；即在服务器端再进

行负载均衡算法分配。



![image-20210924094125748](https://gitee.com/studentgitee/note-picture/raw/master/image-20210924094125748.png)

#### 使用Riobbin

**！！！如果是使用了eureka,那么就不需要引入Riobbin依赖**

1、编写配置类，创建一个RestTemplate

```
package com.wgp.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

@Configuration
public class BeanConfig {

    @Bean
    @LoadBalanced
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

```

2、使用Ribbon实现负载均衡 三种方式DiscoveryClient、LoadBalancerClient、LoadBalanced

a、DiscoveryClient  

- 服务发现客户端对象，根据服务id去服务注册中心获取对应服务列表到本地中
- 缺点：没有负载均衡，需要自己实现

b、LoadBalancerClient

- 负载均衡客户端对象，根据服务id去服务注册中心获取对应服务列表到本地中,根据默认负载均衡策略选择列表中的一台机器进行返回
- 缺点：使用时每次都要根据服务id去获取一个负载均衡机器之后再通过restTemplate调用服务

c、LoadBalanced注解

- 作用：让当前对象具有ribbon负载均衡特性
- ![image-20210925153117802](https://gitee.com/studentgitee/note-picture/raw/master/image-20210925153117802.png)



d、以上3中方法虽然解决了负载均衡问题，但是还是不能解决路径写死的问题，不利于我们进行维护

```
// 路径写死问题
String invoke = restTemplate.getForObject("http://"+"order-server" + "/order", String.class);
```

e、代码实现

```
package com.wgp.controller;


import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.cloud.client.loadbalancer.LoadBalancerClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.client.RestTemplate;



@RestController
public class UserController {

    @Autowired
    private RestTemplate restTemplate;
    @Autowired
    private DiscoveryClient discoveryClient;
    @Autowired
    private LoadBalancerClient loadBalancerClient;

    @GetMapping("/user")
    public String invokeTest() {
        RestTemplate restTemplate = new RestTemplate();
        String invoke = restTemplate.getForObject("http://localhost:8002/order", String.class);
        return "user调用服务成功，返回值为:" + invoke;
    }

    @GetMapping("/ribbon")
    public String invokeDemo() {
        // 实现负载均衡 DiscoveryClient、LoadBalancerClient、LoadBalanced
        // DiscoveryClient 方式 需要手动实现负载均衡
//        List<ServiceInstance> instances = discoveryClient.getInstances("order-server");
//        String invoke = restTemplate.getForObject(instances.get(0).getUri()+"/order", String.class);

        // LoadBalancerClient 方式
//        ServiceInstance choose = loadBalancerClient.choose("order-server");
//        String invoke = restTemplate.getForObject(choose.getUri() + "/order", String.class);

        // LoadBalanced 方式，需要在配置类里面加上 @LoadBalanced
        String invoke = restTemplate.getForObject("http://"+"order-server" + "/order", String.class);


        return "user调用服务成功，返回值为:" + invoke;
    }
}

```

#### 源码解析

a、我们可以通过choose作为入口进行分析

```
ServiceInstance choose = loadBalancerClient.choose("order-server");
```

b、我们可以发现ServiceInstanceChooser是LoadBalancerClient父接口

c、查看RibbonLoadBalancerClient![image-20210925155530537](https://gitee.com/studentgitee/note-picture/raw/master/image-20210925155530537.png)

d、在RibbonLoadBalancerClient有一个choose方法带有两个参数这里面进行负载均衡实现

![image-20210925155708382](https://gitee.com/studentgitee/note-picture/raw/master/image-20210925155708382.png)

e、查看getServer方法

![image-20210925155828682](https://gitee.com/studentgitee/note-picture/raw/master/image-20210925155828682.png)

f、通过源码可以知道IRule是底层负载均衡接口

![image-20210925155332008](https://gitee.com/studentgitee/note-picture/raw/master/image-20210925155332008.png)

#### 常见负载均衡算法

```
# 1.ribbon负载均衡算法

- RoundRobinRule   轮训策略按顺序循环选择Server

- RandomRule    随机策略随机选择server

- AvailabilityFilteringRule  可用过滤策略会先过滤由于多次访问故障而处于断路器跳闸状态的服务，还有并发的连接数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问

- weightedResponseTimeRule  响应时间加权策略，根据平均响应的时间计算所有服务的权重，响应时间越快服务权重越大被选中的概率越高，刚启动时如果统计信息不足，则使用

- RoundRobinRule  策略,等统计信息足够会切换到

- RetryRule   重试策略先按照RoundRobinRule的策略获取服务，如果获取失败则在制定时间内进行重试，获取可用的服务。

- BestAviabTeRule    最低并发策略会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务

```

#### 修改默认负载均衡策略

！！！需要在调用方的配置文件里面写

```
properties写法：
服务id.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RandomRule

yaml写法
服务id:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```



## 5.OpenFeign组件

### 5.1什么是Feign

- Feign是Netflix开发的声明式、模板化的HTTP客户端，其灵感来自Retrofit、JAXRS-2.0以及WebSocket。Feign可帮助我们更加便捷、优雅地调用HTTP API。Feign支持多种注解，例如Feign自带的注解或者JAX-RS注解等。
- Spring Cloud  openfeign 对Feign进行了增强，使其支持Spring MVC注解，另外还整合了Ribbon和Nacos，从而使得Feign的使用更加方便
- 它是一个伪http，实质是对RestTemplate进行再封装，解决RestTemplate的问题

| 物理层 | 数据链路层 | 网络层 | 传输层 | 会话层 | 表示层 | 应用层 |
| :----: | :--------: | :----: | :----: | :----: | :----: | :----: |
|        |            |        |  RPC   |        |        |  http  |

**优势**

Feign可以做到使用 HTTP 请求远程服务时就像调用本地方法一样的体验，开发者完全感知不到这是远程方法，更感知不到这是个 HTTP 

请求。它像 Dubbo 一样，consumer 直接调用接口方法调用 provider，而不需要通过常规的 Http Client 构造请求再解析返回数据。它

解决了让开发者调用远程接口就跟调用本地方法一样，无需关注与远程的交互细节，更无需关注分布式环境开发。

### 5.2快速使用OpenFeign

```java
openfeign的负载均衡可以看 FeignRibbonClientAutoConfiguration
```

1、调用端引入依赖

```
<!-- openfeign-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```



2、调用端编写调用接口+@FeignClient注解

```
@FeignClient(value = "cms-user-service", path = "/wgp")  #value 被调用端的服务名称   #path  调用的路径
public interface CmsUserFeignClient {
    @GetMapping("/test")
    String test(@RequestParam("name")String name);
}
```



3、调用端在启动类上添加@EnableFeignClients注解

```
@SpringBootApplication
@EnableFeignClients
public class WgpOpenfeignApplication {
    public static void main(String[] args) {
        SpringApplication.run(WgpOpenfeignApplication.class, args);
    }
}
```



4、发起调用，像调用本地方式一样调用远程服务

```
@RestController
@RequestMapping("/openfeign")
public class UsreController {
    @Autowired
    private CmsUserFeignClient cmsUserFeignClient;

    @GetMapping("/test")
    public String test(@RequestParam("name")String name) {
        return cmsUserFeignClient.test(name);
    }
}
```

 

### 5.3Openfeign超时处理

在使用 openFeign  进行服务间调用时，由于业务代码复杂可能导致响应时间会比较久，提示 read timeout，解决方案如下

```
#配置指定服务设置超时时间
feign:
  client:
    config:
      order-server:
        connectTimeout: 5000   #连接时间
        readTimeout: 6000 #等待时间       
```

```
#配置所有服务设置超时时间
feign:
  client:
    config:
      default:
        connectTimeout: 5000   #连接时间
        readTimeout: 6000 #等待时间
```



## 6.微服务网关

### 6.1微服务存在的问题

每个业务都会需要鉴权、限流、权限校验、跨域等逻辑，如果每个业务都各自为战，自己造轮子实现一遍，会很麻烦，完全可以抽出来，

放到一个统一的地方去做。如果业务量比较简单的话，这种方式前期不会有什么问题，但随着业务越来越复杂，比如淘宝、亚马逊打开一

个页面可能会涉及到数百个微服务协同工作，如果每一个微服务都分配一个域名的话，一方面客户端代码会很难维护，涉及到数百个域

名，另一方面是连接数的瓶颈，想象一下你打开一个APP，通过抓包发现涉及到了数百个远程调用，这在移动端下会显得非常低效。

后期如果需要对微服务进行重构的话，也会变的非常麻烦，需要客户端配合你一起进行改造，比如商品服务，随着业务变的越来越复杂，

后期需要进行拆分成多个微服务，这个时候对外提供的服务也需要拆分成多个，同时需要客户端配合你进行改造，非常麻烦。

### 6.2为什么使用网关？

- 单体应用:浏览器发起请求到单体应用所在的机器，应用从数据库查询数据原路返回给浏览器,对于单体应用来说是不需要网关的。
- 微服务:微服务的应用可能部署在不同机房,不同地区，不同域名下.此时客户端(浏览器/手机/软件工具)想要请求对应的服务,都需要知道机器的具体IP或者域名URL，当微服务实例众多时，这是非常难以记忆的，对于客户端来说也太复杂难以维护。此时就有了网关，客户端相关的请求直接发送到网关，由网关根据请求标识解析判断出具体的微服务地址，再把请求转发到微服务实例。这其中的记忆功能就全部交由网关来操作了。



![image-20210926094539562](https://gitee.com/studentgitee/note-picture/raw/master/image-20210926094539562.png)





### 6.3zuul网关

#### 简介

官网地址：https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.1.RELEASE/reference/html/#router-and-filter-zuul

Zuul是从设备和网站到应用程序后端的所有请求的前门。作为边缘服务应用程序，Zuul 旨在实现动态路由，监视，弹性和安
全性。Zuul 包含了对请求的路由和过滤两个最主要的功能。

Zuul是Netlix开源的微服务网关，它可以和Eureka. Ribbon. Hystrix 等组件配合使用。Zuul 的核心是一系列的过滤器， 这些

过滤器可以完成以下功能:

●身份认证与安全:识别每个资源的验证要求，并拒绝那些与要求不符的请求

●审查与监控:在边缘位置追踪有意义的数据和统计结果，从而带来精确的生产试图

●动态路由:动态地将请求路由到不同的后端集群

●压力测试:逐渐增加只想集群的流量，以了解性能

●负载分配:为每一种负载类型分配对应容量，并弃用超出限定值的请求

●静态响应处理:在边缘位置直接建立部份响应，从而避免其转发到内部集群

●多区域弹性:跨越AWS Region进行请求路由,旨在实现ELB (Elastic Load Balancing) 使用的多样化,以及让系统的边缘更贴近系统的使用者

#### 使用Zuul

**我们在使用zuul的同时，一般跟eureka一起整合使用，另外zuul本身已经实现了负载均衡的功能，默认是轮训方式**

a、引入依赖

```
 <dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
 </dependency>
```

b、编写启动类

```
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.cloud.netflix.zuul.EnableZuulProxy;

@SpringBootApplication
@EnableEurekaClient
@EnableZuulProxy
public class ZullServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ZullServerApplication.class, args);
    }
}
```

c、编写配置文件application.yml

```
zuul:
  routes:
    order-server:  # 服务名称
      path: /order/** # 匹配路径
      serviceId: order-server # 服务名称
```



#### 路由排除

##### url排除

```
zuul:
  ignored-patterns: /**/order/**
```

##### 服务名称排除

```
zuul:
  ignored-services: order-server  #服务名称，多用服务需要时候逗号隔开
```



#### 路由前缀

```
zuul:
  prefix: /api  #路由前缀,访问时要加上此前缀
  stripPrefix: true #代理前缀默认会从请求路径中移除，通过该设置关闭移除功能， 

#比如 http://127.0.0.1:9999/api/order-server/order
当stripPrefix: true      请求路径为：http://127.0.0.1:9999/order-server/order    api会移除
当stripPrefix: false     请求路径为：http://127.0.0.1:9999/api/order-server/order   api不会移除
```





#### 网关过滤器

![image-20210926104843817](https://gitee.com/studentgitee/note-picture/raw/master/image-20210926104843817.png)

##### 关键名词

- 类型：定义路由流程中应用过滤器的阶段。共pre、 routing、post、error4个类型。
- 执行顺序：在同类型中，定义过滤器执行的顺序。比如多个pre类型的执行顺序。
- 条件：执行过滤器所需的条件。true开启，false关闭。
- 动作：如果符合条件，将执行的动作。具体操作。

##### 过滤器类型

- pre：请求被路由到源服务器之前执行的过滤器

1. 身份认证
2. 选路由
3. 请求日志

- routing：处理将请求发送到源服务器的过滤器
- post：响应从源服务器返回时执行的过滤器

1. 对响应增加HTTP头
2. 收集统计和度量指标
3. 将响应以流的方式发送回客户端

- error：上述阶段中出现错误时执行的过滤器

### 

### 6.4Gateway网关

#### 简介

- 网关作为流量的入口，常用的功能包括路由转发，权限校验，限流等。

- Spring Cloud Gateway 是Spring Cloud官方推出的第二代网关框架，定位于取代 Netflix Zuul1.0。相比 Zuul 来说，Spring Cloud

  Gateway 提供更优秀的性能，更强大的有功能。

- Spring Cloud Gateway 是由 WebFlux + Netty + Reactor 实现的响应式的 API 网关。它不能在传统的 servlet 容器中工作，也不能构

  建成 war 包。

- Spring Cloud Gateway 旨在为微服务架构提供一种简单且有效的 API 路由的管理方式，并基于 Filter 的方式提供网关的基本功能，

  例如说安全认证、监控、限流等等。

- 其他的网关组件：

  在SpringCloud微服务体系中，有个很重要的组件就是网关，在1.x版本中都是采用的Zuul网关；但在2.x版本中，zuul的升级一直跳

  票，SpringCloud最后自己研发了一个网关替代Zuul，那就是SpringCloud Gateway

#### Gateway功能特征

- 基于Spring Framework 5, Project Reactor 和 Spring Boot 2.0 进行构建；
- 动态路由：能够匹配任何请求属性；
- 支持路径重写;
- 集成 Spring Cloud 服务发现功能（Nacos、Eruka）；
- 可集成流控降级功能（Sentinel、Hystrix）；
- 可以对路由指定易于编写的 Predicate（断言）和 Filter（过滤器）；

#### 核心概念

**路由（route)**

路由是网关中最基础的部分，路由信息包括一个ID、一个目的URI、一组断言工厂、一组Filter组成。如果断言为真，则说明请求的URL和

配置的路由匹配。



**断言(predicates)**

Java8中的断言函数，SpringCloud Gateway中的断言函数类型是Spring5.0框架中的ServerWebExchange。断言函数允许开发者去定义

匹配Http request中的任何信息，比如请求头和参数等。



**过滤器（Filter)**

SpringCloud Gateway中的filter分为Gateway FilIer和Global Filter。Filter可以对请求和响应进行处理。

#### 集成gateway网关

1、引入依赖

```
<!-- spring-cloud网关-->
<dependency>
 <groupId>org.springframework.cloud</groupId>
 <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```



2、bootstrap.yml文件

```
server:
  port: 8888
spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      globalcors:   # 跨域配置
        cors-configurations:
          '[/**]':   # 允许跨域访问的资源
            allowedOrigins: "*"   #跨域允许来源
            allowedMethods:
              - GET
              - POST
      routes:
        - id: cms-user-service  # 路由的唯一标识，路由到order
          uri: lb://cms-user-service     #需要转发的地址   lb: 使用nacos中的本地负载均衡策略cms-user-service服务名
          predicates:    #断言规则 用于路由规则的匹配
            - Path=/wgp/**
        - id: wgp-openfeign
          uri: lb://wgp-openfeign
          predicates:
            - Path=/openfeign/**
    nacos:
#      config:
#        file-extension: yaml   #配置文件后缀
#        group: DEFAULT_GROUP   #组
#        server-addr: ${spring.cloud.nacos.server-addr}
      discovery:
        server-addr: 192.168.1.28:8848
      server-addr: 192.168.1.28:8848


```



## 7.Hystrix断路器

官方网址：https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.1.RELEASE/reference/html/#circuit-breaker-hystrix-clients

### 7.1服务雪崩

**在微服务之间进行服务调用是由于某一个服务故障， 导致级联服务故障的现象，称为雪崩效应。雪崩效应描述的是提供方不可用，导致**

**消费方不可用并将不可用逐渐放大的过程。**

![image-20210926112033190](https://gitee.com/studentgitee/note-picture/raw/master/image-20210926112033190.png)



### 7.2服务熔断

“熔断器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器(hystrix)的故障监控， 某个异常条件被触发，直接熔断整个服

务。向调用方法返回一个符合预期的、可处理的备选响应，而不是长时间的等待或者抛出调用方法无法处理的异常，就保证了服务调用方

的线程不会被长时间占法返回一个符合预期的、可处理的备选响应，而不是长时间的等待或者抛出调用方法无法处理的异常，就保证了服

务调用方的线程不会被长时间占用，避免故障在分布式系统中蔓延，乃至雪崩。如果目标服务情况好转则恢复调用。服务熔断是解决服务

雪崩的重要手段。**（熔断器相当于保险丝，相对于被调用端来说是服务熔断）**


**熔断条件：**

1、当对特定服务的10s内20次请求失败

2、10s内超过50%的请求失败



![image-20210926113616477](https://gitee.com/studentgitee/note-picture/raw/master/image-20210926113616477.png)



![image-20210927172522562](https://gitee.com/studentgitee/note-picture/raw/master/image-20210927172522562.png)



### 7.3服务降级

服务压力剧增的时候根据当前的业务情况及流量对一些服务和页面有策略的降级，以此缓解服务器的压力，以保证核心任务的进行。同时

保证部分甚至大部分任务客户能得到正确的响应。也就是当前的请求处理不了或者出错了，给一个默认的返回。**(相对于调用端来说是服务降级)**



### 7.4整合Hystrix

a、引入依赖

```
<!-- hystrix依赖 -->
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

b、在启动类上添加@EnableCircuitBreaker注解，代表开启服务熔断

```
@SpringBootApplication
@EnableCircuitBreaker
@EnableEurekaClient
public class HystrixServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(HystrixServerApplication.class, args);
    }
}

```

c、分两种方式实现服务熔断，第一种是自定义返回错误提示，第二种是默认返回错误提示

```
package com.wgp.controller;

import com.netflix.hystrix.contrib.javanica.annotation.HystrixCommand;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HystrixController {

    /**
     * 第一种：指定返回方法，返回方式参数列表和返回值需要和开启服务熔断的方法保持一致
     * @param id
     * @return
     */
    @GetMapping("/hystrix")
    @HystrixCommand(fallbackMethod = "hystrixFallbackMethod")
    public String hystrix(Integer id){
       if(id<0){
           throw new RuntimeException("id值小于0");
       }
       return "success："+id;
    }
    public String hystrixFallbackMethod(Integer id){
      return "活动火爆，请稍后重试！！！！";
    }

    /**
     * 第二种：默认返回方法，默认方法只需要返回错误结果即可
     * @param id
     * @return
     */
    @GetMapping("/hystrixdefaultFallback")
    @HystrixCommand(defaultFallback = "hystrixdefaultFallback")
    public String hystrixdefaultFallback(Integer id){
        if(id<0){
            throw new RuntimeException("id值小于0");
        }
        return "success："+id;
    }

    public String hystrixdefaultFallback(){
        return "网络异常，请重试";
    }
}

```

d、在浏览器调用开启服务熔断的方法，根据所写代码让程序短时间内产生大量异常，导致线程资源不能释放，然后再已正确的参数调用方法，就可以看到熔断的效果

![image-20210927212156089](https://gitee.com/studentgitee/note-picture/raw/master/image-20210927212156089.png)

### 7.5Openfeign整合Hystrix

a、因为openfeign依赖中已经引入了hystrix的依赖，所以我们不需要再重复引入hystrix的依赖，只要引入openfeign即可

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```



b、在application.yml配置文件中开启Openfeign对Hystrix的支持

```
feign:
  hystrix:
    enabled: true
```



c、编写Openfeign服务调用失败默认处理实现类

```
package com.wgp.hystrixfallback;

import com.wgp.hystrixclient.HystrixClinet;
import org.springframework.stereotype.Component;

@Component
public class HystrixFallbackFactory implements HystrixClinet {
    @Override
    public String hystrixdefaultFallback(Integer id) {
        return "Hystrix-server服务太火爆了！！";
    }
}

```



d、在openfeign中加入@FeignClient(value = "hystrix-server",fallback = 默认实现类名.class)

```
package com.wgp.hystrixclient;

import com.wgp.hystrixfallback.HystrixFallbackFactory;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@FeignClient(value = "hystrix-server",fallback = HystrixFallbackFactory.class)
public interface HystrixClinet {
    @GetMapping("/hystrixdefaultFallback")
    String hystrixdefaultFallback(@RequestParam(value ="id" ) Integer id);
}

```



e、当服务不可用时，会给我们返回默认提示

![image-20210927220834969](https://gitee.com/studentgitee/note-picture/raw/master/image-20210927220834969.png)

## 8.SpringcloudConfig配置中心

官网地址：https://spring.io/projects/spring-cloud-config

### 8.1整合配置中心服务端

a、引入依赖

```
<!-- 配置中心服务-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

b、启动类添加配置中心服务注解@EnableConfigServer 

```
@SpringBootApplication
@EnableEurekaClient
@EnableConfigServer // 开启配置中心服务
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

c、配置application.yml

```
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/studentgitee/springcloudconfig.git  #git仓库地址
          default-label: master  #拉去git的master分支
```

d、验证是否成功，在gitee上新建两个文件application.yml 和application-dev.yml         (一般配置文件我们都是：**服务名-环境.yml**)

![image-20210929093942124](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929093942124.png)

application.yml

```
server:
  port: 8007
spring:
  application:
    name: config-client
```

application-dev.yml

```
username: devzhangsan
```

e、使用我们的configserver去访问文件，如果能看到自己刚才新建的文件说明成功了

![image-20210929094035929](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929094035929.png)

### 8.2整合配置中心client

a、引入依赖

```
<!-- config配置中心客户端-->
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```



b、配置application.yml

```
spring:
  cloud:
    config:
      discovery:
        enabled: true #开启根据服务id去注册中心获取服务
        service-id: config-server  #根据服务id去注册中心查找配置服务
      label: master  #git的master分支
      name: application   #拉去的配置文件名称
      profile: dev   #拉去的环境，不如生产环境或者是测试环境
eureka:
  client:
    service-url:
      defaultZone: http://localhost:10001/eureka  #eureka地址     访问地址http://localhost:10001
```



c、需要将application.yml改成bootstrap.yml，这是因为application.yml是可知的配置文件，但是我们现在需要到config上面去拉去配置

后再进行服务启动，所有我们要先加载配置文件后再启动服务。

![image-20210929100917748](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929100917748.png)



d、编写测试是否成功  @Value("${username}")是在配置文件application-dev.yml中定义的，我们只需要看能不能取到变量值。

```
@RestController
public class ConfigController {

    @Value("${username}")
    private String name;

    @GetMapping("/username")
    public String invokeTest() {
        return "success" + name;
    }
}
```

e、结果

![image-20210929101216254](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929101216254.png)



## 9.bus组件

官网地址：https://spring.io/projects/spring-cloud-bus

### 9.1什么是bus(AMQP RibbitMQ)

> **定义: bus称之为springcloud中消息总线,主要用来在微服务系统中实现远端配置更新时通过广播形式通知所有客户端刷新配置信息，避免手动重启服务的工作**

![image-20210929120458129](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929120458129.png)

### 9.2整合bus消息总线

#### 在docker中安装rabbitMQ

- 下载镜像

```
docker pull rabbitmq:3.7.7-management
```

- 启动镜像（用户名和密码设置为guest guest）

```
docker run -dit --name rabbitmq3.7.7 -e RABBITMQ_DEFAULT_USER=guest -e RABBITMQ_DEFAULT_PASS=guest -p 15672:15672 -p 5672:5672 rabbitmq:3.7.7-management
```

- 访问rabbitmq管理界面

```rabbitmq
注意访问后台用的是端口15672，springcloud程序连接的是5672。   ip:15672
```

![image-20210929120203367](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929120203367.png)

#### config服务端整合bus

a、引入bus消息总线依赖

```
<!--bus消息总线-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

b、配置application.yml文件

```
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/studentgitee/springcloudconfig.git
          default-label: master
  rabbitmq:
    host: 192.168.1.27  #rabbitmq地址
    port: 5672  #rabbitmq端口
    username: guest   #用户名
    password: guest   #密码
    virtual-host: /   #虚拟主机
management:
  endpoints:
    web:
      exposure:
        include: "*"  #开启所有的web端口
```

c、启动服务后再rabbitmq查看是否有连接

![image-20210929123431652](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929123431652.png)



#### config客户端整合bus

a、引入bus消息总线依赖

```
<!--bus消息总线-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

b、配置application.yml文件

```
spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/studentgitee/springcloudconfig.git
          default-label: master
  rabbitmq:
    host: 192.168.1.27  #rabbitmq地址
    port: 5672  #rabbitmq端口
    username: guest   #用户名
    password: guest   #密码
    virtual-host: /   #虚拟主机
management:
  endpoints:
    web:
      exposure:
        include: "*"  #开启所有的web端口
```

 c、在controller上添加@RefreshScope刷新

```
import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
@RefreshScope
public class ConfigController {

    @Value("${username}")
    private String name;

    @GetMapping("/username")
    public String invokeTest() {
        return "success" + name;
    }
}
```

d、修改云端的配置文件application-dev.yml的username

```
spring:
  rabbitmq:
    host: 192.168.1.27
    port: 5672
    username: guest
    password: guest
    virtual-host: /
username: devzhangsan111111122222333333
```



e、使用postman发送post请求，让config配置中心从gitee上拉去配置文件

```
curl -X POST http://localhost:8848/actuator/bus-refresh
```

![image-20210929141803439](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929141803439.png)



f、观察config服务端和客户端的控制台

**服务端：**

![image-20210929144257111](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929144257111.png)

**客户端：**

![image-20210929144350499](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929144350499.png)



g、刷新客户端页面可以看到，修改配置文件已经生效了

![image-20210929144825170](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929144825170.png)



### 9.3webhooks实现配置自动刷新

#### 配置webhooks
- 说明: git仓库提供一种特有机制:这种机制就是一 个监听机制，监听就是仓库提交事件--------------触发对应事件执行
- javascript:事件事件源htm1标签 事件:触发特定动作c1ick ... 事件处理程序:函数
- 添加webhooks
- 在webhooks 中添加刷新配置接口
- 内网穿透的网站: https://natapp.cn/

![image-20210929145519944](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929145519944.png)



#### 利用natapp实现内网穿透（如果没有域名）

网址：https://natapp.cn/

a、购买隧道

![image-20210929153452741](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929153452741.png)



b、配置隧道

![image-20210929153538877](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929153538877.png)



c、下载客户端

![image-20210929153615368](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929153615368.png)



d、进入客户端安装目录

![image-20210929153715918](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929153715918.png)



e、建立通道

![image-20210929153921887](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929153921887.png)

![image-20210929153902920](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929153902920.png)

f、结果

![image-20210929153800234](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929153800234.png)

**在gitiee填写地址**

![image-20210929160016876](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929160016876.png)



当我们配置好WebHooks仍然不能实现动态刷新配置，就去WebHooks查看提交修改发送POST请求的操作信息，出现400的错误。错误的意思 POST请求后会给配置中心服务器发送一些内容，这些内容使用JSON解析的时候出错，无法反序列化这些类容，然后就抛异常，返回400错误。

> {“timestamp”:“2020-4-8T12:33:34.975+0000”,“status”:400,“error”:“Bad Request”,“message”:“JSON parse error: Cannot deserialize instance of java.lang.String out of START_ARRAY token; nested exception is com.fasterxml.jackson.databind.exc.MismatchedInputException: Cannot deserialize instance of java.lang.String out of START_ARRAY token\n at [Source: (PushbackInputStream); line: 1, column: 326] (through reference chain: java.util.LinkedHashMap[“commits”])”,“path”:"/actuator/bus-refresh"}
> 

**解决方案：**

a、编写配置类：

```
package com.wgp.fiter;

import org.springframework.stereotype.Component;
import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.ByteArrayInputStream;
import java.io.IOException;

@Component
public class WebHooksFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void destroy() {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest httpServletRequest = (HttpServletRequest) servletRequest;
        String url = new String(httpServletRequest.getRequestURI());

        if(!url.endsWith("/bus-refresh")){
            filterChain.doFilter(servletRequest,servletResponse);
            return;
        }
        RequestWrapper requestWrapper = new RequestWrapper(httpServletRequest);
        filterChain.doFilter(requestWrapper, servletResponse);
    }

    private class RequestWrapper extends HttpServletRequestWrapper {
        public RequestWrapper(HttpServletRequest request) {
            super(request);
        }

        @Override
        public ServletInputStream getInputStream() throws IOException {
            byte[] bytes = new byte[0];
            ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
            ServletInputStream servletInputStream = new ServletInputStream() {
                @Override
                public int read() throws IOException {
                    return byteArrayInputStream.read();
                }

                @Override
                public boolean isFinished() {
                    return byteArrayInputStream.read() == -1 ? true : false;
                }

                @Override
                public boolean isReady() {
                    return false;
                }

                @Override
                public void setReadListener(ReadListener listener) {

                }
            };
            return servletInputStream;
        }
    }
}

```

b、在启动类上添加@ServletComponentScan(basePackages="com.wgp.fiter")

```
@SpringBootApplication
@EnableEurekaClient
@EnableConfigServer // 开启配置中心服务
@ServletComponentScan(basePackages="com.wgp.fiter")
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

c、查看gitee

![image-20210929160339904](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929160339904.png)



d、测试发现已实现配置自动刷新



## 10.Sleuth链路跟踪-zipkin系统监控

### 10.1简介

> SpringCloud从F版起已不需要自己构建Zipkin Server了，只需调用jar包即可
>
> 

安装zipkin

a、下载jar后直接运行即可

地址：https://dl.bintray.com/openzipkin/maven/io/zipkin/java/zipkin-server/（是时候会失效）

![image-20210929164131013](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929164131013.png)

### 10.2集成zipkin

a、引入依赖

```
<!--zipkin系统监控包含了sleuth-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```

b、配置application.yml

```
spring:
  application:
    name: user-server
  zipkin:
    base-url: http://localhost:9411  #zipkin地址
    sleuth:
      sampler:
        probability: 1 #采样率值介于0到1之间，1则表示全部采集
```

c、测试调用后查看zipkin

![image-20210929164256442](https://gitee.com/studentgitee/note-picture/raw/master/image-20210929164256442.png)



## 11.SpringCloud Alibaba介绍

### 文档地址

**github地址**：https://github.com/alibaba/spring-cloud-alibaba/wiki

**官网地址**：https://spring.io/projects/spring-cloud-alibaba



- Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用服务的必需组件，方便开发者通过 

  Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。

- 依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里分布式应用解决方案，通过

  阿里中间件来迅速搭建分布式应用系统。

### 主要功能

- **服务限流降级**：默认支持 WebServlet、WebFlux、OpenFeign、RestTemplate、Spring Cloud Gateway、Zuul、Dubbo 和 

  RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。

- **服务注册与发现**：适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持。

- **分布式配置管理**：支持分布式系统中的外部化配置，配置更改时自动刷新。

- **消息驱动能力**：基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。

- **分布式事务**：使用 @GlobalTransactional 注解， 高效并且对业务零侵入地解决分布式事务问题。

- **阿里云对象存储**：阿里云提供的海量、安全、低成本、高可靠的云存储服务。支持在任何应用、任何时间、任何地点存储和访问任意

  类型的数据。

- **分布式任务调度**：提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，

  如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。

- **阿里云短信服务**：覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达通道。

## 12.Nacos注册中心

### 文档地址

**nacos官方文档**：https://nacos.io/zh-cn/docs/what-is-nacos.html

### 什么是nacos

- Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服

  务元数据及流量管理。

- Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原

  生范式) 的服务基础设施

**Nacos 的关键特性包括:**

- 服务发现和服务健康监测
- 动态配置服务
- 动态 DNS 服务
- 服务及其元数据管理

### 注册中心演变及设计思想

管理所有微服务、解决微服务之间调用关系错综复杂、难以维护的问题；



![image-20210923154242161](https://gitee.com/studentgitee/note-picture/raw/master/image-20210923154242161.png)

### 核心功能

**服务注册**：Nacos Client会通过发送REST请求的方式向Nacos Server注册自己的服务，提供自身的元数据，比如ip地
址、端口等信息。Nacos Server接收到注册请求后，就会把这些元数据信息存储在一个双层的内存Map中。

**服务心跳**：在服务注册后，Nacos Client会维护一个定时心跳来持续通知Nacos Server，说明服务一直处于可用状态，防
止被剔除。默认5s发送一次心跳。

**服务同步**：Nacos Server集群之间会互相同步服务实例，用来保证服务信息的一致性。  leader   raft   

**服务发现**：服务消费者（Nacos Client）在调用服务提供者的服务时，会发送一个REST请求给Nacos Server，获取上面
注册的服务清单，并且缓存在Nacos Client本地，同时会在Nacos Client本地开启一个定时任务定时拉取服务端最新的注
册表信息更新到本地缓存

**服务健康检查**：Nacos Server会开启一个定时任务用来检查注册服务实例的健康情况，对于超过15s没有收到客户端心跳
的实例会将它的healthy属性置为false(客户端服务发现时不会发现)，如果某个实例超过30秒没有收到心跳，直接剔除该
实例(被剔除的实例如果恢复发送心跳则会重新注册)

### centos7安装nacos

1、官网下载对应版本的nacos

地址：https://github.com/alibaba/Nacos/releases

2、解压，进入nacos目录,进入到bin目录下，修改startup.sh

```
standalone
```

![image-20210923182905383](https://gitee.com/studentgitee/note-picture/raw/master/image-20210923182905383.png)

3、进入conf目录找到nacos-mysql.sql,新建数据库nacos,执行脚本

![image-20210923183106249](https://gitee.com/studentgitee/note-picture/raw/master/image-20210923183106249.png)

再找到application.properties,修改成自己的数据库

![image-20210923183230327](https://gitee.com/studentgitee/note-picture/raw/master/image-20210923183230327.png)

4、进入到bin目录,启动nacos

```
sh startup.sh    #启动nacos

访问地址：http://ip:8848/nacos

账号：nacos
密码：nacos
```

### 微服务引入nacos

1、pom文件

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>springcloudalibaba</artifactId>
        <groupId>com.wgp</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>cms-user-service</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
         <!--  服务配置 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
        <!--  服务发现 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
    </dependencies>
</project>

```

2、在resourcess下新建bootstrap.yml 

![image-20210924111318823](https://gitee.com/studentgitee/note-picture/raw/master/image-20210924111318823.png)

```
server:
  port: 8001
spring:
  application:
    name: cms-user-service
  cloud:
    nacos:
      config:
        file-extension: yaml   #配置文件后缀
        group: DEFAULT_GROUP   #组
        server-addr: ${spring.cloud.nacos.server-addr} #nacos配置地址
      discovery:
        server-addr: 192.168.1.28:8848 #服务发现地址
      server-addr: 192.168.1.28:8848 #nacos服务地址

```

3、进入nacos控制台新建一个文件cms-user-service.yaml

在 Nacos Spring Cloud 中，`dataId` 的完整格式如下：

```
${prefix}-${spring.profiles.active}.${file-extension}
```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 `spring.profiles.active` 为空时，对应的连接符 `-` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型

