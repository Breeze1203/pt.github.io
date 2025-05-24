---
title: "Explain深度剖析"
date: 2024-11-18 15:41:52
---

## **Explain介绍**

使用EXPLAIN关键字可以模拟优化器执行SQL语句，分查询语句或是结构的性能瓶颈

在select 语句之前增加 explain 关键字，MySQL 会在查询上设置一个标记，执行查询会返回执行计划的信息，而不是执行这条SQL。

如果from 中包含子查询，仍会执行该子查询，将结果放入临时表中 。

------------------------------------------------------------------------

## **测试数据**

DBVersion

``` sql
 mysql> select version();
+------------+
| version()  |
+------------+
| 5.7.29-log |
+------------+
1 row in set (0.00 sec)

mysql> 
```

``` sql
DROP TABLE IF EXISTS actor;

CREATE TABLE actor (
 id int(11) NOT NULL,
 name varchar(45) DEFAULT NULL,
 update_time datetime DEFAULT NULL,
 PRIMARY KEY (id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO actor (id, name, update_time) VALUES (1,'a','2017-12-22 15:27:18'), (2,'b','2017-12-22 15:27:18'), (3,'c','2017-12-22 15:27:18');

###############################

DROP TABLE IF EXISTS film;

CREATE TABLE film (
 id int(11) NOT NULL AUTO_INCREMENT,
 name varchar(10) DEFAULT NULL,
 PRIMARY KEY (id),
 KEY idx_name (name)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO film (id, name) VALUES (3,'film0'),(1,'film1'),(2,'film2');

###############################

DROP TABLE IF EXISTS film_actor;

CREATE TABLE film_actor (
 id int(11) NOT NULL,
 film_id int(11) NOT NULL,
 actor_id int(11) NOT NULL,
 remark varchar(255) DEFAULT NULL,
 PRIMARY KEY (id),
 KEY idx_film_actor_id (film_id,actor_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO film_actor (id, film_id, actor_id) VALUES (1,1,1),(2,1,2),(3,2,1);
```

------------------------------------------------------------------------

## **explain 使用**

explain 两个扩展的使用

explain extended: 提供： 额外一些查询优化的信息 （**‘EXTENDED’ is deprecated and will be removed in a future release.**）

``` sql
mysql> explain extended select * from film where id=1;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | film  | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
1 row in set, 2 warnings (0.00 sec)
#   2 warnings

# 可以通过 show warnings 命令查看
mysql> show warnings;
+---------+------+----------------------------------------------------------------------------------+
| Level   | Code | Message                                                                          |
+---------+------+----------------------------------------------------------------------------------+
| Warning | 1681 | 'EXTENDED' is deprecated and will be removed in a future release.                |
| Note    | 1003 | /* select#1 */ select '1' AS id,'film1' AS name from dbtest.film where 1 |
+---------+------+----------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql>  
```

filtered 列： 百分比，计算公式 rows \* filtered/100 可以估算出将要和 explain 中前一个表进行连接的行数（前一个表指 explain 中的id值比当前表id值小的表） ， 供参考

------------------------------------------------------------------------

第二个\*\*‘PARTITIONS’ is deprecated and will be removed in a future\*\*

``` sql
mysql> explain partitions select * from film where id=1;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | film  | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
1 row in set, 2 warnings (0.00 sec)

mysql> show warnings;
+---------+------+----------------------------------------------------------------------------------+
| Level   | Code | Message                                                                          |
+---------+------+----------------------------------------------------------------------------------+
| Warning | 1681 | 'PARTITIONS' is deprecated and will be removed in a future release.              |
| Note    | 1003 | /* select#1 */ select '1' AS id,'film1' AS name from dbtest.film where 1 |
+---------+------+----------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

mysql> 
```

所以只使用explain就足够了 。

------------------------------------------------------------------------

## **explain重要列说明**

``` sql
mysql> explain select * from film_actor a where a.actor_id  = (select id from actor where name = 'a');
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | a     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |    33.33 | Using where |
|  2 | SUBQUERY    | actor | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |    33.33 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
```

### **id**

id列的编号是 select 的序列号，有几个 select 就有几个id，并且id的顺序是按 select 出现的顺序增长的。

