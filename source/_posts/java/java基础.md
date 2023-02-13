---
title: java基础
date: 2023-02-12 23:33:02
tags: java
categories: java
cover: https://gitee.com/studentgitee/note-picture/raw/master/917121c7d8615458f6c6039069fc6255.png
---
![image-20230210143018927](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210143018927.png)

## 反射

### 什么是反射？

> 主要是指程序可以访问、检测和修改它本身状态或行为的一种能力

### 反射机制流程

![image-20230210145249032](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210145249032.png)

### 反射能干什么？

- 在运行时判断任意一个对象所属的类
- 在运行时构造任意一个类的对象
- 在运行时得到任意一个类所具有的成员变量和方法
- 在运行时调用任意一个对象的成员变量和方法
- 生成动态代理

### 获取Class的实例(三种)

```
Class clazz = 类名.class

Class clazz = Class.forName("类的全限定类名");

Class clazz = 对象.getClass();
```

### 获取Field(四个方法)

```
//该方法只能通过属性名获取public的属性
Field field = c.getField("属性名");　　　

//获取所有的public属性数组
Field[] field = c.getFields();　　

//获取类的属性，包括protected/private
Field field = c.getDeclaredField("属性名"); 
```

### 设置可访问性

```
//默认为false只能对public修饰的操作，设置true可对private修饰的操作
//可以用在被访问修饰符修饰的地方，
- setAccessible(true);　　
```

### 获取Field的信息

```
//获取属性名
- String name = field.getName();　　

//获取属性的类型
- Class<?> type = field.getType(http://www.amjmh.com/v/);　　

//获取obj对象的field属性的值
- Object value = field.get(obj);　　

//给obj对象的field属性赋值value
- field.set(obj,Object value);　　　　
```

### 创建对象

```
//1通过newInstence()
Phone instance1 = (Phone) clazz1.newInstance();

//2先调用构造器，再通过newInstence()创建
Object instance2 = clazz1.getConstructor().newInstance();
```

