---
layout: post
title:  "java对象在内存中的存储分析"
date:   2020-07-22 18:14:54
categories: java
tags: java
excerpt: java对象在内存中的存储分析
mathjax: true
---

* content
{:toc}

## 在centos7.2 X64位上安装mysql 笔记

1.创建两个文件
cd /
mkdir tools
mkdir application
2.进入tools目录，下载mysql安装包
cd tools
wget http://mirrors.sohu.com/mysql/MySQL-5.7/mysql-5.7.16-linux-glibc2.5-x86_64.tar.gz
3.进入安装目录，解压并且改名
cd application
tar -xzvf /tools/mysql-5.7.16-linux-glibc2.5-x86_64.tar.gz
ls
mv mysql-5.7.16-linux-glibc2.5-x86_64 mysql
4.创建mysql用户组，使得mysql用户禁止登录，只能数据库连接
groupadd mysql
useradd -M -s /sbin/nologin  mysql

修改/etc/my.cnf配置文件，如果当期没有，则copy一个
copy,先进入mysql
cd /application/mysql
cp support-files/my-default.cnf  /etc/my.cnf
然后使用vi或者vim进行修改，centos好像不支持vim命令
本地采用配置文件如下

```
[client]
port = 3306
socket = /tmp/mysql.sock
default-character-set = utf8mb4

[mysql]
prompt="MySQL [\d]> "
no-auto-rehash

[mysqld]
port = 3306
socket = /tmp/mysql.sock

basedir = /application/mysql
datadir = /application/mysql/var
pid-file = /application/mysql/var/mysql.pid
user = mysql
bind-address = 0.0.0.0
server-id = 1

init-connect = 'SET NAMES utf8mb4'
character-set-server = utf8mb4

skip-name-resolve
#skip-networking
back_log = 300

max_connections = 330
max_connect_errors = 6000
open_files_limit = 65535
table_open_cache = 128
max_allowed_packet = 500M
binlog_cache_size = 1M
max_heap_table_size = 8M
tmp_table_size = 16M

read_buffer_size = 2M
read_rnd_buffer_size = 8M
sort_buffer_size = 8M
join_buffer_size = 8M
key_buffer_size = 4M

thread_cache_size = 8

query_cache_type = 1
query_cache_size = 8M
query_cache_limit = 2M

ft_min_word_len = 4

log_bin = mysql-bin
binlog_format = mixed
expire_logs_days = 7

log_error = /application/mysql/var/mysql-error.log
slow_query_log = 1
long_query_time = 1
slow_query_log_file = /application/mysql/var/mysql-slow.log

performance_schema = 0
explicit_defaults_for_timestamp

#lower_case_table_names = 1

skip-external-locking

default_storage_engine = InnoDB
innodb_file_per_table = 1
innodb_open_files = 500
innodb_buffer_pool_size = 64M
innodb_write_io_threads = 4
innodb_read_io_threads = 4
innodb_thread_concurrency = 0
innodb_purge_threads = 1
innodb_flush_log_at_trx_commit = 2
innodb_log_buffer_size = 2M
innodb_log_file_size = 32M
innodb_log_files_in_group = 3
innodb_max_dirty_pages_pct = 90
innodb_lock_wait_timeout = 120

bulk_insert_buffer_size = 8M
myisam_sort_buffer_size = 8M
myisam_max_sort_file_size = 10G
myisam_repair_threads = 1

interactive_timeout = 28800
wait_timeout = 28800

[mysqldump]
quick
max_allowed_packet = 500M

[myisamchk]
key_buffer_size = 8M
sort_buffer_size = 8M
read_buffer = 4M
write_buffer = 4M
```

其中安装目录是/application/mysql 数据源目录是/application/mysql/var
copy server配置文件
cp support-files/mysql.server /etc/init.d/mysqld
vi /etc/init.d/mysqld   //编辑或者修改
basedir=/application/mysql 
datadir=/application/mysql/var

在mysql目录下执行
bin/mysqld --initialize --user=mysql --basedir=/application/mysql --datadir=/application/mysql/var
会报错，解决办法，使用yum命令加载libaio必要
yum install libaio*
yum -y install libaio.so.1
执行这两个后再执行
bin/mysqld --initialize --user=mysql --basedir=/application/mysql --datadir=/application/mysql/var


修改进入mysql密码，在新版本mysql数据库中，默认用户root密码已经指定成为一个随机，所以直接用
root登录会报错，解决办法

在mysql关闭情况下，执行
/etc/init.d/mysqld start --skip-grant-tables
执行成功后就可以通过该命令进入到mysql控制台
bin/mysql -u root mysql
接下来进行修改root密码
use mysql
UPDATE user SET Password=PASSWORD('123456') where USER='root';
FLUSH PRIVILEGES;


在执行update的时候会报错
需要在my.cnf中加上 一行skip-grant-tables 或者用下面命令
/etc/init.d/mysqld restart --skip-grant-tables

错误原因：mysql数据库下已经没有password这个字段了，password字段改成了authentication_string。
使用下面的语句进行更改密码
update mysql.user set authentication_string=PASSWORD('Test_12345') where User='root';

flush privileges;
MySQL5.7 设置的密码中必须至少包含一个大写字母、一个小写字母、一个特殊符号、一个数字，
密码长度至少为8个字符

退出重启mysql
/etc/init.d/mysqld restart
在第一次登录进去后会被要求改密码，使用alter
ALTER USER 'root'@'localhost'IDENTIFIED BY 'Test_1234';
开放外网权限访问数据库

root使用密码从任何主机连接到mysql服务器
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'Test_1234' WITH GRANT OPTION;
flush privileges;
如果你想允许用户root从ip为192.168.1.104的主机连接到mysql服务器
GRANT ALL PRIVILEGES ON *.* TO ‘myuser’@'192.168.1.104′ IDENTIFIED BY ‘admin123′  WITH GRANT OPTION;
flush privileges;