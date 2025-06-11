---
title: "orderby和groupby优化初探"
date: 2025-06-11 19:23:40
categories: MySQL
tags: MySQL
---

#### **文章目录**

- DB Version

- Table

- 数据量

- 案例一 ：explain select \* from employees where name = 'LiLei' and position = 'dev' order by age

- 案例二： explain select \* from employees where name = 'LiLei' order by position

- 案例三：explain select \* from employees where name = 'LiLei' order by age , position

- 案例四：explain select \* from employees where name = 'LiLei' order by position , age

- 案例五：explain select \* from employees where name = 'LiLei' and age = 18 order by position , age ;

- 案例六：explain select \* from employees where name = 'LiLei' order by age asc , position desc ;

- 案例七：explain select \* from employees where name in ('HanMeiMei' , 'LiLei') order by age , position ;

- 案例八： explain select \* from employees where name \> 'HanMeiMei' order by name ;

- group by 优化

- 小结

## **DB Version**

``` sql
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.7.28    |
+-----------+
1 row in set

mysql> 
```

------------------------------------------------------------------------

## **Table**

``` sql
CREATE TABLE employees (
  id int(11) NOT NULL AUTO_INCREMENT,
  name varchar(24) NOT NULL DEFAULT '' COMMENT '姓名',
  age int(11) NOT NULL DEFAULT '0' COMMENT '年龄',
  position varchar(20) NOT NULL DEFAULT '' COMMENT '职位',
  hire_time timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '入职时间',
  PRIMARY KEY (id),
  KEY idx_name_age_position (name,age,position) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COMMENT='员工记录表';
```

两个索引

**1、** 主键索引；  
**2、** 二级索引KEY`idx_name_age_position`(`name`,`age`,`position`)USINGBTREE；

重点就是这个二级索引 ，记号了哈。

------------------------------------------------------------------------

## **数据量**

``` sql
mysql> select count(1) from employees ;
+----------+
| count(1) |
+----------+
|   100002 |
+----------+
1 row in set

mysql> 
```

------------------------------------------------------------------------

## **案例一 ：explain select \* from employees where name = ‘LiLei’ and position = ‘dev’ order by age**

``` sql
explain select * from employees where name = 'LiLei' and position = 'dev' order by age ;
```

先想一下这个order by 会不会走索引 ？

<img src="https://cloud.cxykk.com/images/2024/2/2/1554/1706860497077.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

会走索引

原因呢？

脑海中要有这个联合索引在MySQL底层的B+Tree的数据结构 ， 索引 排好序的数据结构。

<img src="https://cloud.cxykk.com/images/2024/2/2/1555/1706860502551.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

name = ‘LiLei’ and position = ‘dev’ order by age

**name 为 LiLei , name 确定的情况下， age 肯定是有序的 ，age 有序不能保证position 有序**

所以这个order by age 是可以走索引的

继续分析下这个explain

``` sql
mysql> explain select * from employees where name = 'LiLei' and position = 'dev' order by age ;
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-----------------------+
| id | select_type | table     | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra                 |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-----------------------+
|  1 | SIMPLE      | employees | NULL       | ref  | idx_name_age_position | idx_name_age_position | 74      | const |    1 |       10 | Using index condition |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-----------------------+
1 row in set
```

order by 走的索引 是不会体现在key_len上的, 这个74 = 3 \* 24 + 2 ， 是计算的name 。 最左匹配原则 ，中间字段不能断，因此查询用到了name索引。

但是Extra直接里面可以看出来 Using index condition ，说明age索引列用在了排序过程中 。 如果没有走索引的话，那就是 Using FileSort 了

接下来继续看几个例子，加深理解，**重点是脑海中的 索引B+Tree结构**

------------------------------------------------------------------------

## **案例二： explain select \* from employees where name = ‘LiLei’ order by position**

``` sql
mysql> explain select * from employees where  name = 'LiLei' order by position ;     
```

想一想，这个order by 会走索引吗？

我们来看下索引 KEY `idx_name_age_position` (`name`,`age`,`position`) USING BTREE

再来看下查询SQL

``` sql
 where  name = 'LiLei' order by position ;  
```

name = LiLei , name 值能确定下来， 符合最左匹配原则 所以查询会走索引 ， 用了联合索引中的name字段， key len = 74 . 所以 Using index condition

