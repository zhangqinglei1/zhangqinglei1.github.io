---
layout: post
title:  "k8s移除kubernetes-dashboard"
date:   2021-12-05 20:00:00
categories: k8s
tags: k8s
excerpt: k8s移除kubernetes-dashboard
mathjax: true

---

在安装kubernetes-dashboard的时候，可能会出现dashboard版本和kubernetes不一致的问题，具体的版本对应关系请查看

https://github.com/kubernetes/dashboard/releases

因此需要卸载掉dashboard

以下给出卸载命令

```
kubectl -n kubernetes-dashboard delete $(kubectl -n kubernetes-dashboard get pod -o name)
```

删除找到所有的kubernetes-dashboard，

```
kubectl delete deployment kubernetes-dashboard --namespace=kubernetes-dashboard
kubectl delete service kubernetes-dashboard  --namespace=kubernetes-dashboard
kubectl delete role kubernetes-dashboard-minimal --namespace=kubernetes-dashboard
kubectl delete rolebinding kubernetes-dashboard-minimal --namespace=kubernetes-dashboard
kubectl delete sa kubernetes-dashboard --namespace=kubernetes-dashboard
kubectl delete secret kubernetes-dashboard-certs --namespace=kubernetes-dashboard
kubectl delete secret kubernetes-dashboard-csrf --namespace=kubernetes-dashboard
kubectl delete secret kubernetes-dashboard-key-holder --namespace=kubernetes-dashboard
```

最后删除命名空间

```
kubectl delete namespace kubernetes-dashboard
```

如果dashbord不能删除的话，强制删除以下

```
kubectl -n kubernetes-dashboard get pod -o name
kubectl delete pod kubernetes-dashboard-775cbc6f45-7mkqh -n kubernetes-dashboard --force --grace-period=0
//kubernetes-dashboard-775cbc6f45-7mkqh为获取的名称
```

最后再进行安装

```
kubectl create -f recommended.yaml
```

