##1.分类
每次查询都有先明确数据源。
###1.按位置分类
按照子查询语句在外部查询语句中出现的位置分为3类：
- FROM子查询：子查询语句跟在FROM后
- WHERE子查询：子查询语句在WHERE条件中
- EXISTS子查询：子查询语句出现在EXISTS语句里面
###2.按结果分类
根据子查询得到的数据进行分类：
- 标量子查询：子查询得到的结果是一行一列，子查询语句在WHERE之后。
- 列子查询：子查询得到的结果是一列多行，子查询语句在WHERE之后。
- 行子查询：子查询得到的结果是一行多列（或多行多列），子查询语句在WHERE之后。
- 表子查询：子查询得到的结果是多行多列，子查询语句在FROM之后。
##2.标量子查询
查询结果当普通值用。
需求：知道班级名字，找出班级的所有学生。
过程：用班级名字查询班级表，获取班级ID，然后查询学生表，找出所有班级ID对应的学生。
`SELECT * FROM students WHERE ClassID = (SELECT ClassID FROM classes WHERE ClassName='PHP0702');`
##3.列子查询
查询结果当集合用。
返回一列多行数据，需要IN，=ANY，=SOME，=ALL作为匹配条件。NULL结果排除在结果集之外。
肯定结果：
=ANY：和IN一样，满足任意一个即可
=SOME：和=ANY一样
=ALL：全部，一般只用否定查询
否定结果：
！=ANY：和NOT IN一样，不满足任意一个即可
！=SOME：和！=ANY一样
！=ALL：不等于所有的
```sql
-- 肯定结果
SELECT * FROM student WHERE ClassID IN (SELECT ClassID FROM classes);
SELECT * FROM student WHERE ClassID =ANY (SELECT ClassID FROM classes);
SELECT * FROM student WHERE ClassID =SOME (SELECT ClassID FROM classes);
SELECT * FROM student WHERE ClassID =ALL (SELECT ClassID FROM classes);
-- 否定结果
SELECT * FROM student WHERE ClassID NOT IN (SELECT ClassID FROM classes);
SELECT * FROM student WHERE ClassID !=ANY (SELECT ClassID FROM classes);
SELECT * FROM student WHERE ClassID !=SOME (SELECT ClassID FROM classes);
SELECT * FROM student WHERE ClassID !=ALL (SELECT ClassID FROM classes);
```
##4.行子查询
查询结果当一行或多行元素用。
子查询返回的结果可以是一行多列或多行多列。多个字段的行元素作为WHERE条件时，必须用括号括起来。
```sql
SELECT * FROM students WHERE (name, age) = (SELECT MAX(name), MAX(age) from students); -- (name, age)是行元素
-- 也可以用下面的语句实现（2个列子查询组成）
SELECT * FROM students WHERE name = (SELECT MAX(name) FROM students) and WHERE age = (SELECT MAX(age) FROM students);
```
##5.表子查询
查询结果当二维表用。
表子查询的结果作为**FROM的数据源**。主要必须要给数据源别名`AS 别名`。
例如：选出每个班级最高的学生：先降序排序得到新数据表，再分组取每个班级第一个数据。
```sql
-- 正确语句
SELECT * FROM (SELECT * FROM students ORDER BY Heighe DESC) AS student GROUP BY ClassID;
-- 错误语句，先分组再排序则排序无效，每组只返回最靠前的记录
SELECT * FROM students GROUP BY ClassID ORDER BY Heighe DESC;
```
##6.EXISTS子查询
EXISTS子查询用来判断某些条件是否满足（跨表），EXISTS语句接在WHERE之后才有意义。
EXISTS只返回0或1。
``` SQL
-- 判断某语句是否有数据，没啥卵用
SELECT EXISTS(SELECT * FROM students);
-- 如果有班级数据，则查询所有学生。。。
SELECT * FROM students WHERE EXISTS(SELECT * FROM classes);
```