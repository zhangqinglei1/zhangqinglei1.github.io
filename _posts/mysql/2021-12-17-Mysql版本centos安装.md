

本文针对Mysql8版本，系统为centos

## Mysql下载

Mysql的下载地址如下

[https://downloads.mysql.com/archives/community/](https://downloads.mysql.com/archives/community/){:target="_blank"}

如下图

![](https://zhangqinglei1.github.io/img/mysql/mysql-path.png)

如何选择合适的rpm bundle版本进行下载

查看当前Linux的版本

```
[root@vm-node1 ~]# cat /etc/system-release
CentOS Linux release 8.5.2111
```

表示系统需要el8的版本

查询CPU的类别，主要是aarch要下载aarch的专用下载包

```
[root@vm-node1 ~]# cat /proc/cpuinfo | grep 'model name' |uniq
model name	: Intel(R) Core(TM) i9-10900 CPU @ 2.80GHz
```

下载

| **Red Hat Enterprise Linux 8 / Oracle Linux 8 (x86, 64-bit), RPM Bundle** | Jul 1, 2021 | 741.6M |      |
| ------------------------------------------------------------ | ----------- | ------ | ---- |
| (mysql-8.0.26-1.el8.x86_64.rpm-bundle.tar)                   |             |        |      |

下载解压后如下图，是我们需要的

![](https://zhangqinglei1.github.io/img/mysql/mysql-rpm.png)

其中**devel 包**，正式环境可以不用安装
devel 包主要是供开发用，至少包括以下2个东西:

1. 头文件
2. 链接库
   有的还含有开发文档或演示代码。

## Mysql检测

安装之前，可先检测系统是否安装过mysql或者mariadb，如果有可以先卸载移除

```
[root@vm-node1 home]# rpm -qa|grep mysql
[root@vm-node1 home]# rpm -qa|grep mariadb
[root@vm-node1 home]# 

移除，找到检测冲突，进行移除
yum remove XXX
```

## 安装Mysql

按照如下顺序进行安装

```
rpm -ivgh mysql-community-common-8.0.26-1.el8.x86_64.rpm 
rpm -ivgh mysql-community-client-plugins-8.0.26-1.el8.x86_64.rpm
rpm -ivgh mysql-community-libs-8.0.26-1.el8.x86_64.rpm
rpm -ivgh mysql-community-client-8.0.26-1.el8.x86_64.rpm 
rpm -ivgh mysql-community-server-8.0.26-1.el8.x86_64.rpm

以上顺序有依赖关系，不要跨顺序安装
```

安装完成后进行初始化mysql

```
mysqld --initialize --lower_case_table_names=1
```

安装完成后启动mysql

```
[root@vm-node1 home]# systemctl start mysqld
```

查看mysql的root的密码

```
[root@vm-node1 home]# tail /var/log/mysqld.log 
2021-12-17T12:47:45.454111Z 6 [Note] [MY-010454] [Server] A temporary password is generated for root@local
host: oKcftk:nl3x=
2021-12-17T12:47:48.505290Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.26) starting as p
rocess 57834
2021-12-17T12:47:48.572780Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2021-12-17T12:47:49.485166Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2021-12-17T12:47:50.355685Z 0 [Warning] [MY-013746] [Server] A deprecated TLS version TLSv1 is enabled for
 channel mysql_main
2021-12-17T12:47:50.355778Z 0 [Warning] [MY-013746] [Server] A deprecated TLS version TLSv1.1 is enabled f
or channel mysql_main
2021-12-17T12:47:50.356390Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2021-12-17T12:47:50.356503Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. 
Encrypted connections are now supported for this channel.
2021-12-17T12:47:50.369699Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: 
'::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
2021-12-17T12:47:50.369746Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Versi
on: '8.0.26'  socket: '/var/lib/mysql/mysql.sock'  port: 3306  MySQL Community Server - GPL.
```

通过第一行日志

A temporary password is generated for root@localhost: oKcftk:nl3x=

可以得知密码是oKcftk:nl3x=

下一步就是修改密码啦

## 修改密码

使用刚刚的密码登录mysql

```
[root@vm-node1 home]# mysql -uroot -poKcftk:nl3x=
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.26

Copyright (c) 2000, 2021, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>alter user user() identified by "!QAZ2wsx";
Query OK, 0 rows affected (0.02 sec)
exit
```

第一次安装必须先修改密码

后面再次修改时，可以

```
mysql> use mysql; 
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> ALTER user 'root'@'localhost' IDENTIFIED BY 'Zhang123#';
Query OK, 0 rows affected (0.00 sec)
```

## Mysql配置文件优化配置

默认安装的mysql目录，配置的参数如下

```
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

一般情况下，我们需要增加后的配置如下

```
[mysqld]
port=3306
# 设置3306端口
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

innodb_buffer_pool_size = 128M
# INNODB的一级缓存，默认是128M，在专用数据库服务器上，可以配置内存的80%。
lower_case_table_names = 1
# 是否支持数据库大小写敏感，默认为0，1表示不敏感，所以也是必须要配置
max_connections=1000
# 允许的最大连接数，默认为100，所以必须配置
# 实际起作用的最大值（实际最大可连接数）为16384，该参数在服务器资源够用的情况下应该尽量设置大，以满足多个客户端同时连接的需求。否则将会出现类似”Too many # connections”的错误。配置可以根据业务1000-10000。具体和业务规模，是否专用数据库服务器(CPU,内存)相关,查询密度。
# 该值配置大不会占用服务器系统资源。
max_connect_errors=100
# 允许连接失败的次数。这是为了防止有人从该主机试图攻击数据库系统
character-set-server=utf8mb4
# 服务端使用的字符集默认为UTF8
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8mb4
```

