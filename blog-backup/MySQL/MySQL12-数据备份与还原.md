##1.数据表备份（仅限MyISAM引擎）
每一个用MyISAM做引擎的表，在数据库目录下有三个文件，`*.frm`数据表结构文件，`*.MYD`数据表数据文件，`*.MYI`数据表索引文件。直接备份这三个文件即可完成数据表的备份。
##2.单表数据备份
###1.备份
将表中的一部分或所有的数据保存到外部文件。每次只备份一张表，且只备份数据（不备份表结构和索引）。
`SELECT */字段列表 INTO OUTFILE '文件所在路径' FIELDS 字段处理 LINES 行处理 FROM 数据源;`
参数FIELDS：字段处理
- ENCLOSED BY：字段用什么包裹，默认是''，空字符串。
- TERMINATED BY：字段用什么结束，默认是'\t'，tab键。
- ESCAPED BY：特殊符号用什么方式处理，默认是'\\'，用反斜杠转义。
参数LINES：行处理
- STARTING BY：每行用什么开始，默认是''，空字符串。
- TERMINATED BY：每行用什么结束，默认是'\r\n'，换行符。
```sql
SELECT * INTO OUTFILE '1.txt'
FIELDS
  ENCLOSED BY '"' -- 字段用双引号包裹
  TERMINATED BY '|' -- 字段用'|'结束
LINES
  STARTING BY '666' -- 每行用'666'开始
FROM students;
```
###2.权限设置
如果报错'ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement'，说明当前数据库限制数据导入和导出操作（如LOAD DATA、SELECT ... INTO OUTFILE语句和LOAD_FILE()函数）。
####1.查看secure-file-priv
`SHOW VARIABLES LIKE '%secure%';`
如果secure-file-priv是一个目录名，MySQL服务只允许在这个目录中执行文件的导入和导出操作。这个目录必须存在，MySQL服务不会创建它。
如果secure-file-priv为空或NULL，禁止导入和导出操作。
####2.修改配置文件并重启MySQL
修改my.cnf（Linux）或my.ini（Windows）文件。在[mysqld]最后面添加`secure_file_priv='/'`，即可将文件保存到任意位置。
###3.还原
`LOAD DATA INFILE 文件路径 INTO TABLE 表名 [(字段列表)] FIELDS 字段处理 LINES 行处理;`
##3.SQL备份
备份的是SQL语句：系统会对表结构和数据进行处理，变成对应的SQL语句备份。还原的时候只需要执行SQL语句即可。
###1.备份
MySQL没有提供备份指令，需要理由MySQL提供的软件mysqldump。
备份制定库：`mysqldump -uroot -proot 库名 > 路径名/备份的库名.sql`
备份制定表：`mysqldump -uroot -proot 库名 表名 > 路径名/备份的表名.sql`
###2.还原
用mysql命令还原：`mysql -uroot -proot 库名 < 备份文件`
用SQL指令还原SQL备份：`SOURCE 备份文件;`
##4.增量备份
不是真的数据或SQL指令进行备份，而是针对MySQL服务器的日志文件进行备份。
增量备份：只备份制定时间段。备份数据不会重复，且所有的操作都会备份。
大项目基本都用增量备份。