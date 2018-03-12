#1.名词
数据库：database
数据库管理系统：DBMS(Database Management System）
数据库系统：DBS（Database System） = DBMS + DB
数据库管理员：DBA（Database Administrator）
行/记录：row/record，表中的一行（一条记录）
列/字段：column/field，字段
结构化查询语言SQL：Structured Query Language
#2.关系型数据库和非关系型数据库
数据库可以分为两类：
##1. 关系型数据库SQL（指用关系模型来组织数据的数据库，关系模型指的就是二维表格模型）：数据存储在磁盘上。安全。支持复杂查询，可以用SQL语句在多个表之间做复杂的数据查询。支持事务（数据库事务必须具备ACID特性，ACID是Atomic原子性，Consistency一致性，Isolation隔离性，Durability持久性），使得对于安全性能很高的数据访问要求得以实现。

关系模型包括三个方面：
- 数据结构：数据存储问题，二维表
- 操作指令集合：所有SQL语句
- 完整性约束：表内数据约束（字段与字段），表与表之间约束（外键）

代表产品有：
- 大型：Oracle，DB2
- 中型：MySQL
- 小型：access,SQLLite

##2. 非关系型数据库NoSQL(Not Only SQL或No Relation SQL)：数据存储在内存上。效率高。是传统关系型数据库的功能阉割版本，通过减少很少用的功能，来大幅度提高产品性能。基于键值对，数据之间没有耦合性，所以非常容易水平扩展。
memcached,mongodb,redis（可以同步到磁盘，重启后数据可以恢复）。

参考：从关系型数据库到非关系型数据库：http://blog.csdn.net/robinjwong/article/details/18502195/
#3.SQL指令分类
1. DDL：Data Definition Language，数据定义语言，用来维护存储数据的结构（数据库，表）。指令有：create,drop,alter
2. DML：Data Manipulation Language，数据操作语言，对表中的数据进行操作。指令有：insert,delete,update,select
3. DCL：Data Control Language，数据控制语言，负责权限管理。指令有：grant，revoke
#4.MySQL数据库基本操作
##1. 连接与断开
 `mysql -u用户名 -p -h主机IP地址 -P端口号（默认3306的话可以不写）  #连接数据库，需要密码`
 `show databases;  #显示所有数据库`
 `use 数据库;  #使用某一个数据库`
 `show tables;  #显示所有数据表`
 `exit`或`quit`或`\q`退出数据库
MySQL服务器内部对象分为四层：系统（DBMS）->数据库（DB）->数据表（Table）->字段（field）
##2. 增删改查CRUD
根据操作对象不同，SQL基本操作分为三类：库操作，表操作（字段操作），数据操作。
MySQL单行注释用双中划线加空格，或#，多行注释/**/。
###1.数据库操作
####1.1. 新增数据库
CREATE DATABASE 数据库名【库选项】;
其中，库选项用来约束数据库，分两个选项：
- 字符集设定：charset/character set 具体字符集。注意UTF-8在MySQL中不加中横线（UTF8）。
- 校对集设定：collate 具体校对集。
关键字和保留字最好不要用做数据库名字，非要用时，必须用反引号（ESC键的引号）扩起来。
例如：`CREATE DATABASE mydb charset utf8;`
数据库创建后，会在保存数据库的文件夹下，创建一个对应数据库名字的文件夹，并添加db.opt配置文件，配置文件中有字符集和校对集设定。
####1.2.查看数据库
SHOW DATABASES;
SHOW DATABASES LIKE 'pattern';
pattern是匹配模式，用'%'百分号匹配多个字符，用'_'下划线匹配一个字符，转义字符'\'。
####1.3.查看数据库创建语句
SHOW CREATE DATABASE 数据库名;
数据库在执行SQL语句前会先优化，所以数据库创建语句会有所不同。
####1.4.更新数据库
数据库名字不可以改。
数据库只可以修改库选项：字符集和校对集（校对集依赖字符集）
ALTER DATABASE 数据库名【库选项】;
CHARSET/CHARACTER SET [=] 字符集
COLLATE [=] 校对集
####1.5.删除数据库
DROP DATABASE 数据库名；
数据库删除后，在保存数据库的文件夹下，删除对应数据库的文件夹。不可逆，必须先备份再删除。
###2.表操作
表和字段密不可分。
####2.1.新增数据表
CREATE TABLE [IF NOT EXITSTS] 表名（
字段名称    数据类型，
字段名称    数据类型             #最后一行不需要逗号
）[表选项]；
IF NOT EXITSTS：如果表不存在就创建，否则不创建。
表选项：控制表的表现。
- 字符集：CHARSET/CHARACTER SET 具体字符集
- 校对集：COLLATE 具体校对集
- 存储引擎：ENGINE 具体存储引擎（ MyISAM或InnoDB）
```sql
CREATE TABLE IF NOT EXISTS student(
name varchar(10),
age int
)charset utf8;
```
####2.2.查看数据表
SHOW TABLES;-- 查看数据库中的所有表
SHOW TABLES LIKE 'pattern';-- 模糊匹配
SHOW CREATE TABLE 表名; -- 查看创建表的SQL语句。分号可用'\g'（相当于分号）或'\G'（将查询到的结果旋转90度纵向显示）替换，但部分版本用\g或\G会报错。
####2.3.查看表结构（表中字段信息）
DESC 表名；
DESCRIBE 表名；
SHOW COLUMNS FROM 表名；
字段名|字段类型|字段属性（是否允许为NULL）|索引（类型有PRI主键，UNI唯一键等）|默认值|附加字段属性（AUTO INCREMENT自增长）
####2.4.修改数据表（修改表本身或修改字段）
#####修改表
RENAME TABLE 旧表名 TO 新表名；-- 改表名
ALTER TABLE 表名 表选项 [=] 值；-- 修改表选项（字符集，校对集，存储引擎）
例如：RENAME TABLE student TO my_student;
#####修改字段（新增，修改，重命名，删除）
1. 增加字段
ALTER TABLE 表名 ADD [COLUMN] 字段名 数据类型[列属性][位置];
位置：FIRST（作为第一个字段），AFTER 字段名（在某个字段之后）。
例如：ALTER TABLE my_student ADD COLUMN id INT FIRST;
2. 修改字段属性或类型
ALTER TABLE 表名 MODIFY 字段名 数据类型 [属性][位置];
例如：ALTER TABLE my_student MODIFY number CHAR(10) AFTER id;
3. 重命名字段
ALTER TABLE 表名 CHANGE 旧字段名 新字段名 数据类型 [属性][位置];
例如：ALTER TABLE my_student CHANGE xingbie sex varchar(10);
4. 删除字段
ALTER TABLE 表名 DROP 字段名；
例如：ALTER TABLE my_student DROP age;
####2.5.删除表
DROP TABLE 表名1，表名2.......;
删除数据表后，对应文件系统的数据库文件夹下，表文件*.frm被删除。
###3.数据操作
####3.1.新增数据
INSERT INTO 表名 VALUES（值列表）[，（值列表）...];-- 给全表字段插入数据，必须顺序一致，非数值数据要用单引号包裹。可以一次插入多条记录。
INSERT INTO 表名（字段1，字段2，....） VBALUES（值1，值2，...）[，（值1，值2）...]；-- 给指定字段插入数据。
####3.2.查看数据
SELECT */字段列表 FROM 表名 [WHERE条件];
例如：
SELECT * FROM my_student; -- 查看所有数据
SELECT age,name FROM my_student; --查看指定字段，指定条件的数据
####3.3.更新数据
UPDATE 表名 SET 字段 = 值 [WHERE条件];
####3.4.删除数据
DELETE FROM 表名 [WHERE条件];
#5.附加问题
##1.中文数据问题
MySQL支持多种字符集。
###1.1.查看字符集
`SHOW VARIABLES LIKE '%char%';` -- 查看系统设定的字符集
`SHOW CHARACTER SET;`-- 查看所有支持的字符集
+--------------------------+---------------------------------------------------------+
| Variable_name            | Value                                                   |
+--------------------------+---------------------------------------------------------+
| character_set_client     | utf8                                                      |
| character_set_connection | utf8                                                    |
| character_set_database   | utf8                                                    |
| character_set_filesystem | binary                                                  |
| character_set_results    | utf8                                                    |
| character_set_server     | utf8                                                    |
| character_set_system     | utf8                                                    |
| character_sets_dir       | C:\Program Files\MySQL\MySQL Server 5.7\share\charsets\ |
+--------------------------+---------------------------------------------------------+
其中，系统设定的字符集中，`character_set_client`表示MySQL服务器默认的客户端发送数据的字符集，当该项设置与客户端实际字符集不一致时会报错，需要修改该项设置。
###1.2.设置字符集
`SET CHARACTER_SET_CLIENT=GBK;`-- 修改服务器认为的客户端字符集
`SET CHARACTER_SET_RESULTS=GBK;`-- 修改服务器返回数据的编码字符集为GBK
SET 变量名 = 变量值；这种方式设置的变量仅本次回话有效。
设置服务器对客户端字符集的认识，可以用快捷方式：`SET NAMES 字符集`。
`SET NAMES GBK;`  
CHARACTER_SET_CLIENT,CHARACTER_SET_RESULTS,CHARACTER_SET_CONNECTION这三个选项受影响。其中CHARACTER_SET_CONNECTION是字符集转变的中间值，统一后可以提高效率。
##2.校对集
校对集指数据的比较方式，必须在有数据之前设置。依赖于字符集。只有数据比较或排序时，校对集才会生效。对于UTF-8编码，默认校对集是UTF8_GENERAL_CI，可以设定为UTF8_BIN。
_bin：binary，二进制比较，取出二进制位，逐位比较。区分大小写。
_cs：case sensitive，大小写敏感。
_ci：case insensitive，大小写不敏感。
_ci和_bin经常使用。
`SHOW COLLATION;`-- 显示所有校对集，带YES的表示是默认校对集。
