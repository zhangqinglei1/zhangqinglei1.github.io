---
layout: post
title:  "CentOS 安装rz和sz命令 lrzsz"
date:   2021-12-05 20:00:00
categories: Linux
tags: Linux
excerpt: CentOS 安装rz和sz命令 lrzsz
mathjax: true

---

## lrzsz介绍

lrzsz 官网入口：http://freecode.com/projects/lrzsz/
lrzsz是一个unix通信套件提供的X，Y，和ZModem文件传输协议
windows 需要向centos服务器上传文件，可直接在centos上执行命令yum -y install lrzsz 程序会自动安装好，然后如你要下载者sz [找到你要下载的文件] 如果你要上传，者rz 浏览找到你本机要上传的文件。
需要注意的事这个命令无法在putty界面使用哦！

步骤：
 一、首先安装lrzsz

```
yum -y install lrzsz 
```

二、 上传文件，执行命令rz，会跳出文件选择窗口，选择好文件，点击确认即可
rz
三、下载文件，执行命令sz
sz

 这样子就可以很简单的上下传文件了。