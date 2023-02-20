---
title: canal文档教程
tags: 常用框架
categories: 常用框架
cover: https://gitee.com/studentgitee/note-picture/raw/master/md007.png
---
![image-20230210143018927](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210143018927.png)

# canal学习大纲

## canal是什么？

> canal是阿里开源的数据同步工具，基于bin log可以将数据库同步到其他各类数据库中，目标数据库支持mysql,postgresql,oracle,redis,MQ,ES等

## 准备

环境是在centos7,在win10安装可能会出现问题

mysql8.0

elasticsearch7.10.1

canal1.1.5

canal下载地址：https://github.com/alibaba/canal/releases

![image-20230210201616363](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210201616363.png)

## 配置mysql以及创建es索引

因为canal是基于bin log来实现的，所以要开启binlog，并且设置binlog模式为行模式。

### **开启bin log**


修改mysql8的配置文件my.cnf

```javascript
sudo vim /etc/my.cnf
```

**添加以下内容**

```
log-bin=mysql-bin
binlog_format=ROW
```

**登录mysql查看是否开启成功**

```java
SHOW variables like '%log_bin%'
```

![image-20230210202152999](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210202152999.png)

### 创建canal用户

我们创建一个canal用户，专门用于canal同步数据库使用
登录mysql，执行

```javascript
# 注意大小写

CREATE USER canal IDENTIFIED BY 'canaL';

GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';

FLUSH PRIVILEGES;
```

### 创建es索引

```
PUT cs_line
{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "line_name":{
         "type": "keyword"
      },
      "line_code":{
         "type": "keyword"
      },
      "description":{
        "type": "keyword"
      }
    }
  }
}
```

## 安装服务端 canal-deployer