执行顺序：

id越大执行优先级越高，id相同则从上往下执行，id为NULL最后执行

------------------------------------------------------------------------

### **select_type**

表示的对应行是简单还是复杂的查询

#### **simple**

简单查询，查询不包含子查询和union

``` sql
mysql> explain select * from film where id=1;
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | film  | NULL       | const | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+-------+---------------+---------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

mysql> 
```

------------------------------------------------------------------------

#### **primary**

复杂查询中最外层的 select

``` sql
mysql> explain select * from film_actor a where a.actor_id  = (select id from actor where name = 'a');
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | a     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |    33.33 | Using where |
|  2 | SUBQUERY    | actor | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |    33.33 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
```

------------------------------------------------------------------------

#### **subquery**

包含在select 中的子查询（不在 from 子句中）

``` sql
mysql> explain select * from film_actor a where a.actor_id  = (select id from actor where name = 'a');
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | PRIMARY     | a     | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |    33.33 | Using where |
|  2 | SUBQUERY    | actor | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |    33.33 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
```

------------------------------------------------------------------------

#### **derived**

**包含在 from 子句中的子查询。**

**MySQL会将结果存放在一个临时表中，也称为派生表**

derived: 衍生的

``` sql
mysql> set session optimizer_switch='derived_merge=off'; #关闭mysql5.7新特性对衍生表的合并优化
Query OK, 0 rows affected (0.00 sec)

mysql> explain select (select 1 from  actor where id = 1 ) from (select * from film where id =1 ) t ;
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------------+
| id | select_type | table      | partitions | type   | possible_keys | key     | key_len | ref   | rows | filtered | Extra       |
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------------+
|  1 | PRIMARY     | <derived3> | NULL       | system | NULL          | NULL    | NULL    | NULL  |    1 |   100.00 | NULL        |
|  3 | DERIVED     | film       | NULL       | const  | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL        |
|  2 | SUBQUERY    | actor      | NULL       | const  | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | Using index |
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------------+
3 rows in set, 1 warning (0.00 sec)

mysql> 
```

------------------------------------------------------------------------

#### **union**

在union 中的第二个和随后的 select

``` sql
mysql> EXPLAIN select 1 union select 1 ;
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| id | select_type  | table      | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra           |
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
|  1 | PRIMARY      | NULL       | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | No tables used  |
|  2 | UNION        | NULL       | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | No tables used  |
| NULL | UNION RESULT | <union1,2> | NULL       | ALL  | NULL          | NULL | NULL    | NULL | NULL |     NULL | Using temporary |
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
3 rows in set, 1 warning (0.00 sec)

mysql> 
```

------------------------------------------------------------------------

### **table**

表示explain 的一行正在访问哪个表

``` sql
mysql> explain select (select 1 from  actor where id = 1 ) from (select * from film where id =1 ) t ;
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------------+
| id | select_type | table      | partitions | type   | possible_keys | key     | key_len | ref   | rows | filtered | Extra       |
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------------+
|  1 | PRIMARY     | <derived3> | NULL       | system | NULL          | NULL    | NULL    | NULL  |    1 |   100.00 | NULL        |
|  3 | DERIVED     | film       | NULL       | const  | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL        |
|  2 | SUBQUERY    | actor      | NULL       | const  | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | Using index |
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------------+
3 rows in set, 1 warning (0.00 sec)
```

**当from 子句中有子查询时，table列是** `<derivenN>` 格式，表示当前查询依赖 id=N 的查询，于是先执行 id=N 的查询。

``` sql
mysql> EXPLAIN select 1 union select 1 ;
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| id | select_type  | table      | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra           |
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
|  1 | PRIMARY      | NULL       | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | No tables used  |
|  2 | UNION        | NULL       | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | No tables used  |
| NULL | UNION RESULT | <union1,2> | NULL       | ALL  | NULL          | NULL | NULL    | NULL | NULL |     NULL | Using temporary |
+----+--------------+------------+------------+------+---------------+------+---------+------+------+----------+-----------------+
3 rows in set, 1 warning (0.00 sec)

mysql> 
```

