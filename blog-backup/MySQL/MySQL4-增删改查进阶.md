#1.主键冲突
新增数据时，如果主键对应的数据已经存在，则会报错：
`ERROR 1062 (23000): Duplicate entry '2' for key 'PRIMARY'`。可以进行更新或替换操作。
###1.更新
INSERT INTO 表名[(字段列表，包括主键)] VALUES(值列表) ON DUPLICATE KEY UPDATE 字段=值[,字段=值...];
###2.替换replace
REPLACE INTO 表名[(字段列表，包括主键)] VALUES(值列表);
**注意：不管是更新还是替换，主键冲突时会影响2行数据。可以用`SELECT last_insert_id();`查询影响的行数。**
#2.蠕虫复制
从已有的表中获取数据，然后将数据插入对应表中。
###1.复制表结构
CREATE TABLE 新表名 LIKE 旧表名;
此时只有结构，没有数据。
###2.蠕虫复制
INSERT INTO 表名[(字段列表)] SELECT 字段列表/* FROM 表名;
蠕虫复制的意义：
- 从已有表拷贝数据到新表中。
- 让表中数据成倍增长迅速膨胀，测试表的效率及压力。
#3.限制新增，限制删除
###1.限制新增数量：
UPDATE 表名 SET 字段=值 [WHERE条件] [LIMIT 数量];
###2.限制删除数量：
DELETE FROM 表名 [WHERE条件] [LIMIT 数量];
###3.全部删除：
DELETE FROM 表名;
delete会删除表中的所有数据，但是不会重置自增长的当前计数值。
###4.全部删除：
TRUNCATE 表名;
    **如果有ROLLBACK语句，DELETE操作将被撤销，但TRUNCATE不会撤销。**
    TRUNCATE在功能上与不带WHERE子句的DELETE语句相同，但TRUNCATE重置自增长的当前计数值。
　　DELETE语句每次删除一行，并在事务日志中为所删除的每行记录一项。TRUNCATE通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放。
　　对于由 FOREIGN KEY 约束引用的表，不能使用 TRUNCATE TABLE，而应使用不带 WHERE 子句的 DELETE 语句。由于 TRUNCATE TABLE 不记录在日志中，所以它不能激活触发器。
　　TRUNCATE不能用于参与索引视图的表。
#4.查询
查询语句完整格式：
**SELECT [SELECT选型] 字段列表[字段别名]/* FROM 数据源 [WHERE子句] [GROUP BY子句] [HAVING子句] [ORDER BY子句] [LIMIT子句]**
###1.SELECT选型
####1.默认不写时，就是ALL
####2.DISTINCT去重（所有字段都相同时，才被认为是重复数据）
SELECT DISTINCT * FROM 表名;-- 全部字段查询并去重
SELECT DISTINCT(字段名) FROM 表名；-- 查询一个字段并去重
SELECT DISTINCT(字段名1),字段名2... FROM 表名 GROUP BY 字段名1；-- 查询多个字段并去重，此时需要用GROUP BY分组才能是字段名1不重复。
###2.字段别名
多表联合查询时，可能会有字段同名，此时可以用字段别名解决。
字段名 [AS] 别名
SELECT 字段名 [AS] 别名,字段名 [AS] 别名... FROM 表名;
###3.数据源
####1.单表数据源
SELECT * FROM table;
####2.多表数据源（没啥用）
表名用逗号分隔，最终得到的记录数是每个表的记录数的乘积（笛卡尔积）。
SELECT * FROM table1,table2,.....
####3.子查询（数据来源是一条查询语句）
SELECT * FROM(SELECT 语句) AS 别名;
###4.WHERE子句
WHERE 条件语句
WHERE子句返回0（false）或1（true），用来筛选数据。因为MySQL没有布尔类型，用0代表false（枚举类型从1开始，字符串也是从1开始计算）。
####1.判断条件：
- 比较运算符：>,<,>=,<=,!=,<>,=,LIKE（模糊匹配），BETWEEN AND（闭区间，左值必须小于等于右值），IN/NOT IN（多数据）
- 逻辑运算符：&&(AND),||(OR),!(NOT)
a
####2.WHERE原理
WHERE是唯一一个在从磁盘获取数据的时候就开始判断的条件。从磁盘取出的每一条记录，先判断是否满足WHERE条件，满足则保存到内存，否则舍弃。
####3.示例
SELECT * FROM myTable WHERE ID=1 || ID=3;-- ID等于1或3
SELECT * FROM myTable WHERE ID=1 OR ID=3;
SELECT * FROM myTable WHERE ID IN (1,3);

SELECT * FROM myTable WHERE age >= 20 AND age <= 30; -- age在[20,30]之间
SELECT * FROM myTable WHERE age between 20 AND 30;
###5.GROUP BY子句
SELECT 字段，统计函数 FROM 表 GROUP BY 字段 [ASC | DESC]
分组是为了按组统计数据。分组后，默认对得到的组进行升序排序。分成几组就会展示几条记录（每组只展示第一条记录）。
####1.统计函数
count()：统计分组后每一组有多少条记录。参数可以是*（统计记录数）或字段（当字段为NULL时忽略）。
max()：求分组中的最大值。
min()：求分组中的最小值。
avg()：求平均值。
sum()：求和。
SELECT sex,count(*),max(height),min(height),avg(age),sum(age) FROM myTable GROUP BY sex;
####2.多字段分组
GROUP BY 字段1，字段2......；
SELECT class,sex,count(*) FROM myTable GROUP BY class,sex;
最终得到的数据是将表中所有的数据，先用class分组，然后在每一个分组中用sex分组。
####3.group_concat()函数
对分组结果中的某个字段进行字符串连接。
SELECT class,sex,count(*),group_concat(name) FROM myTable GROUP BY class,sex;
####4.回溯统计WITH ROLLUP，多字段分组回溯统计
分组后，将当前分组信息向上级分组进行汇报统计。
SELECT class,sex,count(*),group_concat(name) FROM myTable GROUP BY class,sex WITH ROLLUP;
###6.HAVING子句
进行条件判断。
####1.WHERE子句与HAVING子句区别：
- 分组统计的结果只有HAVING能用。WHERE处理硬盘上的数据，但数据读到内存后进行分组统计，此时仍存在于内存上，只有HAVING能处理。
`SELECT class,sex,count(*),group_concat(name) FROM myTable GROUP BY class,sex HAVING count(*) > 2;`
- HAVING可以使用字段别名，但WHERE不能用。WHERE从磁盘读数据，只能用字段名。别名是在数据读入内存后才有的。
`SELECT class,sex,count(*) as total,group_concat(name) FROM myTable GROUP BY class,sex HAVING total > 2;`
###7.ORDER BY子句
排序，依赖校对集。
ORDER BY 字段名 [ASC | DESC]
其中ASC是升序，DESC是降序。
####1.单字段排序
`SELECT * FROM myTable ORDER BY id；`
####2.多字段排序
先用第一个字段排序，然后对分组后的每一段数据再做排序。
`SELECT * FROM myTable ORDER BY sex,name；`
###8.LIMIT子句
限制结果，限制数量
####1.限制长度
LIMIT 数据量
`SELECT * FROM myTable LIMIT 2;-- 只查询2条记录`
####2.限制起始位置，限制数量（分页）
LIMIT 起始位置，长度
其中，起始位置OFFSET=（页码-1）*每页显示条数
长度=每页显示条数
`SELECT * FROM myTable LIMIT 4,2;-- 从第4条记录开始，查询2条记录`
