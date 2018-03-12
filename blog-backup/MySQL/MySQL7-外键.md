外键：表中某个字段指向另一张表中的主键时，这个字段就是外键。外键字段所在表是子表，另一张表是父表。
创建外键时，要求此字段必须是索引，如果不是索引会在创建外键前自动为其添加索引。但删除外键时，不会自动删除索引。
##1.创建外键
###1.创建表的时候创建外键
`foreigne key(字段名) references 表名(字段名)`
例如：
```sql
CREATE TABLE students(
StudentID int primary key auto_increment,
ClassID int comment '班级',
foreign key(ClassID) preferences class(ClassID)
)charset utf8;
```
然后用`SHOW CREATE TABLE students`查看。`CONSTRAINT`后面是外键名字，`FORENGN KEY`后面是外键字段，`REFERENCES`后面是外键引用。
用`desc student`查看时，外键的Key属性列显示为`MUL`多索引。
###2.为已有的表增加外键
`ALTER TABLE 表名 ADD [CONSTRAINT 外键名字] FOREIGN KEY（字段名） REFERENCES 表名（主键字段）;`
其中constraint指定外键名字，可以不写。
##2.删除外键
外键不能修改，只能先删除再增加。
`ALTER TABLE 表名 DROP FOREIGN KEY 外键名;`
一张表可以有多个外键，名字必须不同。
外键删除后，不会删除索引，所以结构上看不出效果（desc table），必须用创建语句查看（show create table ）。
##3.外键默认约束
###1.对子表的约束
子表数据进行写操作（增和改）时，如果外键字段的数据无法匹配到父表主键，则操作失败：
`ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`school`.`student`, CONSTRAINT `student_ibfk_1` FOREIGN KEY (`c_id`) REFERENCES `class` (`id`))`
###2.对父表的约束
父表数据进行写操作（删除和改：都必须涉及主键本身）时，如果对应的主键在子表中已经被数据引用，则操作失败：
`ERROR 1451 (23000): Cannot delete or update a parent row: a foreign key constraint fails (`school`.`student`, CONSTRAINT `student_ibfk_1` FOREIGN KEY (`c_id`) REFERENCES `class` (`id`))`
##4.创建外键的条件
####1.存储引擎
表的存储引擎必须是InnoDB（默认存储引擎）才可以。如果不是InnoDB，可以创建外键，但没有约束效果。
####2.外键字段的字段类型必须和父表的主键类型完全一致。
####3.同一张表中，外键名字不能重复。
####4.如果要为已有数据的表增加外键约束，必须保证已有的数据与父表主键的数据可以对应，否则增加外键约束失败。