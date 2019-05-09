# InnoDB学习之表(四)
## 分区概述
不管创建何种类型的分区,如果表中存在主键和唯一的索引时,分区列必须要是唯一索引的组成部分,我们看一下下面的这个例子:
```
mysql> create table t1(
    -> col1 int not null,
    -> col2 date not null,
    -> col3 int not null,
    -> col4 int not null,
    -> unique key(col1,col2))
    -> partition by hash(col3)
    -> partition 4;
ERROR 1064 (42000): You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'partition 4' at line 8

mysql> create table t1(col1 int null,col2 int null,col3 int null,col4 int not null,unique key(col1,col2,col3,col4))
    -> partition by hash(col3)
    -> partitions 4;
Query OK, 0 rows affected (0.24 sec)
```
当没有指定唯一的索引的时候,可以指定任何一列为分区列,我们看一下下面的表的创建
```
mysql> create table t2(col1 int null,col2 date null,col3 int null,col4 int null,key(col4))engine =innodb partition by hash(col3) partitions 4;
Query OK, 0 rows affected (0.23 sec)
```
## 分区类型
### range分区
```
mysql> create table t(
    -> id int)engine=Innodb
    -> partition by range(id)
    -> (partition p0 values less than (10),
    -> partition p1 values less than(20));
Query OK, 0 rows affected (0.12 sec)
```
然后我们看一下表空间
```
root@chengcongyue:/var/lib/mysql/LearningInnoDb3# ls
db.opt  t.frm  t#P#p0.ibd  t#P#p1.ibd
```
我们看到其中的存储方式就发生变化了,当前的这个表分成两个表空间进行存储,然后我们插入一些数据,进行测试
```
mysql> insert into t select 9;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> insert into t select 10;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> insert into t select 15;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0
```
然后我们通过一下的查询来看一下数据的存储,分别在哪儿
```
mysql> select * from information_schema.partitions where 
    -> table_schema=database() and table_name='t';
+---------------+-----------------+------------+----------------+-------------------+----------------------------+-------------------------------+------------------+---------------------+----------------------+-------------------------+-----------------------+------------+----------------+-------------+-----------------+--------------+-----------+---------------------+---------------------+------------+----------+-------------------+-----------+-----------------+
| TABLE_CATALOG | TABLE_SCHEMA    | TABLE_NAME | PARTITION_NAME | SUBPARTITION_NAME | PARTITION_ORDINAL_POSITION | SUBPARTITION_ORDINAL_POSITION | PARTITION_METHOD | SUBPARTITION_METHOD | PARTITION_EXPRESSION | SUBPARTITION_EXPRESSION | PARTITION_DESCRIPTION | TABLE_ROWS | AVG_ROW_LENGTH | DATA_LENGTH | MAX_DATA_LENGTH | INDEX_LENGTH | DATA_FREE | CREATE_TIME         | UPDATE_TIME         | CHECK_TIME | CHECKSUM | PARTITION_COMMENT | NODEGROUP | TABLESPACE_NAME |
+---------------+-----------------+------------+----------------+-------------------+----------------------------+-------------------------------+------------------+---------------------+----------------------+-------------------------+-----------------------+------------+----------------+-------------+-----------------+--------------+-----------+---------------------+---------------------+------------+----------+-------------------+-----------+-----------------+
| def           | LearningInnoDb3 | t          | p0             | NULL              |                          1 |                          NULL | RANGE            | NULL                | id                   | NULL                    | 10                    |          1 |          16384 |       16384 |            NULL |            0 |         0 | 2019-04-25 17:27:06 | 2019-04-25 17:30:10 | NULL       |     NULL |                   | default   | NULL            |
| def           | LearningInnoDb3 | t          | p1             | NULL              |                          2 |                          NULL | RANGE            | NULL                | id                   | NULL                    | 20                    |          2 |           8192 |       16384 |            NULL |            0 |         0 | 2019-04-25 17:27:06 | 2019-04-25 17:30:15 | NULL       |     NULL |                   | default   | NULL            |
+---------------+-----------------+------------+----------------+-------------------+----------------------------+-------------------------------+------------------+---------------------+----------------------+-------------------------+-----------------------+------------+----------------+-------------+-----------------+--------------+-----------+---------------------+---------------------+------------+----------+-------------------+-----------+-----------------+
2 rows in set (0.00 sec)
```
我们发现分区p0只有一个记录,而分区p1有两个记录.当执行下面的操作时,就会报出异常
```
mysql> insert into t select 30;
ERROR 1526 (HY000): Table has no partition for value 30
```
如何不报出上面的错误呢?这个时候我们就可以扩大区间,执行下面的操作
```
mysql> alter table t
    -> add partition(
    -> partition p2 values less than maxValue);
```
添加一个区间是无穷大,这样的话就完成了

