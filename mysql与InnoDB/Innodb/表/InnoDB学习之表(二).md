# InnoDB学习之表(二)
## char的行结构存储
![](_v_images/20190424235310930_490556207.png)

内部还是被视为变长字符类型,对于没有填充满的会填充字段，在实际存储的方式下，是没有区别的(varchar和char)
## InnoDb的数据页结构
我们以及知道页是Innodb管理数据库的最小单位,页类型中B_tree_node就是数据库中的实际数据了.



