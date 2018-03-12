##1.概述
事务：一系列要连续发生的操作。
事务安全transaction security：保护连续操作同时发生或同时不发生的机制。保护数据的完整性。
**MyISAM引擎不支持事务，得用InnoDB。**
##2.事务操作
事务分两种：自动事务（默认的）和手动事务。
在数据库目录的data目录下，有ib_logfile0和ib_logfile1两个InnoDB引擎的事务日志文件（日志个数由`innodb_log_files_in_group`参数决定）。文件为顺序写入，当写至最后一个文件末尾时，会从第一个文件开始复用。开启事务后，所有的操作都会经过事务日志文件中转。
参考链接：http://dinglin.iteye.com/blog/906761

建立测试数据表并插入测试数据：
```sql
CREATE TABLE account(
id int not null primary key auto_increment,
number varchar(20) not null unique comment '账号',
name varchar(20) not null,
money decimal(10,3) not null default 0.0
)charset utf8;

INSERT INTO account values(null, '0001', 'jack', 8000);
INSERT INTO account values(null, '0002', 'rose', null);
```
###1.手动事务
####1.开启事务
`start transaction;`
####2.进行事务操作：一系列数据库操作
此时的操作并没有立刻改变数据库中的数据，而是记录在事务日志文件中。可以通过重新登录数据库查看数据来验证。
```sql
UPDATE account SET money = money - 1000 WHERE id = 1;
SELECT * FROM account; -- 此时id=1的用户金额减了1000，但实际数据并没有减，只是这次操作记录到了事务日志里且本次查询的数据经过事务日志文件处理了。
UPDATE account SET money = money + 1000 WHERE id = 2;
SELECT * FROM account; -- 同理，此时id=2的用户金额加了1000，但实际数据并没有加。
```
####3.关闭事务
- 提交事务：commit，同步数据表（操作成功）
- 回滚事务：rollback，直接清空日志表，撤销事务的所有操作（操作失败）
###2.自动事务
MySQL默认开启自动事务（系统变量autocommit=1），每一步用户操作都立即执行，同步到数据表中。除非显式地开始一个事务（start transaction）。
如果关闭自动事务，每一步用户操作都是一个事务，用户将一直处在事务，除非commit提交或rollback回滚。
```sql
show variables like 'autocommit'; -- 查看是否开启自动提交事务
set autocommit = 0; -- 关闭自动提交事务
```
##3.回滚点
事务比较复杂时，如果需要在出现错误时回滚到指定操作，就需要设置回滚点。
设置回滚点：`savepoint 回滚点名字;`
回滚至指定回滚点：`rollback to 回滚点名字;`
```sql
start transaction;
UPDATE account SET money = money + 1000 WHERE id = 2;
savepoint sp1;
UPDATE account SET money = money - 1000 * 0.1 WHERE id = 3;
rollback to sp1;
UPDATE account SET money = money - 1000 * 0.1 WHERE id = 2;
commit;
```
##4.事务特性ACID
参考链接：http://blog.csdn.net/shuaihj/article/details/14163713
A：Atomicity原子性，事务的整个操作是一个整体，不可分割，要么全部成功，要么全部失败。
C： Consistency一致性，事务开始之前和事务结束以后，数据库的完整性约束没有被破坏。
I：Isolation隔离性， 多个事务并发访问时，事务之间是隔离的，一个事务不应该影响其它事务运行效果。
D：Durability持久性，事务一旦提交，永久改变数据库中的数据。

InnoDB默认是行锁，但如果事务操作中没有使用索引，系统会自动全表检索数据，自动升级为表锁。此时如果一个事务改动某表，则另一个事务无法再改动此表。