**当有 union 时，UNION RESULT 的 table 列的值为\<union1,2\>，1和2表示参与 union 的 select 行id。**

------------------------------------------------------------------------

### **type**

表示关联类型或访问类型，即MySQL决定如何查找表中的行，查找数据行记录的大概范围。

依次从最优到最差分别为：**system \> const \> eq_ref \> ref \> range \> index \> ALL**

**一般来说，得保证查询达到range级别，最好达到ref**

------------------------------------------------------------------------

#### **NULL**

**mysql能够在优化阶段分解查询语句，在执行阶段用不着再访问表或索引.**

例如：在索引列中选取最小值，可以单独查找索引来完成，不需要在执行时访问表

``` sql
mysql> explain select min(id) from actor;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra                        |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
|  1 | SIMPLE      | NULL  | NULL       | NULL | NULL          | NULL | NULL    | NULL | NULL |     NULL | Select tables optimized away |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+------------------------------+
1 row in set, 1 warning (0.00 sec)
```

------------------------------------------------------------------------

#### **const, system**

**mysql能对查询的某部分进行优化并将其转化成一个常量（可以看show warnings 的结果）。用于primary key 或 unique key 的所有列与常数比较时，所以表最多有一个匹配行，读取1次，速度比较快。**

**system是const的特例，表里只有一条元组匹配时为system.**

``` sql
mysql> EXPLAIN select * from (select * from film where id=1) t ;
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------+
| id | select_type | table      | partitions | type   | possible_keys | key     | key_len | ref   | rows | filtered | Extra |
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------+
|  1 | PRIMARY     | <derived2> | NULL       | system | NULL          | NULL    | NULL    | NULL  |    1 |   100.00 | NULL  |
|  2 | DERIVED     | film       | NULL       | const  | PRIMARY       | PRIMARY | 4       | const |    1 |   100.00 | NULL  |
+----+-------------+------------+------------+--------+---------------+---------+---------+-------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)

mysql> show warnings;
+-------+------+---------------------------------------------------------------+
| Level | Code | Message                                                       |
+-------+------+---------------------------------------------------------------+
| Note  | 1003 | /* select#1 */ select '1' AS id,'film1' AS name from dual |
+-------+------+---------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> 
```

------------------------------------------------------------------------

#### **eq_ref**

**primary key 或 unique key 索引的所有部分被连接使用 ，最多只会返回一条符合条件的记录。这可能是在const 之外最好的联接类型了，简单的 select 查询不会出现这种 type。**

``` sql
mysql> EXPLAIN select * from film_actor a  left join film b on a.film_id = b.id ;
+----+-------------+-------+------------+--------+---------------+---------+---------+------------------+------+----------+-------+
| id | select_type | table | partitions | type   | possible_keys | key     | key_len | ref              | rows | filtered | Extra |
+----+-------------+-------+------------+--------+---------------+---------+---------+------------------+------+----------+-------+
|  1 | SIMPLE      | a     | NULL       | ALL    | NULL          | NULL    | NULL    | NULL             |    3 |   100.00 | NULL  |
|  1 | SIMPLE      | b     | NULL       | eq_ref | PRIMARY       | PRIMARY | 4       | dbtest.a.film_id |    1 |   100.00 | NULL  |
+----+-------------+-------+------------+--------+---------------+---------+---------+------------------+------+----------+-------+
2 rows in set, 1 warning (0.00 sec)

mysql> show warnings;
+-------+------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Level | Code | Message                                                                                                                                                                                                                                                                                                                                   |
+-------+------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Note  | 1003 | /* select#1 */ select dbtest.a.id AS id,dbtest.a.film_id AS film_id,dbtest.a.actor_id AS actor_id,dbtest.a.remark AS remark,dbtest.b.id AS id,dbtest.b.name AS name from dbtest.film_actor a left join dbtest.film b on((dbtest.b.id = dbtest.a.film_id)) where 1 |
+-------+------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> 
```

------------------------------------------------------------------------

#### **ref**

**相比 eq_ref，不使用唯一索引，而是使用普通索引或者唯一性索引的部分前缀，索引要和某个值相比较，可能会找到多个符合条件的行.**

【简单 select 查询，name是普通索引（非唯一索引）】

