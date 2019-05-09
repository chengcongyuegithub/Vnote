# InnoDb学习之文件
### 参数文件
Mysql在启动时,需要读取参数文件,但是和Oracle不同的是,Mysql在没有参数文件时,也可以启动,但是Oracle就不能启动了

Mysql的参数文件是以纯文本的形式存在的,所以可以通过文件编辑器进行修改.
####参数
我们先来看几个简单的命令
```
mysql> set global show_compatibility_56=on;
Query OK, 0 rows affected (0.00 sec)

mysql> select * from GLOBAL_VARIABLES where variable_name like 'innodb_buffer';
Empty set, 1 warning (0.00 sec)

mysql> select * from GLOBAL_VARIABLES where variable_name like 'innodb_buffer%'; 
+-------------------------------------+----------------+
| VARIABLE_NAME                       | VARIABLE_VALUE |
+-------------------------------------+----------------+
| INNODB_BUFFER_POOL_FILENAME         | ib_buffer_pool |
| INNODB_BUFFER_POOL_LOAD_ABORT       | OFF            |
| INNODB_BUFFER_POOL_DUMP_NOW         | OFF            |
| INNODB_BUFFER_POOL_DUMP_PCT         | 25             |
| INNODB_BUFFER_POOL_CHUNK_SIZE       | 134217728      |
| INNODB_BUFFER_POOL_DUMP_AT_SHUTDOWN | ON             |
| INNODB_BUFFER_POOL_INSTANCES        | 1              |
| INNODB_BUFFER_POOL_LOAD_AT_STARTUP  | ON             |
| INNODB_BUFFER_POOL_SIZE             | 134217728      |
| INNODB_BUFFER_POOL_LOAD_NOW         | OFF            |
+-------------------------------------+----------------+
10 rows in set, 1 warning (0.00 sec)
```
#### 参数类型
* 动态参数
* 静态参数


```
mysql> set read_buffer_size=524288;
Query OK, 0 rows affected (0.00 sec)

mysql> select @@session.read_buffer_size;
+----------------------------+
| @@session.read_buffer_size |
+----------------------------+
|                     524288 |
+----------------------------+
1 row in set (0.00 sec)

mysql> select @@global.read_buffer_size;
+---------------------------+
| @@global.read_buffer_size |
+---------------------------+
|                    131072 |
+---------------------------+
1 row in set (0.00 sec)

mysql> set @@global.read_buffer_size=1048576;
Query OK, 0 rows affected (0.00 sec)

mysql> select @@session.read_buffer_size;
+----------------------------+
| @@session.read_buffer_size |
+----------------------------+
|                     524288 |
+----------------------------+
1 row in set (0.00 sec)

mysql> select @@global.read_buffer_size;
+---------------------------+
| @@global.read_buffer_size |
+---------------------------+
|                   1048576 |
+---------------------------+
1 row in set (0.00 sec)
```

这里注意到有global和session两个参数,表示的有效范围
### 日志文件
#### 错误日志
错误日志对mysql的启动运行关闭进行了记录.
```
mysql> show variables like 'log_error';
+---------------+--------------------------+
| Variable_name | Value                    |
+---------------+--------------------------+
| log_error     | /var/log/mysql/error.log |
+---------------+--------------------------+
1 row in set (0.00 sec)

```
#### 慢查询日志
![](_v_images/20190423150922505_214903208.png =630x)

默认情况下,并不启动这个文件
这个log是dba用来优化SQL语句的.我们来看一个参数
```
mysql> show variables like 'log_queries_not_using_indexes';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| log_queries_not_using_indexes | OFF   |
+-------------------------------+-------+
1 row in set (0.00 sec)

```
这个参数开启时,没有用到索引的sql语句就会被记录.
我们可以把慢记录的日志记录放入到一个表中,是的用户的查询更加的方便.

超过10秒执行的sql语句也会被记录到慢查询日志中
#### 查询日志
这个日志记录了所有mysql数据区请求的信息
#### 二进制日志
记录所有对数据库执行更改的操作.这些操作没有导致数据库发生变化,也会被记录
二进制日志的作用: 恢复,复制.审计
max_binlog_size指定了单个的日志文件的最大值,如果超过了该值,就会产生新的二进制日志文件,

事务和二进制日志的关系,当事务还没提交,这个时候,我们就把执行的操作写入到了一个缓存中去,等待事务提交之后,直接讲缓存中的二进制信息写入到二进制文件中,
bin_cache_size,这是基于会话的,所以当你开启一个线程,mysql就会自动给你分配一个大小为bin_cache_size的缓存,当事务的记录过大,这个时候就会讲多的信息写入到一个临时文件中区
```
mysql> show variables like 'binlog_cache_size';
+-------------------+-------+
| Variable_name     | Value |
+-------------------+-------+
| binlog_cache_size | 32768 |
+-------------------+-------+
1 row in set (0.00 sec)

mysql> show global status like 'binlog_cache%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Binlog_cache_disk_use | 0     |
| Binlog_cache_use      | 0     |
+-----------------------+-------+
2 rows in set (0.00 sec)
```
二进制日志不是每次写的时候都会同步到磁盘中区, sync_binlog=[N],当为1时,就是同步写.
当设置为1时,也会有问题,当早早的将事务写到了二进制日志中并写到磁盘上去之后,这个时候发生了异常,事务回滚了,这个时候事务的记录已经存在了磁盘上了,这个时候就需要另外一个参数Innodb_support_xa的帮助了,


然后再来介绍一下binlog_format
它分为三种形式 Statement  , ROW,MIXED
不同的binlog_format会影响不同的事务隔离级别
同时row的格式会比Statement的格式大好多.
### 表结构定义文件
对于一个表,都有对应的frm文件
```
root@chengcongyue:/var/lib/mysql# ls
auto.cnf         ib_buffer_pool  ib_logfile1  performance_schema
debian-5.7.flag  ibdata1         ibtmp1       sys
easyzhihu        ib_logfile0     mysql
root@chengcongyue:/var/lib/mysql# cd easyzhihu/
root@chengcongyue:/var/lib/mysql/easyzhihu# ls
comment.frm  login_ticket.frm  message.ibd   user.frm
comment.ibd  login_ticket.ibd  question.frm  user.ibd
db.opt       message.frm       question.ibd

```

### 存储引擎文件
#### 表空间文件
在默认配置下会有一个
ibdata1
用户可以通过多个文件组成一个表空间,同时制定文件的属性
![](_v_images/20190423161602294_1176946983.png =781x)


独立的表空间.ibd,存放的是表的数据索引和插入缓冲,其余的信息都是放在默认的表空间中