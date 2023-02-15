---
title: mysql面试题
tags: 面试题
categories: 面试题
cover: https://gitee.com/studentgitee/note-picture/raw/master/115.jpg
---
## **数据库事务的四个特性及含义**

**原子性：**整个事务中的所有操作，要么全部完成，要么全部不完成，不可能停滞在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。

**一致性：**在事务开始之前和事务结束以后，数据库的完整性约束没有被破坏。

**隔离性：**隔离状态执行事务，使它们好像是系统在给定时间内执行的唯一操作。如果有两个事务，运行在相同的时间内，执行 相同的功能，事务的隔离性将确保每一事务在系统中认为只有该事务在使用系统。这种属性有时称为串行化，为了防止事务操作间的混淆，必须串行化或序列化请 求，使得在同一时间仅有一个请求用于同一数据。

**持久性：**在事务完成以后，该事务所对数据库所作的更改便持久的保存在数据库之中，并不会被回滚。

## **什么是视图**

> 视图是一种虚拟的表，具有和物理表相同的功能。可以对视图进行增，改，查，操作，试图通常是有一个表或者多个表的行或列的子集。对视图的修改不影响基本表。它使得我们获取数据更容易，相比多表查询。

**如下两种场景一般会使用到视图：**

1、不希望访问者获取整个表的信息，只暴露部分字段给访问者，所以就建一个虚表，就是视图。

2、查询的数据来源于不同的表，而查询者希望以统一的方式查询，这样也可以建立一个视图，把多个表查询结果联合起来，查询者只需要直接从视图中获取数据，不必考虑数据来源于不同表所带来的差异。

## **drop、delete、truncate的区别**

- `drop`直接删掉数据表 。 
- `truncate`删除表中数据，再插入时自增长id又从1开始，不可恢复 。 
- `delete`删除表中数据，可以加where字句。

1、DELETE语句执行删除的过程是每次从表中删除一行，并且同时将该行的删除操作作为事务记录在日志中保存以便进行进行回滚操作。TRUNCATE TABLE 则一次性地从表中删除所有的数据并不把单独的删除操作记录记入日志保存，删除行是不能恢复的。并且在删除的过程中不会激活与表有关的删除触发器。执行速度快。

2、表和索引所占空间。当表被TRUNCATE 后，这个表和索引所占用的空间会恢复到初始大小，而DELETE操作不会减少表或索引所占用的空间。drop语句将表所占用的空间全释放掉。

3、 一般而言，drop > truncate > delete

4、 应用范围。TRUNCATE 只能对TABLE；DELETE可以是table和view

5、TRUNCATE 和DELETE只删除数据，而DROP则删除整个表（结构和数据）。

6、truncate与不带where的delete ：只删除数据，而不删除表的结构（定义）drop语句将删除表的结构被依赖的约束（constrain),触发器（trigger)索引（index)；依赖于该表的存储过程/函数将被保留，但其状态会变为：invalid。

7、 delete语句为DML（data maintain Language),这个操作会被放到 rollback segment中,事务提交后才生效。如果有相应的 tigger,执行的时候将被触发。

8、 truncate、drop是DLL（data define language),操作立即生效，原数据不放到 rollback segment中，不能回滚。

9、 在没有备份情况下，谨慎使用 drop 与 truncate。要删除部分数据行采用delete且注意结合where来约束影响范围。回滚段要足够大。要删除表用drop;若想保留表而将表中数据删除，如果于事务无关，用truncate即可实现。如果和事务有关，或老师想触发trigger,还是用delete。

10、Truncate table 表名 速度快,而且效率高,因为: 
truncate table 在功能上与不带 WHERE 子句的 DELETE 语句相同：二者均删除表中的全部行。但 TRUNCATE TABLE 比 DELETE 速度快，且使用的系统和事务日志资源少。DELETE 语句每次删除一行，并在事务日志中为所删除的每行记录一项。TRUNCATE TABLE 通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放。