``` sql
mysql> show INDEX  from  film ;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| film  |          0 | PRIMARY  |            1 | id          | A         |           3 |     NULL | NULL   |      | BTREE      |         |               |
| film  |          1 | idx_name |            1 | name        | A         |           3 |     NULL | NULL   | YES  | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
2 rows in set (0.00 sec)

mysql> 
mysql> EXPLAIN select * from film a where a.name = 'film0';
+----+-------------+-------+------------+------+---------------+----------+---------+-------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key      | key_len | ref   | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | a     | NULL       | ref  | idx_name      | idx_name | 33      | const |    1 |   100.00 | Using index |
+----+-------------+-------+------------+------+---------------+----------+---------+-------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> 
```

【关联表查询，idx_film_actor_id是film_id和actor_id的联合索引，这里使用到了film_actor的左边前缀film_id部分。】

``` sql
mysql> show index from film_actor;
+------------+------------+-------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table      | Non_unique | Key_name          | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+------------+------------+-------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| film_actor |          0 | PRIMARY           |            1 | id          | A         |           3 |     NULL | NULL   |      | BTREE      |         |               |
| film_actor |          1 | idx_film_actor_id |            1 | film_id     | A         |           2 |     NULL | NULL   |      | BTREE      |         |               |
| film_actor |          1 | idx_film_actor_id |            2 | actor_id    | A         |           3 |     NULL | NULL   |      | BTREE      |         |               |
+------------+------------+-------------------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
3 rows in set (0.00 sec)

mysql> 
mysql> EXPLAIN select film_id from film  left join  film_actor  on film.id =film_actor.film_id ;
+----+-------------+------------+------------+-------+-------------------+-------------------+---------+----------------+------+----------+-------------+
| id | select_type | table      | partitions | type  | possible_keys     | key               | key_len | ref            | rows | filtered | Extra       |
+----+-------------+------------+------------+-------+-------------------+-------------------+---------+----------------+------+----------+-------------+
|  1 | SIMPLE      | film       | NULL       | index | NULL              | idx_name          | 33      | NULL           |    3 |   100.00 | Using index |
|  1 | SIMPLE      | film_actor | NULL       | ref   | idx_film_actor_id | idx_film_actor_id | 4       | dbtest.film.id |    1 |   100.00 | Using index |
+----+-------------+------------+------------+-------+-------------------+-------------------+---------+----------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)

mysql> 
```

------------------------------------------------------------------------

#### **range**

**范围扫描通常出现在 in(), between ,\> ,\<, \>= 等操作中。使用一个索引来检索给定范围的行。**

``` sql
mysql> explain select * from actor  where id > 1 ;
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | actor | NULL       | range | PRIMARY       | PRIMARY | 4       | NULL |    2 |   100.00 | Using where |
+----+-------------+-------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

------------------------------------------------------------------------

#### **index**

**扫描全索引就能拿到结果，一般是扫描某个二级索引，这种扫描不会从索引树根节点开始快速查找，而是直接对二级索引的叶子节点遍历和扫描，速度还是比较慢的，这种查询一般为使用覆盖索引，二级索引一般比较小，所以这种通常比ALL快一些**

``` sql
mysql> explain select * from film;
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | film  | NULL       | index | NULL          | idx_name | 33      | NULL |    3 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

------------------------------------------------------------------------

#### **ALL**

**全表扫描，扫描你的聚簇索引的所有叶子节点。通常情况下这需要增加索引来进行优化了**

``` sql
mysql> explain select * from actor ;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
|  1 | SIMPLE      | actor | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |   100.00 | NULL  |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```

------------------------------------------------------------------------

### **possible_keys**

**显示查询可能使用哪些索引来查找**

explain 时可能出现 possible_keys 有列，而 key 显示 NULL 的情况，这种情况是因为表中数据不多，mysql认为索引对此查询帮助不大，选择了全表查询。

如果该列是NULL，则没有相关的索引。在这种情况下，可以通过检查 where 子句看是否可以创造一个适当的索引来提高查询性能，然后用 explain 查看效果。

------------------------------------------------------------------------

### **key**

**mysql实际采用哪个索引来优化对该表的访问。**

