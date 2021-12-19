---
layout: post
title:  "Artifactory仓库的管理功能"
date:   2021-12-07 20:00:00
categories: 持续集成
tags: 持续集成
excerpt: Artifactory仓库的管理功能
mathjax: true
---

## 1.artifactory的介绍

此部分转载于：[https://blog.csdn.net/afandaafandaafanda/article/details/81735013](https://blog.csdn.net/afandaafandaafanda/article/details/81735013){:target="_blank"}

Jfrog(这是公司名）的Artifactory是一款**Maven仓库服务端软件**，可以用来在内网搭建maven仓库，供公司内部公共库的上传和发布，以提高公共代码使用的便利性。

它也是目前全球唯一一个支持所有开发语言，任意维度的元数据检索、跨语言正反向依赖分析，并同时拥有深度递归、支持多活异地灾备的企业级、高可用**二进制制品管理仓库**。世界五百强中93%的企业已经将Artifactory作为自己DevOps的核心系统。

### Artifactory 仓库类型

Artifactory 仓库主要有四种类型，远程仓库、本地仓库、虚拟仓库及分发仓库，分别应用在如下不同的场景。

远程仓库
Artifactory 仓库支持代理公网或内网二进制软件制品仓库(Artifactory, Nexus，Harbor等)，按需获取后在本地进行缓存，可大幅度提升构建效率。

本地仓库
Artifactory 本地仓库用来存储本地构建产出的软件制品。本地仓库中的软件制品通常都带有丰富的元数据，并且通过基于角色的访问控制(RBAC)实现资源隔离。

虚拟仓库
为满足制品管理的多团队协作需求，虚拟仓库通过打包任意数量的远程仓库和本地仓库，暴露唯一的访问入口的方式，将制品提供者和消费者之间的耦合度降到最低，提升协作效率。

分发仓库
分发仓库通过JFrog Bintray SaaS服务满足软件制品公网分发的需求，提供默认的全球CDN加速服务。

## 2.jfrog-artifactory-oss的安装

官方下载地址：

https://www.jfrogchina.com/open-source/

### 1.yum安装

通过yum方式安装jfrog，比较简单。需要下载jfrog的yum仓库源，然后进行安装即可，如下：

```
wget -O /etc/yum.repos.d/frog-artifactory.repo https://bintray.com/jfrog/artifactory-rpms/rpm
yum -y install jfrog-artifactory-oss
systemctl start artifactory.service
systemctl status artifactory.service

ps -ef |grep jfrog
sudo netstat -tlunp | grep 51306
```

### 2.zip包安装

zip包下载地址，选择下载的版本:

[https://bintray.com/jfrog/artifactory/jfrog-artifactory-oss-zip](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fbintray.com%2Fjfrog%2Fartifactory%2Fjfrog-artifactory-oss-zip)

也可以使用

wget “[https://bintray.com/jfrog/artifactory/download_file?file_path=jfrog-artifactory-oss-6.12.2.zip](https://www.oschina.net/action/GoToLink?url=https%3A%2F%2Fbintray.com%2Fjfrog%2Fartifactory%2Fdownload_file%3Ffile_path%3Djfrog-artifactory-oss-6.12.2.zip)”

上传到服务器后解压

unzip -q jfrog-artifactory-oss-6.12.2.zip

进入目录后，启动程序

```
./bin/artifactoryctl start

ps -ef |grep jfrog
sudo netstat -tlunp | grep 15557
```

访问

http://192.168.19.134:8082/ui/login/

### 3.rpm安装

先在上面地址下载rpm包，或者

```
wget “https://bintray.com/jfrog/artifactory-rpms/download_file?file_path=jfrog-artifactory-oss-6.12.2.rpm” -O jfrog-artifactory-oss.rpm
yum -y install jfrog-artifactory-oss.rpm

查看
rpm -qa |grep jfrog
rpm -ql jfrog-artifactory-oss
systemctl start artifactory.service
systemctl status artifactory.service

```



## 3.artifactory的使用

jfrog默认的端口是8081，默认的用户名和密码是:admin/password。

下面介绍下artifactory和jenkins构建的关联

```
stage ('Artifactory configuration') {
			steps{
				timestamps {
					script{ 
						def SERVER_ID = '6888' 
						def server = Artifactory.server SERVER_ID
						def uploadSpec = 
						"""
						{
						"files": [
							{
								"pattern": "epochOpen/epoch.zip",
								"target": "epoch/${BUILD_NUMBER}/"
							},
							{
								"pattern": "epoch.zip",
								"target": "epoch/${BUILD_NUMBER}/"
							}
						  ]
						}
						"""
						def buildInfo = Artifactory.newBuildInfo()
						buildInfo.env.capture = true
						buildInfo = server.upload(uploadSpec)
						server.publishBuildInfo(buildInfo)
					}
				}
			}
		}
```

以上介绍了，将uploadSpe包的内容上传到Artifactory进行管理



