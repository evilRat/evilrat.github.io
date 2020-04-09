---
layout: post
title: "sql学习笔记"
date: 2016-06-10
excerpt: "love you forever"
tags: [sql,select,distinct,group by,order by]
comments: true
---


## sql学习笔记

### 关于DISTINCT

DISTINCT可以去除重复的内容，但是，如果查询的数据是多个列，那么只有在这多个列的数据都相同的时候才可以消除。如果一个列重复，另一个列不重复，那么这一行也不会被消除。

### 四则运算可以作为SELECT参数

### 给计算结果设计别名

SELECT empno，ename，sal*12 income FROM emp;

这里打印出来的结果中sal*12那一列的列名就是income。

### 常量如果是字符串要使用单引号而不是双引号，如果是数字不用加引号，如果是日期，就要按照日期格式编写。

### 两列内容的连接使用||

select empno||ename from emp;

这里的使用方法很像java里面的“+”：

<font face="黑体">例如：</font>

	select '雇员编号：'|| empno ||',姓名：'|| ename ||',收入：'|| income from student;


### BETWEEN 最小值 AND 最大值;

<font color="red">这里一定要注意是闭区间！！！</font>

### 空判断 IS NULL和IS NOT NULL

### IN和NOT IN

BETWEEN AND 给了一个大的可选范围，IN也用来规定一个范围，不过用起来更灵活。

<font face="黑体">例如：</font>

SELECT * FROM emp WHERE empno=1 OR empno=2 OR empno=3;
这句代码使用IN来做就是：
SELECT * FROM emp WHERE empno IN (1,2,3);

指定值查找使用IN会比较方便


### 关于NOT IN和NULL的问题

使用NOT IN进行范围判断的时候，如果范围里面包括NULL，那么就不会有任何结果。

<font face="黑体">例如：</font>
SELECT * FROM emp WHERE empno NOT IN(1,2,3,NULL);

之所以使用WHERE，就是要抓取有用信息，没有限制，显示所有行，对于大型数据库根本没有意义。

使用NOT IN的目的是为了查询部分数据行，但是如果有了NULL（某些数据永远不可能为NULL）,就成了查询全部了。



为什么sql里面NOT IN后面的子查询如果有记录为NULL的，主查询就查不到记录？？？原因很简单：
SELECT *
FROM dbo.TableA AS a
WHERE a.id NOT IN ( 2, NULL )

等同于：
SELECT *
FROM Table_A AS a
WHERE a.id <> 2
AND a.ID <> NULL



<font color="red">于NULL值不能参与比较运算符，导致条件不成立，查询不出来数据。</font>


### LIKE

"_":匹配任意以为字符；
"%":匹配任意的零位，多位字符。

<font face="黑体">注意：</font>
LIKE 可以应用在各种数据类型上，不一定是字符串；
LIKE 如果不设置关键字，那么表示查询全部信息，就像LIKE '%%'。虽然这样可以查询全部数据，但是与不使用WHERE子句相比，不使用WHERE子句的效率更高。

### ORDER BY

排序方式有两种ASC(默认)和DESC。


### COUNT(),MAX(),MIN(),SUM(),AVG()

count是统计个数，里面可以跟上<font color="red">distinct</font>字段。
max和min也可以用于<font color="red">日期</font>类型的数据。

<font face="黑体">注意：</font>

COUNT(*),COUNT(字段),COUNT(DISTINCT 字段)的区别？

* COUNT(*):明确的返回表中的数据个数，是最准确的；
* COUNT(字段):不统计为null的数据个数，如果某一列的数据不可能为null，那么结果与COUNT(*)相同；
* COUNT(DISTINCT 字段):统计消除掉重复数据后的数据个数。



### GROUP BY

```

SELECT job,COUNT(empno),AVG(sal)
FROM emp
GROUP BY job;

```


```

SELECT DEPTNO,COUNT(empno),MAX(SAL),MIN(SAL)
FROM EMP
GROUP BY DEPTNO;


```
* 没有编写group by子句的时候（全表作为一组），那么select子句之中只允许出现统计函数，不允许出现其他字段。
例如：
select count(empno),ename from emp;
这里查询结果里面第一列已经显示了empno的数目了，这肯定只有一行，所以第二列不可能列出很多行ename的数据了，因为这不符合数据库的表达形式。


* 在使用group by子句分组的时候，select子句之中只允许出现分组字段与统计函数，其他字段不允许出现。

正确代码：
```
select job,count(empno) from emp group by job;
```

错误代码：
```
select job,count(empno),ename from group by job; 
```

* 统计函数允许嵌套查询，但是嵌套后的统计查询中，select子句中不允许再出现任何的字段，包括分组字段，只能够使用嵌套的统计函数。

正确代码：

```

SELECT deptno,AVG(sal)
FROM emp 
GROUP BY deptno;

```

错误代码：

```
SELECT deptno,MAX(AVG(sal))
FROM emp
GROUP BY deptno;
```
这里已经有了嵌套的统计函数，就不能再有deptno了。


修改：

```
SELECT MAX(AVG(sal))
FROM emp
GROUP BY deptno;

```

### 多表查询

<font face="黑体">示例：</font>

查询出每个部门的名称、人数、平均工资：

分析：
1.确定要使用的表：
（1）dept:部门名称
（2）emp:统计出人数，平均工资
2.确定已知的关联字段：
雇员与部门：emp.deptno=dept.deptno


#### 第一步：查询每个雇员的编号，部门名称，工资
```

SELECT e.empno,d.dnama,e.sal
FROM emp e,dept d
WHERE e.deptno=d.deptno;

```

#### 第二步：通过以上的查询可以发现dname字段上出现了重复查询，有重复数据才可以分组。另外我们的查询明确要求是根据部门名称分组，现在对查询结果分组。（上面查询出来的结果可以看作是一张临时数据表）

```

SELECT e.empno,d.dnama,e.sal
FROM emp e,dept d
WHERE e.deptno=d.deptno
GROUP BY d.name;

```

#### 第三步：部门一共有三个，但是我们现在只出现了三个，加入外连接控制

```

SELECT e.empno,d.dnama,e.sal
FROM emp e,dept d
WHERE e.deptno(+)=d.deptno
GROUP BY d.name;

```

#### 查询成功。











<html>
<div class="ds-thread" data-thread-key="http://kongzheng1993.github.io/kongzheng1993-sql/" data-title="lover" data-url="http://kongzheng1993.github.io/kongzheng1993-sql/"></div>
<!-- 多说评论框 end -->
<!-- 多说公共JS代码 start (一个网页只需插入一次) -->
<script type="text/javascript">
var duoshuoQuery = {short_name:"kongzheng1993"};
	(function() {
		var ds = document.createElement('script');
		ds.type = 'text/javascript';ds.async = true;
		ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
		ds.charset = 'UTF-8';
		(document.getElementsByTagName('head')[0] 
		 || document.getElementsByTagName('body')[0]).appendChild(ds);
	})();
</script>
</html>
