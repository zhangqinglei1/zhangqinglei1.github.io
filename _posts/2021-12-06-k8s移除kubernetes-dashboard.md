---
layout: post
title:  "k8s中dashboard权限anonymous问题"
date:   2021-12-06 20:00:00
categories: k8s
tags: k8s
excerpt: k8s中dashboard权限anonymous问题
mathjax: true

---

## 介绍

在安装kubernetes-dashboard的时候，低版本中可能会有system-anonymons 权限问题

获取POD，进行日志查看

```
kubectl -n kubernetes-dashboard get pod -o name
kubectl logs -f -n kubernetes-dashboard pod/kubernetes-dashboard-576cb95f94-gf7m6
```

日志出现User “system:anonymous“ cannot list nodes at the cluster scope，表示需要授权.

授权

```
kubectl create clusterrolebinding cluster-system-anonymons --clusterrole=cluster-admin --user=system:anonymous
```

