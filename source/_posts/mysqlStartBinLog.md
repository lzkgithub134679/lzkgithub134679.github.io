---
title: MySQL 开启binlog日志
date: 2019-06-25 14:34:06
header-img: "Demo.png"
tags:
- Show All
- 数据库
---

**1.查看MySQL的binlog日志状态**

show variables like '%log_bin%';

![1](1.jpg)

**2.开始MySQL的binlog日志**

在安装的MySQL路径的my.cnf的[mysqld]标签下加入;

server_id=1
log_bin = mysql-bin
binlog_format = ROW

**3.重启MySQL**

必须要重启，否则不生效，然后再执行show variables like '%log_bin%',查询一下看是否是NO;