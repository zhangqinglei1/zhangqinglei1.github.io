---
layout: post
title:  "centos8安装docker出现的问题"
date:   2021-12-05 20:00:00
categories: Docker
tags: Docker
excerpt: centos8安装docker出现的问题
mathjax: true

---

## 问题

centos8.5在安装docker的containerd.io的时候，可能会发生以下问题，起因是安装操作系统的时候，使用了默认虚拟化技术导致的。

报错1

```
错误：
 问题: 安装的软件包的问题 podman-3.3.1-9.module_el8.5.0+988+b1f0b741.x86_64
  - 软件包 podman-3.3.1-9.module_el8.5.0+988+b1f0b741.x86_64 需要 runc >= 1.0.0-57，但没有提供者可以被安装
  - 软件包 containerd.io-1.3.7-3.1.el8.x86_64 与 runc（由 runc-1.0.2-1.module_el8.5.0+911+f19012f9.x86_64 提供）冲突
  - 软件包 containerd.io-1.3.7-3.1.el8.x86_64 取代了 runc（由 runc-1.0.2-1.module_el8.5.0+911+f19012f9.x86_64 提供）
  - 冲突的请求
  - 软件包 runc-1.0.0-66.rc10.module_el8.5.0+1004+c00a74f5.x86_64 被模块过滤过滤掉
  - 软件包 runc-1.0.0-72.rc92.module_el8.5.0+1006+8d0e68a2.x86_64 被模块过滤过滤掉
(尝试在命令行中添加 '--allowerasing' 来替换冲突的软件包 或 '--skip-broken' 来跳过无法安装的软件包 或 '--nobest' 来不只使用软件包的最佳候
选)
```

这是因为和podman发生了冲突，可以通过卸载podman方式

```
dnf remove podman //卸载podman包
```

报错2

```
错误：
 问题: 软件包 podman-3.3.1-9.module_el8.5.0+988+b1f0b741.x86_64 需要 runc >= 1.0.0-57，但没有提供者可以被安装
  - 软件包 containerd.io-1.4.12-3.1.el8.x86_64 与 runc（由 runc-1.0.2-1.module_el8.5.0+911+f19012f9.x86_64 提供）冲突
  - 软件包 containerd.io-1.4.12-3.1.el8.x86_64 取代了 runc（由 runc-1.0.2-1.module_el8.5.0+911+f19012f9.x86_64 提供）
  - 无法为软件包安装最佳更新候选 runc-1.0.2-1.module_el8.5.0+911+f19012f9.x86_64
  - 无法为软件包安装最佳更新候选 podman-3.3.1-9.module_el8.5.0+988+b1f0b741.x86_64
  - 软件包 runc-1.0.0-66.rc10.module_el8.5.0+1004+c00a74f5.x86_64 被模块过滤过滤掉
  - 软件包 runc-1.0.0-72.rc92.module_el8.5.0+1006+8d0e68a2.x86_64 被模块过滤过滤掉
(尝试在命令行中添加 '--allowerasing' 来替换冲突的软件包 或 '--skip-broken' 来跳过无法安装的软件包 或 '--nobest' 来不只使用软件包的最佳候
选)
```

和runc发生了冲突

```
dnf remove runc
```

然后再安装containerd.io即可。

下载地址

https://download.docker.com/linux/centos/8/x86_64/edge/Packages/

