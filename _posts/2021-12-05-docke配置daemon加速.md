---
layout: post
title:  "docke配置daemon加速"
date:   2021-12-05 20:00:00
categories: Mysql
tags: Mysql
excerpt: docke配置daemon加速
mathjax: true

---

docker使用daemon.json进行加速配置

路径在/etc/docker/daemon.json

配置阿里云加速

```
{
  "registry-mirrors": ["https://cfaccnwy.mirror.aliyuncs.com"]
}
```

配置完成后进行重启

```
systemctl daemon-reload && systemctl restart docker
```

