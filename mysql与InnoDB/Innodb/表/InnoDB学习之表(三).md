# InnoDB学习之表(三)
## 数据完整性
关系型数据库和文件系统不同的是,关系型数据库本身可以存储数据的完整性,不需要应用程序的控制,而文件系统需要应用程序的控制.关系型数据库都提供了约束机制.
* 实体完整性 也就是保证一个表中有主键.其中通过定义primary key 和 unique key约束来保证实体的完整性
* 域外完整性 合适的数据类型 外键 编写触发器 default*

### 约束的创建和查找
```
mysql> create table u ( id int , name varchar(20),id_card char(18),primary key(id),unique key(name));
Query OK, 0 rows affected (0.07 sec)
```
```
mysql> select constraint_name,constraint_type from information_schema.table_constraints where table_schema='LearingInnoDb' and table_name= 'u';
+-----------------+-----------------+
| constraint_name | constraint_type |
+-----------------+-----------------+
| PRIMARY         | PRIMARY KEY     |
| name            | UNIQUE          |
+-----------------+-----------------+
2 rows in set (0.00 sec)

```
对于主键约束来说默认的约束名为primary,对于unique来说,unique的name和列名保持一致,当然也可以自己指定
```
mysql> alter table u add unique key uk_id_card(id_card);
Query OK, 0 rows affected (0.04 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> select constraint_name,constraint_type from information_schema.table_constraints where table_schema='LearingInnoDb' and table_name= 'u';
+-----------------+-----------------+
| constraint_name | constraint_type |
+-----------------+-----------------+
| PRIMARY         | PRIMARY KEY     |
| name            | UNIQUE          |
| uk_id_card      | UNIQUE          |
+-----------------+-----------------+
3 rows in set (0.00 sec)

```
然后我们看一看外键的约束
```
mysql> create table p( id int ,u_id int,primary key(id),foreign key(u_id) references p (id));
Query OK, 0 rows affected (0.07 sec)


mysql> select constraint_name,constraint_type from information_schema.table_constraints where table_schema='LearingInnoDb' and table_name= 'u';
+-----------------+-----------------+
| constraint_name | constraint_type |
+-----------------+-----------------+
| PRIMARY         | PRIMARY KEY     |
| name            | UNIQUE          |
| uk_id_card      | UNIQUE          |
+-----------------+-----------------+
3 rows in set (0.00 sec)
```
### 约束和索引的区别
唯一的索引就创建了唯一的约束.约束是一个逻辑上的概念,是为了保证数据的完整性,而索引是一个数据结构,既有逻辑上的概念,也代表这物理存储的方式.
### 对enum的约束
```
mysql> create table a2(id int,sex enum('male','female'));
Query OK, 0 rows affected (0.06 sec)

mysql> insert into a2 select 1, 'female';
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> insert into a2 select 2, 'bi';
ERROR 1265 (01000): Data truncated for column 'sex' at row 1

```
### 对触发器的约束
```
mysql> insert into usercash select 1,1000;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> update usercash set cash=cash-(-20) where userid=1;
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> create table usercash_err_log( userid int not null,old_cash int unsigned not null,new_cash int unsigned not null,user varchar(30),time datetime);
Query OK, 0 rows affected (0.06 sec)

mysql> delimiter $$
mysql> create trigger tgr_usercash_update before update on usercash
    -> for each row
    -> begin 
    -> if new.cash-old.cash > 0 then
    -> insert into usercash_err_log
    -> select old.userid,old.cash,new.cash,USER(),NOW();
    -> set new.cash=old.cash;
    -> end if;
    -> end;
    -> $$
Query OK, 0 rows affected (0.01 sec)

mysql> delete from usercash;
    -> $$
Query OK, 1 row affected (0.02 sec)

mysql> delimiter ;
mysql> insert into usercash select 1,1000;
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> update usercash set cash=cash-(-20) where userid=1;
Query OK, 0 rows affected (0.01 sec)
Rows matched: 1  Changed: 0  Warnings: 0

mysql> select * from usercash;
+--------+------+
| userid | cash |
+--------+------+
|      1 | 1000 |
+--------+------+
1 row in set (0.00 sec)

mysql> select * from usercash_err_log;
+--------+----------+----------+----------------+---------------------+
| userid | old_cash | new_cash | user           | time                |
+--------+----------+----------+----------------+---------------------+
|      1 |     1000 |     1020 | root@localhost | 2019-04-25 09:31:30 |
+--------+----------+----------+----------------+---------------------+
1 row in set (0.00 sec)

```
### 视图
视图是一个虚表,它由SQL查询来定义,可以当做表来使用,但是视图并没有实际的物理存储
#### 视图的作用
视图的作用就是被当做一个抽象装置,特别是对一些应用程序,程序本身并不关心基表的结构,只通过视图进行取数据和更新数据,视图在一定程度上起到了安全层的作用
虽然视图是一个虚拟的表,但是我们依旧可以对视图进行更新操作,其本质就是通过视图定义来更新基本表,一般称进行更新操作的视图称为可更新视图,
```
mysql> create table t5(id int);
Query OK, 0 rows affected (0.06 sec)

mysql> create view v_t
    -> as 
    -> select * from t5 where id<10;
Query OK, 0 rows affected (0.01 sec)
```

