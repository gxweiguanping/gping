---
title: nginx常用命令
tags: nginx
categories: nginx
cover: https://gitee.com/studentgitee/note-picture/raw/master/48540923dd54564e717962685c908285d1584f1d.jpg
---
![image-20230210143018927](https://gitee.com/studentgitee/note-picture/raw/master/image-20230210143018927.png)

## 1、nginx常用命令

```
/usr/local/nginx/sbin/nginx -tc /usr/local/nginx/conf/nginx.conf  #验证配置文件nginx.conf

/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf #指定配置文件启动nginx

/usr/local/nginx/sbin/nginx -s reload -c /usr/local/nginx/conf/nginx.conf #指定配置文件重启nginx

注：/usr/local/nginx/ 目录视自己的安装情况而定。配置文件同样根据自己的命名习惯指定
```
