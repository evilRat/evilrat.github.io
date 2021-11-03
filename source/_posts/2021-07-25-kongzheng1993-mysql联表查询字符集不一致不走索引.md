---
title: MySQL联表查询表字符集不一致索引失效问题
excerpt: 'MySQL'
tags: [MySQL]
categories: [MySQL]
comments: true
date: 2021-07-27 16:30:10
---

### 背景

新建的表字符集用的是utf8mb4，而之前的两张主表的字符集设置的是utf8，查询进行表关联，导致索引失效。

### 表结构

```mysql
-- 表1utf8字符集，并建立code和name列索引
CREATE TABLE `t1` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(20) DEFAULT NULL,
`code` varchar(50) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `idx_code` (`code`),
KEY `idx_name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;

-- 表2 utf8mb4字符集，并建立code和name列索引
CREATE TABLE `t2` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(20) DEFAULT NULL,
`code` varchar(50) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `idx_code` (`code`),
KEY `idx_name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4;

INSERT INTO `t1` (`name`,`code`) VALUES ('zzz','00006'),('xxx','00002'),('aaa','000003'),('sss','000004'),('ddd','000005');

INSERT INTO `t2` (`name`,`code`) VALUES ('zzz','00001'),('xxx','00002'),('aaa','000003'),('sss','000004'),('ddd','000005');
```

### 慢SQL

```mysql
desc select * from t1 left join t2 on t1.code = t2.code where t2.name = 'ddd';
```

### 查看执行计划

```mysql
EXPLAIN EXTENDED select * from t2 left join t1 on t1.code = t2.code where t2.name = 'ddd';
SHOW WARNINGS;
-- 结果
/* select#1 */ select `demo`.`t2`.`id` AS `id`,`demo`.`t2`.`name` AS `name`,`demo`.`t2`.`code` AS `code`,`demo`.`t1`.`id` AS `id`,`demo`.`t1`.`name` AS `name`,`demo`.`t1`.`code` AS `code` from `demo`.`t2` left join `demo`.`t1` on((convert(`demo`.`t1`.`code` using utf8mb4) = `demo`.`t2`.`code`)) where (`demo`.`t2`.`name` = 'ddd')
```

存在一个convert的过程

### 查看告警信息

```mysql
show warnings;
```

`show warnings;`可以显示上一条sql的告警信息。

**字符集转换遵循由小到大的原则，因为utf8mb4是utf8的超集，所以这里把utf8转换成utf8mb4。而实际上t1表中的索引是utf8格式的，所以会导致t1表全表扫描。**

