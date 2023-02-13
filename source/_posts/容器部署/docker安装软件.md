---
title: docker安装软件
tags: 容器部署
categories: 容器部署
cover: https://gitee.com/studentgitee/note-picture/raw/master/6e1baf91e53fce11a3e242797e747424.webp
---
![image-20230210143018927](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210143018927.png)

# 离线安装docker

**（1）下载地址**https://download.docker.com/linux/static/stable/x86_64/

**（2）通过 FTP工具上传 docker-18.06.1-ce.tgz 到服务器上**

**（3）解压安装包**

```
tar zxf docker-18.06.1-ce.tgz
```

 **（4）将docker 相关命令拷贝到 /usr/bin，方便直接运行命令**

```
sudo cp docker/* /usr/bin/
```

**（5）启动Docker守护程序**

```
sudo dockerd &
```

**（6） 验证是否安装成功，执行docker info命令，若正常打印版本信息则安装成功。**

```
docker info
```

**（7）将docker注册成系统服务（记得kill docker服务后，再执行这一步哦）**

```
sudo vi /usr/lib/systemd/system/docker.service
```

```
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
 
[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
 
[Install]
WantedBy=multi-user.target
```

**（8）启动 / 停止 docker 服务**

```
systemctl start/stop docker
```

**（9）开机自启/取消开机自启 docker 服务**

```
systemctl enable/disable docker
```

# 离线安装redis

```
docker run -it -d --name redis -p6379:6379 redis --bind 0.0.0.0 --protected-mode no  --requirepass 1qaz@2wsx

-it						以交互式模式运行容器，并为容器分配一个伪输入终端(其实是两个命令，不过大多数在一起使用)
-d						表示后台启动
--name					设置容器名称
-p6379:6379				设置端口映射(前面的端口是本地端口，后面的端口是docker的端口)
--bind					设置redis配置中的bind(redis用来限制访问的ip地址 默认为127.0.0.1)
--protected-mode		设置保护模式(默认为yes 开启了保护模式，限制只能本地访问)
redis					镜像名称
```

# 离线安装mysql8

（1）现在有网络的环境下载mysql8镜像包

（2）保存mysql8镜像包到本地后，上传到目标服务器

（3）如果安装的是mysql8那么需要多加一个启动参数：--privileged=true

--lower-case-table-names=1在首次启动的时候就需要加上

如果忘记加上，那么必须新建映射的data目录的（/docker/mysql_master/data）

```
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
```

# 离线上传镜像包到docker

**（1）先在网络的的docker上面pull需要的镜像**

**（2）导出镜像**

```
docker save -o mysql8.tar mysql:8.0

# mysql8.tar tar的名称
# mysql:8.0  mysql是镜像的名称，8.0是版本
```

**（3）将镜像包mysql8.tar上传至无网络的环境上，执行**

```
docker load -i mysql8.tar
```

# 打包运维项目制作镜像

（1）在根目录下aiurt-platform打包项目

```
mvn clean install -Dmaven.test.skip=true -Pdocker

# -p 当前环境
```

（2）制作镜像

```
docker build -t aiurt-platfrom:3.2.0 .

# docker bulild必须和dockerfile在同一目录下使用
```

（3）保存镜像到本地

```
docker save -o xxx.tar 镜像名称:镜像id
```

（4）通过ftp工具将xxx.tar 上传到目标服务器

（5）导入镜像

```
docker load -i xxx.tar
```



# 在docker上部署springcloud

1、打包微服务成为一个镜像

```
eureka:
  client:
    service-url:
      defaultZone: http://192.168.1.30:10001/eureka  #eureka地址     访问地址http://localhost:10001
  instance:
    prefer-ip-address: true   #eureka注册需要加上,以ip的方式注册
```

