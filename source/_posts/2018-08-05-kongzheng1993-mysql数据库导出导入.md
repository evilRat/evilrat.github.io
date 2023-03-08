---
layout: post
title: "mysql数据库导出导入"
date: 2018-08-05
excerpt: "mysql 导库"
tags: [mysql,备份]
categories: [mysql]
comments: true
---

一直在用oracle，今天学习了一下mysql如何导库。

## mysqldump

mysqldump一般在/usr/bin/目录下，装晚mysql后就可以使用了。

## 导出

1. 一般形式：mysqldump -h IP -u 用户名 -p 数据库名 > 导出的文件名

mysqldump和mysql登陆数据库的格式差不多，-h指定ip，-u指定登陆用户，-p指定输入密码。后面加上输出重定向到文件名。

这种一般形式是导出此数据库中的所有表和数据。

2. 导出数据库所有表结构，但不导出数据。

```bash
mysqldump -h localhost -u root -p -d test > /home/evilrat/dbbak/test.sql
```

3. 导出某张表的表结构，不含数据。

```bash
mysqldump -h localhost -u root -p -d test t_user > /home/evilrat/dbbak/tuser.sql
```

4. 备份多个数据库

```bash
mysqldump -h localhost -u root -p --databases test1 test2 > /home/evilrat/dbbak/test1_test2.sql
```

5. 备份所有数据库的方法

```bash
mysqldump -h localhost -u root -p --all-databases > /home/evilrat/dbbak/localhost.sql
```

## 导入

```bash
mysql -h localhost -u root -p
*******
create database test;
show databases;
use test;
show tables;
source /home/evilrat/dbbak/test.sql;
show tables;
exit;
```