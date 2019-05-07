---
layout: post
title:  "在github上配置图片链接"
date:   2019-05-07 14:41:55
categories: blog
tags: blog github
---

* content
{:toc}

# 简述
有的时候我们上传到github上的图片，如果是链接地址的有的会被对应服务器上的权限给屏蔽，那么最好的方式就是使用相对路径，这样就可以放心的引用图片等资源了。

# 引用方式
会出现下面这种不能展示的方式

![image](http://note.youdao.com/yws/public/resource/2bd42504386d1ea5762e205c3ffa818c/48D63E030FA5403391734A78F57CF87E?ynotemdtimestamp=1557210331307)
```bash
![image](http://note.youdao.com/yws/res/11631/6B0595D0CB224F64B8C6CE8C89777BC3)
# 哪怕是可以在浏览器显示的也可能会屏蔽
![image](http://note.youdao.com/yws/public/resource/2bd42504386d1ea5762e205c3ffa818c/48D63E030FA5403391734A78F57CF87E?ynotemdtimestamp=1557210331307)
```
# 修改为相对路径
1. 首先在项目中建立resource文件夹
2. 然后建立Image文件夹分类
3. 最后根据业务建立blog,spring-boot 等细分
4. 将图片拷贝到文件夹**blog**
5. 使用自己项目的引用图片地址

![image](https://github.com/whydejavu/whydejavu.github.io/resource/image/blog/blog-maven-pic.jpg)

**规则**
```bash
# 参考
![image](https://github.com/whydejavu/whydejavu.github.io/resource/image/blog/blog-maven-pic.png)
# 解释
![image](https://github.com/用户名称/项目名称/相对路径/图片)
```