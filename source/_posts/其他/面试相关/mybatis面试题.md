---
title: mybatis面试题
tags: 面试题
categories: 面试题
cover: https://gitee.com/studentgitee/note-picture/raw/master/md005.png
---
## #{}和${}的区别是什么？

- `${}`是 Properties 文件中的变量占位符，它可以用于标签属性值和 sql 内部，属于静态文本替换，比如${driver}会被静态替换为`com.mysql.jdbc. Driver`。
- `#{}`是 sql 的参数占位符，MyBatis 会将 sql 中的`#{}`替换为? 号，在 sql 执行前会使用 PreparedStatement 的参数设置方法，按序给 sql 的? 号占位符设置参数值，比如 ps.setInt(0, parameterValue)，`#{item.name}` 的取值方式为使用反射从参数对象中获取 item 对象的 name 属性值，相当于 `param.getItem().getName()`。

## 既然$不安全，为什么还需要$，什么时候会用到它？

它可以解决一些特殊情况下的问题。例如，在一些动态表格（根据不同的条件产生不同的动态列）中，我们要传递SQL的列名，根据某些列进行排序，或者传递列名给SQL都是比较常见的场景，这就无法使用预编译的方式了。

## Xml 映射文件中，除了常见的 select|insert|update|delete 标签之外，还有哪些标签？

**答：**还有很多其他的标签， `<resultMap>` 、 `<parameterMap>` 、 `<sql>` 、 `<include>` 、 `<selectKey>` ，加上动态 sql 的 9 个标签， `trim|where|set|foreach|if|choose|when|otherwise|bind` 等，其中 `<sql>` 为 sql 片段标签，通过 `<include>` 标签引入 sql 片段， `<selectKey>` 为不支持自增的主键生成策略标签。

## MyBatis的xml文件和Mapper接口是怎么绑定的？

是通过xml文件中，<mapper> 根标签的namespace属性进行绑定的，即namespace属性的值需要配置成接口的全限定名称，MyBatis内部就会通过这个值将这个接口与这个xml关联起来。，当调用接口方法时，接口全限名+方法名拼接字符串作为 key 值，可唯一定位一个 `MappedStatement`。举例： `com.mybatis3.mappers. StudentDao.findStudentById` ，可以唯一找到 namespace 为 `com.mybatis3.mappers. StudentDao` 下面 `id = findStudentById` 的 `MappedStatement` 。在 MyBatis 中，每一个 `<select>` 、 `<insert>` 、 `<update>` 、 `<delete>` 标签，都会被解析为一个 `MappedStatement` 对象。

## MyBatis 是如何进行分页的？分页插件的原理是什么？

**(1)** MyBatis 使用 RowBounds 对象进行分页，它是针对 ResultSet 结果集执行的内存分页，而非物理分页；

**(2)** 可以在 sql 内直接书写带有物理分页的参数来完成物理分页功能，

**(3)** 也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。

举例： `select _ from student` ，拦截 sql 后重写为： `select t._ from （select \* from student）t limit 0，1`

## 物理分页和逻辑分支

- 物理分页依赖的是某一物理实体，这个物理实体就是数据库，比如MySQL数据库提供了limit关键字，程序员只需要编写带有limit关键字的SQL语句，数据库返回的就是分页结果。
- 逻辑分页依赖的是程序员编写的代码。数据库返回的不是分页结果，而是全部数据，然后再由程序员通过代码获取分页数据，常用的操作是一次性从数据库中查询出全部数据并存储到List集合中，因为List集合有序，再根据索引获取指定范围的数据。

**对比**
**(1)**数据库负担
物理分页每次都访问数据库，逻辑分页只访问一次数据库，物理分页对数据库造成的负担大。

**(2)**服务器负担
逻辑分页一次性将数据读取到内存，占用了较大的内容空间，物理分页每次只读取一部分数据，占用内存空间较小。

**(3)**实时性
逻辑分页一次性将数据读取到内存，数据发生改变，数据库的最新状态不能实时反映到操作中，实时性差。物理分页每次需要数据时都访问数据库，能够获取数据库的最新状态，实时性强。

**(4)**适用场合
逻辑分页主要用于数据量不大、数据稳定的场合，物理分页主要用于数据量较大、更新频繁的场合。

## MyBatis 能执行一对一、一对多的关联查询吗？都有哪些实现方式，以及它们之间的区别

一对多映射有两种配置方式，都是使用collection标签实现的。在此之前，为了能够存储一对多的数据，需要在主表对应的实体类中增加集合属性，用于封装子表对应的实体类。

嵌套查询：

1. 通过select标签定义查询主表的SQL，返回结果通过reusltMap进行映射。
2. 在resultMap中，除了映射主表属性，还要通过collection标签映射子表属性，该标签需包含如下内容：
   - 通过property属性指定子表属性名；
   - 通过javaType属性指定封装子表属性的集合类型；
   - 通过ofType属性指定子表的实体类型；
   - 通过select属性指定查询子表所依赖的SQL，这个SQL需单独定义，内部包含查询子表的语句。

