# InnoDB学习之表(一)
表就是特定结构的数据集合,是关系型数据库的核心
## 索引组织表
在Innodb的存储引擎中,表都是根据主键顺序存放的,这种存放方式叫做索引组织表,在Innodb的索引组织表中,每个表都有一个主键,如果在创建的时候没有显示的定义主键,那么Innodb会按照这样的方式来定义主键
```
* 判断表中是否有非空的唯一的主键
* 如果不符合条件,Innodb会自动创建一个6字节大小的指针
```
我们来举个例子,我们创建一个表,如下图
```
create table z (
   a int not null,
   b int null,
   c int not null,
   d int not null,
   Unique key (b),Unique key (d),Unique key (c)
)
```
其中b d c都是唯一的,然后我们向其中插入数据,并查看
```
insert into z select 1,2,3,4;
insert into z select 5,6,7,8;
insert into z select 9,10,11,12;
select a,b,c,d,_rowid from z;
```
_rowid就是查看当前行的主键是哪一个,我们发现主键就是4,,8,12,所以d就是这个表的主键,因为它是第一个定义的**非空,并且唯一的列**
## 逻辑存储结构
### 表空间
所有的数据都是存在与表空间的,也就是数据库表的最高层结构
```
mysql> system sudo ls -lh /var/lib/mysql/ibdata1
-rw-r----- 1 mysql mysql 12M 4月  24 09:30 /var/lib/mysql/ibdata1

```
所有的表的信息都要存在这个表中,然后当我们开启一个选项时,
```
mysql> show variables like 'innodb_file_per_table';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_file_per_table | ON    |
+-----------------------+-------+
1 row in set (0.00 sec)

```
当开启这个开关时,不同的表都要独立出一部分,**我们注意就算独立出来,每张表存放的都是数据索引和缓冲**,回滚信息,索引和插入缓冲bitmap页等都是存在在共享空间中,我们来做一个实验
```
mysql> system sudo ls -lh /var/lib/mysql/ibdata1
-rw-r----- 1 mysql mysql 12M 4月  24 09:51 /var/lib/mysql/ibdata1
mysql> call load_t1(30000);
Query OK, 1 row affected (8.48 sec)

mysql> system sudo ls -lh /var/lib/mysql/ibdata1
-rw-r----- 1 mysql mysql 76M 4月  24 09:52 /var/lib/mysql/ibdata1
mysql> rollback;
Query OK, 0 rows affected (3.00 sec)

mysql> system sudo ls -lh /var/lib/mysql/ibdata1
-rw-r----- 1 mysql mysql 76M 4月  24 09:52 /var/lib/mysql/ibdata1
```
当回滚时,undo并不会回到上次的日志情况,这个时候就需要full purge这个线程的参与,进行undo页的删除   
### 区
我们来看一下区的操作
每个区的大小都是1mb,为了保持连续,区就是连续的几个页,一个区64个页,然后磁盘一次申请4---5个区,后来有了页的压缩,区的大小还是1mb,默认情况下,创建的表的默认大小是96KB,我们来算一下,一个区中有64个页,一个页的大小是16KB,这样这个区的大小就应该是1mb,表的大小应该就是1mb.我们注意到默认的最开始的表的大小是96kb,最开始分配数据的地方是一些碎片页,使用完这些碎片页才会申请64个连续的页.我们来创建一个表来验证一下
```
mysql> create table t1 
      ( col1 int not null auto_increment,
         col2 varchar(7000),
         primary key ( col1 )) engine=innodb;
       Query OK, 0 rows affected (0.06 sec)
```
然后
```
mysql> system sudo ls -lh /var/lib/mysql/LearingInnoDb/t1.ibd
[sudo] chengcongyue 的密码： 
-rw-r----- 1 mysql mysql 96K 4月  24 08:47 /var/lib/mysql/LearingInnoDb/t1.ibd
```
我们看到创建一个表,最开始默认的大小是96k
```
insert t1 select null,repeat('a',7000);
insert into t1 select null,repeat('a',7000);
```
插入两条数据,这个时候我们在查看一下表的空间,还是96k,因为这个时候用的碎片页,
```
chengcongyue@chengcongyue:~/下载/innodb$ sudo python py_innodb_page_info.py -v /var/lib/mysql/LearingInnoDb/t1.ibd
page offset 00000000, page type <File Space Header>
page offset 00000001, page type <Insert Buffer Bitmap>
page offset 00000002, page type <File Segment inode>
page offset 00000003, page type <B-tree Node>, page level <0001>
page offset 00000004, page type <B-tree Node>, page level <0000>
page offset 00000005, page type <B-tree Node>, page level <0000>
Total number of page: 6:
Insert Buffer Bitmap: 1
File Space Header: 1
B-tree Node: 3
File Segment inode: 1

```
我们通过一个脚本文件,来看一下表空间,这个时候有三个树节点,有两个叶子节点和一个非叶子节点.
然后我们来使用一个存储过程,通过这个过程我们插入60个上面的记录
```
create procedure load_t1(count int unsigned)
begin
declare s int unsigned default 1;
declare c varchar(7000) default repeat('a',7000);
while s<=count do
insert into t1 select null,c;
set s=s+1;
end while;
end;
```
然后在实行它,接着我们查看表空间的大小
```
mysql> system sudo ls -lh /var/lib/mysql/LearingInnoDb/t1.ibd
-rw-r----- 1 mysql mysql 592K 4月  24 09:15 /var/lib/mysql/LearingInnoDb/t1.ibd
mysql> call load_t1(1);

```
这个时候还是没有申请64个连续的页,这个时候还是小于1mb的,我们通过脚本工具来看一下节点的情况
```
chengcongyue@chengcongyue:~/下载/innodb$ sudo python py_innodb_page_info.py -v /var/lib/mysql/LearingInnoDb/t1.ibd
page offset 00000000, page type <File Space Header>
page offset 00000001, page type <Insert Buffer Bitmap>
page offset 00000002, page type <File Segment inode>
page offset 00000003, page type <B-tree Node>, page level <0001>
page offset 00000004, page type <B-tree Node>, page level <0000>
page offset 00000005, page type <B-tree Node>, page level <0000>
page offset 00000006, page type <B-tree Node>, page level <0000>
page offset 00000007, page type <B-tree Node>, page level <0000>
page offset 00000008, page type <B-tree Node>, page level <0000>
page offset 00000009, page type <B-tree Node>, page level <0000>
page offset 0000000a, page type <B-tree Node>, page level <0000>
page offset 0000000b, page type <B-tree Node>, page level <0000>
page offset 0000000c, page type <B-tree Node>, page level <0000>
page offset 0000000d, page type <B-tree Node>, page level <0000>
page offset 0000000e, page type <B-tree Node>, page level <0000>
page offset 0000000f, page type <B-tree Node>, page level <0000>
page offset 00000010, page type <B-tree Node>, page level <0000>
page offset 00000011, page type <B-tree Node>, page level <0000>
page offset 00000012, page type <B-tree Node>, page level <0000>
page offset 00000013, page type <B-tree Node>, page level <0000>
page offset 00000014, page type <B-tree Node>, page level <0000>
page offset 00000015, page type <B-tree Node>, page level <0000>
page offset 00000016, page type <B-tree Node>, page level <0000>
page offset 00000017, page type <B-tree Node>, page level <0000>
page offset 00000018, page type <B-tree Node>, page level <0000>
page offset 00000019, page type <B-tree Node>, page level <0000>
page offset 0000001a, page type <B-tree Node>, page level <0000>
page offset 0000001b, page type <B-tree Node>, page level <0000>
page offset 0000001c, page type <B-tree Node>, page level <0000>
page offset 0000001d, page type <B-tree Node>, page level <0000>
page offset 0000001e, page type <B-tree Node>, page level <0000>
page offset 0000001f, page type <B-tree Node>, page level <0000>
page offset 00000020, page type <B-tree Node>, page level <0000>
page offset 00000021, page type <B-tree Node>, page level <0000>
page offset 00000022, page type <B-tree Node>, page level <0000>
page offset 00000023, page type <B-tree Node>, page level <0000>
page offset 00000000, page type <Freshly Allocated Page>
Total number of page: 37:
Freshly Allocated Page: 1
Insert Buffer Bitmap: 1
File Space Header: 1
B-tree Node: 33
File Segment inode: 1
```
其中有33个树节点,1个是非叶子节点,32个叶子节点,这个时候对于数据段就有32个页了,之后在申请就是会使用64个连续的页了,
我们在插入一条记录,这个时候就申请了64个连续的页了
```
mysql> system sudo ls -lh /var/lib/mysql/LearingInnoDb/t1.ibd
-rw-r----- 1 mysql mysql 2.0M 4月  24 09:17 /var/lib/mysql/LearingInnoDb/t1.ibd
```

