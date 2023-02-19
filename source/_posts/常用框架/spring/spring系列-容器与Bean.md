---
title: spring系列容器与Bean
tags: 常用框架
categories: 常用框架
cover: https://gitee.com/studentgitee/note-picture/raw/master/f9a0132ef7bd81b06d86d07b6879f13d.jpg
---
![image-20230210143018927](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210143018927.png)

# 对Spring的理解

1、Spring是实现了工厂模式的工厂类（在这里有必要解释清楚什么是工厂模式），这个类名为BeanFactory（实际上是一个接口），在程序中通常BeanFactory的子类ApplicationContext。Spring相当于一个大的工厂类，在其配置文件中通过<bean>元素配置用于创建实例对象的类名和实例对象的属性。

2、Spring提供了对IOC良好支持，IOC是一种编程思想，是一种架构艺术，利用这种思想可以很好地实现模块之间的解耦，IOC也称为DI（Depency Injection）。

3、Spring提供了对AOP技术的良好封装， AOP称为面向切面编程，就是系统中有很多各不相干的类的方法，在这些众多方法中要加入某种系统功能的代码，例如，加入日志，加入权限判断，加入异常处理，这种应用称为AOP。

 4、实现AOP功能采用的是代理技术，客户端程序不再调用目标，而调用代理类，代理类与目标类对外具有相同的方法声明，有两种方式可以实现相同的方法声明，一是实现相同的接口，二是作为目标的子类。

5、 在JDK中采用Proxy类产生动态代理的方式为某个接口生成实现类，如果要为某个类生成子类，则可以用CGLI B。在生成的代理类的方法中加入系统功能和调用目标类的相应方法，系统功能的代理以Advice对象进行提供，显然要创建出代理对象，至少需要目标类和Advice类。spring提供了这种支持，只需要在spring配置文件中配置这两个元素即可实现代理和aop功能。

6、Spring最主要就是提高我们的开发效率，以及模块之间解耦

# 容器接口

## 什么是BeanFactory?

<img src="https://gitee.com/studentgitee/note-picture/raw/master/image-20230219160002802.png"/>

> 它是`ApplicationContext`的父接口
>
> 它才是 `Spring` 的核心容器，主要的 `ApplicationContext` 实现都 [组合]了他的功能，【组合】是指 ApplicationContext 的一个重要成员变量就是 BeanFactory

## BeanFactory能做什么？

> BeanFactory的定义(选中BeanFactory后按住Ctrl+F12可以查看类里面的所有方法)

![image-20230219115854082](https://gitee.com/studentgitee/note-picture/raw/master/image-20230219115854082.png)

> - 表面上只有 getBean
> - 实际上控制反转、基本的依赖注入、直至 Bean 的生命周期的各种功能，都由它的实现类提供

## ApplicationContext的功能

## BeanFactory与ApplicationContext的区别

1、BeanFactory是Spring的早期接口，称为Spring的Bean工厂，ApplicationContext是后期更高级接口，称之为Spring容器；
ApplicationContext在BeanFactory基础上对功能进行了扩展，例如：监听功能、国际化功能等。BeanFactory的API更偏向底层，ApplicationContext的API大多数是对这些底层API的封装；

![image-20230212120519435](https://gitee.com/studentgitee/note-picture/raw/master/image-20230212120519435.png)

2、Bean创建的主要逻辑和功能都被封装在BeanFactory中，ApplicationContext不仅继承了BeanFactory，而且ApplicationContext内部还维护着BeanFactory的引用，所以，ApplicationContext与BeanFactory既有继承关系，又有融合关系。

3、Bean的初始化时机不同，原始BeanFactory是在首次调用getBean时才进行Bean的创建，而ApplicationContext则是配置文件加载，容器一创建就将Bean都实例化并初始化好

# 容器实现

* DefaultListableBeanFactory，是 BeanFactory 最重要的实现，像**控制反转**和**依赖注入**功能，都是它来实现
* ClassPathXmlApplicationContext，从类路径查找 XML 配置文件，创建容器（旧）
* FileSystemXmlApplicationContext，从磁盘路径查找 XML 配置文件，创建容器（旧）
* XmlWebApplicationContext，传统 SSM 整合时，基于 XML 配置文件的容器（旧）
* AnnotationConfigWebApplicationContext，传统 SSM 整合时，基于 java 配置类的容器（旧）
* AnnotationConfigApplicationContext，Spring boot 中非 web 环境容器（新）
* AnnotationConfigServletWebServerApplicationContext，Spring boot 中 servlet web 环境容器（新）
* AnnotationConfigReactiveWebServerApplicationContext，Spring boot 中 reactive web 环境容器（新）

另外要注意的是，后面这些带有 ApplicationContext 的类都是 ApplicationContext 接口的实现，但它们是**组合**了 DefaultListableBeanFactory 的功能，并非继承而来

## BeanFactory的实现

## ApplicationContext的实现

# Bean的生命周期

# Bean后置处理器

# BeanFactory后置处理器

# Aware接口

# 初始化与销毁

# Scope

## Scop的类型




