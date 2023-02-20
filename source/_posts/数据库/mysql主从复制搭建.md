---
title: mysql主从赋值搭建
tags: 数据库
categories: 数据库
cover: https://gitee.com/studentgitee/note-picture/raw/master/xd009.png
---
![image-20230210143018927](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210143018927.png)

# 一、架构图

![image-20221216142535184](https://gitee.com/studentgitee/note-picture/raw/master/image-20221216142535184.png)

# 二、mysql主从复制搭建

## 原理	

![image-20221213114549605](https://gitee.com/studentgitee/note-picture/raw/master/image-20221213114549605.png)



1、主库会生成一个 log dump 线程,用来给从库 I/O 线程传 Binlog 数据。

2、从库的 I/O 线程会去请求主库的 Binlog，并将得到的 Binlog 写到本地的 relay log (中继日志)文件中。

3、SQL 线程,会读取 relay log 文件中的日志，并解析成 SQL 语句逐一执行。

**主从复制用途**

1） 实时灾备，用于故障切换（高可用）

2） 读写分离，提供查询服务（读扩展）

3） 数据备份，避免影响业务（高可用）

**疑问：**

为什么要先写一遍relay log然后再写从库，直接将数据写入到从库不好吗？ 在这里relay log的作用就类似于一个中间层，主库是多线程并发写的，从库

的sql线程是单线程串行执行的，所以这两边的生产和消费速度肯定不同。当主库发的binlog消息过多时，从库的relay log可以起到暂存主库数据的作

用，接着从库的sql线程再慢慢消费这些relay log数据，这样既不会限制主库发消息的速度，也不会给从库造成过大压力。

## 实战

**主从部署必要条件**

1） 从库服务器能连通主库

2） 主库开启binlog日志（设置log-bin参数）

3） 主从server-id不同

我们需要准备两台mysql，我这里使用docker，那么我就需要开启两个mysql容器就好了。docker安装mysql自行查找资料。

```
# 主库
docker run -p 9088:3306 --name mysql_master -v /docker/mysql_master/conf:/etc/mysql/conf.d -v /docker/mysql_master/logs:/logs -v /docker/mysql_master/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7

# 从库
docker run -p 9089:3306 --name mysql_from -v /docker/mysql_from/conf:/etc/mysql/conf.d -v /docker/mysql_from/logs:/logs -v /docker/mysql_from/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

**注意：**如果安装的是mysql8那么需要多加一个启动参数：--privileged=true

--lower-case-table-names=1在首次启动的时候就需要加上

如果忘记加上，那么必须新建映射的data目录的（/docker/mysql_master/data）

mysql8

```
# 主库
docker run  
-p 9088:3306 \
--name mysql_master \
-v /docker/mysql_master/conf:/etc/mysql/conf.d \
-v /docker/mysql_master/logs:/logs\ 
-v /docker/mysql_master/data:/var/lib/mysql\ 
--privileged=true \
-e MYSQL_ROOT_PASSWORD=123456 \
--restart=always\
-d mysql:8.0 \
--lower-case-table-names=1

# 从库
docker run  
-p 9089:3306 \
--name mysql_from \
-v /docker/mysql_from/conf:/etc/mysql/conf.d \
-v /docker/mysql_from/logs:/logs \
-v /docker/mysql_from/data:/var/lib/mysql \
--privileged=true \
-e MYSQL_ROOT_PASSWORD=123456 \
--restart=always\
-d mysql:8.0\
--lower-case-table-names=1
```



### 1、准备

mysql：8.0

docker

| 主库ip | 192.168.1.27 |
| ------ | ------------ |
| 端口   | 9088         |
| 账号   | root         |
| 密码   | 123456       |

| 从库ip | 192.168.1.27 |
| ------ | ------------ |
| 端口   | 9089         |
| 账号   | root         |
| 密码   | 123456       |

**做同步之前要保证两个数据库数据一致.**

如果主数据库有数据的话。 数据库锁表操作，不让数据再进行写入动作。

```
# 操作之前执行
mysql> FLUSH TABLES WITH READ LOCK;

# 开启同步之前执行
mysql> UNLOCK TABLES;
```

### 2、配置主服务器

#### 配置主服务器的 my.cnf 添加以下内容：

```
[mysqld]
## 同一局域网内注意要唯一
server-id=88
## 开启二进制日志功能
log-bin=mysql-bin
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'
ngram_token_size=2
innodb_ft_min_token_size=1
ft_min_word_len=1
## GROUP_CONCAT函数的长度
group_concat_max_len=4294967295

auto-increment-increment = 2
auto-increment-offset = 1

```

**配置解释：**

**server-id：**标识，唯一，值范围在：1至2^23-1

**log-bin：**配置是否在数据库有变动时写二进制日志

**lower_case_table_names：**忽略大小写

**auto_increment_offset**和**auto_increment_increment**：当同步断开，两台服务器分别有新数据进入，那么主键ID是自增长列会出现冲突的情况，会导致同步无法继续。加上上面两个设置后server1的auto_increment字段产生的数值是：1, 3, 5, 7, …等奇数，server2的为偶数。

#### 重启主服务mysql

```
 service mysql restart
```

#### 查看 skip_networking 的状态

确保在主服务器上 skip_networking 选项处于 OFF 关闭状态, 这是默认值。如果是启用的，则从站无法与主站通信，并且复制失败。

```
mysql> show variables like '%skip_networking%';
+-----------------+-------+
| Variable_name   | Value |
+-----------------+-------+
| skip_networking | OFF   |
+-----------------+-------+
1 row in set (0.00 sec)
```

#### 创建一个专门用来复制的用户

```
CREATE USER 'repl'@'%' identified by '123456';

GRANT REPLICATION SLAVE ON *.*  TO  'repl'@'%';
```

#### 在主服务器配置连接到从服务器的相关信息 

```
CHANGE MASTER TO MASTER_HOST='192.168.1.27', MASTER_PORT=9089,MASTER_USER='repl',MASTER_PASSWORD='123456',master_log_file='mysql-bin.000001',master_log_pos=154;
```

注意：master_log_file和master_log_pos需要在从服务器上查看后，根据实际情况修改。

在从服务器上执行：

```
show master status;
```

![image-20221213112600625](https://gitee.com/studentgitee/note-picture/raw/master/image-20221213112600625.png)



### 3、配置从服务器

#### 配置从服务器的 my.cnf 添加以下内容：

```
[mysqld]
## 同一局域网内注意要唯一
server-id=89
## 开启二进制日志功能
log-bin=mysql-bin
lower_case_table_names=1
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'
ngram_token_size=2
innodb_ft_min_token_size=1
ft_min_word_len=1
group_concat_max_len=4294967295

auto-increment-increment = 2
auto-increment-offset = 2
```

配置解释：

**server-id：**标识，唯一，值范围在：1至2^23-1

**log-bin：**配置是否在数据库有变动时写二进制日志

**lower_case_table_names：**忽略大小写

**auto_increment_offset**和**auto_increment_increment**：当同步断开，两台服务器分别有新数据进入，那么主键ID是自增长列会出现冲突的情况，会导致同步无法继续。加上上面两个设置后server1的auto_increment字段产生的数值是：1, 3, 5, 7, …等奇数，server2的为偶数。

#### 重启从服务mysql

```
 service mysql restart
```

#### 创建一个专门用来复制的用户

```
CREATE USER 'repl'@'%' identified by '123456';

GRANT REPLICATION SLAVE ON *.*  TO  'repl'@'%';
```

#### 在从服务器配置连接到主服务器的相关信息 

```
CHANGE MASTER TO MASTER_HOST='192.168.1.27', MASTER_PORT=9088,MASTER_USER='repl',MASTER_PASSWORD='123456',master_log_file='mysql-bin.000002',master_log_pos=154;
```

注意：master_log_file和master_log_pos需要在从服务器上查看后更换。

在从服务器上执行：

```
show master status;
```

![image-20221213112852370](https://gitee.com/studentgitee/note-picture/raw/master/image-20221213112852370.png)

### 4、主库和从库启动同步

主库和从库先后执行

```
mysql> start slave;
Query OK, 0 rows affected (0.00 sec)
```

注意：如果想关闭同步，则可以使用（这一步不是必须的）

```
mysql> stop slave;
```

### 5、查看同步状态

#### 主库mysql同步状态

```
mysql>  show slave status \G;
```

![image-20221213113308906](https://gitee.com/studentgitee/note-picture/raw/master/image-20221213113308906.png)

#### 从库mysql同步状态

```
mysql>  show slave status \G;
```

![image-20221213113427067](https://gitee.com/studentgitee/note-picture/raw/master/image-20221213113427067.png)

### 6、测试

1、在主库新建表后，去从库查看是否同步，如果有对应的表就说明成功了

2、在从库的一张表插入数据后，去主库的对应的表查看，如果有对应的数据就说明成功了

## 问题

### 异步复制

1、逻辑上

MySQL默认的复制即是异步的，主库在执行完客户端提交的事务后会立即将结果返给给客户端，并不关心从库是否已经接收并处理，这样就会有一个问题，主如果挂掉了，此时主上已经提交的事务可能并没有传到从库上，如果此时，强行将从提升为主，可能导致新主上的数据不完整。

2、技术上

主库将事务 Binlog 事件写入到 Binlog 文件中，此时主库只会通知一下 Dump 线程发送这些新的 Binlog，然后主库就会继续处理提交操作，而此时不会保证这些 Binlog 传到任何一个从库节点上。

3、原理图

![image-20221213114549605](https://gitee.com/studentgitee/note-picture/raw/master/image-20221213114549605.png)

### 并行复制（全同步复制）

1、逻辑上

指当主库执行完一个事务，所有的从库都执行了该事务才返回给客户端。因为需要等待所有从库执行完该事务才能返回，所以全同步复制的性能必然会收到严重的影响。

### **半同步复制**

1、逻辑上

是介于并行复制与异步复制之间的一种，主库只需要等待至少一个从库节点收到并且 Flush Binlog 到 Relay Log 文件即可，主库不需要等待所有从库给

主库反馈。同时，这里只是一个收到的反馈，而不是已经完全完成并且提交的反馈，如此，节省了很多时间。

# 三、mycat数据库中间件搭建

我们现在普遍的Java应用程序都是直接连接了MySQL软件进行读写操作，也就是我们在Java中的配置文件等定义了mysql的数据源，直接连接到了我们的

mysql软件，但是当某些情况下我们可能需要用到了多个数据库，这个时候我们可能就需要配多个数据源去连接我们的多个数据库，这个时候我们进行

sql操作的时候就会很麻烦，因为Java与数据库有了一个紧密的耦合度，但是如果我们在Java应用程序与mysql中间使用了mycat，我们只需要访问mycat

就可以了，至于数据源等问题，mycat会直接帮我们搞定。

## mycat下载地址

```
https://github.com/MyCATApache/Mycat-download/tree/master/1.6.5-DEV

# 官网地址
http://www.mycat.org.cn/
```

## **原理**

![image-20221214155807553](https://gitee.com/studentgitee/note-picture/raw/master/image-20221214155807553.png)

使用mycat进行主从切换，当一台mysql服务器宕机之后，mycat会切换至另一台进行连接，两台mysql互为主从，这样可以使两台mysql服务器互相备

份，使其数据一致。

## 实战

mycat是基于java开发的，所以需要先安装jdk1.8

### 下载mycat

```
https://github.com/MyCATApache/Mycat-download/tree/master/1.6.5-DEV
```

![image-20221214164143637](https://gitee.com/studentgitee/note-picture/raw/master/image-20221214164143637.png)

### 安装mycat

1、将下载到的包传输到linux服务器上的software上。

2、对压缩包进行解压。

```
tar -zxvf Mycat-server-1.6.5-DEV-20161231120132-linux.tar.gz
```

3、将mycat目录移动到 /usr/local下**(强制要求)**

```
mv mycat /usr/local
```

4、配置环境变量，vi ~/.bash_profile，增加以下 export

```
MYCAT_HOME=/usr/local/mycat
export PATH=$PATH:$MYCAT_HOME/bin
```

使配置文件生效

```
source ~/.bash_profile
```

5、配置连接信息

mycat的server.xml配置逻辑库的名称访问的账号密码

```
<user name="root" defaultAccount="true">  <!-- 这里是给虚拟库设定一个账号叫root，并且作为默认账号 -->
        <property name="password">123456</property>   <!-- 账号root的密码 -->
        <property name="schemas">aiurt-platform</property>   <!-- 账号root对应的虚拟库,这个库保持默认比较好 -->
</user>
<user name="test">                                   <!-- 这里是给虚拟库设定一个账号叫test，并且作为默认账号 -->
        <property name="password">123456</property>    <!-- 账号test的密码 -->
        <property name="schemas">aiurt-platform</property>   <!-- 账号test对应的虚拟库,这个库保持默认比较好 -->
        <property name="readOnly">true</property>    <!-- 说明这个账号是只读账号 -->
</user>
```

6、配置真实数据库信息

打开`schema.xml`，编辑如下地方：

```
<!--schema定义逻辑库的标签,配置逻辑库 name：表示逻辑库的名称  dataNode表示逻辑库关联的节点名称 -->
<schema name="aiurt-platform" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1"></schema>
     <!-- dataNode表示定义数据节点的标签 name：节点名称，必须和上面schema的dataNode值保持一致  dataHost关联的主机名称
	 database=表示真实数据库的名称-->
	<dataNode name="dn1" dataHost="localhost1" database="aiurt-platform" />
	<!-- 定义真是服务所在的地址 name：表示数据主机的名称  -->
	<dataHost name="localhost1" maxCon="1000" minCon="10" balance="1"
			  writeType="0" dbType="mysql" dbDriver="native" switchType="1"  slaveThreshold="100">
		<!-- 定义真是服务所在的地址 name：表示数据主机的名称  -->
		<heartbeat>select user()</heartbeat>
		<!-- 配置主节点的信息 host随便起-->
		<writeHost host="hostM1" url="192.168.1.27:9088" user="root" password="123456">
		  <!-- 配置读节点的信息 可以配置多个-->
			<readHost host="hostS1" url="192.168.1.27:9088" user="root" password="123456" />
		</writeHost>
		<writeHost host="hostM2" url="192.168.1.27:9089" user="root" password="123456" />
		  <!-- 配置读节点的信息 可以配置多个-->
			<readHost host="hostS2" url="192.168.1.27:9089" user="root" password="123456" />
		</writeHost>
	</dataHost>
```

**参数说明：**

`schema`:`name`属性是自定义的  dataNode表示数据库的节点信息  dn1表示逻辑库

`datahost`:

```text
<!--参数介绍-->
		<!--balance 0表示所有的读操作都会发往writeHost主机 -->  
		<!--1表示所有的读操作发往readHost和闲置的主节点中-->
		<!--writeType=0 所有的写操作都发往第一个writeHost主机，当第一个writeHost宕机时，切换到第二个writeHost-->	
		<!--writeType=1 所有的写操作随机发往writeHost中-->
		<!--dbType 表示数据库类型 mysql/oracle-->
		<!--dbDriver="native"  固定参数 不变-->
		<!--switchType=-1 表示不自动切换, 主机宕机后不会自动切换从节点-->
		<!--switchType=1  表示会自动切换(默认值)如果第一个主节点宕机后,Mycat会进行3次心跳检测,如果3次都没有响应,则会自动切换到第二个主节点-->
		<!--并且会更新/conf/dnindex.properties文件的主节点信息 localhost1=0 表示第一个节点.该文件不要随意修改否则会出现大问题-->
```

7、启动mycat

检查好格式并保存之后，就到 mycat 目录下的/bin/里`./mycat start`就启动 mycat 了。启动成功之后，8066 和 9066 都是被监听的，如图

![image-20221214165120918](https://gitee.com/studentgitee/note-picture/raw/master/image-20221214165120918.png)

### 启动故障排错

如果启动 mycat 失败，可以去 logs 文件夹里看日志.

### 验证

使用navicat软件连接到mycat虚拟库

```
# mysql的端口号默认为8066，所以我们连接的时候也需要把端口写成8066
```

通过后台服务连接到mycat后，通关postman访问某个接口，此时可以从mycat.log这个文件日志中看出，该条sql走的是哪个数据库的。

![image-20221215103847319](https://gitee.com/studentgitee/note-picture/raw/master/image-20221215103847319.png)

# 四、nginx负载均衡搭建

## 原理

简单来说所谓的负载均衡就是把很多请求进行分流，将他们分配到不同的服务器去处理。比如我有3个服务器，分别为A、B、C，然后使用Nginx进行负

载均衡，使用轮询策略，此时如果收到了9个请求，那么会均匀的将这9个请求分发给A、B、C服务器，每一个服务器处理3个请求，这样的话我们可以利

用多台机器集群的特性减少单个服务器的压力。简言之，负载均衡实际上就是将大量请求进行分布式处理的策略。

### 正向代理

正向代理（Forward Proxy）最大的特点是，客户端非常明确要访问的服务器地址，它代理客户端，替客户端发出请求。比如：FQ（警告⚠️：FQ操作违

反相关法律规定，本文只是为了解释正向代理向读者举个例子，仅供学习参考，切勿盲目FQ）。

![1800425-20200427222938602-1590961015](https://gitee.com/studentgitee/note-picture/raw/master/1800425-20200427222938602-1590961015.png)

假设客户端想要访问 Google，它明确知道待访问的服务器地址是 https://www.google.com/，但由于条件限制，它找来了一个能够访问到 Google 的”朋

友”：代理服务器。客户端把请求发给代理服务器，由代理服务器代替它请求 Google，最终再将响应返回给客户端。这便是一次正向代理的过程，该过程

中服务器并不知道真正发出请求的是谁。

### 反向代理

那么，随着请求量的爆发式增长，服务器觉得自己一个人始终是应付不过来，需要兄弟服务器们帮忙，于是它喊来了自己的兄弟以及代理服务器朋友。此

时，来自不同客户端的所有请求实际上都发到了代理服务器处，再由代理服务器按照一定的规则将请求分发给各个服务器。

这就是反向代理（Reverse Proxy），反向代理隐藏了服务器的信息，它代理的是服务器端，代其接收请求。换句话说，反向代理的过程中，客户端并不

知道具体是哪台服务器处理了自己的请求。如此一来，既提高了访问速度，又为安全性提供了保证。

![1800425-20200427223008777-503217438](https://gitee.com/studentgitee/note-picture/raw/master/1800425-20200427223008777-503217438.png)

在这之中，反向代理需要考虑的问题是，如何进行均衡分工，控制流量，避免出现局部节点负载过大的问题。通俗的讲，就是如何为每台服务器合理的分

配请求，使其整体具有更高的工作效率和资源利用率。

### 常用算法

**源地址哈希法：**根据获取客户端的IP地址，通过哈希函数计算得到一个数值，用该数值对服务器列表的大小进行取模运算，得到的结果便是客服端要访问

服务器的序号。采用源地址哈希法进行负载均衡，同一IP地址的客户端，当后端服务器列表不变时，它每次都会映射到同一台后端服务器进行访问。

```
upstream backend {
    ip_hash;
    server 192.168.136.136 ;
    server 192.168.136.136:81;
    server 192.168.136.136:82 ;
    server 192.168.136.136:83;
}
```

**轮询法：**将请求按顺序轮流地分配到后端服务器上，它均衡地对待后端的每一台服务器，而不关心服务器实际的连接数和当前的系统负载。

```
upstream backend {
    server 192.168.136.136;
    server 192.168.136.136:81;
    server 192.168.136.136:82;
    server 192.168.136.136:83;
}
```

**随机法：**通过系统的随机算法，根据后端服务器的列表大小值来随机选取其中的一台服务器进行访问。

```
upstream server_group { 
   random; 
   server backend1.example.com; 
   server backend2.example.com; 
   server backend3.example.com; 
   server backend4.example.com;
}
```

**加权轮询法：**不同的后端服务器可能机器的配置和当前系统的负载并不相同，因此它们的抗压能力也不相同。给配置高、负载低的机器配置更高的权重，

让其处理更多的请；而配置低、负载高的机器，给其分配较低的权重，降低其系统负载，加权轮询能很好地处理这一问题，并将请求顺序且按照权重分配

到后端。

```
upstream backend {
    server 192.168.136.136 weight=1;
    server 192.168.136.136:81 weight=2;
    server 192.168.136.136:82 weight=3;
    server 192.168.136.136:83 weight=4;
}
```

**加权随机法：**与加权轮询法一样，加权随机法也根据后端机器的配置，系统的负载分配不同的权重。不同的是，它是按照权重随机请求后端服务器，而非

顺序。

**最小连接数法：**由于后端服务器的配置不尽相同，对于请求的处理有快有慢，最小连接数法根据后端服务器当前的连接情况，动态地选取其中当前积压连

接数最少的一台服务器来处理当前的请求，尽可能地提高后端服务的利用效率，将负责合理地分流到每一台服务器。

```
upstream backend {
    least_conn;
    server 192.168.136.136 ;
    server 192.168.136.136:81;
    server 192.168.136.136:82 ;
    server 192.168.136.136:83;
}
```



## 修改配置

nginx.conf修改成如下：

```
upstream myServer {
        server 192.168.1.189:9001;
        server 192.168.1.189:9002;
    }

    server {
        listen       8887;
        server_name  localhost;

      	location ^~/api/ {
      	# myServer后面的/不能少
            proxy_pass http://myServer/;
            root   html;
            index  index.html index.htm;
       }
```

# 五、Keepalived+Nginx 集群(主从模式)搭建

## 原理

keepalived是基于VRRP协议实现的保证集群高可用的一个服务软件，主要功能是实现真机的故障隔离和负载均衡器间的失败切换，防止单点故障。一个

LVS服务会有2台服务器运行Keepalived，一台为主服务器（MASTER），一台为备份服务器（BACKUP），但是对外表现为一个虚拟IP，主服务器会发送

特定的消息给备份服务器，当备份服务器收不到这个消息的时候，即主服务器宕机的时候， 备份服务器就会接管虚拟IP，继续提供服务，从而保证了高

可用性。Keepalived是VRRP的完美实现。

## VRRP 协议简介

在现实的网络环境中，两台需要通信的主机大多数情况下并没有直接的物理连接。对于这样的情况，它们之间路由怎样选择？主机如何选定到达目的主机

的下一跳路由，这个问题通常的解决方法有二种：

- 在主机上使用动态路由协议(RIP、OSPF等)
- 在主机上配置静态路由

很明显，在主机上配置动态路由是非常不切实际的，因为管理、维护成本以及是否支持等诸多问题。配置静态路由就变得十分流行，但路由器(或者说默

认网关default gateway)却经常成为单点故障。VRRP的目的就是为了解决静态路由单点故障问题，VRRP通过一竞选(election)协议来动态的将路由任务交

给LAN中虚拟路由器中的某台VRRP路由器。

## Keepalived+Nginx实现高可用的思路

1. 请求不要直接打到Nginx上，应该先通过Keepalived（这就是所谓虚拟IP，VIP）。
2. Keepalived应该能监控Nginx的生命状态，提供一个用户自定义的脚本，定期检查Nginx进程状态，进行权重变化,，从而实现Nginx故障切换。

## VRRP 工作流程

### 初始化

路由器启动时，如果路由器的优先级是255(最高优先级，路由器拥有路由器地址)，要发送VRRP通告信息，并发送广播ARP信息通告路由器IP地址对应的

MAC地址为路由虚拟MAC，设置通告信息定时器准备定时发送VRRP通告信息，转为MASTER状态；否则进入BACKUP状态，设置定时器检查定时检查是

否收到MASTER的通告信息。

### Master

- 设置定时通告定时器；

- 用VRRP虚拟MAC地址响应路由器IP地址的ARP请求；

- 转发目的MAC是VRRP虚拟MAC的数据包；

- 如果是虚拟路由器IP的拥有者，将接受目的地址是虚拟路由器IP的数据包，否则丢弃；

- 当收到shutdown的事件时删除定时通告定时器，发送优先权级为0的通告包，转初始化状态；

- 如果定时通告定时器超时时，发送VRRP通告信息；

- 收到VRRP通告信息时，如果优先权为0，发送VRRP通告信息；否则判断数据的优先级是否高于本机，或相等而且实际IP地址大于本地实际IP，设置定

  时通告定时器，复位主机超时定时器，转BACKUP状态；否则的话，丢弃该通告包；

### Backup

- 设置主机超时定时器；

- 不能响应针对虚拟路由器IP的ARP请求信息；

- 丢弃所有目的MAC地址是虚拟路由器MAC地址的数据包；

- 不接受目的是虚拟路由器IP的所有数据包；

- 当收到shutdown的事件时删除主机超时定时器，转初始化状态；

- 主机超时定时器超时的时候，发送VRRP通告信息，广播ARP地址信息，转MASTER状态；

- 收到VRRP通告信息时，如果优先权为0，表示进入MASTER选举；否则判断数据的优先级是否高于本机，如果高的话承认MASTER有效，复位主机超

  时定时器；否则的话，丢弃

## 实战

### 1、准备两台虚拟机分别安装 nginx 和 keepalived

192.168.1.21

192.168.1.27

1、安装nginx

2、安装 keepalived

```
yum install keepalived –y
```

安装完成之后,默认在 etc 里面生成 keepalived 目录,该目录下存在配置文件 keepalived.conf

![image-20221215181240066](https://gitee.com/studentgitee/note-picture/raw/master/image-20221215181240066.png)

### 2、修改 keepalived.conf 配置文件

主服务器和从服务器只有两个地方不一样,两个不一样的配置都在 vrrp_instance VI_1 中

state: 主服务器的值是 MASTER ,副本服务器的值是 BACKUP

priority: 主服务器的值比副本服务器的值大即可(例如:主服务器设置为 100,副本服务器设置为 80)

```
global_defs {
   notification_email { # keepalived 服务器宕机异常出现的时候,发送通知邮件,可以配置多个邮箱地址
    1412556053@qq.com # 收件人邮箱1
    # ***@**.com      # 收件人邮箱2
   }
   notification_email_from 1412556053@qq.com # 邮箱发件人
   smtp_server stmp.qq.com # 邮箱服务器地址
   smtp_connect_timeout 30 # 超时时间
   router_id LVS_DEVEL # 路由 id
   vrrp_skip_check_adv_addr # 默认不跳过检查
   # vrrp_strict # 这个东西要注释掉,这个东西要注释掉,这个东西要注释掉...
   vrrp_gna_interval 0 # 单位秒,在一个网卡上每组消息之间的延迟时间,默认为 0
   script_user root  
   enable_script_security
}


vrrp_script chk_haproxy
{
    script "/etc/keepalived/chk_nginx.sh" # keepalived 监测 nginx 的监本路径和名称
    interval 2 #检测时间间隔
    weight 2  #权重
}


# vrrp 实例,我们集群设置,多机配置,除了 state 和 priority 不一样之外,其它的配置都是一样的
# 主实例 state 为 MASTER , priority 的值高于副本实例的值
# 副本实例 state 为 BACKUP , priority 的值低于主实例的值
vrrp_instance VI_1 {
    state BACKUP  # 服务器状态,MASTER 代表主服务器, BACKUP 是备份服务器
    interface ens192 # 通信端口,通过 ifconfig 命令可以看到,根据自己的机器配置
    virtual_router_id 51 # 虚拟路由 ID ,主实例和副本实例保持一致
    priority 100 # 权重比,主服务器的 priority 比副本服务器大即可
    advert_int 1 # 心跳间隔,单位秒, keepalived 多机器集群通过心跳检测,如果发送心跳没反应,就立刻接管
    authentication { # 服务器之间的通信密码
        auth_type PASS
        auth_pass 1111
    }
	
    track_script { # keepalived 的监测脚本,与 vrrp_script 定义的名称一致
      chk_haproxy
    }
	
    virtual_ipaddress { # 自定义虚拟 ip ,可以配置多个,每行一个
        192.168.1.165
    }
}
```

### 3、编写 keepalived 监测 nginx 的脚本

我们在 keepalived.conf 中 vrrp_script chk_haproxy 配置的监测脚本路径和名称如下: /etc/keepalived/chk_nginx.sh

所以我们需要在 /etc/keepalived 目录下新建一个 chk_nginx.sh 的脚本文件,文件内容如下

```
#!/bin/bash
# nginx 挂掉之后, keepalived 重新启动 nginx ,若不能启动 则关闭当前 keepalived
status=`ps -ef|grep -w  nginx|grep -v grep|wc -l`
echo ${status}
if [ ${status} -eq 0 ]; then
    systemctl start nginx.service
    sleep 2
    status2=`ps -ef|grep -w  nginx|grep -v grep|wc -l`
    echo ${status2}
    if [ ${status2} -eq 0  ]; then
            systemctl stop keepalived.service
    fi
fi
```

然后运行 chmod +x chk_nginx.sh 命令为脚本文件添加可执行权限

```
chmod +x chk_nginx.sh
```

### 4、启动 nginx 和 keepalived

分别启动两台服务器上的 nginx 和 keepalived

```
// 启动 nginx
./nginx

// 启动 keepalived
systemctl start keepalived.service

// 查看 nginx 是否成功启动
ps -ef | grep nginx

// 查看 keepalived 是否成功启动
ps -ef | grep keepalived


#扩展
// 重启keepalived
systemctl restart keepalived.service

// 停止keepalived
systemctl stop keepalived.service

// 查看keepalived状态
systemctl status keepalived.service
```

### **5、测试**

配置信息如下:

```
# 主服务器:　　192.168.1.27

# 从服务器:　　192.168.1.21

# 虚拟IP:　　192.168.1.165
```

1、浏览器访问 192.168.1.165

2、关闭主机 192.168.1.27 上的 nginx 和 keepalived ,再次访问虚拟 IP ,发现可以正常访问,并且访问的是从机

如果主服务器上的nginx 和 keepalived恢复后，虚拟ip会漂移到主服务器上。

### 6、扩展

如果你的服务器有很多的nginx进程，那么上面chk_nginx.sh脚本就不生效，则可以使用下面的脚本

```
#!/bin/bash
# nginx 挂掉之后, keepalived 重新启动 nginx ,若不能启动 则关闭当前 keepalived
file=/usr/local/nginx/logs/nginx.pid
if [ ! -f "$file" ]; then
     #尝试启动nginx
    /usr/local/nginx/sbin/nginx
    if [ ! -f "$file" ]; then
        systemctl stop keepalived.service
    fi
fi
```