order by position , 在索引中 中间缺失了age , 用position ，跳过了age , 那索引树能是有序的吗？ 肯定不是。。。所以 position肯定不是排好序的 ， 无法走索引排序，因此 Extra信息 有 Using filesort

来看下执行计划

``` sql
mysql> explain select * from employees where  name = 'LiLei' order by position ;     
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
| id | select_type | table     | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra                                 |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
|  1 | SIMPLE      | employees | NULL       | ref  | idx_name_age_position | idx_name_age_position | 74      | const |    1 |      100 | Using index condition; Using filesort |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
1 row in set

mysql> 
```

正如分析~

有感觉了吗？ 再来看一个

------------------------------------------------------------------------

## **案例三：explain select \* from employees where name = ‘LiLei’ order by age , position**

这个SQL和案例二的很相似 ， 仅仅在排序的时候在前面多了一个age字段参与排序 ， 那分析分析 order by 会走索引吗

``` sql
mysql> explain select * from employees where  name = 'LiLei' order by age , position ;
```

**时刻不要那个索引树** ，来分析一下

name = LiLei , name 固定，结合 建立的索引， 最左原则，所以查询肯定会走联合索引中的部分索引 name .

在name都是LiLei 的情况下 ， order by age , position 结合索引树 ，age和position用于排序 也是有序的，应该不会走using filesort

我们来看下执行计划

<img src="https://cloud.cxykk.com/images/2024/2/2/1555/1706860508386.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

``` sql
mysql> explain select * from employees where  name = 'LiLei' order by age , position ;
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-----------------------+
| id | select_type | table     | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra                 |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-----------------------+
|  1 | SIMPLE      | employees | NULL       | ref  | idx_name_age_position | idx_name_age_position | 74      | const |    1 |      100 | Using index condition |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+-----------------------+
1 row in set

mysql> 
```

------------------------------------------------------------------------

## **案例四：explain select \* from employees where name = ‘LiLei’ order by position , age**

再分析一个，和案例上也很像。 把 order by的排序顺序 调整一下，我们来分析一下 order by会不会走索引

``` sql
explain select * from employees where  name = 'LiLei' order by position , age ;   
```

<img src="https://cloud.cxykk.com/images/2024/2/2/1555/1706860514047.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

``` sql
mysql> explain select * from employees where  name = 'LiLei' order by position , age ;   
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
| id | select_type | table     | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra                                 |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
|  1 | SIMPLE      | employees | NULL       | ref  | idx_name_age_position | idx_name_age_position | 74      | const |    1 |      100 | Using index condition; Using filesort |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
1 row in set

mysql> 
```

咦，执行计划中有 using filesort

为什么呢？

**看看我们二级索引的建立的字段顺序 ， 创建顺序为name,age,position，但是排序的时候age和position颠倒位置了， 那排好序的特性肯定就无法满足了，那你让MySQL怎么走索引？**

------------------------------------------------------------------------

## **案例五：explain select \* from employees where name = ‘LiLei’ and age = 18 order by position , age ;**

这个order by 会走索引吗？

<img src="https://cloud.cxykk.com/images/2024/2/2/1555/1706860519865.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

``` sql
mysql> explain select * from employees where name = 'LiLei' and age = 18 order by position , age ;
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------------+------+----------+-----------------------+
| id | select_type | table     | partitions | type | possible_keys         | key                   | key_len | ref         | rows | filtered | Extra                 |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------------+------+----------+-----------------------+
|  1 | SIMPLE      | employees | NULL       | ref  | idx_name_age_position | idx_name_age_position | 78      | const,const |    1 |      100 | Using index condition |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------------+------+----------+-----------------------+
1 row in set

mysql> 
```

**走了dx_name_age_position 索引中的 name 和 age , order by 其实也走了索引，你看extra中并没有 using filesort ,因为age为常量，在排序中被MySQL优化了，所以索引未颠倒，不会出现Using filesort**

------------------------------------------------------------------------

## **案例六：explain select \* from employees where name = ‘LiLei’ order by age asc , position desc ;**

