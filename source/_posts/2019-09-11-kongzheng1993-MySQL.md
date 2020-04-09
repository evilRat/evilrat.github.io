---
title: MySQL
excerpt: 'mysql'
tags: [mysql]
categories: [mysql]
comments: true
date: 2019-09-11 18:30:52
---

## MySQL

之前一只觉得数据库没什么，会用就行了，最近面试总是碰壁，很多又是数据库的问题打不上来，不够深入是我的大问题。所以准备总结一下。


### 索引
索引是数据库查询操作中提升速度的一种手段，索引是一种数据结构。
索引是一个排序的列表，这个列表中存储着索引的值和包含这个值的数据所在的物理地址，数据量庞大的时候，索引可以快速定位需要查找的数据对应的物理地址，不需要扫描全表的数据。

1. 建标时创建索引
```sql
CREATE TABLE t_table(
    ID INT NOT NULL,
    USER_NAME VARCHAR(16) NOT NULL,
    INDEX USER_NAME_INDEX (USER_NAME), #单列索引
    INDEX (ID,USER_NAME) #组合索引
) ENGINE = INNODB DEFAULT CHARSET = utf8 COMMENT '注释';
```

2. 建表后创建索引
```sql
ALTER TABLE t_TABLE ADD UNIQUE INDEX (ID);
ALTER TABLE T_TABLE ADD INDEX (ID,USER_NAME);
ALTER TABLE T_TABLE ADD PRIMARY KEY (ID);
```

3. 查看已经创建的索引
```sql
show index from t_table;
```

4. 删除索引
```sql
drop index user_name_index on t_table;
alter table t_table drop index user_name_index;
```

5. 查看索引使用情况（执行计划）
```sql
explain select * from t_table where user_name = 'Tom';
```

