---
layout: post
title:  "k8s问题解决check non-fatal"
date:   2021-12-03 20:00:00
categories: k8s
tags: k8s
excerpt: k8s问题解决check non-fatal
mathjax: true

---

## 问题

```
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
[ERROR CRI]: container runtime is not running: output: time="2021-12-03T22:51:54+08:00" level=fatal msg="getting status of runtime: rpc error: code =
 Unimplemented desc = unknown service runtime.v1alpha2.RuntimeService", error: exit status 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
```

解决办法

```bash
rm /etc/containerd/config.toml
systemctl restart containerd
```

