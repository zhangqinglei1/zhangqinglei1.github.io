---
layout: post
title:  "docker安装mysql"
date:   2021-12-22 10:00:00
categories: Docker
tags: Docker
excerpt: docker安装mysql
mathjax: true
---
* content
{:toc}

MySQL 是世界上最受欢迎的开源数据库。凭借其可靠性、易用性和性能，MySQL 已成为 Web 应用程序的数据库优先选择。

## 1、查看可用的 MySQL 版本

mysql的镜像地址：[镜像地址](https://hub.docker.com/_/mysql?tab=tags){:target="_blank"}

此外，也可以通过 docker search mysql来搜索可用版本。

从官方描述来看，安装mysql可以通过或者或者yaml等进行安装

## 2.拉取mysql镜像

```
docker pull mysql:latest

或者拉取其他版本
docker pull mysql:8.0.27
docker pull mysql:5.7.36
docker pull mysql:5.6.51
```

## 3.运行mysql

启动 MySQL 实例很简单

```console
docker run -itd --name mysql-test -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql
```

参数说明：

- **-p 3306:3306** ：映射容器服务的 3306 端口到宿主机的 3306 端口，外部主机可以直接通过 **宿主机ip:3306** 访问到 MySQL 的服务。
- **MYSQL_ROOT_PASSWORD=123456**：设置 MySQL 服务 root 用户的密码。

通过yml文件启动mysql

如下为stack.yml

```
# Use root/example as user/password credentials
version: '3.1'

services:
  db:
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example

  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
```

运行`docker stack deploy -c stack.yml mysql`（或`docker-compose -f stack.yml up`），等待它完全初始化，然后访问`http://swarm-ip:8080`、`http://localhost:8080`、 或`http://host-ip:8080`（视情况而定）。

## 4.容器shell访问和查看MySQL日志

exec命令可以进入到mysql容器内执行命令

```
docker exec -it some-mysql bash
```

通过logs获取容器内日志

```
docker logs some-mysql
```

## 5.使用自定义 MySQL 配置文件

MySQL 的默认配置可以在 中找到`/etc/mysql/my.cnf`

如果`/my/custom/config-file.cnf`是你的自定义配置文件的路径和名称，你可以这样启动你的`mysql`容器

```
docker run -itd --name some-mysql -v /my/custom:/etc/mysql/conf.d -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql
```

将所有表的默认编码和排序规则更改为使用 UTF-8 ( `utf8mb4`)，只需运行以下命令：

```
docker run -itd --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

查看完整的支持列表

```
docker run -it --rm mysql:tag --verbose --help
```

## 环境变量

| 参数                         | 是否必须 | 说明                                                         |
| ---------------------------- | -------- | ------------------------------------------------------------ |
| MYSQL_ROOT_PASSWORD          | 必须     | 作为 MySQL`root`超级用户帐户设置的密码。                     |
| MYSQL_DATABASE               | 可选     | 指定启动时创建数据库的名称，如果提供了用户/密码（见下文），则该用户将被授予对该数据库的超级用户访问权限 |
| MYSQL_USER  MYSQL_PASSWORD   | 可选     | 创建新用户和设置该用户的密码，该用户将被授予对`MYSQL_DATABASE`变量指定的数据库的超级用户权限。创建用户时需要这两个变量。不用这个来创建root超级用户，默认情况下会使用MYSQL_ROOT_PASSWORD。 |
| MYSQL_ALLOW_EMPTY_PASSWORD   | 可选     | 默认no，设置如果为yes的时候，则允许root使用空密码登录。      |
| `MYSQL_RANDOM_ROOT_PASSWORD` | 可选     | 默认no，设置为yes的时候，为root生成随机密码，生成的密码打印到stdout( `GENERATED ROOT PASSWORD: .....`)。 |
| MYSQL_ONETIME_PASSWORD       | 可选     | 初始化完成后，将root（*不是*在`MYSQL_USER`! 中指定的用户）用户设置为过期，强制在首次登录时更改密码。任何非空值都将激活此设置。*注意*：此功能仅在 MySQL 5.6+ 上受支持。在 MySQL 5.5 上使用此选项将在初始化期间引发适当的错误。 |
| MYSQL_INITDB_SKIP_TZINFO     | 可选     | 默认情况下，入口点脚本会自动加载`CONVERT_TZ()`函数所需的时区数据。如果不需要，任何非空值都会禁用时区加载 |

## 密码文件传输

比如从`/run/secrets/<secret_name>`文件中的加载密码。例如：

```
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/mysql-root -d mysql:tag
```

目前，这仅支持`MYSQL_ROOT_PASSWORD`，`MYSQL_ROOT_HOST`，`MYSQL_DATABASE`，`MYSQL_USER`，和`MYSQL_PASSWORD`。

## 数据存储

这里比较推荐的是在主机(容器外)创建一个目录，将其挂载到容器数据目录

比如

1. 在主机系统上的合适卷上创建数据目录，例如`/my/own/datadir`.
2. `mysql`像这样启动你的容器：

```
docker run -it -d --name some-mysql -v /my/own/datadir:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

对于已使用的数据目录，可以省去$MYSQL_ROOT_PASSWORD

## 创建数据库转储

以下通过exec将数据转存到文件

```
docker exec some-mysql sh -c 'exec mysqldump --all-databases -uroot -p"$MYSQL_ROOT_PASSWORD"' > /some/path/on/your/host/all-databases.sql
```

## 从数据文件恢复数据

用于恢复数据。您可以使用`docker exec`带有`-i`标志的命令，类似于以下内容：

```
docker exec -i some-mysql sh -c 'exec mysql -uroot -p"$MYSQL_ROOT_PASSWORD"' < /some/path/on/your/host/all-databases.sql
```

