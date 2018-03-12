##1.基本语法UNION
将多次查询（多条SELECT语句）在记录上进行拼接，不会增加字段。
每条SELECT语句获取记录的字段数必须一样多，但类型可以不一致。
语法：`SELECT语句1 UNION [UNION选项] SELECT语句2;`
UNION选项和SELECT选项一样，分为ALL和DISTINCT，区别是联合查询时默认为DISTINCT去重，一般查询则默认ALL所有。
```sql
SELECT * FROM students UNION ALL SELECT * FROM students;
```
##2.联合查询与连接查询的区别
联合查询是将任意表查询的结果合并在一起输出，是一种强制人为操作。
连接查询（内连接，外连接）的表通过外键进行关联，查询结果能自动合成一张表。
##3.意义
####1.查询同一张表，但需求比较复杂：查学生信息，男生身高升序排列，女生身高降序排列。
####2.多表查询：数据量太大水平分表时，组合各表查询到的数据。
##4.ORDER BY使用
联合查询中，ORDER BY不能直接使用，需要对查询语句加括号，且必须搭配LIMIT和最大限制数量（如果选择全部数据，这个限制数量写的很大即可）。
```SQL
(SELECT * FROM students WHERE sex='male' ORDER BY age DESC LIMIT 999999) 
UNION 
(SELECT * FROM students WHERE sex='female' ORDER BY age ASC LIMIT 999999);
```