---
layout: post
title:  "使用docker部署Fuseki服务"
data:   2019-05-20 11:04:58
categories: docker
tags: fuseki docker 部署
---

* content
{:toc}


# 参考资料

```
# docker 官方jena-fuseki 源
https://hub.docker.com/r/stain/jena-fuseki/
```
# 开始部署
## 简单版本
```
# 下载镜像
docker pull stain/jena-fuseki
# 运行容器
# 这里-p 3030:3030 将本机3030端口映射为容器3030端口
docker run -p 3030:3030 stain/jena-fuseki
# 访问地址
http://localhost:3030/
```





### 成功后会显示用户名密码
```
localhost:~ zhangqiankun$ docker run -p 3030:3030 stain/jena-fuseki
###################################
Initializing Apache Jena Fuseki

Randomly generated admin password:

admin=cS42A7o1Niv1QOM

###################################
```

![image](https://github.com/whydejavu/whydejavu.github.io/raw/master/resource/image/blog/blog-docker-fuseki-01.png)
## 修改端口

```
# 将8080映射到Fuseki
localhost:~ zhangqiankun$ docker run -p 8080:3030 stain/jena-fuseki
###################################
Initializing Apache Jena Fuseki

Randomly generated admin password:

admin=UtD2iHHin2PpvTt

###################################
[2019-05-16 09:51:13] Server     INFO  Apache Jena Fuseki 3.10.0
[2019-05-16 09:51:13] Config     INFO  FUSEKI_HOME=/jena-fuseki
[2019-05-16 09:51:13] Config     INFO  FUSEKI_BASE=/fuseki
[2019-05-16 09:51:13] Config     INFO  Shiro file: file:///fuseki/shiro.ini
[2019-05-16 09:51:15] Server     INFO  Started 2019/05/16 09:51:15 GMT on port 3030

# 登录地址
http://localhost:8080/index.html
```
![image](https://github.com/whydejavu/whydejavu.github.io/raw/master/resource/image/blog/blog-docker-fuseki-02.png)

## 最终启动方式
### 启动命令介绍
```
# 后台启动+修改名称+修改端口+ 修改内存+更改映射地址
docker run -d --name fuseki -p 9030:3030  -e JVM_ARGS=-Xmx2g -e ADMIN_PASSWORD=admin -v /Users/zhangqiankun/docker/dockerdata/jena-fuseki/data/fuseki:/fuseki stain/jena-fuseki
# 访问地址
http://localhost:9030/index.html  admin/admin
```
### 所有信息会保存到本地
```
localhost:fuseki zhangqiankun$ pwd
/Users/zhangqiankun/docker/dockerdata/jena-fuseki/data/fuseki
localhost:fuseki zhangqiankun$ ls
backups       config.ttl    configuration databases     logs          shiro.ini     system        system_files  templates

```
## docker 命令

```
# --name =fuseki 如果你的名字是fuseki的话
# 查看日志
docker logs fuseki
# 停止
docker stop fuseki
# 启动
docker start fuseki
# 重启
docker restart fuseki
```
## 更新stain/jena-fuseki

```
docker pull stain/jena-fuseki
docker stop fuseki
docker rm fuseki
# 启动
docker run 你前面的命令
```
## 详细请参考docker-jena-fuseki官网
[参考地址](https://hub.docker.com/r/stain/jena-fuseki/)

# 使用jena-fuseki
