---
title: docker常用命令
tags: 容器部署
categories: 容器部署
cover: https://gitee.com/studentgitee/note-picture/raw/master/u=202403325,594744356&fm=253&fmt=auto&app=138&f=PNG.webp
---
![image-20230210143018927](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210143018927.png)

## 官网地址

docker文档官网地址：https://vuepress.mirror.docker-practice.com/

## docker安装

- 所有平台通用安装(bash方式）

  ```
  $ curl -fsSL get.docker.com -o get-docker.sh
  $ sudo sh get-docker.sh --mirror Aliyun
  ```

- 启动docker

  ```  
  $ sudo systemctl enable docker  #开机自启
  $ sudo systemctl start docker   #启动docker
  $ sudo systemctl restart docker   #重启docker
  $ sudo systemctl stop docker   #停止docker
  $ sudo systemctl status docker   #查看docker状态
  ```

- 默认情况下，`docker` 命令会使用 [Unix socket (opens new window)](https://en.wikipedia.org/wiki/Unix_domain_socket)与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

  ```
  $ sudo groupadd docker  #创建一个docker组
  $ sudo usermod -aG docker $USER #将当前用户加入组
  ```

- 配置加速镜像

  目前主流 Linux 发行版均已使用 [systemd (opens new window)](https://systemd.io/)进行服务管理，这里介绍如何在使用 systemd 的 Linux 发行版中配置镜像加速器。

  1、请首先执行以下命令，查看是否在 `docker.service` 文件中配置过镜像地址。

  ```
  $ systemctl cat docker | grep '\-\-registry\-mirror'
  ```

  2、如果该命令有输出，那么请执行 `$ systemctl cat docker` 查看 `ExecStart=` 出现的位置，修改对应的文件内容去掉 `--registry-mirror` 参数及其值，并按接下来的步骤进行配置。

  3、如果以上命令没有任何输出，那么就可以在 `/etc/docker/daemon.json` 中写入如下内容（如果文件不存在请新建该文件）：

  ```
  {
    "registry-mirrors": [
      "https://hub-mirror.c.163.com",
      "https://mirror.baidubce.com"
    ]
  }
  ```

  > 注意，一定要保证该文件符合 json 规范，否则 Docker 将不能启动。

  4、重新启动服务。

  ```bash
  $ sudo systemctl daemon-reload
  $ sudo systemctl restart docker
  ```

​       5、查看镜像是否配置成功

​       执行 `$ docker info`，如果从结果中看到了如下内容，说明配置成功。

![image-20210902115057612](https://gitee.com/studentgitee/note-picture/raw/master/image-20210902115057612.png)

## docker的核心概念

- **镜像**（`Image`）：定义:一个镜像代表着一个软件如: mysql镜像、redis镜像、nginx镜像。也可以理解成印钱的模板

- **容器**（`Container`）：定义:基于某个镜像运行一次就是生成一个程序实例，一个程序实例称之为一个容器。可以理解成钱

- **仓库**（`Repository`）：存储docker中的所有镜像。可以理解成放印钱模板的地方（远程仓库地址:https://registry.hub.docker.com/）

  **docker拉取镜像并启动容器**

![image-20210902121101783](https://gitee.com/studentgitee/note-picture/raw/master/image-20210902121101783.png)

**docker运行流程图**

![image-20210902121446529](https://gitee.com/studentgitee/note-picture/raw/master/image-20210902121446529.png)

**docker   run的运行图**

![image-20210902143306179](https://gitee.com/studentgitee/note-picture/raw/master/image-20210902143306179.png)

## 辅助命令

### 查看更详细的信息

```
docker info
```

### 命令帮助

```
docker --help
```

### 查看docker的版本号

```
docker version 
```

## 镜像命令

### 查找镜像

```
docekr search 
```

### 拉取镜像

```
docker pull 镜像名称:版本号  
```

### 查看本地仓库的镜像

```
docker images 
-a   #闲置镜像所有信息
-q   #只显示镜像的id
```

### 删除镜像

```
docker image rm 镜像id    
# -f  强制删除
```

### 强制删除所有镜像

```
docker image rm -f $(docker images -a)   
```

### 删除none的镜像

```
docker rmi $(docker images | grep "none" | awk '{print $3}')
```

### 进入镜像内部

```
docker run -it --entrypoint sh 镜像id
```

## 容器命令

### 导入容器

```
# 导入容器
docker load -i tar名称
```

### 导出容器

```
# 将tar包导入到docker本地仓库中
docker save -o xxx.tar 镜像名称:镜像版本
```

### 运行容器

```
docker run 镜像名---------------# 镜像名新建并启动容器
--name                         # 别名为容器起一个名字
-d                             # 启动守护式容器(在后台启动容器）
-p                             # 映射端口号:原始端口号
-it                            # it:交互式启动
--restart=always               # 重启docker时自动启动该容器
-v                             # 挂载容器目录  宿主机目录:容器内目录
```

### 查看运行的容器

```
docker ps                       # 列出所有正在运行的容器
-a                              # 正在运行的和历史运行过的容器
-9                              # 静默模式，只显示容器编号
```

### 查看容器的日志

```
docker logs 容器id
```

### 停止|关闭|重启容器

```
docker start  容器名字或者容器id        # 开启容器
docker restart   容器名或者容器id      # 重启容器
docker stop   容器名或者容器id        # 正常停止容器运行
docker kill  容器名或者容器id         #立即停止容器运行
```

### 进入容器内部

```
docker exec -it 容器id /bin/bash
```

###  删除容器

```
docker rm -f  容器id           #-f强制删除

# 删除所有容器
docker rm -f $(docker ps -aq)    

# 清理所有处于终止状态的容器
docker container prune
```

## 数据卷配置

数据卷供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

- 数据卷 可以在容器之间共享和重用
- 对 数据卷的修改会立马生效
- 对 数据卷的更新，不会影响镜像
- 数据卷 默认会一直存在，即使容器被删除

**作用：用来实现容器和宿主机之间的数据通信**

```
#自定义数据卷
docker run -v 绝对路径:容器内的路径

#自动创建数卷
docker run -v 卷名（会自动创建）:容器内的路径

#查看数据卷
docker volume ls 

#查看某个数据卷的详细内容
docker volume inspect 卷名

#创建数据卷
docker volume create 卷名

#删除目前没有使用的数据卷
docker volume prune 
docker volume rm 卷名
```

## dockerfile

### 构建镜像

```
docker build -t aiurt-platfrom:3.2.0 .

# docker build需要在dockerfile同目录执行
```

