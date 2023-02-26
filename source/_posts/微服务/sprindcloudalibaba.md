---
title: springcloudalibaba
tags: 微服务
categories: 微服务
cover: https://gitee.com/studentgitee/note-picture/raw/master/md0024.png
---

# SpringCloud Alibaba

## 介绍

### 文档地址

> **github地址**：https://github.com/alibaba/spring-cloud-alibaba/wiki

>  **官网地址**：https://spring.io/projects/spring-cloud-alibaba

> Spring Cloud Alibaba 致力于提供微服务开发的一站式解决方案。此项目包含开发分布式应用服务的必需组件，方便开发者通过 Spring Cloud 编程模型轻松使用这些组件来开发分布式应用服务。依托 Spring Cloud Alibaba，您只需要添加一些注解和少量配置，就可以将 Spring Cloud 应用接入阿里分布式应用解决方案，通过阿里中间件来迅速搭建分布式应用系统。

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

## Nacos注册中心

> **nacos官方文档**：https://nacos.io/zh-cn/docs/what-is-nacos.html

### 什么是nacos

> Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。

> Nacos 帮助您更敏捷和容易地构建、交付和管理微服务平台。 Nacos 是构建以“服务”为中心的现代应用架构 (例如微服务范式、云原生范式) 的服务基础设施

**Nacos 的关键特性包括:**

- 服务发现和服务健康监测
- 动态配置服务
- 动态 DNS 服务
- 服务及其元数据管理

### 核心功能

**服务注册**：Nacos Client会通过发送REST请求的方式向Nacos Server注册自己的服务，提供自身的元数据，比如ip地
址、端口等信息。Nacos Server接收到注册请求后，就会把这些元数据信息存储在一个双层的内存Map中。

**服务心跳**：在服务注册后，Nacos Client会维护一个定时心跳来持续通知Nacos Server，说明服务一直处于可用状态，防
止被剔除。默认5s发送一次心跳。

**服务同步**：Nacos Server集群之间会互相同步服务实例，用来保证服务信息的一致性。

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

### nacos之namespace、group、dataid说明



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



## Gateway

## sentinel