### 页
Innodb最小的磁盘管理单位,默认的页的大小是16K

## 行格式记录
### compact
我们创建一个表,表的定义如下
```
mysql> create table mytest(
    -> t1 varchar(10),
    -> t2 varchar(10),
    -> t3 char(10),
    -> t4 varchar(10))
    -> engine=innodb charset= latin1 row_format=compact;
Query OK, 0 rows affected (0.05 sec)

mysql> insert into mytest values('a','bb','bb','ccc');
Query OK, 1 row affected (0.01 sec)

mysql> insert into mytest values('d','ee','ee','fff');
Query OK, 1 row affected (0.01 sec)

mysql> insert into mytest values('d',null,null,'fff');
Query OK, 1 row affected (0.01 sec)

```
其中的row_format定义为compact,然后我们要把mytest.ibd的文件打开,来查看它的二进制格式,具体操作
```
vim -b mytest.iba
然后在末行命令模式下使用, :%!xxd
```
我们把需要分析的复制到笔记中:
```
0000c070: 7375 7072 656d 756d 0302 0100 0000 1000  supremum........
0000c080: 2c00 0000 0002 0000 0000 0021 72e5 0000  ,..........!r...
0000c090: 0172 0110 6162 6262 6220 2020 2020 2020  .r..abbbb
0000c0a0: 2063 6363 0302 0100 0000 1800 2b00 0000   ccc........+...
0000c0b0: 0002 0100 0000 0021 73e6 0000 0173 0110  .......!s....s..
0000c0c0: 6465 6565 6520 2020 2020 2020 2066 6666  deeee        fff
0000c0d0: 0301 0600 0020 ff98 0000 0000 0202 0000  ..... ..........
0000c0e0: 0000 2178 e900 0002 0301 1064 6666 6600  ..!x.......dfff.
```
看一下表中的数据
```
mysql> select * from mytest;
+------+------+------+------+
| t1   | t2   | t3   | t4   |
+------+------+------+------+
| a    | bb   | bb   | ccc  |
| d    | ee   | ee   | fff  |
| d    | NULL | NULL | fff  |
+------+------+------+------+
3 rows in set (0.00 sec)
```
第一行的数据
```
0302 0100 0000 1000  supremum........
2c00 0000 0002 0000 0000 0021 72e5 0000  ,..........!r...
0172 0110 6162 6262 6220 2020 2020 2020  .r..abbbb
2063 6363
```
我们发现其中的实际数据是按照acsii存放的,然后对于定长的char类型,如果不满10个字节,其他的按照0x20的方式存放.然后其中的00 1000 2c,其中的2c指向的是下一行的数据,第一行分析完成.
从第三行我们可以知道,compact格式下null是不存储的,不占用任何的存储空间.
### Redundant
### 行溢出
```
mysql> create table test(
    -> a varchar(65535))charset=latin1 engine=innodb;
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs

```
在mysql中varchar是可以存放65535的数据的,定义是这样定义的,可是实际上并不支持.实际上应该是65532,我们来试一下
```
mysql> create table test( a varchar(65532))charset=latin1 engine=innodb;
Query OK, 0 rows affected (0.06 sec)
```
这样就成功了
上面创建的test表中的字段a是varchar长度为65532的字段,那么长度65532指的是什么呢.我们换成其他的字符集,来看一下
```
mysql> create table test2(a varchar(65532))charset=GBK engine=Innodb;
ERROR 1074 (42000): Column length too big for column 'a' (max = 32767); use BLOB or TEXT instead
```
我们使用的字符集GBK,这个时候a的最大长度就是32767.
然后我们在使用utf-8
```
mysql> create table test3(a varchar(65532))charset=utf8 engine=innodb;
ERROR 1074 (42000): Column length too big for column 'a' (max = 21845); use BLOB or TEXT instead
```
根据上面的可以知道varchar(N)指的是字符的数量,字节的数量肯定是65532,这个时候如果使用latin1,一个字符对于着一个字节,所以最大值为65532,而当我们使用GBK的时候,一个字符对应这两个字节,utf8中一个字符对应这3个字节
**所以65535的大小指的是字节,varchar(N)其中的N是字符**

