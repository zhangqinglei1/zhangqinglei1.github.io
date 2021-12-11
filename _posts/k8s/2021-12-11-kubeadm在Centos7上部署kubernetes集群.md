---
layout: post
title:  "kubeadm在Centos7上部署kubernetes集群.md"
date:   2021-12-11 22:14:54
categories: k8s
tags: k8s
excerpt: kubeadm在Centos7上部署kubernetes集群.md
mathjax: true
---

* content
{:toc}
## 准备工作

准备了三台虚拟机，配置是2核4G，40G硬盘，分配IP主机如下

| IP             | 主机名 | 作用       |
| -------------- | ------ | ---------- |
| 192.168.19.135 | master | K8S主节点  |
| 192.168.19.136 | node1  | K8S从节点1 |
| 192.168.19.137 | node2  | K8S从节点2 |

系统版本

```
[root@master ~]# cat /etc/centos-release
CentOS Linux release 7.5.1804 (Core) 
```

### 1.关闭防火墙

在所有节点关闭防火墙

主要操作如下

```
systemctl stop firewalld
systemctl disable firewalld
sed -i 's/enforcing/disabled/' /etc/selinux/config
setenforce 0
```

### 2.设置NTP服务器

ntp是一个时钟源，能确保所有服务器时间一致，这是在企业级必须使用的一种手段。

检测ntp是否安装

```
rpm -q ntp
```

如果没有安装，使用如下命令进行安装

```
yum -y install ntp
```

安装后设置启动和开机启动如下

```
systemctl enable ntpd
systemctl start ntpd
```

选定主服务器master作为ntp服务器地址，保证node和主服务器时间一致。

**在ntp_sever 和 ntp_client均需要开放123/udp端口**

### 3.设置SSH互通

在每个主机上设置主

```
vi /etc/hosts  增加如下内容
192.168.19.135 master
192.168.19.136 node1
192.168.19.137 node2
```

在每台服务器上生成私钥/公钥（默认一路回车，在家目录.ssh文件下）

```
ssh-keygen -t rsa
```

在每台服务器执行如下，分别拷贝到其他的服务器，拷贝时需要输入拷贝服务器root密码，按指令输入

```
cd ~/.ssh
scp ~/.ssh/authorized_keys master:~/.ssh/
scp ~/.ssh/authorized_keys node1:~/.ssh/
scp ~/.ssh/authorized_keys node2:~/.ssh/
```

测试是否已支持ssh通信

ssh node1 不用输入密码，直接进入到命令即可。

要指定端口和IP如下ssh -p 22 root@192.168.137

使用exit进行登出

### 4.关于swap分区

```
[root@master .ssh]# swapoff -a
[root@master .ssh]# cat /etc/fstab 
#
# /etc/fstab
# Created by anaconda on Sat Dec 11 17:29:36 2021
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/centos-root /                       xfs     defaults        0 0
UUID=0f9771c4-64b1-4e29-b8ee-ae24949bc2db /boot                   xfs     defaults        0 0
#/dev/mapper/centos-swap swap                    swap    defaults        0 0
```

etc/fstab注释掉最后一行

### 5.配置内核参数

配置内核参数，将桥接的IPv4流量传递到iptables的链

```
[root@master ~]# cat > /etc/sysctl.d/k8s.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### 5.配置yum原

配置阿里云源，docker源和k8s源。

阿里源配置请查看

[https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.22c11b11Tb5Yh4](https://developer.aliyun.com/mirror/centos?spm=a2c6h.13651102.0.0.22c11b11Tb5Yh4){:target="_blank"}

```
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
```

增加docker源

```
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

添加kubernetes源