（11） TRUNCATE TABLE 删除表中的所有行，但表结构及其列、约束、索引等保持不变。新行标识所用的计数值重置为该列的种子。如果想保留标识计数值，请改用 DELETE。如果要删除表定义及其数据，请使用 DROP TABLE 语句。

（12） 对于由 FOREIGN KEY 约束引用的表，不能使用 TRUNCATE TABLE，而应使用不带 WHERE 子句的 DELETE 语句。由于 TRUNCATE TABLE 不记录在日志中，所以它不能激活触发器。

## DDL、DML、DQL、DCL

> 1.DDL（DataDefinitionLanguage）：数据定义语言，用来定义数据库对象：库、表、列等；
>
> 2.DML（DataManipulationLanguage）：数据操作语言， 用来定义数据库记录（数据）,例如insert，delete，update，select；
>
> 3.DQL（DataQueryLanguage）：数据查询语言，用来查询记录（select 数据）；
>
> 4.DCL（DataControlLanguage）：数据控制语言，用来定义访问权限和安全级别。

## B+树最多只需1~3次磁盘IO找到记录

> InnoDB存储引擎中页的大小为16KB，一般表的主键类型为INT(占用4个字节）或BIGINT(占用8个字节)，指针类型也一般为4或8个字节，也就是说一个页(B+Tree中的一个节点)中大概存储16KB/(8B+8B)=1K个键值（因为是估值，为方便计算，这里的K取值为10^3。也就是说一个深度为3的B+Tree索引可以维护 10^3 * *10^3*  * 10^3= 10亿条记录。(这里假定一个数据页也存储10^3条行记录数据了)
>
> 实际情况中每个节点可能不能填充满，因此在数据库中，B+Tree 的高度一般都在2-4层，MySQL中InnoDB存储引擎在设计时是将根节点常驻内存的，也就是说查找某一键值的行记录时最多只需要1~3次磁盘1/o操作。

## 为什么使用B+树

1）首先，B+树查询效率更稳定。因为B+树每次只有访问到叶子节点才能找到对应的数据，而在B树中，非叶子节点也会存储数据，这样就会造成查询效率不稳定的情况，有时候访问到了非叶子节点就可以找到关键字，而有时需要访问到叶子节点才能找到关键字。

2）其次，B+树的查询效率更高。这是因为通常B+树比B树更矮胖（(阶数更大，深度更低)，查询所需要的磁盘I/o也会更少。同样的磁盘页大小，B+树可以存储更多的节点关键字。

3）不仅是对单个关键字的查询上，在查询范围上，B+树的效率也比B树高。这是因为所有关键字都出现在B+树的叶子节点中，叶子节点之间会有指针，数据又是递增的，这使得我们范围查找可以通过指针连接查找。而在B树中需要通过中序遍历才能完成范围查找，效率更低

## Hash索引和B+树索引的区别

1、 Hash 索引`不支持范围查找`，仅能满足(=)()和IN查询。如果进行范围查询，哈希型的索引，时间复杂度会退化为o(n)；而树型的“有序”特性，依然能够保持o(log2N)的高效率。

2、Hash索引不支持`ORDER BY排序`，数据的存储是没有顺序的，在ORDER BY的情况下，使用Hash索引还需要对数据重新排序。

3、Hash索引`不支持联合索引的最左匹配原则`(联合索引的部分索引会失效)，对于联合索引的情况，Hash值是将联合索引键合并后一起来计算的，无法对单独的一个键或者几个索引键进行查询。

4、InnoDB不支持Hash索引

## 索引失效的11种情况

### 最佳左前缀法则

> 索引文件具有 B-Tree 的最左前缀匹配特性，如果左边的值未确定，那么无法使用此索引。最左前缀法则是针对于联合来说的。**最佳左前缀法则：带头大哥不能死、中间兄弟不能断**

```mysql
CREATE TABLE `emp` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `empno` mediumint(8) unsigned NOT NULL DEFAULT '0',
  `ename` varchar(20) NOT NULL DEFAULT '',
  `job` varchar(9) NOT NULL DEFAULT '',
  `mgr` mediumint(8) unsigned NOT NULL DEFAULT '0',
  `hiredate` date NOT NULL,
  `sal` decimal(7,2) NOT NULL,
  `comm` decimal(7,2) NOT NULL,
  `deptno` mediumint(8) unsigned NOT NULL DEFAULT '0',
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
```

```mysql
INSERT INTO `emp` VALUES (1, 100002, 'ItTpTw', 'SALESMAN', 1, '2022-06-21', 2000.00, 400.00, 287);
INSERT INTO `emp` VALUES (2, 100003, 'zMncAv', 'SALESMAN', 1, '2022-06-21', 2000.00, 400.00, 282);
INSERT INTO `emp` VALUES (3, 100004, 'yKdnfL', 'SALESMAN', 1, '2022-06-21', 2000.00, 400.00, 150);
INSERT INTO `emp` VALUES (4, 100005, 'nxzdUo', 'SALESMAN', 1, '2022-06-21', 2000.00, 400.00, 367);
INSERT INTO `emp` VALUES (5, 100006, 'LzmMXD', 'SALESMAN', 1, '2022-06-21', 2000.00, 400.00, 499);
INSERT INTO `emp` VALUES (6, 100007, 'kcFVNb', 'SALESMAN', 1, '2022-06-21', 2000.00, 400.00, 443);
INSERT INTO `emp` VALUES (7, 100008, 'nKKwdd', 'SALESMAN', 1, '2022-06-21', 2000.00, 400.00, 47);
INSERT INTO `emp` VALUES (8, 100009, 'kMhAgf', 'SALESMAN', 1, '2022-06-21', 2000.00, 400.00, 89);
INSERT INTO `emp` VALUES (9, 100010, 'xQMndA', 'SALESMAN', 1, '2022-06-21', 2000.00, 400.00, 208);
INSERT INTO `emp` VALUES (10, 100011, 'wZIsOS', 'SALESMAN', 1, '2022-06-21', 2000.00, 400.00, 480);

```

```mysql
# 创建联合索引
create index  idx_empno_ename_deptno on emp(empno,ename,deptno);
```

根据deptno列单独查询，发现所以联合索引并没有用上

```java
EXPLAIN select * from emp where  deptno='287'
```

![image-20220719214913794](https://gitee.com/studentgitee/note-picture/raw/master/image-20220719214913794.png)

根据empno和deptno一起查询，发现只有empno用上了索引

```mysql
EXPLAIN select * from emp where  deptno='287' and empno = '100002' 
```

![image-20220719215129057](https://gitee.com/studentgitee/note-picture/raw/master/image-20220719215129057.png)

根据empno和deptno和ename一起查询,发现索引都用上了

```java
EXPLAIN select * from emp where  deptno='287' and empno = '100002' and ename='ItTpTw'
```

![image-20220719215242669](https://gitee.com/studentgitee/note-picture/raw/master/image-20220719215242669.png)

### like以通配符%开头索引失效

```mysql
EXPLAIN select * from emp where ename like '%e'
```

LIKE查询以%开头使用了索引的原因就是使用了索引覆盖。
针对二级索引MySQL提供了一个[**优化**](https://www.bmabk.com/index.php/post/tag/77)技术。即从辅助索引中就可以得到查询的记录，就不需要回表再根据聚集索引查询一次完整记录。使用索引覆盖的一个好处是辅助索引不包含整行记录的所有信息，故其大小要远小于聚集索引，因此可以减少大量的IO操作，但是前提是要查询的所有列必须都加了索引。

LIKE查询以%开头会导致全索引扫描或者全表扫描，如果没有索引覆盖的话，查询到的数据会回表，多了一次IO操作，当MySQL预估全表扫描或全索引扫描的时间比走索引花费的时间更少时，就不会走索引。有了索引覆盖就不需要回表了，减少了IO操作，花费的时间更少，所以就使用了索引。

### OR 前后存在非索引的列，索引失效

如果想使用or的时候索引也生效，那么or的每个字段都需要加上索引。

### 如果列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引

## sql优化

### 优化步骤