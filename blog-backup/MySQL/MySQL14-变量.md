##1.系统变量
系统变量用来控制服务器的表现，大部分用不到。例如version，auto_commit，auto_increment_offset，character_set_result。
###1.查看系统变量
```sql
show variables; -- 查看所有的系统变量
show warnings; -- 查看系统警告信息
select @@version,@@character_set_result; --  查看指定的系统变量
```
###2.修改系统变量
修改系统变量有两种类型：
- 会话级别（当前会话有效）：`set 变量名 = 值；`或`set @@变量名 = 值；`
- 全局级别（影响所有用户，对于已登录的用户需要重新登录才有效）。set global 变量名 = 值；`
```sql
set autocommit = 1; -- 会话级别
set @@autocommit = 1; -- 会话级别
set global autocommit = 1; -- 全局级别
```
##2.自定义变量
所有自定义变量都是会话级别，当前会话始终有效。
###1.定义和查看
自定义变量的名字用@开头。
定义：`set @name = '李四';`
查看：`select @name;`
###2.MySQL的等号问题
在MySQL中，等号‘=‘只在两种情况下（都是跟在SET后）作为赋值符号用：
- 设置变量时，例如set autocommit = 1;
- 更新数据时，例如`UPDATE student SET name = 'rose' WHERE id = 1;`。
**其他情况下，等号当做比较符号用，要用等号时，得用‘：=’。**
例如：`set @age := 29;`
###3.MySQL中可以从数据表中获取数据，赋值给变量
####1.边赋值，边查看结果（如果有多条记录，最终赋值最后一条）
语法：`SELECT @变量名 := 字段名 FROM 数据源; -- 必须用:=赋值`
例如：`SELECT @name := name FROM student;`
####2.直接赋值，必须只能有一条记录，否则报错
`SELECT 字段名 FROM 数据源 WHERE条件 INTO @变量名;`
