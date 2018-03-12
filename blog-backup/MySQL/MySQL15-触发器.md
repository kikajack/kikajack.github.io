不建议使用，[参考这里](https://segmentfault.com/q/1010000004907411)

##1.概述
触发器trigger：事先为某张表绑定好一段代码，当表中的某些内容发生改变的时候（增删改），系统自动触发代码执行。

- 事件类型：增insert，删delete，改update三种类型。
- 触发时间：前before，后after。
- 触发对象：表中的每一条记录（行）。

在一张表中，同样触发时间同样类型的触发器只能有一个，所以一张表中最多有6个触发器（BEFORE INSERT、BEFORE UPDATE、BEFORE DELETE、AFTER INSERT、AFTER UPDATE、AFTER DELETE）。
##2.创建触发器
MySQL高级结构中没有大括号，用字符 begin，end 代替。
```SQL
# 创建触发器
CREATE TRIGGER trigger_name
trigger_time
trigger_event ON table_name
FOR EACH ROW
trigger_stmt

# BEGIN … END 语句的语法：
BEGIN
[statement list]
END
```
trigger_name：触发器名称，用户自行指定；
trigger_time：触发时机，BEFORE 或 AFTER；
trigger_event：触发事件，INSERT、UPDATE 或 DELETE；
table_name：在哪张表上建立触发器；
trigger_stmt：触发器程序体，可以是一条 SQL 语句，或者用 BEGIN 和 END 包含的多条语句。
