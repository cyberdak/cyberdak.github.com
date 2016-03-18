---
layout : post
title : mysql count 的性能讨论
---

刚好看到v2ex一个关于mysql count性能的讨论。

帖子内容如下：

```
mysql> select count(*) from threads; 

InnoDB ，大约 150 万数据，有主键，六个字段，用时 7s 。我觉得应该能在一秒内完成查询。这个查询会不会临时锁表？因为执行的同时还有另外三个连接在做其它的查询。还是因为阿里云 ECS 的磁盘 IO 太慢？
```

这还是让我很震惊的，因为mysql到2016年进化到5.7以来，在我的印象中，起码千万级的数据还是可以单机从容处理的。

而楼下的群众的回复更是让我目瞪口呆，有的告诉楼主不要使用`count(*)`.但是这我在《高性能MySQL》中说的完全冲突。

书中原文如下：

```
我们发现一个常见的错误就是，在括号内指定了一个列却希望统计结果集的行数，如果希望知道的是结果集的行数，最好使用count(*)，这样写意义清晰，性能也会很好。
```

注意最后几个字，并没有提到使用count(*)会降低性能。所以在innodb表中，使用count(*)肯定是没有问题的。

公说公有理，婆说婆有理。那么就自己动手测试一下就好了。

这次还准备一共比较一下innodb和myisam的count。

inndo表
```
Create Table: CREATE TABLE `model_150m` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(200) NOT NULL,
  `title` varchar(200) NOT NULL,
  `content` varchar(200) NOT NULL,
  `create_date` datetime NOT NULL,
  `xx` varchar(200) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1919616 DEFAULT CHARSET=utf8
```

myisam表
```
Create Table: CREATE TABLE `model_150m_my` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(200) NOT NULL,
  `create_date` datetime NOT NULL,
  `xx` varchar(200) NOT NULL,
  `title` varchar(200) NOT NULL,
  `content` varchar(200) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=MyISAM AUTO_INCREMENT=561619 DEFAULT CHARSET=utf8
```

然后用java插入数据，这里偷懒用了之前一个项目的代码，写了一下model，然后用代码生成器生成了crud的代码，本来打算用junit直接插入了，结果项目还是没有junit，只好写了一个controller来保存数据。

model:
```
@Table(name="model_150m")
@Entity
public class Model150m {
		private int id;
		private String name;
		private Date create_date;
		private String content;
		private String title;
		private String xx;
        
        /*-- getter & setter -- */
		
		public Model150m() {
			this.name = UUID.randomUUID().toString();
			this.create_date = new Date();
			this.content = UUID.randomUUID().toString();;
			this.title = UUID.randomUUID().toString();;
			this.xx = UUID.randomUUID().toString();;
		}
		
```

往两个表个插入150w的数据。因为这里我让程序跑着就去睡觉了，一觉睡醒，innodb有191w条，myisam有200w条。

首先是inndo的查询结果。

count(*):
```
mysql> select count(*) from model_150m;
+----------+
| count(*) |
+----------+
|  1919615 |
+----------+
1 row in set (8.60 sec)
```
count(1):
```
mysql> select count(1) from model_150m;
+----------+
| count(1) |
+----------+
|  1919615 |
+----------+
1 row in set (8.81 sec)
```

count(id):
```
mysql> select count(id) from model_150m;
+-----------+
| count(id) |
+-----------+
|   1919615 |
+-----------+
1 row in set (8.62 sec)
```

并没有什么本质区别。。

然后是myisam的count:
```
mysql> select count(*) from model_150m_my;
+----------+
| count(*) |
+----------+
|  2061618 |
+----------+
1 row in set (0.00 sec)
```

实际上，对于myisam表，没有where条件的count查询，可以直接从引擎的特性中得到行数。

在mysql的information_schema.tables表中，储存着mysql所有表的数据。

inndo表的保存信息
```
mysql> select * from information_schema.tables where table_schema  = 'nxspiii' a
nd table_name = 'model_150m'\G
*************************** 1. row ***************************
  TABLE_CATALOG: def
   TABLE_SCHEMA: nxspiii
     TABLE_NAME: model_150m
     TABLE_TYPE: BASE TABLE
         ENGINE: InnoDB
        VERSION: 10
     ROW_FORMAT: Compact
     TABLE_ROWS: 2035929
 AVG_ROW_LENGTH: 220
    DATA_LENGTH: 449691648
MAX_DATA_LENGTH: 0
   INDEX_LENGTH: 0
      DATA_FREE: 1068498944
 AUTO_INCREMENT: 1919616
    CREATE_TIME: 2016-03-17 22:09:09
    UPDATE_TIME: NULL
     CHECK_TIME: NULL
TABLE_COLLATION: utf8_general_ci
       CHECKSUM: NULL
 CREATE_OPTIONS:
  TABLE_COMMENT:
1 row in set (0.10 sec)
```

可以看到的，inndo引擎的表的table_rows照样有数据，但是错的太离谱了。

myisam表的信息
```
mysql> select * from information_schema.tables where table_schema  = 'nxspiii' a
nd table_name = 'model_150m_my'\G
*************************** 1. row ***************************
  TABLE_CATALOG: def
   TABLE_SCHEMA: nxspiii
     TABLE_NAME: model_150m_my
     TABLE_TYPE: BASE TABLE
         ENGINE: MyISAM
        VERSION: 10
     ROW_FORMAT: Dynamic
     TABLE_ROWS: 2061618
 AVG_ROW_LENGTH: 96
    DATA_LENGTH: 197915328
MAX_DATA_LENGTH: 281474976710655
   INDEX_LENGTH: 21190656
      DATA_FREE: 0
 AUTO_INCREMENT: 2061619
    CREATE_TIME: 2016-03-18 09:27:14
    UPDATE_TIME: 2016-03-18 10:35:07
     CHECK_TIME: NULL
TABLE_COLLATION: utf8_general_ci
       CHECKSUM: NULL
 CREATE_OPTIONS:
  TABLE_COMMENT:
1 row in set (0.02 sec)
```
而myisam表的myisam的table_rows就完整准确。

innodb为何这么慢，是因为它支持MVCC，不同的事务能查询到的结果是不一样的，每一次查询都要开一个事务，然后查看这个事务能看到的数据，就导致每一次都是扫描全表。只能去把索引的数量计算一遍，导致了无法避免的消耗。