```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

最后更新yum源

```
yum makecache
```

## 开始安装

### 1.安装docker

本地安装k8s1.21版本

查看官方[https://github.com/kubernetes/kubernetes](https://github.com/kubernetes/kubernetes){:target="_blank"}的CHANGELOG

[https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.21.md](https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.21.md){:target="_blank"}

Update the latest validated version of Docker to 20.10 ([#98977](https://github.com/kubernetes/kubernetes/pull/98977), [@neolit123](https://github.com/neolit123)) [SIG CLI, Cluster Lifecycle and Node]

从如上文档得知，docker的版本必须在20.10+

查看docker版本

```
 yum list docker-ce.x86_64 --showduplicates | sort -r
 [root@node1 .ssh]# yum list docker-ce.x86_64 --showduplicates | sort -r
已加载插件：fastestmirror, langpacks
可安装的软件包
 * updates: mirrors.aliyun.com
Loading mirror speeds from cached hostfile
 * extras: mirrors.aliyun.com
docker-ce.x86_64            3:20.10.9-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.7-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.6-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:20.10.11-3.el7                    docker-ce-stable
docker-ce.x86_64            3:20.10.10-3.el7                    docker-ce-stable
docker-ce.x86_64            3:20.10.0-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.9-3.el7                     docker-ce-stable
。。。
```

直接安装最新版本

```
安装必要的一些系统工具
yum install -y yum-utils device-mapper-persistent-data lvm2
yum -y install docker-ce
```

添加阿里仓库加速器

```
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://cfaccnwy.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
sudo systemctl enable docker
```

### 2.安装kubelet

安装，指定版本，否则是最新版本，所有主机都需要安装

```
(备注:1.21.0有个BUG，/coredns:v1.8.0版本号的问题,测试安装了1.21.0有问题，所以安装到1.21.2
[ERROR ImagePull]: failed to pull image registry.aliyuncs.com/google_containers/coredns/coredns:v1.8.0: output: Error response f
rom daemon: pull access denied for registry.aliyuncs.com/google_containers/coredns/coredns)
```

开始安装kubeam

```
yum install kubectl-1.21.2 kubelet-1.21.2 kubeadm-1.21.2
```

安装完成后启动

```
 systemctl daemon-reload && systemctl enable kubelet
```

在master节点进行初始化

```
kubeadm init --kubernetes-version=1.21.2  \
--apiserver-advertise-address=192.168.19.135   \
--image-repository registry.aliyuncs.com/google_containers  \
--service-cidr=10.10.0.0/16 --pod-network-cidr=10.122.0.0/16
```

成功后显示

```
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.19.135:6443 --token huf49h.jxnn9669lvvmt2uc \
	--discovery-token-ca-cert-hash sha256:0f1a2df654aff15af824b2989fb2e86b0af33e178ad0541852b06e472cf8b367
```

表示master节点安装成功，记得留下kubeadm join信息留下备用。

创建kubectl

```
 mkdir -p $HOME/.kube
 sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
 sudo chown $(id -u):$(id -g) $HOME/.kube/config
 echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
```

### 3.安装网络插件

网络插件可以选择calico和flannel，这里我们使用calico

```
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

安装完成后如下，即可代表安装成功

```
[root@master yum.repos.d]# kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/caliconodestatuses.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipreservations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/kubecontrollersconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
Warning: policy/v1beta1 PodDisruptionBudget is deprecated in v1.21+, unavailable in v1.25+; use policy/v1 PodDisruptionBudget
poddisruptionbudget.policy/calico-kube-controllers created
```

### 4.添加集群节点

在从节点上加入集群，可以查看第二部成功后预留的信息

```
kubeadm join 192.168.19.135:6443 --token huf49h.jxnn9669lvvmt2uc \
	--discovery-token-ca-cert-hash sha256:0f1a2df654aff15af824b2989fb2e86b0af33e178ad0541852b06e472cf8b367
```

在从节点上执行，完成后显示