``` sql
mysql> explain select * from employees where name = 'LiLei'  order by  age asc , position desc ;  
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
| id | select_type | table     | partitions | type | possible_keys         | key                   | key_len | ref   | rows | filtered | Extra                                 |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
|  1 | SIMPLE      | employees | NULL       | ref  | idx_name_age_position | idx_name_age_position | 74      | const |    1 |      100 | Using index condition; Using filesort |
+----+-------------+-----------+------------+------+-----------------------+-----------------------+---------+-------+------+----------+---------------------------------------+
1 row in set
```

<img src="https://cloud.cxykk.com/images/2024/2/2/1555/1706860525579.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

我们可以看到虽然排序的字段列与建立索引的顺序一样， order by默认升序排列，而SQL中的 position desc变成了降序排列，导致与索引的排序方式不同，从而产生Using filesort。

Note: Mysql8以上版本有降序索引可以支持该种查询方式。

------------------------------------------------------------------------

## **案例七：explain select \* from employees where name in (‘HanMeiMei’ , ‘LiLei’) order by age , position ;**

``` sql
mysql> explain select * from employees where name in ('HanMeiMei' , 'LiLei')  order by  age  , position  ;    
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+---------------------------------------+
| id | select_type | table     | partitions | type  | possible_keys         | key                   | key_len | ref  | rows | filtered | Extra                                 |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+---------------------------------------+
|  1 | SIMPLE      | employees | NULL       | range | idx_name_age_position | idx_name_age_position | 74      | NULL |    2 |      100 | Using index condition; Using filesort |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+------+----------+---------------------------------------+
1 row in set

mysql> 
```

对order by 来讲 ，多个相等的条件也是 范围查询。 既然是范围查询， 可能对于每个值在索引中是有序的，但多个合并在一起，就不是有序的了，所以 using filesort .

------------------------------------------------------------------------

## **案例八： explain select \* from employees where name \> ‘HanMeiMei’ order by name ;**

``` sql
mysql> explain select * from employees where name > 'HanMeiMei'  order by  name  ;       
+----+-------------+-----------+------------+------+-----------------------+------+---------+------+-------+----------+-----------------------------+
| id | select_type | table     | partitions | type | possible_keys         | key  | key_len | ref  | rows  | filtered | Extra                       |
+----+-------------+-----------+------------+------+-----------------------+------+---------+------+-------+----------+-----------------------------+
|  1 | SIMPLE      | employees | NULL       | ALL  | idx_name_age_position | NULL | NULL    | NULL | 96845 |       50 | Using where; Using filesort |
+----+-------------+-----------+------------+------+-----------------------+------+---------+------+-------+----------+-----------------------------+
1 row in set

mysql> 
```

MySQL自己内部有一套优化机制，且数据量不同、版本不一样，结果也可能有差异

**一般情况下， 联合索引第一个字段用范围不一定会走索引 ， 可以采用 覆盖索引进行优化，避免回表带来的性能开销 。**

``` sql
mysql> explain select name
 from employees where name > 'a'  order by  name  ;       
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+-------+----------+--------------------------+
| id | select_type | table     | partitions | type  | possible_keys         | key                   | key_len | ref  | rows  | filtered | Extra                    |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+-------+----------+--------------------------+
|  1 | SIMPLE      | employees | NULL       | range | idx_name_age_position | idx_name_age_position | 74      | NULL | 48422 |      100 | Using where; Using index |
+----+-------------+-----------+------------+-------+-----------------------+-----------------------+---------+------+-------+----------+--------------------------+
1 row in set

mysql> 
```

<img src="https://cloud.cxykk.com/images/2024/2/2/1555/1706860531033.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

------------------------------------------------------------------------

## **group by 优化**

- group by与order by类似，其实质是**先排序后分组**，**遵照索引创建顺序的最左前缀法则**。

- 对于group by的优化如果不需要排序的可以加上**order by null禁止排序**。

- where高于having，能写在where中的限定条件就不要去having限定了。

------------------------------------------------------------------------

## **小结**

- MySQL支持两种方式的排序filesort和index，Using index是指MySQL扫描索引本身完成排序

- order by满足两种情况会使用Using index  
  A: order by语句使用索引最左前列。  
  B: 使用where子句与order by子句条件列组合满足索引最左前列

- 尽量在索引列上完成排序，遵循索引建立（索引创建的顺序）时的最左前缀法则

- 如果order by的条件不在索引列上，就会产生Using filesort

- 能用覆盖索引尽量用覆盖索引