```
mysql> create table test4(a varchar(22000),b varchar(22000),c varchar(22000))
    -> charset=latin1 engine=innodb;
ERROR 1118 (42000): Row size too large. The maximum row size for the used table type, not counting BLOBs, is 65535. This includes storage overhead, check the manual. You have to change some columns to TEXT or BLOBs

```
其实varchar(65532)指的是全部的列的总和为65532,如果有多个varchar列,它们字符和的长度超过了65532也会报错.
我们知道一个页的大小最大为16kb,那么是如何存储65532字节大小的字节的呢?一般情况下varchar都是存放在页类型的B-tree Node中,但是发生行溢出之后,我们就把数据放在Uncompress Blok页中,
我们来做一个实验,我们创建一个表
```
mysql> create table t(a varchar(65532))engine=Innodb charset=latin1;
Query OK, 0 rows affected (0.06 sec)
```
然后添加一行
```
mysql> insert into t select repeat('a',65532);
Query OK, 1 row affected (0.03 sec)
Records: 1  Duplicates: 0  Warnings: 0
```
然后我们通过工具来查看表空间的信息
```
chengcongyue@chengcongyue:~/下载/innodb$ sudo ./py_innodb_page_info.py -v /var/lib/mysql/LearingInnoDb/t.ibd
page offset 00000000, page type <File Space Header>
page offset 00000001, page type <Insert Buffer Bitmap>
page offset 00000002, page type <File Segment inode>
page offset 00000003, page type <B-tree Node>, page level <0000>
page offset 00000004, page type <Uncompressed BLOB Page>
page offset 00000005, page type <Uncompressed BLOB Page>
page offset 00000006, page type <Uncompressed BLOB Page>
page offset 00000007, page type <Uncompressed BLOB Page>
page offset 00000008, page type <Uncompressed BLOB Page>
Total number of page: 9:
Insert Buffer Bitmap: 1
Uncompressed BLOB Page: 5
File Space Header: 1
B-tree Node: 1
File Segment inode: 1
```
其中只有一个B-tree Node 节点,然后有5个Uncompressed BLOB Page,然后我们进入到这个表空间中查看它的二进制文件
其中就是有一个偏移量还有一个指向,指向的就是Uncompressed BLOB Page
我们现在就来确定什么时候需要使用到Uncompressed BLOB Page
我们来创建一个表
```
mysql> create table t2( a varchar(9000))Engine= Innodb;
Query OK, 0 rows affected (0.06 sec)
mysql> insert into t2 select repeat('a',9000);
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> insert into t2 select repeat('a',9000);
Query OK, 1 row affected (0.00 sec)
Records: 1  Duplicates: 0  Warnings: 0

```
创建一个表,向其中插入两个行,然后我们通过工具类查看
```
root@chengcongyue:/home/chengcongyue/下载/innodb# sudo ./py_innodb_page_info.py -v /var/lib/mysql/LearingInnoDb/t2.ibd 
page offset 00000000, page type <File Space Header>
page offset 00000001, page type <Insert Buffer Bitmap>
page offset 00000002, page type <File Segment inode>
page offset 00000003, page type <B-tree Node>, page level <0000>
page offset 00000004, page type <Uncompressed BLOB Page>
page offset 00000005, page type <Uncompressed BLOB Page>
Total number of page: 6:
Insert Buffer Bitmap: 1
Uncompressed BLOB Page: 2
File Space Header: 1
B-tree Node: 1
File Segment inode: 1
```
其中就使用到了Uncompressed BLOB Page,说明这个时候就行溢出了
我们找到了一个阈值8098,当varchar的大小等于8098时,就可以在一个页中放两条数据,我们来测试一下
```
mysql> create table t3(a varchar(8098))Engine=innodb;
Query OK, 0 rows affected (0.06 sec)

mysql> insert into t3 select repeat('a',8098);
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

mysql> insert into t3 select repeat('a',8098);
Query OK, 1 row affected (0.01 sec)
Records: 1  Duplicates: 0  Warnings: 0

```
然后我们通过工具来查看t3的表空间
```
root@chengcongyue:/home/chengcongyue/下载/innodb# sudo ./py_innodb_page_info.py -v /var/lib/mysql/LearingInnoDb/t3.ibd 
page offset 00000000, page type <File Space Header>
page offset 00000001, page type <Insert Buffer Bitmap>
page offset 00000002, page type <File Segment inode>
page offset 00000003, page type <B-tree Node>, page level <0000>
page offset 00000000, page type <Freshly Allocated Page>
page offset 00000000, page type <Freshly Allocated Page>
Total number of page: 6:
Freshly Allocated Page: 2
Insert Buffer Bitmap: 1
File Space Header: 1
B-tree Node: 1
File Segment inode: 1
```
这个时候,就不会使用其他的页.
对于text block类型,也不一定是使用<Uncompressed BLOB Page>的页
```
root@chengcongyue:/home/chengcongyue/下载/innodb# sudo ./py_innodb_page_info.py -v /var/lib/mysql/LearingInnoDb/t4.ibd 
page offset 00000000, page type <File Space Header>
page offset 00000001, page type <Insert Buffer Bitmap>
page offset 00000002, page type <File Segment inode>
page offset 00000003, page type <B-tree Node>, page level <0001>
page offset 00000004, page type <B-tree Node>, page level <0000>
page offset 00000005, page type <B-tree Node>, page level <0000>
page offset 00000006, page type <B-tree Node>, page level <0000>
page offset 00000000, page type <Freshly Allocated Page>
Total number of page: 8:
Freshly Allocated Page: 1
Insert Buffer Bitmap: 1
File Space Header: 1
B-tree Node: 4
File Segment inode: 1
```
这里就没有使用到<Uncompressed BLOB Page>,因为此时的blob比较小