```
mysql> explain select * from t_test where username = 'Tom';
+----+-------------+--------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-------------+
| id | select_type | table  | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra       |
+----+-------------+--------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | t_test | NULL       | ref  | t_test_index_username | t_test_index_username | 67      | const |    1 |   100.00 | Using index |
+----+-------------+--------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

说明：

id：SELECT识别符。这是SELECT的查询序列号。

select_type：SELECT类型。

    SIMPLE： 简单SELECT(不使用UNION或子查询)
    PRIMARY： 最外面的SELECT
    UNION：UNION中的第二个或后面的SELECT语句
    DEPENDENT UNION：UNION中的第二个或后面的SELECT语句，取决于外面的查询
    UNION RESULT：UNION的结果
    SUBQUERY：子查询中的第一个SELECT
    DEPENDENT SUBQUERY：子查询中的第一个SELECT，取决于外面的查询
    DERIVED：导出表的SELECT(FROM子句的子查询)

table：表名

type：联接类型。是SQL性能的非常重要的一个指标，结果值从好到坏依次是：system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL。
一般来说，得保证查询至少达到range级别。

    system：表仅有一行(=系统表)。这是const联接类型的一个特例。
    const：表最多有一个匹配行，它将在查询开始时被读取。因为仅有一行，在这行的列值可被优化器剩余部分认为是常数。const用于用常数值比较PRIMARY KEY或UNIQUE索引的所有部分时。
    eq_ref：对于每个来自于前面的表的行组合，从该表中读取一行。这可能是最好的联接类型，除了const类型。它用在一个索引的所有部分被联接使用并且索引是UNIQUE或PRIMARY KEY。eq_ref可以用于使用= 操作符比较的带索引的列。比较值可以为常量或一个使用在该表前面所读取的表的列的表达式。
    ref：对于每个来自于前面的表的行组合，所有有匹配索引值的行将从这张表中读取。如果联接只使用键的最左边的前缀，或如果键不是UNIQUE或PRIMARY KEY(换句话说，如果联接不能基于关键字选择单个行的话)，则使用ref。如果使用的键仅仅匹配少量行，该联接类型是不错的。ref可以用于使用=或<=>操作符的带索引的列。
    ref_or_null：该联接类型如同ref，但是添加了MySQL可以专门搜索包含NULL值的行。在解决子查询中经常使用该联接类型的优化。
    index_merge：该联接类型表示使用了索引合并优化方法。在这种情况下，key列包含了使用的索引的清单，key_len包含了使用的索引的最长的关键元素。
    unique_subquery：该类型替换了下面形式的IN子查询的ref：value IN (SELECT primary_key FROMsingle_table WHERE some_expr);unique_subquery是一个索引查找函数，可以完全替换子查询，效率更高。
    index_subquery：该联接类型类似于unique_subquery。可以替换IN子查询，但只适合下列形式的子查询中的非唯一索引：value IN (SELECT key_column FROM single_table WHERE some_expr)
    range：只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引。key_len包含所使用索引的最长关键元素。在该类型中ref列为NULL。当使用=、<>、>、>=、<、<=、IS NULL、<=>、BETWEEN或者IN操作符，用常量比较关键字列时，可以使用range
    index：该联接类型与ALL相同，除了只有索引树被扫描。这通常比ALL快，因为索引文件通常比数据文件小。
    all：对于每个来自于先前的表的行组合，进行完整的表扫描。如果表是第一个没标记const的表，这通常不好，并且通常在它情况下很差。通常可以增加更多的索引而不要使用ALL，使得行能基于前面的表中的常数值或列值被检索出。

possible_keys：possible_keys列指出MySQL能使用哪个索引在该表中找到行。注意，该列完全独立于EXPLAIN输出所示的表的次序。这意味着在possible_keys中的某些键实际上不能按生成的表次序使用。

key：key列显示MySQL实际决定使用的键(索引)。如果没有选择索引，键是NULL。要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。

key_len：key_len列显示MySQL决定使用的键长度。如果键是NULL，则长度为NULL。注意通过key_len值我们可以确定MySQL将实际使用一个多部关键字的几个部分。

ref：ref列显示使用哪个列或常数与key一起从表中选择行。

rows：rows列显示MySQL认为它执行查询时必须检查的行数。

Extra：该列包含MySQL解决查询的详细信息。

    Distinct：MySQL发现第1个匹配行后，停止为当前的行组合搜索更多的行。
    Not exists：MySQL能够对查询进行LEFT JOIN优化，发现1个匹配LEFT JOIN标准的行后，不再为前面的的行组合在该表内检查更多的行。
    range checked for each record (index map: #)：MySQL没有发现好的可以使用的索引，但发现如果来自前面的表的列值已知，可能部分索引可以使用。对前面的表的每个行组合，MySQL检查是否可以使用range或index_merge访问方法来索取行。
    Using filesort：MySQL需要额外的一次传递，以找出如何按排序顺序检索行。通过根据联接类型浏览所有行并为所有匹配WHERE子句的行保存排序关键字和行的指针来完成排序。然后关键字被排序，并按排序顺序检索行。
    Using index：从只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的列信息。当查询只使用作为单一索引一部分的列时，可以使用该策略。
    Using temporary：为了解决查询，MySQL需要创建一个临时表来容纳结果。典型情况如查询包含可以按不同情况列出列的GROUP BY和ORDER BY子句时。
    Using where：WHERE子句用于限制哪一个行匹配下一个表或发送到客户。除非你专门从表中索取或检查所有行，如果Extra值不为Using where并且表联接类型为ALL或index，查询可能会有一些错误。
    Using sort_union(...), Using union(...), Using intersect(...)：这些函数说明如何为index_merge联接类型合并索引扫描。
    Using index for group-by：类似于访问表的Using index方式，Using index for group-by表示MySQL发现了一个索引，可以用来查询GROUP BY或DISTINCT查询的所有列，而不要额外搜索硬盘访问实际的表。并且，按最有效的方式使用索引，以便对于每个组，只读取少量索引条目。

6. 模糊查询时，%如果在前面，那么不会使用索引。涉及到多个索引字段时,如果这些索引字段中，不存在主键索引的话，那么就会使用该使用的索引。多个索引时，先使用哪个索引后使用哪个索引，是由MySQL的优化器经过一些列计算后作出的抉择。当对索引字段进行 >， <，>=， <=，not in，between …… and ……，函数(索引字段)，like模糊查询%在字段前时，不会使用该索引.在实际使用时，如果涉及到多列，我们一般都不会将这些列一 一创建为单列索引，而是将这些列创建为组合索引。


7. 组合索引的使用
    最左原则
    假设组合索引为：a,b,c的话;那么当SQL中对应有：a或a，b或a，b，c的时候，可称为完全满足最左原则；当SQL中对应只有a，c的时候，可称为部分满足最左原则；当SQL中没有a的时候，可称为不满足最左原则。
    注：SQL语句中的对应条件的先后顺序与创建组合索引中列的顺序无关。如果完全满足最左原则，所有的列都会走索引，部分满足最左原则，那么最左的列会走索引，剩下的不会走索引。不满足最左原则的话就不会走索引。

8. 索引无法存储null值
        
    a. 单列索引无法储null值，复合索引无法储全为null的值。
    b. 查询时，采用is null条件时，不能利用到索引，只能全表扫描。
    为什么索引列无法存储Null值？
    a.索引是有序的。NULL值进入索引时，无法确定其应该放在哪里。（将索引列值进行建树，其中必然涉及到诸多的比较操作，null值是不确定值，无法比较，无法确定null出现在索引树的叶子节点位置。）　
    b.如果需要把空值存入索引，方法有二：其一，把NULL值转为一个特定的值，在WHERE中检索时，用该特定值查找。其二，建立一个复合索引。例如`create index ind_a on table(col1,1);`通过在复合索引中指定一个非空常量值，而使构成索引的列的组合中，不可能出现全空值。　
    
9. 不适合键值较少的列（重复数据较多的列）
    假如索引列TYPE有5个键值，如果有1万条数据，那么`WHERE TYPE = 1`将访问表中的2000个数据块。再加上访问索引块，一共要访问大于200个的数据块。如果全表扫描，假设10条数据一个数据块，那么只需访问1000个数据块，既然全表扫描访问的数据块少一些，肯定就不会利用索引了。
    
    3.前导模糊查询不能利用索引(like '%XX'或者like '%XX%')
    假如有这样一列code的值为'AAA','AAB','BAA','BAB' ,如果`where code like '%AB'`条件，由于前面是模糊的，所以不能利用索引的顺序，必须一个个去找，看是否满足条件。这样会导致全索引扫描或者全表扫描。如果是这样的条件`where code like 'A%'`，就可以查找CODE中A开头的CODE的位置，当碰到B开头的数据时，就可以停止查找了，因为后面的数据一定不满足要求。这样就可以利用索引了。

10. 索引失效的几种情况
    1.如果条件中有or，即使其中有条件带索引也不会使用(这也是为什么尽量少用or的原因)要想使用or，又想让索引生效，只能将or条件中的每个列都加上索引
    2.对于多列索引，不是使用的第一部分，则不会使用索引
    3.like查询以%开头
    4.如果列类型是字符串，那一定要在条件中将数据使用引号引用起来,否则不使用索引
    5.如果mysql估计使用全表扫描要比使用索引快,则不使用索引

11. MySQL主要提供2种方式的索引：B-Tree索引，Hash索引
    B树索引具有范围查找和前缀查找的能力，对于有N节点的B树，检索一条记录的复杂度为O(LogN)。相当于二分查找。哈希索引只能做等于查找，但是无论多大的Hash表，查找复杂度都是O(1)。
    显然，如果值的差异性大，并且以等值查找（=、 <、>、in）为主，Hash索引是更高效的选择，它有O(1)的查找复杂度。
    如果值的差异性相对较差，并且以范围查找为主，B树是更好的选择，它支持范围查找。