如果没有使用索引，则该列是 NULL。如果想强制mysql使用或忽视possible_keys列中的索引，在查询中使用 force index、ignore index。

------------------------------------------------------------------------

### **key_len**

**显示了mysql在索引里使用的字节数，通过这个值可以算出具体使用了索引中的哪些列。**

举个例子 ：

film_actor的联合索引 idx_film_actor_id 由 film_id 和 actor_id 两个int列组成，并且每个int是4字节。

``` sql
mysql> explain select * from film_actor where film_id=1;
+----+-------------+------------+------------+------+-------------------+-------------------+---------+-------+------+----------+-------+
| id | select_type | table      | partitions | type | possible_keys     | key               | key_len | ref   | rows | filtered | Extra |
+----+-------------+------------+------------+------+-------------------+-------------------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | film_actor | NULL       | ref  | idx_film_actor_id | idx_film_actor_id | 4       | const |    2 |   100.00 | NULL  |
+----+-------------+------------+------------+------+-------------------+-------------------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)

mysql> 
```

通过结果中的key_len=4可推断出查询使用了第一个列：film_id列来执行索引查找。

#### **key_len计算规则**

【字符串】

- char(n)：n字节长度

- varchar(n)：如果是utf-8，则长度 3n + 2 字节，加的2字节用来存储字符串长度

【数值类型】

- tinyint：1字节

- smallint：2字节

- int：4字节

- bigint：8字节

【时间类型】

- date：3字节

- timestamp：4字节

- datetime：8字节

**如果字段允许为 NULL，需要1字节记录是否为 NULL**

索引最大长度是768字节，当字符串过长时，mysql会做一个类似左前缀索引的处理，将前半部分的字符提取出来做索引

------------------------------------------------------------------------

### **ref**

显示了在key列记录的索引中，表查找值所用到的列或常量，**常见的有：const（常量），字段名（例：film.id）**

``` sql
mysql> explain select * from film_actor where film_id=1;
+----+-------------+------------+------------+------+-------------------+-------------------+---------+-------+------+----------+-------+
| id | select_type | table      | partitions | type | possible_keys     | key               | key_len | ref   | rows | filtered | Extra |
+----+-------------+------------+------------+------+-------------------+-------------------+---------+-------+------+----------+-------+
|  1 | SIMPLE      | film_actor | NULL       | ref  | idx_film_actor_id | idx_film_actor_id | 4       | const |    2 |   100.00 | NULL  |
+----+-------------+------------+------------+------+-------------------+-------------------+---------+-------+------+----------+-------+
1 row in set, 1 warning (0.00 sec)
```

``` sql
mysql> EXPLAIN select film_id from film  left join  film_actor  on film.id =film_actor.film_id ;
+----+-------------+------------+------------+-------+-------------------+-------------------+---------+----------------+------+----------+-------------+
| id | select_type | table      | partitions | type  | possible_keys     | key               | key_len | ref            | rows | filtered | Extra       |
+----+-------------+------------+------------+-------+-------------------+-------------------+---------+----------------+------+----------+-------------+
|  1 | SIMPLE      | film       | NULL       | index | NULL              | idx_name          | 33      | NULL           |    3 |   100.00 | Using index |
|  1 | SIMPLE      | film_actor | NULL       | ref   | idx_film_actor_id | idx_film_actor_id | 4       | dbtest.film.id |    1 |   100.00 | Using index |
+----+-------------+------------+------------+-------+-------------------+-------------------+---------+----------------+------+----------+-------------+
2 rows in set, 1 warning (0.00 sec)
```

------------------------------------------------------------------------

### **rows**

**mysql估计要读取并检测的行数，注意这个不是结果集里的行数。**

------------------------------------------------------------------------

### **Extra**

展示的是额外信息。

列举几个常见的值

#### **Using index**

**使用覆盖索引** ： 无需回表

mysql执行计划explain结果里的key有使用索引，如果select后面查询的字段都可以从这个索引的树中获取，这种情况一般可以说是用到了覆盖索引，extra里一般都有using index；

覆盖索引一般针对的是辅助索引，整个查询结果只通过辅助索引就能拿到结果，不需要通过辅助索引树找到主键，再通过主键去主键索引树里获取其它字段值