Range分区的话,可以存放不同的年份,我们创建一个这样的表
```
create table sales(
     money int unsigned not null,
     date datetime)engine=innodb
     partition by range (year(date))
     (partition p2008 values less than(2009),
     partition p2009 values less than(2010),
     partition p2010 values less than(2011));
```
然后向其中添加信息
```
mysql> insert into sales select 100,'2008-01-01';
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> insert into sales select 100,'2008-02-01';
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> insert into sales select 200,'2008-01-02';
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> insert into sales select 100,'2009-03-01';
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> insert into sales select 200,'2010-03-01';
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0
```
这个时候我们剖析一下下面的查询操作
```
mysql> explain partitions  select * from sales where date>='2008-01-01' and date<='2008-12-31';
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | sales | p2008      | ALL  | NULL          | NULL | NULL    | NULL |    4 |    25.00 | Using where |
+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 2 warnings (0.00 sec)

```
我们发现这次的操作是使用到了一个p2008,但是如果把date<='2008-12-31'变成 date<'2009-01-01',就会变成下面
```

mysql> explain partitions  select * from sales where date>='2008-01-01' and date<='2009-01-01';
+----+-------------+-------+-------------+------+---------------+------+---------+------+------+----------+-------------+
| id | select_type | table | partitions  | type | possible_keys | key  | key_len | ref  | rows | filtered | Extra       |
+----+-------------+-------+-------------+------+---------------+------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | sales | p2008,p2009 | ALL  | NULL          | NULL | NULL    | NULL |    4 |    25.00 | Using where |
+----+-------------+-------+-------------+------+---------------+------+---------+------+------+----------+-------------+
1 row in set, 2 warnings (0.01 sec)
```
这个时候查的分区就变成了两个
当我们对月进行分区的时候,发现并不可能,是因为range并不会优化Month这写函数,我们可以通过下面的方式,实现对月的分区




## 子分区
```
mysql> create table ts2(a int ,b date)
    -> partition by range(year(b))
    -> subpartition by hash(to_days(b))
    -> (
    -> partition p0 values less than (1990)(subpartition s0,subpartition s1),
    -> partition p1 values less than (2000)(subpartition s2,subpartition s3),
    -> partition p2 values less than maxvalue(subpartition s4,subpartition s5)
    -> );

```
这样创建是6个分区,range会创建三个大的分区,然后每个大分区就会创建2个小分区,所有小分区的名字不能相同,而却要创建子分区那么所有的分区都要创建子分区
同时指定不同的目录,让不同的分区存在不同的位置是不能行的
## 分区对null的处理
```
mysql> create table t_range( a int,b int)engine=Innodb partition by range(b)( partition p0 values less than(10), partition p1 values less than(20), partition p3 values less than maxValue);
Query OK, 0 rows affected (0.15 sec)

mysql> insert into t_range select 1,1;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> insert into t_range select 1,null;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

```
```
mysql> select * from t_range;
+------+------+
| a    | b    |
+------+------+
|    1 |    1 |
|    1 | NULL |
+------+------+
2 rows in set (0.00 sec)

```
```
mysql> select table_name,partition_name,table_rows 
    -> from information_schema.partitions
    -> where table_schema=database() and table_name='t_range';
+------------+----------------+------------+
| table_name | partition_name | table_rows |
+------------+----------------+------------+
| t_range    | p0             |          2 |
| t_range    | p1             |          0 |
| t_range    | p3             |          0 |
+------------+----------------+------------+
3 rows in set (0.00 sec)
```
我们发现对于null的处理就是讲null放在最左的区域
如果这个时候我们删除p0上的值,那么null也会跟着删除
然后我们来看一下list
```
mysql> create table t_List( a int, b int)engine=innodb partition by list(b)( partition p0 values in(1,3,5,7,9), partition p1 values in(0,2,4,6,8));
Query OK, 0 rows affected (0.12 sec)

```
向其中插入数据如果有null,必须要在partition中指定
对于Hash和key任何包含null的都会返回0
```
mysql> create table t_hash(
    -> a int,
    -> b int) engine=Innodb
    -> partition by hash(b)
    -> partitions 4;
Query OK, 0 rows affected (0.21 sec)

mysql> insert into t_hash select 1,0;
Query OK, 1 row affected (0.00 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> insert into t_hash select 1,null;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

```
然后我们查看
```
mysql> select table_name,partition_name,table_rows
    -> from information_schema.partitions
    -> where table_schema=database() and table_name='t_hash';
+------------+----------------+------------+
| table_name | partition_name | table_rows |
+------------+----------------+------------+
| t_hash     | p0             |          2 |
| t_hash     | p1             |          0 |
| t_hash     | p2             |          0 |
| t_hash     | p3             |          0 |
+------------+----------------+------------+
4 rows in set (0.00 sec)

```