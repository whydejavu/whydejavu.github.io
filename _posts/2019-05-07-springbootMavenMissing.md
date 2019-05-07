---
layout: post
title:  "springboot遇到无法引用maven库的JAR包解决方式"
data:   2019-05-07 13:52:04
categories: spring-boot
tags: spring-boot maven 问题
---

* content
{:toc}


# 问题描述
springboot引用JAR包一般使用国内的镜像源（为啥不用国外的，你懂的-_-|），使用国内的镜像源有时候会出现没有这个JAR包的情况。

解决方式有2种：
1. 切换到国外源
2. 引用本地JAR包





# 错误信息
这里可以看到，错误提示是ali的镜像库没有这个包
```
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building spring-boot-fastDFS 1.0
[INFO] ------------------------------------------------------------------------
Downloading: http://maven.aliyun.com/nexus/content/groups/public/org/csource/fastdfs-client-java/1.27-RELEASE/fastdfs-client-java-1.27-RELEASE.pom
[WARNING] The POM for org.csource:fastdfs-client-java:jar:1.27-RELEASE is missing, no dependency information available
Downloading: http://maven.aliyun.com/nexus/content/groups/public/org/csource/fastdfs-client-java/1.27-RELEASE/fastdfs-client-java-1.27-RELEASE.jar
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 5.372 s
[INFO] Finished at: 2019-05-07T10:40:23+08:00
[INFO] Final Memory: 22M/307M
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal on project spring-boot-fastDFS: Could not resolve dependencies for project com.neo:spring-boot-fastDFS:jar:1.0: Could not find artifact org.csource:fastdfs-client-java:jar:1.27-RELEASE in alimaven (http://maven.aliyun.com/nexus/content/groups/public/) -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/DependencyResolutionException
```
# 方法一：多配置一个镜像源
多配置一个镜像源，重新加载
```
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
     <!-- 多配置一个镜像源-->
    <mirror>
        <id>yonyoucloud</id>
        <name>yonyoucloud</name>
        <url>http://maven.yonyoucloud.com/nexus/content/groups/public/</url>
        <mirrorOf>central</mirrorOf>
    </mirror>

    <mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>        
    </mirror>
  
  </mirrors>
```


# 方法二：配置本地
1. 新建lib包，将jar包拷入
2. 配置pom.xml文件

```
        <!-- 如果找不到更改数据源的方式 -->
        <!-- https://mvnrepository.com/artifact/org.csource/fastdfs-client-java -->
        <!--<dependency>-->
            <!--<groupId>org.csource</groupId>-->
            <!--<artifactId>fastdfs-client-java</artifactId>-->
            <!--<version>1.27-RELEASE</version>-->
        <!--</dependency>-->
        <!-- 使用本地Jar包的方式 -->
        <dependency>
            <groupId>org.csource</groupId>
            <artifactId>fastdfs-client-java</artifactId>
            <scope>system</scope>
            <systemPath>${basedir}/lib/fastdfs-client-java-1.27-RELEASE.jar</systemPath>
            <version>1.27-RELEASE</version>
        </dependency>
```


# 备注-官方网址推荐
```
# maven官方引用引用地址
- 不仅仅是包含maven，还包含其他种类的引用方式，如：gradle,sbt等
https://mvnrepository.com/artifact/org.csource/fastdfs-client-java/1.27-RELEASE
```
![image](http://note.youdao.com/yws/res/11631/6B0595D0CB224F64B8C6CE8C89777BC3)