嵌套结果：

1. 通过select标签定义关联查询主表和子表的SQL，返回结果通过resultMap进行映射。
2. 在resultMap中，除了映射主表属性，还要通过collection标签映射子表属性，该标签需包含如下内容：
   - 通过property属性指定子表属性名；
   - 通过ofType属性指定子表的实体类型；
   - 通过result子标签定义子表字段和属性的映射关系。

## MyBatis 都有哪些 Executor 执行器？它们之间的区别是什么？

MyBatis 有三种基本的 Executor 执行器， `SimpleExecutor`（简单执行器） 、 `ReuseExecutor`（可重用执行器） 、 `BatchExecutor`（批处理执行器）

`SimpleExecutor` ：每执行一次 update 或 select，就开启一个 Statement 对象，用完立刻关闭 Statement 对象。

`ReuseExecutor` ：**执行 update 或 select，以 sql 作为 key 查找 Statement 对象，存在就使用，不存在就创建，用完后，不关闭 Statement 对象，而是放置于 Map<String, Statement>内，供下一次使用。简言之，就是重复使用 Statement 对象。

`BatchExecutor` ：**执行 update（没有 select，JDBC 批处理不支持 select），将所有 sql 都添加到批处理中（addBatch()），等待统一执行（executeBatch()），它缓存了多个 Statement 对象，每个 Statement 对象都是 addBatch()完毕后，等待逐一执行 executeBatch()批处理。与 JDBC 批处理相同。

## MyBatis缓存机制吗？

![image-20200603165448345](https://gitee.com/studentgitee/note-picture/raw/master/image-20200603165448345.png)

### **一级缓存**

也叫做会话级缓存，生命周期仅存在于当前会话，不可以直接关关闭。但可以通过flushCache和localCacheScope对其做相应控制，**一级缓存是线程不安全的，不能跨线程使用**

缓存命中的条件：

1. SQL与参数相同：
2. 同一个会话：
3. 相同的MapperStatement ID：
4. RowBounds行范围相同：

触发清空缓存：

1. 手动调用clearCache
2. 执行提交回滚
3. 执行update
4. 配置flushCache=true
5. 缓存作用域为Statement

实现：

当会话接收到查询请求之后，会交给执行器的Query方法，在这里会通过 Sql、参数、分页条件等参数创建一个缓存key，在基于这个key去 PerpetualCache中查找对应的缓存值，如果有主直接返回。没有就会查询数据库，然后在填充缓存。

![image-20200603175917082](https://gitee.com/studentgitee/note-picture/raw/master/image-20200603175917082.png)

### **二级缓存**

二级缓存也称作是应用级缓存，与一级缓存不同的，是它的作用范围是整个应用，**而且可以跨线程使用**。所以二级缓存有更高的命中率，适合缓存一些修改较少的数据。在流程上是先访问二级缓存，在访问一级缓存。

![image-20200609104535077](https://gitee.com/studentgitee/note-picture/raw/master/image-20200609104535077.png)

我们在设计一个二级缓存的时候，需要考虑他应该有哪些功能？

**核心功能：**

**存储：**我们的缓存应该存在哪里？

内存：程序重启后，存储在内存的东西将会丢失。

硬盘：可以持久化，容量大。但访问速度不如内存，一般会结合内存一起使用。

第三方缓存数据库：比如redis，如果是分布式的情况下，一般使用这种方案。

**溢出淘汰**：无论哪种存储都必须有一个容易，当容量满的时候就要进行清除，清除的算法即溢出淘汰机制。

常见算法如下：

1. FIFO：先进先出
2. LRU：最近最少使用（mybatis使用的是这种）
3. WeakReference: 弱引用，将缓存对象进行弱引用包装，当Java进行gc的时候，不论当前的内存空间是否足够，这个对象都会被回收
4. SoftReference：软件引用，基机与弱引用类似，不同在于只有当空间不足时GC才才回收软引用对象。

**其他功能：**

1.线程安全

2.过期清理

3.序列化

................................

![image-20200609115516417](https://gitee.com/studentgitee/note-picture/raw/master/image-20200609115516417.png)

### 二级缓存责任链设计

这么多的功能，如何才能简单的实现，并保证它的灵活性与扩展性呢？这里MyBatis抽像出Cache接口，其只定义了缓存中最基本的功能方法：

- 设置缓存
- 获取缓存
- 清除缓存
- 获取缓存数量

然后上述中每一个功能都会对应一个组件类，并基于装饰者加责任链的模式，将各个组件进行串联。在执行缓存的基本功能时，其它的缓存逻辑会沿着这个责任链依次往下传递。

![image-20200609121931489](https://gitee.com/studentgitee/note-picture/raw/master/image-20200609121931489.png)

这样设计有以下优点：

1. 职责单一：各个节点只负责自己的逻辑，不需要关心其它节点。
2. 扩展性强：可根据需要扩展节点、删除节点，还可以调换顺序保证灵活性。
3. 松耦合：各节点之间不没强制依赖其它节点。而是通过顶层的Cache接口进行间接依赖。