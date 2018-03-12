##1.用途
视图是一个虚拟表，只有结构没有数据，其内容由查询定义。视图的结构从已有的表（称为基表underlying table）中产生。
####1.视图可以节省SQL语句：将复杂的查询语句用视图保存，以后可以对视图进行操作。
####2.数据安全：视图操作主要针对查询，如果删除视图不会影响数据。
####3.视图往往在大项目中使用，而且是多系统使用：可以隐藏指定的数据。
####4.视图可以更好的进行权限控制：保护数据库的信息。
##2.创建视图
```sql
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    VIEW view_name [(column_list)]
    AS select_statement
    [WITH [CASCADED | LOCAL] CHECK OPTION]
```
- CREATE：新建视图；
- REPLACE：替换已有视图
- ALGORITHM ：视图算法
- view_name ：视图名
- column_list：属性列
- select_statement：查询语句，可以是普通查询，连接查询，联合查询，子查询。
- [WITH [CASCADED | LOCAL] CHECK OPTION]：字段更新限制。任何对该字段的操作都必须满足操作后可查询的数据仍然可以查询到。

根据基表个数，视图分为2种：
-- 单表视图：基表只有一个。
-- 多表视图：基表有多个。注意不能有同名字段。
视图一旦创建，数据库文件夹下会创建对应的*.FRM文件。
```sql
CREATE VIEW view1 AS SELECT * FROM students WHERE age > 20 WITH CHECK OPTION;-- 只能查询到age>20的数据且不能将这些数据的age改的小于20（字段更新限制）。
-- 多表查询不能有同名字段
CREATE VIEW view2 AS SELECT students.*,classes.ClassID FROM students LEFT JOIN classes ON students.ClassID = classes.ClassID;
```
##3.查看视图结构
视图也是表，可以用查看表的所有命令（SHOW TABLES或DESC 视图名或SHOW CREATE 视图名）
```sql
SHOW TABLES;
DESC 视图名;
SHOW CREATE TABLE 视图名;
SHOW CREATE VIEW 视图名;-- 视图专有的命令
```
##4.使用视图
视图主要用于查询，当成表用即可。
视图本质的封装的SELECT语句，实现代码的封装复用。
`SELECT * FROM 视图名;`
##5.修改视图
视图本身没有啥数据，不可修改，但可以修改视图的来源。修改视图即修改视图的来源语句（SELECT语句）。
`ALTER VIEW 视图名 AS SELECT语句;`
##6.删除视图
`DROP VIEW 视图名;`
##7.视图操作
###1.新增数据
####1.多表视图不能用来增加数据。
####2.单表视图可以增加数据，前提是视图中包含基表中不能为空的字段，否则报错。
`INSERT INTO 视图名 VALUES(...);`
###2.删除数据
####1.多表视图不能删除数据。
####2.单表视图可以删除数据。
`DELETE FROM 视图名 WHERE 条件;`
###3.更新数据
单表视图和多表视图都可以更新数据。
用视图更新数据时，只能更新视图可以查到的数据。
`UPDATE 视图名 SET 字段名 = 值 WHERE 条件;`
更新限制：`WITH CHECK OPTION`：如果在创建视图的时候限制了某个字段，在通过视图更新数据的时候，系统会进行验证，保证更新后的数据仍然可以被视图查到。
##8.视图算法
参考文档：https://dev.mysql.com/doc/refman/5.7/en/view-algorithms.html
实例：http://blog.itpub.net/25704976/viewspace-1319994/
视图的处理算法有三种：
    1.MERGE：会将引用视图的语句的文本与视图定义合并起来，使得视图定义的某一部分取代语句的对应部分。
    2.TEMPTABLE：视图结果将被置于临时表中，然后使用它执行语句。临时表视图是不可更新的。
    3.UNDEFINED：默认算法。视图没有定义算法，系统自动选择。系统通常会优先选择MERGE，因为MERGE通常更有效。

如果视图中包含如下的函数，则MERGE算法将不能被使用：
    1.聚合函数 COUNT()，SUM()等
    2.DISTINCT
    3.GROUP BY
    4.HAVING
    5.LIMIT
    6.UNION OR UNION ALL
    7.select列中含有子查询
    8.只有 literal values，即（create view as select 1 as aa，类似这样没有基础表）