![image-20211011174031459](https://gitee.com/studentgitee/note-picture/raw/master/image-20211011174031459.png)

2、将jar包传到linux上，后编写Dockerfile文件

```
FROM kdvolder/jdk8
ADD cgkj-eureka-0.0.1-SNAPSHOT.jar /cgkj-eureka-0.0.1-SNAPSHOT.jar
EXPOSE 10001
ENTRYPOINT ["java","-jar","/cgkj-eureka-0.0.1-SNAPSHOT.jar"]
```

```
FROM       -- 依赖哪个环境
ADD        -- 将宿主机的jar包拷贝到docker容器内
EXPOSE     -- 暴露的端口
ENTRYPOINT -- 执行的命令
```

3、构建微服务镜像

```
docker build -t eureka .
```

```
eureka   -- 镜像的名称
.        -- dokcerfile同一个目录
```

4、启动容器

```
docker run -di  --name eureka -p 10001:10001 eureka

如果是配置中心需要添加 --network=host
docker run -d --name configserver --network=host d4eab0e5a068
```

5、查看日志

```
docker logs -f --tail=30 zuul
```

```
zuul   -- 是容器的id
```

# 使用docker-maven-plugin进行部署

1、修改 docker 的配置，开放远程部署端口

```
vi /lib/systemd/system/docker.service

其中 ExecStart=后添加配置
-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```

2、在每个微服务加入 dockermaven 插件

```
 <build>
        <!-- jar包名 -->
        <finalName>zuul</finalName>
        <plugins>
            <!-- docker的maven插件，官网：https://github.com/spotify/docker-maven-plugin -->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.4.13</version>
                <configuration>
                    <!-- 注意imageName一定要是符合正则[a-z0-9-_.]的，否则构建不会成功 -->
                    <!-- 镜像名称包名 -->
                    <imageName>zuul</imageName>
                    <!-- 基础镜像，springboot运行要依赖于jdk -->
                    <baseImage>openjdk:8-jre</baseImage>
                    <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                    <!--远程docker地址 -->
                    <dockerHost>http://192.168.1.30:2375</dockerHost>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

注意：每个微服务的<imageName>是不一样的！

3、进项要构建的项目中，在 cmd 运行

```
mvn clean package docker:build           #打包后构建
```

4、查看 docker 看镜像是否构建成功

# Rancher 管理部署微服务

**简介**

Rancher 是一个开源的企业级容器管理平台。通过 Rancher，企业再也不必自己使用一系列的开源软件去从头搭建容器服务平台。Rancher 提供了在生产环境中使用的管理 Docker 和 Kubernetes 的全栈化容器部署与管理平台。

1、下载镜像

```
docker pull rancher/server
```

2、运行容器

```
docker run -d --name rancher -p 10000:8080 镜像id
```

3、访问rancher

```
http://ip:10000/
```

4、中文描述

![image-20211012172454821](https://gitee.com/studentgitee/note-picture/raw/master/image-20211012172454821.png)

5、创建一个环境后切换到该环境

![image-20211012172542383](https://gitee.com/studentgitee/note-picture/raw/master/image-20211012172542383.png)

6、添加远程docker主机

![image-20211012172626620](https://gitee.com/studentgitee/note-picture/raw/master/image-20211012172626620.png)

# 安装Docker 私服-registry

1、下载 registry

```
docker pull registry
```

2、运行 registry

```
docker run -di --name=registry -p 5000:5000 registry
```

3、访问

```
http://ip:5000/v2/_catalog
```

4、配置 registry

```
vi /etc/docker/daemon.json
"insecure-registries":["ip:5000"]   #添加之后重启 docker，这时 docker 就信任 registry 地址了。
```



**两种上传方式**

1、手动上传

```
docker tag eureka 192.168.1.30:5000/eureka
eureka                                #镜像名称
192.168.1.30:5000/eureka              #打成的镜像名称

docker push 192.168.1.30:5000/eureka  #上传
```

2、DockerMaven 上传

```
<build>
        <finalName>zuul</finalName>
        <plugins>
            <!-- docker的maven插件，官网：https://github.com/spotify/docker-maven-plugin -->
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.4.13</version>
                <configuration>
                    <!-- 注意imageName一定要是符合正则[a-z0-9-_.]的，否则构建不会成功 -->
                    <!-- 192.168.1.30:5000 是私服地址-->
                    <imageName>192.168.1.30:5000/zuul</imageName>
                    <baseImage>openjdk:8-jre</baseImage>
                    <entryPoint>["java", "-jar", "/${project.build.finalName}.jar"]</entryPoint>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                    <dockerHost>http://192.168.1.30:2375</dockerHost>
                </configuration>
            </plugin>
        </plugins>
    </build>
```

# 安装logstash

```
docker run \
--name logstash \
--restart=always \
-p 5044:5044 \
-p 9600:9600 \
-e ES_JAVA_OPTS="-Duser.timezone=Asia/Shanghai" \
-v /home/docker/logstash_backup/logstash/config:/usr/share/logstash/config \
-v /home/docker/logstash_backup/logstash/data:/usr/share/logstash/data \
-v /home/docker/logstash_backup/logstash/logstash/pipeline:/usr/share/logstash/pipeline \
-d logstash:7.10.1
```

