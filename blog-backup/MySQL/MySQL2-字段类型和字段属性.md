
#1.字段类型
MySQL的数据类型分为3类：数值型，字符串型，日期时间型。

##1.数值型
数值型分为整数型和小数型。可以有符号（Signed）和无符号（Unsigned）。默认是有符号类型，可以在类型后加UNSIGNED指定为无符号类型`int unsigned`。
###1.整数型
![这里写图片描述](http://img.blog.csdn.net/20170430182833729?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
TINYINT：用1个字节存储，范围0~255或-128~127。
SMALLINT：用2个字节存储，范围0~65535或-32768~32767。
MEDIUMINT：用3个字节存储。
INT：用4个字节存储。
BIGINT：用8个字节存储。
+-------+-------------+------+-----+---------+-------+
| Field | Type        | Null | Key | Default | Extra |
+-------+-------------+------+-----+---------+-------+
| id    | int(10)     | YES  |     | NULL    |       |
| name  | varchar(10) | YES  |     | NULL    |       |
| sex   | tinyint(1)  | YES  |     | NULL    |       |
+-------+-------------+------+-----+---------+-------+
- 常用的整数型有TINYINT,SMALLINT,INT。
- 用`SHOW CREATE TABLE 表名`查看表结构时，类型后面的括号中的数字表示’显示宽度‘。当数据不够显示宽度时，此时如果指定填充0`ZEROFILL`，则数据会填充前导0来达到显示宽度。注意此时数据只能是无符号型。
例如：`ALTER TABLE ADD student COLUMN TINYINT(2) ZEROFILL;` -- 显示宽度为2，填充0
###2.浮点型
浮点型数据，小数点浮动，精度有限且会丢失精度。

![这里写图片描述](http://img.blog.csdn.net/20170430183333465?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

FLOAT：单精度，占4个字节，精度范围大概7位左右。
DOUBLE：双精度，占8个字节，精度范围大概15位左右。
- FLOAT(7,3) 规定显示的值不会超过 7 位数字，小数点后面带有 3 位数字。
- 对于小数点后面的位数超过允许范围的值，MySQL 会自动将它四舍五入为最接近它的值。但是如果整数范围超长度会报错。
- 超出精度范围时，浮点数会四舍五入。如果此时进位导致整数部分长度超过指定长度，系统也允许成立。
```SQL
create table my_float(
    -> f1 float,
    -> f2 float(10, 2), --10位，超过精度范围
    -> f3 float(6, 2) --6位，在精度范围内
    -> );
```
输入数据可以直接是小数，也可以用科学记数法`3e38`，`3.01e7`

###3.定点型
定点型数据，小数点固定，精度固定且不会丢失精度（不会四舍五入）。整数部分绝对不会四舍五入，不会丢失精度。
DECIMAL（M，D）或DEC（M，D）：占M+2个字节。
##2.时间日期类型
![这里写图片描述](http://img.blog.csdn.net/20170430183056379?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
###1.DATETIME
日期时间，格式是YYYY-mm-dd HH:ii:ss。表示范围从1000至9999年，有0值0000-00-00 00:00:00。
###2.DATE
日期部分。
###3.TIME
时间部分。可以是负数。
###4.TIMESTAMP
时间戳，不能为空，默认值是当前时间戳，从1970-01-01 00:00:00开始的日期时间。**该字段所在记录的任何一个字段被更新时，对应的该字段会被刷新为当前时间戳。**只用于SESSION。
###5.YEAR
可以用2位或4位数插入。
##3.字符串
![这里写图片描述](http://img.blog.csdn.net/20170430182930830?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
SQL中，字符串分为六类：CHAR,VARCHAR,TEXT,BLOB,ENUM,SET
###1.CHAR定长字符串
二维表定义的时候，已经确定了最终数据的存储长度。浪费磁盘空间，但效率高。如果数据长度基本一致，就用定长。如手机号，身份证号。
CHAR(L)：L代表可以存储的长度，单位是字符，最大255。
存储中文时，例如CHAR(4)在UTF8环境下，需要分配12个字节的空间。
###2.VARCHAR变长字符串
分配空间时，按最大的空间分配，但实际用多少则根据具体的数据来确定。节省磁盘空间，但效率低。
VARCHAR(L)：L代表可以存储的长度，单位是字符，最大65535，实际上超过255的话，最好用文本字符串TEXT。
需要多1-2个字节用于存储实际占用的空间（实际数据占用字节数小雨256时需要1个字节，大于时需要2个字节）。
###3.TEXT文本字符串
数据长度超过255个字符，就用TEXT。
TEXT：存储文字（存储二进制数据的路径）。
BLOB：存储二进制数据（通常不用，存储到磁盘上，拿到路径）。
###4.ENUM枚举字符串
提前将所有可能出现的结果都设计好，实际存储的数据必须是 规定好的数据之一。实际存储的是数字，从1开始，加一递增。数字与字符串对应的关系，存储在日志中。
定义：ENUM（可能出现的元素列表）；//ENUM('male','female','unknown');
ENUM可以规范数据格式，节省存储空间（实际存储的是数值而不是字符串）。将字段结果取出，做加零运算（+0）`SELECT sex+0, sex from my_enum;`。
###5.SET集合字符串（PHP基本不用）
存储数值而非字符串，多选。实际存储的是数字，从1开始，乘二递增。每一个元素对应一个二进制位，被选中则置一。
插入数据时，一个字符串内可以放多个数据，用逗号分隔，也可以直接插入数字，用对应位置的数字之和（例如3=1+2，即第一个和第二个元素同时选中）。
##4.MySQL记录长度
**MySQL规定，任何一条记录，最长不能超过65535个字节。**
UTF8下VARCHAR最多21844个字符。
TEXT只占用10个字节存放地址，数据不占用记录长度，额外存储。
若一条记录中有任一字段允许为空，则系统会自动从整个记录中保留一个字节来存储NULL（若所有字段都不允许为空，则释放此字节）。
#2.字段属性
NULL/NOT NULL,DEFAULT,Primary key,Unique key,auto_increment,comment
##1.空属性
NULL：默认的，允许为空。
NOT NULL：非空。
任何东西和NULL运算，结果都是NULL。例如`SELECT 1+NULL`。
##2.列描述COMMENT 
COMMENT 描述内容：列的具体信息，用途的描述。
##3.默认值default 
default 默认值：插入记录时，不给已经赋默认值的字段赋值，则使用默认值。
```SQL
CREATE TABLE class(
name varchar(20) not null comment 'classroom name',
money decimal(10,2) not null comment 'salary'
);
```
##4.主键primary key
一张表里面最多只能有一个字段可以使用，即最多有一个主键（可以有复合主键）。主键用于约束数据的唯一性。主键自动增加NOT NULL属性。
###1.增加主键
####1.创建表时用primary key为一个字段创建主键
```SQL
CREATE TABLE my_pri1(
name varchar(20) not null,
id int primary key comment '学号'
)charset utf8 engine innodb;
```
####2.创建表时用primary key创建复合主键
```SQL
CREATE TABLE my_pri2(
id int comment '学号',
course char(10) comment '学科',
score tinyint unsigned default 60 comment '成绩',
primary key(id,course)
);
```
####3.创建表后，追加主键
ALTER TABLE 表名 MODIFY 列名 列类型 PRIMARY KEY 列属性;
ALTER TABLE 表名 ADD PRIMARY KEY（字段列表）;
###2.主键约束
单字段主键或复合主键，对应的字段中，数据不允许重复。一旦重复，数据增加或修改失败。
###3.主键删除（主键不能更新，只能删除后增加）
ALTER TABLE 表名 DROP PRIMARY KEY;
##5.自增长auto_increment
###1.当对应的字段不给值，或给默认值，或给NULL时，系统会自动从当前字段所有值中找出最大值，加一后存入该条记录。
自增长通常跟主键搭配。
- 只有是索引（DESC查看表结构时，key一栏有值）的字段才可以做自增长。
- 自增长字段必须是数字（整型）。
- 一张表最多只能有一个自增长字段。
- 自增长从1开始，每次递增1。
```SQL
CREATE TABLE increment(
id int PRIMARY KEY AUTO_INCREMENT,
name varchar(20) NOT NULL
)charset utf8;
```
###2.自增长下一个值可以修改（只能改大），可以用SHOW CREATE TABLE查看自增长的下一个值
`ALTER TABLE 表名 AUTO_INCREMENT=整数;`
###3.自增长的系统变量
`show variables like '%auto_increment%';`
auto_increment_increment：步长。
auto_increment_offset：初始值。
可以用`SET auto_increment_increment=值`修改步长，当前回话有效。
###4.删除自增长
`ALTER TABLE 表名 MODIFY 字段名 INT;`
PRIMARY KEY不需要重复出现，否则会报错。
##6.唯一键
一张表中，经常有多个字段具有唯一性，数据不能重复。但主键只能有一个。唯一键（unique key）可以解决表中多个字段需要唯一性约束的问题。
唯一键可以为空。空字段不参与唯一性比较。
用DESC查看表结构时，如果没有主键，同时唯一键含有属性NOT NULL，Key列中唯一键会显示为主键PRI而不是唯一键UNI。
唯一键的创建与主键类似：
###1.创建唯一键
####1.创建表时用unique为一个字段创建唯一键
```SQL
CREATE TABLE my_unique1(
name varchar(20) not null,
id int unique comment '学号'
)charset utf8;
```
####2.创建表时用primary key创建复合唯一键
```SQL
CREATE TABLE my_unique2(
id int comment '学号',
course char(10) comment '学科',
score tinyint unsigned default 60 comment '成绩',
unique key(id,course)
);
```
####3.创建表后，追加唯一键
ALTER TABLE 表名 MODIFY 列名 列类型 UNIQUE KEY 列属性;
ALTER TABLE 表名 ADD UNIQUE KEY（字段列表）;
###2.删除唯一键
唯一键默认的索引名就是字段名。
ALTER TABLE 表名 DROP INDEX 索引名;
##7.索引
###1.概述
索引建立在字段上。根据某种算法，将已有的或未来会新增的数据单独建立一个文件，通过文件实现快速匹配数据，找到表中对应的记录。
索引意义：提升查询效率，约束数据的有效性和唯一性。
如果某个字段经常用作查询的条件，最好使用索引。
如果某个字段需要进行数据的有效性约束，可以使用索引（主键，唯一键）。
###2.类型
####1.主键索引：primary key
非空。
####2.唯一索引：unique key
CREATE UNIQUE INDEX 索引名 ON 表名（字段名）;
####3.全文索引：fulltext index（仅可用于 MyISAM 表）
针对文章内部的关键字进行索引。最大问题是如何确定关键字（英文单词之间有空格容易确定，中文分词很难）。
####4.普通索引：index
第一种方式 :
CREATE INDEX 索引名 ON 表名（字段名）;
第二种方式: 
ALTER TABLE 表名 ADD INDEX 索引名(字段名);
####5.组合索引
一个索引中包含多个字段。
CREATE INDEX 索引名 ON 表名（字段名1,字段名2...）;
###3.索引删除
DORP INDEX 索引名 ON 表名;