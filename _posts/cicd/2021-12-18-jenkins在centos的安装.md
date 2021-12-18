---
layout: post
title:  "jenkins在centos的安装"
date:   2021-12-18 18:14:54
categories: 持续集成
tags: 持续集成
excerpt: jenkins在centos的安装
mathjax: true
---

* content
{:toc}


jenkins的下载地址：

[https://www.jenkins.io/doc/pipeline/tour/getting-started/](https://www.jenkins.io/doc/pipeline/tour/getting-started/){:target="_blank"}

从官方的资料得知，Jenkins对服务器的要求：

设备

- 256MB内存以及以上，建议2G以上
- 10GB的磁盘空间

软件

- Java8到java11版本
- Docker可选安装

## yum install 安装

这种适配有网络环境的，可以进行如下安装

官方安装文档路径

[https://www.jenkins.io/doc/book/installing/linux/#red-hat-centos](https://www.jenkins.io/doc/book/installing/linux/#red-hat-centos){:target="_blank"}

配置yum源

```
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
```

这里要注意的JDK环境

开始安装

```
yum install epel-release # repository that provides 'daemonize'
yum install jenkins
```

启动jenkins

```
systemctl daemon-reload
sudo systemctl start jenkins
sudo systemctl status jenkins
```

如果你的防火墙开启，需要开放8080端口号

```
YOURPORT=8080
PERM="--permanent"
SERV="$PERM --service=jenkins"

firewall-cmd $PERM --new-service=jenkins
firewall-cmd $SERV --set-short="Jenkins ports"
firewall-cmd $SERV --set-description="Jenkins port exceptions"
firewall-cmd $SERV --add-port=$YOURPORT/tcp
firewall-cmd $PERM --add-service=jenkins
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```

访问

Http://IP:8080

如下

![](https://zhangqinglei1.github.io/img/cicd/jenkins-page.png)

根据提示输入

/var/lib/jenkins/secrets/initialAdminPassword 的路径密码

下一步后，选择-选择插件来安装

| Organization and Administration (0/4) | 介绍                                                         | 建议 |
| ------------------------------------- | ------------------------------------------------------------ | ---- |
| **Dashboard View**                    | 可自定义的仪表板，可显示各种作业信息视图。                   | 勾选 |
| **Folders**                           | 这个插件允许用户创建“文件夹”来组织作业                       | 勾选 |
| **OWASP Markup Formatter**            | 支持HTML标记，地址：https://owasp.org/www-project-java-html-sanitizer/ |      |
| Build Features (0/10)                 |                                                              |      |
| **Build Name and Description Setter** | 地址：https://www.jenkins.io/solutions/pipeline/             |      |
| **Build Timeout**                     | 地址：https://plugins.jenkins.io/build-timeout/#documentation |      |
|                                       |                                                              |      |
| Source Code Management (3/10)         |                                                              |      |
| **Git**                               | This plugin integrates [Git](https://git-scm.com/) with Jenkins. |      |
| **GitHub**                            | This plugin integrates [GitHub](http://github.com/) to Jenkins. |      |
| GitLab                                | This plugin allows [GitLab](http://gitlab.com/) to trigger Jenkins builds and display their results in the GitLab UI. |      |

