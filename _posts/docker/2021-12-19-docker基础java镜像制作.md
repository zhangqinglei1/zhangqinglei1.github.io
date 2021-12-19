---
layout: post
title:  "docker基础java镜像制作"
date:   2021-12-19 18:00:00
categories: Mysql
tags: Mysql
excerpt: docker基础java镜像制作
mathjax: true
---
* content
{:toc}

本镜像是centos7衍生而来

## centos镜像拉取

```
docker pull centos:7.9.200
//启动容器
docker run -d -it -p 30122:22 --name centos --privileged=true <imageId> /usr/sbin/init

docker run -d -it -p 30122:22 --name centos --privileged=true eeb6ee3f44bd /usr/sbin/init

----privileged 启动后让docker容器具备超级特权。
-itd  交互式、终端、后台运行
-p  把宿主机的30122端口映射到docker的22端口。
--name 给启动的容器命名，方便后续操作
注：--privileged  和/usr/sbin/init是必须的，否则会报错。
Failed to get D-Bus connection: Operation not permitted

//查看容器
docker ps
进入容器
docker exec -it <containerId> /bin/sh

# 安装openssh
yum install -y openssl openssl-devel
yum install -y openssh-server openssh-clients
systemctl start sshd

passwd 
#修改密码，如果命令不识别，则yum install passwd

#查看是否启动22端口 
netstat -antp | grep sshd
```

## 常用命令安装

其他一些命令必须要装的

```
yum install net-tools
yum install -y unzip zip
yum -y install lrzsz
yum -y install wget
yum install firewalld systemd -y
yum install ntpdate
yum -y install vim
yum install gcc-c++ pcre pcre-devel zlib zlib-devel ruby bash-completion zlib.i686 libstdc++.i686 lsof
```

设置远程root可以访问

```
修改sshd_config 为密码登录
vim /etc/ssh/sshd_config
#打开注释 PermitRootLogin yes, 允许密码登录,保存退出
systemctl start sshd
```

下载jdk-8u251-linux-x64.rpm

进行安装

不太建议安装openjdk版本。

## 打包容器创建镜像

docker commit :从容器创建一个新的镜像

```
docker commit [OPTIONS] CONTAINER [REPOSITORY[:TAG]]

OPTIONS说明：
-a :提交的镜像作者；
-c :使用Dockerfile指令来创建镜像；
-m :提交时的说明文字；
-p :在commit时，将容器暂停。
```

```
docker ps
docker commit -a "zhangqinglei" -m "baseimage" fd8bdf8e19ee  baseimg:1.0
查看打包好的镜像
docker images
```

## 推送镜像到仓库

这里以阿里云镜像仓库为主

推送的镜像仓库地址为：registry.cn-qingdao.aliyuncs.com

```
#登录
docker login registry.cn-qingdao.aliyuncs.com

#打标签
docker tag [ImageId] registry.cn-qingdao.aliyuncs.com/zhang32987/zhangqinglei:[镜像版本号]

docker tag XX registry.cn-qingdao.aliyuncs.com/zhang32987/baseimage:1.0

#推送到仓库
docker push registry.cn-qingdao.aliyuncs.com/zhang32987/baseimage:1.0

#删除原有镜像
docker rmi base
```

本次镜像是免费开发的，大家可以拉取

registry.cn-qingdao.aliyuncs.com/zhang32987/baseimage:1.0

镜像大小为399.749 MB