``` sql
mysql> explain select film_id from film_actor where film_id=1;
+----+-------------+------------+------------+------+-------------------+-------------------+---------+-------+------+----------+-------------+
| id | select_type | table      | partitions | type | possible_keys     | key               | key_len | ref   | rows | filtered | Extra       |
+----+-------------+------------+------------+------+-------------------+-------------------+---------+-------+------+----------+-------------+
|  1 | SIMPLE      | film_actor | NULL       | ref  | idx_film_actor_id | idx_film_actor_id | 4       | const |    2 |   100.00 | Using index |
+----+-------------+------------+------------+------+-------------------+-------------------+---------+-------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> 
```

------------------------------------------------------------------------

#### **Using where**

**使用 where 语句来处理结果，并且查询的列未被索引覆盖**

``` sql
mysql> show index from actor;
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
| actor |          0 | PRIMARY  |            1 | id          | A         |           3 |     NULL | NULL   |      | BTREE      |         |               |
+-------+------------+----------+--------------+-------------+-----------+-------------+----------+--------+------+------------+---------+---------------+
1 row in set (0.00 sec)

mysql> explain select * from actor where name = 'a';
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | actor | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |    33.33 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> 
```

------------------------------------------------------------------------

#### **Using index condition**

**查询的列不完全被索引覆盖，where条件中是一个前导列的范围；**

``` sql
mysql> explain select * from film_actor where film_id > 1 ;
+----+-------------+------------+------------+-------+-------------------+-------------------+---------+------+------+----------+-----------------------+
| id | select_type | table      | partitions | type  | possible_keys     | key               | key_len | ref  | rows | filtered | Extra                 |
+----+-------------+------------+------------+-------+-------------------+-------------------+---------+------+------+----------+-----------------------+
|  1 | SIMPLE      | film_actor | NULL       | range | idx_film_actor_id | idx_film_actor_id | 4       | NULL |    1 |   100.00 | Using index condition |
+----+-------------+------------+------------+-------+-------------------+-------------------+---------+------+------+----------+-----------------------+
1 row in set, 1 warning (0.00 sec)

mysql> 
```

------------------------------------------------------------------------

#### **Using temporary**

**mysql需要创建一张临时表来处理查询。出现这种情况一般是要进行优化的，首先是想到用索引来优化。**

【actor.name没有索引，此时创建了张临时表来distinct】

``` sql
mysql> explain select distinct name from actor;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra           |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
|  1 | SIMPLE      | actor | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |   100.00 | Using temporary |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-----------------+
1 row in set, 1 warning (0.00 sec)
```

【film.name建立了idx_name索引，此时查询时extra是using index,没有用临时表】

``` sql
mysql> explain select distinct name from film;
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | film  | NULL       | index | idx_name      | idx_name | 33      | NULL |    3 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> 
```

------------------------------------------------------------------------

#### **Using filesort**

**将用外部排序而不是索引排序，数据较小时从内存排序，否则需要在磁盘完成排序。这种情况下一般也是要考虑使用索引来优化的**

【actor.name未创建索引，会浏览actor整个表，保存排序关键字name和对应的id，然后排序name并检索行记录】

``` sql
mysql> explain select * from actor order by name;
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra          |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
|  1 | SIMPLE      | actor | NULL       | ALL  | NULL          | NULL | NULL    | NULL |    3 |   100.00 | Using filesort |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+----------------+
1 row in set, 1 warning (0.00 sec)

mysql> 
```

【film.name建立了idx_name索引,此时查询时extra是using index】

``` sql
mysql> explain select * from film order by name ;
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type  | possible_keys | key      | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | film  | NULL       | index | NULL          | idx_name | 33      | NULL |    3 |   100.00 | Using index |
+----+-------------+-------+------------+-------+---------------+----------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)

mysql> 
```

------------------------------------------------------------------------

#### **Select tables optimized away**

使用某些聚合函数（比如 max、min）来访问存在**索引**的某个字段

<img src="https://cloud.cxykk.com/images/2024/2/2/160/1706860844809.png" style="display: inline-block;width:100.0%;height:100.0%" alt="*" />
