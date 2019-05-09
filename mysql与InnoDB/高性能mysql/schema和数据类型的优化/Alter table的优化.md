# Alter table的优化
Alter table是修改表的结构,其过程是创建一个新的表,然后将旧表的数据移动到新表上,然后销毁旧表.
但并不是更改表的结构就会引起表的重建,对于表中默认值的修改.不同的方法,表也不一定会重建.
```
mysql> alter table sakila.film modify column rental_duration tinyint(3) not null default 5;
```
比如我们要修改电影表的租赁期,默认值为5,这样使用modify就会导致表的重建,但是对于默认值的存放实在.frm表中的,所以如果使用如下的操作的话,修改的只是文件.
```
mysql> alter table sakila.film alter column rental_duraation set default 5;
```
这样,修改的只是.frm文件,而不是表空间.

如下情况也可能只修改.frm文件
* 如果移除列的自增属性
* 操作枚举类set类型的列
```
mysql> create table film( id int not null primary key, rating enum('G','PG','PG-13','R','NC-17','PG-14'))engine = innodb;
Query OK, 0 rows affected (0.06 sec)

mysql> create table film_new like film;
Query OK, 0 rows affected (0.06 sec)

mysql> alter table film_new 
    -> modify column rating enum('G','PG','PG-13','R','NC-17','PG-14') default'G';
Query OK, 0 rows affected (0.01 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> flush tables with read lock;
Query OK, 0 rows affected (0.00 sec)
```
然后交换.frm文件
```
root@chengcongyue:/var/lib/mysql/learningMysql# mv film.frm film_tmp.frm
root@chengcongyue:/var/lib/mysql/learningMysql# mv film_new.frm film.frm
root@chengcongyue:/var/lib/mysql/learningMysql# mv film_tmp.frm film_new.frm

```
然后在执行,这样就实现了只修改.frm文件实现了alter table
```
mysql> unlock tables ;
Query OK, 0 rows affected (0.00 sec)

mysql> show columns from film like 'rating';
+--------+--------------------------------------------+------+-----+---------+-------+
| Field  | Type                                       | Null | Key | Default | Extra |
+--------+--------------------------------------------+------+-----+---------+-------+
| rating | enum('G','PG','PG-13','R','NC-17','PG-14') | YES  |     | G       |       |
+--------+--------------------------------------------+------+-----+---------+-------+
1 row in set (0.00 sec)

```