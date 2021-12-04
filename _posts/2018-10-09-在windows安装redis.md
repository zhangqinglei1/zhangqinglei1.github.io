---
layout: post
title:  "在windows安装redis"
date:   2018-10-09 18:14:54
categories: redis
tags: redis
excerpt: 在windows安装redis
mathjax: true
---

* content
{:toc}
## Redis安装

本文主要介绍在windows下安装redis，主要做测试使用。生产环境请参考其他文档。

redis官方并不支持在windows下的安装，以下的redis主要是微软进行维护的地址。

下载Redis-x64-3.2.100.msi安装，下一步，直到成功完成。

下载地址：https://github.com/microsoftarchive/redis/releases

   双击redis-server.exe启动redis服务器，双击redis-cli.exe打开redis客户端（用来执行命令，访问服务器的）

![img](https://images2015.cnblogs.com/blog/578448/201611/578448-20161123152255112-1262260125.png)



如何保证后台启动

打开windows服务，停止redis,打开cmd,在命令执行

![输入图片说明](https://images.gitee.com/uploads/images/2018/1009/093537_a0f91776_626204.png)



sc delete redis

删除当前的redis服务，且确保在windows 服务中也不可见。这里不删除会导致下面重新注册服务报错。如果忘记删了，执行sc delete redis后再删除也可以

B.在redis配置目录中，找到redis.window.conf配置文件，修改如下内容

找到bind 127.0.0.1去掉加注释#

port 6381 改造为自己下要的端口号，默认6379

requirepass 123456 设置密码，必须设置

maxmemory 1024000000 设置缓存

以上基本上满足需求了，接下来注册服务，找到redis目录，打开cmd，执行如下

redis-server.exe --service-install redis.windows.conf



![输入图片说明](https://images.gitee.com/uploads/images/2018/0921/102856_312e1326_626204.png)

说明重新注册成功了，在windows服务中可以找到新注册的redis，并且启动就可以使用了