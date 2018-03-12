##1.MyISAM
###1.引擎特性
不支持事务，不支持外键，表级锁，批量插入速度高，支持全文索引，支持B-Tree索引，不支持哈希索引和集群索引，数据可压缩，磁盘空间和内存占用低。
###2.数据存储方式
新建数据表时，如果存储引擎ENGINE选择MyISAM，会在对应数据库下创建三个文件：`*.frm`数据表结构文件frame，`*.MYD`数据表数据文件data，`*.MYI`数据表索引文件index。
MyISAM的表结构，索引和数据是分开的，并且索引是有压缩的。
##2.InnoDB
###1.引擎特性
支持事务，支持外键，行级锁，批量插入速度低，5.5版本后支持全文索引，支持B-Tree索引，哈希索引，集群索引，数据不可压缩，磁盘空间和内存占用高。
**InnoDB的行锁不是绝对的**，在执行SQL语句时如果MySQL不确定扫描的范围，InnoDB表同样会锁全表，例如`UPDATE student SET age=12 WHERE name LIKE '%aaa%';`。SELECT COUNT(*) 和ORDER BY操作，InnoDB也会锁表的。
**只有通过索引条件检索数据，InnoDB才使用行级锁**，否则，InnoDB将使用表锁。
参考链接：http://blog.csdn.net/xiao7ng/article/details/5034013
###2.数据存储方式
新建数据表时，如果存储引擎ENGINE选择InnoDB，会在对应数据库下创建一个文件：`*.frm`数据表结构文件frame。所有的数据都会存储在同一个文件`ibdata1`中。
InnoDB是索引和数据是紧密捆绑的，没有使用压缩。所以InnoDB比MyISAM体积庞大不小。