```
mysql> insert into v_t select 20;
Query OK, 1 row affected (0.00 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> select * from v_t;
Empty set (0.00 sec)
```
上面插入的数据失败,当更新数据时,我们可以采用with check option,这样当插入失败时,就会保持错误
```
mysql> alter view v_t as
    -> select * from t5 where id <10
    -> with check option;
Query OK, 0 rows affected (0.01 sec)

mysql> insert into v_t select 20;
ERROR 1369 (HY000): CHECK OPTION failed 'LearingInnoDb.v_t'
```
虽然视图是虚表,但是在show tables的时候仍然会显示出来,如果我们指向查看基本表
```
mysql> select table_name from information_schema.tables where table_type = 'base table' and table_schema=database();
+------------------+
| table_name       |
+------------------+
| a                |
| a2               |
| child            |
| j                |
| mytest           |
| p                |
| parent           |
| t                |
| t1               |
| t2               |
| t3               |
| t4               |
| t5               |
| test             |
| u                |
| usercash         |
| usercash_err_log |
| z                |
+------------------+
18 rows in set (0.00 sec)
```
如果我们想查看视图的详细信息
```
mysql> select * from information_schema.views where table_schema=database();
+---------------+---------------+------------+-----------------------------------------------------------------------------------------------------------+--------------+--------------+----------------+---------------+----------------------+----------------------+
| TABLE_CATALOG | TABLE_SCHEMA  | TABLE_NAME | VIEW_DEFINITION                                                                                           | CHECK_OPTION | IS_UPDATABLE | DEFINER        | SECURITY_TYPE | CHARACTER_SET_CLIENT | COLLATION_CONNECTION |
+---------------+---------------+------------+-----------------------------------------------------------------------------------------------------------+--------------+--------------+----------------+---------------+----------------------+----------------------+
| def           | LearingInnoDb | v_t        | select `LearingInnoDb`.`t5`.`id` AS `id` from `LearingInnoDb`.`t5` where (`LearingInnoDb`.`t5`.`id` < 10) | CASCADED     | YES          | root@localhost | DEFINER       | utf8                 | utf8_general_ci      |
+---------------+---------------+------------+-----------------------------------------------------------------------------------------------------------+--------------+--------------+----------------+---------------+----------------------+----------------------+

```
#### 物化视图
Oracle数据库支持物化视图,这样的视图就不是基于基表的虚表,而是根据基表实际存在的实表,物化视图可以用于计算并保存多表的连接和聚集等耗时较多的SQL操作.这种视图也被叫做索引视图.
Mysql本身并不支持物化视图.但是可以通过一些机制实现
```
mysql> mysql> create orders(
    -> order_id int unsigned not null auto_increment,
    -> product_name varchar(30) not null,
    -> price decimal(8,2) not null,
    -> amount smallint not null,
    -> primary key(order_id))engine=innodb;
Query OK, 0 rows affected (0.06 sec)
mysql> insert into orders values(null,'cpu',135.5,1); ERROR 1146 (42S02): Table 'LearingInnoDb.orders' doesn't exist
mysql> insert into Orders values(null,'cpu',135.5,1);
Query OK, 1 row affected (0.01 sec)

mysql> insert into Orders values(null,'memory',48.2,3);
Query OK, 1 row affected (0.01 sec)

mysql> insert into Orders values(null,'cpu',125,6,3);
ERROR 1136 (21S01): Column count doesn't match value count at row 1
mysql> insert into Orders values(null,'cpu',125,3);
Query OK, 1 row affected (0.01 sec)

mysql> insert into Orders values(null,'cpu',105.3,4);
Query OK, 1 row affected (0.00 sec)

mysql> select * from orders;
ERROR 1146 (42S02): Table 'LearingInnoDb.orders' doesn't exist
mysql> select * from Orders;
+----------+--------------+--------+--------+
| order_id | product_name | price  | amount |
+----------+--------------+--------+--------+
|        1 | cpu          | 135.50 |      1 |
|        2 | memory       |  48.20 |      3 |
|        3 | cpu          | 125.00 |      3 |
|        4 | cpu          | 105.30 |      4 |
+----------+--------------+--------+--------+
4 rows in set (0.01 sec)
```
我们先创建一张表,然后向其中插入一些数据
```
mysql> create table Order_MV(
    -> product_name varchar(30) not null,
    -> price_sum decimal(8,2) not null,
    -> amount_sum int not null,
    -> price_avg float not null,
    -> orders_cnt int not null,
    -> Unique index(product_name));
Query OK, 0 rows affected (0.06 sec)
mysql> insert into Order_MV select product_name,sum(price),sum(ount),avg(price),count(*) from Orders group by product_name;
Query OK, 2 rows affected (0.01 sec)
Records: 2  Duplicates: 0  Warnings: 0

mysql> select * from Order_MV;
+--------------+-----------+------------+-----------+------------+
| product_name | price_sum | amount_sum | price_avg | orders_cnt |
+--------------+-----------+------------+-----------+------------+
| cpu          |    365.80 |          8 |   121.933 |          3 |
| memory       |     48.20 |          3 |      48.2 |          1 |
+--------------+-----------+------------+-----------+------------+
2 rows in set (0.00 sec
```
创建的这一张表就相当与是逻辑上的"物化视图",如果我们要实现一个功能,每插入一条数据,就更新一下这个记录表,这样的话就要用到触发器,
```
create trigger trg_Orders_insert
after insert On Orders
for each row 
begin
  set @old_price_sum=0;
  set @old_amount_sum=0;
  set @old_price_avg=0;
  set @old_orders_cnt=0;
  
  select IFNULL(price_sum,0),IFNULL(amount_sum,0),IFNULL(price_avg,0),IFNULL(orders_cnt,0)
  from Order_MV
  where product_name=NEW.product_name
  into  @old_price_sum,@old_amount_sum,@old_price_avg,@old_orders_cnt;
  
  set @new_price_sum=@old_price_sum+NEW.price;
  set @new_amount_sum=@old_amount_sum+NEW.amount;
  set @new_orders_cnt=@old_orders_cnt+1;
  set @new_price_avg=@new_price_sum/@new_orders_cnt;
  
  replace into Order_MV
  values(New.product_name,@new_price_sum,@new_amount_sum,@new_price_avg,@new_orders_cnt);
end;
```
当有了上面的触发器之后,我们在执行插入操作,执行完之后就能够同步的更新这个物化视图