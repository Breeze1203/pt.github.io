---
title: "In和Exists的优化案例讲解"
date: 2025-06-11 19:23:41
categories: MySQL
tags: MySQL
---

- Demo Table

- in的逻辑

- 优化原则

- exists的逻辑

## **Demo Table**

``` sql
CREATE TABLE t1 (
  id int(11) NOT NULL AUTO_INCREMENT,
  a int(11) DEFAULT NULL,
  b int(11) DEFAULT NULL,
  PRIMARY KEY (id),
  KEY idx_a (a)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

create table t2 like t1;
```

两个表t1 和 t2 ， 一样的，包括索引信息

数据量t1 ，t2 如下

``` sql
mysql> select count(1) from t1;
+----------+
| count(1) |
+----------+
|    10000 |
+----------+
1 row in set

mysql> select count(1) from t2;
+----------+
| count(1) |
+----------+
|      100 |
+----------+
1 row in set

mysql> 
```

------------------------------------------------------------------------

## **in的逻辑**

``` sql
select * from t1 where id in (select id from t2) ;
```

这个SQL，先执行哪个呢？

看看执行计划

<img src="https://cloud.cxykk.com/images/2024/2/2/1553/1706860398498.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />

可以理解为

``` sql
for(select id from t2){
      select * from t1 where t1.id = t2.id
    }
```

------------------------------------------------------------------------

### **优化原则**

原则：**小表驱动大表**，即小的数据集驱动大的数据集

**当T2表的数据集小于T1表的数据集时，in优于exists**

------------------------------------------------------------------------

## **exists的逻辑**

``` sql
select * from A where exists (select 1 from B where B.id = A.id)
```

可以理解为

``` sql
  for(select * from A){
     select * from B where B.id = A.id
   }
```

**当A表的数据集小于B表的数据集时，exists优于in**

将主查询A的数据，放到子查询B中做条件验证，根据验证结果（true或false）来决定主查询的数据是否保留

**1、** EXISTS(subquery)只返回TRUE或FALSE,因此子查询中的SELECT\*也可以用SELECT1替换,官方说法是实际执行时会忽略SELECT清单,因此没有区别；  
**2、** EXISTS子查询的实际执行过程可能经过了优化而不是我们理解上的逐条对比；  
**3、** EXISTS子查询往往也可以用JOIN来代替，何种最优需要具体问题具体分析；

``` sql
mysql> explain select * from t2 where exists (select 1 from t1 where t1.id = t2.id) ;
+----+--------------------+-------+------------+--------+---------------+---------+---------+---------------+------+----------+-------------+
| id | select_type        | table | partitions | type   | possible_keys | key     | key_len | ref           | rows | filtered | Extra       |
+----+--------------------+-------+------------+--------+---------------+---------+---------+---------------+------+----------+-------------+
|  1 | PRIMARY            | t2    | NULL       | ALL    | NULL          | NULL    | NULL    | NULL          |  100 |      100 | Using where |
|  2 | DEPENDENT SUBQUERY | t1    | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | artisan.t2.id |    1 |      100 | Using index |
+----+--------------------+-------+------------+--------+---------------+---------+---------+---------------+------+----------+-------------+
2 rows in set
```