canal官方提供了一些配置指导，包括mysql同步mysql、mq、es的配置，要擅用[官方指南](https://github.com/alibaba/canal/wiki)

### 1、解压

```
tar -zxvf canal.deployer-1.1.5.tar.gz
```

### 2、修改配置文件

```javascript
vi conf/example/instance.properties
```

修改内容:
设置数据库地址，账号和密码设置为上述创建的专用账号canal

```javascript
##################################################
## slaveId只需要跟mysql的不一致就可以，在同一局域网内要唯一
canal.instance.mysql.slaveId=903

# enable gtid use true/false
canal.instance.gtidon=false

# position info，address是我们要连接的mysql数据库IP和端口
canal.instance.master.address=192.168.1.29:3310
canal.instance.master.journal.name=
canal.instance.master.position=
canal.instance.master.timestamp=
canal.instance.master.gtid=

# rds oss binlog
canal.instance.rds.accesskey=
canal.instance.rds.secretkey=
canal.instance.rds.instanceId=

# table meta tsdb info
canal.instance.tsdb.enable=true
#canal.instance.tsdb.url=jdbc:mysql://127.0.0.1:3306/canal_tsdb
#canal.instance.tsdb.dbUsername=canal
#canal.instance.tsdb.dbPassword=canal

#canal.instance.standby.address =
#canal.instance.standby.journal.name =
#canal.instance.standby.position =
#canal.instance.standby.timestamp =
#canal.instance.standby.gtid=

# username/password这里要改成我们需要连接的数据的账号密码
canal.instance.dbUsername=canal
canal.instance.dbPassword=canal
canal.instance.connectionCharset = UTF-8
# enable druid Decrypt database password
canal.instance.enableDruid=false
#canal.instance.pwdPublicKey=MFwwDQYJKoZIhvcNAQEBBQADSwAwSAJBALK4BUxdDltRRE5/zXpVEVPUgunvscYFtEip3pmLlhrWpacX7y7GCMo2/JM6LeHmiiNdH1FWgGCpUfircSwlWKUCAwEAAQ==

# table regex
canal.instance.filter.regex=.*\\..*
# table black regex
canal.instance.filter.black.regex=mysql\\.slave_.*
# table field filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.field=test1.t_product:id/subject/keywords,test2.t_company:id/name/contact/ch
# table field black filter(format: schema1.tableName1:field1/field2,schema2.tableName2:field1/field2)
#canal.instance.filter.black.field=test1.t_product:subject/product_image,test2.t_company:id/name/contact/ch

# mq config
canal.mq.topic=example
# dynamic topic route by schema or table regex
#canal.mq.dynamicTopic=mytest1.user,mytest2\\..*,.*\\..*
canal.mq.partition=0
# hash partition config
#canal.mq.partitionsNum=3
#canal.mq.partitionHash=test.table:id^name,.*\\..*
#canal.mq.dynamicTopicPartitionNum=test.*:4,mycanal:6
#################################################

```

### **3、启动**

```
./bin/startup.sh
```

查看日志

```
cat logs/canal/canal.log
```

日志出现这样代表启动成功

```
2023-02-10 17:49:16.901 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## set default uncaught exception handler
2023-02-10 17:49:17.000 [main] INFO  com.alibaba.otter.canal.deployer.CanalLauncher - ## load canal configurations
2023-02-10 17:49:17.026 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## start the canal server.
2023-02-10 17:49:17.173 [main] INFO  com.alibaba.otter.canal.deployer.CanalController - ## start the canal server[172.17.0.1(172.17.0.1):11115]
2023-02-10 17:49:20.159 [main] INFO  com.alibaba.otter.canal.deployer.CanalStarter - ## the canal server is running now ......
```

**全量同步（这一步可以忽略不做）**

1、在conf/example/instance.properties中修改

```javascript
# 全量同步

canal.instance.master.journal.name=mysql-bin.000001

canal.instance.master.position=0

#2019-01-01 00:00:00 上一次更新的时间
canal.instance.master.timestamp=1546272000000
```

2、如果之前同步过，想要重新做全量同步，那么需要删除con/example/meta.dat文件，这个文件会记录上次同步的时间和binlog位置

## 安装客户端canal.adapter

### 1、解压

```javascript
tar -zxvf canal.adapter-1.1.5.tar.gz
```

### 2、修改配置文件

```javascript
vim conf/application.yml
```

修改内容

```
server:
  port: 8081
spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
    default-property-inclusion: non_null

canal.conf:
  mode: tcp #tcp kafka rocketMQ rabbitMQ
  flatMessage: true
  zookeeperHosts:
  syncBatchSize: 1000
  retries: 0
  timeout:
  accessKey:
  secretKey:
  consumerProperties:
    # canal tcp consumer
    canal.tcp.server.host: 192.168.1.29:11115
    canal.tcp.zookeeper.hosts:
    canal.tcp.batch.size: 500
    canal.tcp.username:
    canal.tcp.password:
# 这里是我们需要修改的相关的数据库信息
  srcDataSources:
    defaultDS:
      url: jdbc:mysql://192.168.1.29:3310/aiurt-platform?useUnicode=true
      username: canal
      password: canal
  canalAdapters:
  - instance: example # canal instance Name or mq topic name
    groups:
    - groupId: g1
      outerAdapters:
      - name: logger
# 我们的es版本是7.10.1，所以写es7,低于7的版本就写es;hosts如果写9200端口的，mode就要写rest
      - name: es7
        hosts: 192.168.1.29:9200 # 127.0.0.1:9200 for rest mode，集群就使用,隔开写
        properties:
          mode: rest # or rest
          # security.auth: test:123456 #  only used for rest mode
          cluster.name: elasticsearch
#        - name: kudu
#          key: kudu
#          properties:
#            kudu.master.address: 127.0.0.1 # ',' split multi address
```

### 3、修改es配置文件

查看`conf/es7`文件夹会发现里面默认有3个yml文件

如果需要配置mysql到es的同步，那么就需要在es路径下配置字段的映射，adapter默认会加载es路径下的所有yml文件。一个配置文件表示一张表的mapping，我们删除默认的yml，创建一个cs_line.yml用于cs_line表的同步

内容如下：

```
dataSourceKey: defaultDS # 这里的key与上述application.yml中配置的数据源保持一致
destination: example     # 默认为example,与application.yml中配置的instance保持一致
groupId: g1              
esMapping:
  _index: cs_line        # es中索引的名称
  _id: id
#  upsert: true
#  pk: id
  sql: "select id,line_name,line_code,description  from cs_line"
#  objFields:
#    _labels: array:;
 # etlCondition: "where "
  commitBatch: 30
```

4、启动服务

```
./bin/startup.sh
```

同理通过查看日志来看是否启动成功

```javascript
cat logs/adapter/adapter.log 
```

5、启动后，修改数据库数据，查看日志会发现已经有输出了

```javascript
cat logs/adapter/adapter.log
```

日志如下

![image-20230210205151313](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210205151313.png)

## 遇到的问题以及解决办法

### 问题1：CanalClientException: 微服务连接不上windows上的canal

```
CanalClientException: java.io.IOException: end of stream when reading header

defined in com.xpand.starter.canal.config.CanalClientConfiguration: Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.xpand.starter.canal.client.CanalClient]: Factory method 'canalClient' threw exception; nested exception is com.alibaba.otter.canal.protocol.exception.CanalClientException: java.io.IOException: end of stream when reading header
```

经过检查是主机IP和端口错误。

**解决办法：**去canal的安装目录下，logs>>canal下的canal日志文件里面查找 IP和端口号

start the canal server[xx.xx.xx.xx():11111]，端口号为11111

在微服务的配置文件里面用上面的IP和端口号，问题解决

### 问题2：java.util.NoSuchElementException

没有找到对应字段导致

检查下canal配置文件中的字段是否在es mapping中有对应的，大小写是否一致，是否有遗漏

因为我的操作是mysql同步至es，所以这里说明几项容易出错的地方：

1、canal配置文件中的sql中是否大小写一致，canal是区分大小写的

2、sql中设置的别名是否与es mappings中的名称一致，允许es中的部分字段为空，但是不允许sql中查询出来的字段在es mappings中找不到对应的字段

3、canal配置文件中的dataSourceKey是否正确，其对应到canal application.yml配置文件中的数据库是否正确

### 问题3：adapter启动报错

 adapter启动报错：something goes wrong when starting up the canal client adapters: java.lang.NullPointerException: null

1、这个报错是空指针报错，很明显是哪里获取为空的，这种错误没有固定的原因，但大概率上可以锁定配置文件的问题

2、也有可能是你没开启日志的时候，mysql中已经有数据了，这时就出现了问题，所以在同步的时候，要么mysql和es中的数据都是空，要么mysql和es的数据是一样的