```
[root@node2 .ssh]# kubeadm join 192.168.19.135:6443 --token huf49h.jxnn9669lvvmt2uc \
> --discovery-token-ca-cert-hash sha256:0f1a2df654aff15af824b2989fb2e86b0af33e178ad0541852b06e472cf8b367
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please fol
low the guide at https://kubernetes.io/docs/setup/cri/[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

即可代表成功，如果有失败，需要查看上面是否有遗漏在从节点执行的步骤。

## 在k8s集群中安装kubernetes-dashboard

### 安装

该操作仅在master节点

kubernetes-dashboard是k8s的UI看板，可以查看、编辑整个集群状态

官方地址：[https://github.com/kubernetes/dashboard/](https://github.com/kubernetes/dashboard/){:target="_blank"}

查看release，这里要安装和上面的k8s对应的dashboard

[https://github.com/kubernetes/dashboard/releases](https://github.com/kubernetes/dashboard/releases){:target="_blank"}

截止笔者目前，看到有[v2.3.0](https://github.com/kubernetes/dashboard/releases/tag/v2.3.0)，[v2.3.1](https://github.com/kubernetes/dashboard/releases/tag/v2.3.1)，[v2.4.0](https://github.com/kubernetes/dashboard/releases/tag/v2.4.0)，三个版本支持1.21.1版本。

这里安装一个最新的v2.4.0 

```
kubernetesui/dashboard:v2.4.0
```

安装如下

```
wget  https://raw.githubusercontent.com/kubernetes/dashboard/v2.4.0/aio/deploy/recommended.yaml
```

(这里可能会失败，多测试几次即可)

修改kind: Service

```
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort  //增加nodeport
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30000 //监听端口，可以改掉，不被占用即可
  selector:
    k8s-app: kubernetes-dashboard
```

执行安装

```
kubectl create -f recommended.yaml
```

执行完成后如下

```
[root@master home]# kubectl create -f recommended.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

执行完成查看

```
[root@master home]# kubectl  get svc -n kubernetes-dashboard   
NAME                        TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.10.94.203    <none>        8000/TCP        63s
kubernetes-dashboard        NodePort    10.10.211.165   <none>        443:30000/TCP   63s
```

访问地址：http://NodeIP:30000

创建service account并绑定默认cluster-admin管理员集群角色：

```
$ kubectl create serviceaccount dashboard-admin -n kube-system
$ kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
$ kubectl describe secrets -n kube-system $(kubectl -n kube-system get secret | awk '/dashboard-admin/{print $1}')
```

使用输出的token登录Dashboard。

通过集群的任何一个IP，均可访问：

https://192.168.19.135:30000/#/login

如下

![dashboard](https://zhangqinglei1.github.io/img/k8s/dashboard.png)

访问dashboard需要认证，查看dashboard的pod

```
kubectl get pods -n kubernetes-dashboard
[root@master home]# kubectl get pods -n kubernetes-dashboard
NAME                                        READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-c45b7869d-fk7zj   1/1     Running   0          27m
kubernetes-dashboard-576cb95f94-5bcdt       1/1     Running   0          27m
```

### 认证创建

创建一个认证机制，执行如下命令，创建一个ServiceAccount用户dashboard-admin：

```
kubectl create serviceaccount dashboard -n kubernetes-dashboard
kubectl create rolebinding def-ns-admin --clusterrole=admin --serviceaccount=default:def-ns-admin
kubectl create clusterrolebinding dashboard-cluster-admin --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:dashboard
```

查看

```
kubectl describe sa dashboard -n kubernetes-dashboard
[root@master home]# kubectl describe sa dashboard -n kubernetes-dashboard
Name:                dashboard
Namespace:           kubernetes-dashboard
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   dashboard-token-j7z4z
Tokens:              dashboard-token-j7z4z
Events:              <none>
```

通过Tokens查看token

```
kubectl describe secret dashboard-token-j7z4z -n kubernetes-dashboard
```

复制令牌就可以登录了，记得空格也要复制哦

发现报了个错

```
statefulsets.apps is forbidden: User "system:anonymous" cannot list resource "statefulsets" in API group "apps" in the namespace "default"
```

这个在之前博客就已经讲了

[https://zhangqinglei1.github.io/k8s/2021/12/06/k8s%E7%A7%BB%E9%99%A4kubernetes-dashboard/](https://zhangqinglei1.github.io/k8s/2021/12/06/k8s移除kubernetes-dashboard/){:target="_blank"}

