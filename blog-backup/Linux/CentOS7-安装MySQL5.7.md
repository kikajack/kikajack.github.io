本文写于2017年5月9日，有一定的时效性。
#1.介绍
Oracle收购了MySQL后，有不再开源的风险，CentOS7已经不再默认安装MySQL，而是改为默认安装MariaDB（MySQL的一个分支版本，兼容MySQL）。并且CentOS 7的yum源中没有安装MySQL所需的mysql-sever文件，需要去MySQL官网下载rpm文件。
#2.步骤
参考地址：https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/
###1.下载rpm文件
`wget https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm`
或者下载到本地，然后通过SSH终端用`rz`命令传到服务器上。
###2.为yum仓库添加MySQL源
第一次安装的话（i表示install）：`rpm -ivh mysql57-community-release-el7-11.noarch.rpm`
更新的话（U表示update）：`rpm -Uvh mysql57-community-release-el7-11.noarch.rpm`
###3.选择指定版本
从Oracle下载的rpm文件里有很多软件，每一款软件又有不同的版本，需要在下载前指定。
查看可用的软件版本（标注enabled的）`yum repolist enabled | grep mysql`
![这里写图片描述](http://img.blog.csdn.net/20170509205557532?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
上图中表示当前要安装的是mysql57-community/x86_64版本。
如果不是，需要修改，比如我要安装5.6版本：
`yum-config-manager --disable mysql57-community`
`yum-config-manager --enable mysql56-community`
###4.安装
Oracle官网给出的方法是：`yum install mysql-community-server`，实测行不通。我用这个语句可以安装：
`yum install mysql-server`
###5.启动
启动：`systemctl start mysqld.service`
查看状态：`systemctl status mysqld.service`
###6.查看默认密码
MySQL安装好后，有一个root用户的默认密码，登录即可：
`grep 'temporary password' /var/log/mysqld.log`
![这里写图片描述](http://img.blog.csdn.net/20170509211000199?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
###7.改root用户密码
方法一：登录后改
`mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('newpass');`
MySQL有一个对密码强度的校验机制（validate_password 插件），通过`show VARIABLES like "%password%"`查看：
```sql
mysql root@localhost:(none)> show VARIABLES like "%password%"
+---------------------------------------+---------+
| Variable_name                         | Value   |
|---------------------------------------+---------|
| default_password_lifetime             | 0       |
| disconnect_on_expired_password        | ON      |
| log_builtin_as_identified_by_password | OFF     |
| mysql_native_password_proxy_users     | OFF     |
| old_passwords                         | 0       |
| report_password                       |         |
| sha256_password_proxy_users           | OFF     |
| validate_password_dictionary_file     |         |
| validate_password_length              | 8       |
| validate_password_mixed_case_count    | 1       |
| validate_password_number_count        | 1       |
| validate_password_policy              | MEDIUM  |
| validate_password_special_char_count  | 1       |
+---------------------------------------+---------+
13 rows in set
Time: 0.030s
```
对应的政策如下：
| Policy |  Tests Performed |
| ------------------- |:----------|
|0 or LOW |Length
|1 or MEDIUM  |Length; numeric, lowercase/uppercase, and special characters
|2 or STRONG  |Length; numeric, lowercase/uppercase, and special characters; dictionary file
默认是MEDIUM，所以密码的策略是：数字+小写字母+大写字母+特殊字符，长度至少8位 。
对于要求不高的测试环境，可以修改默认密码策略：
```sql
mysql root@localhost:(none)> set global validate_password_policy = 0;
Query OK, 0 rows affected
Time: 0.003s
```
方法二：用mysqladmin
`mysqladmin -u root password oldpass "newpass"`
###8.MySQL安装后服务器默认Latin1编码，需设置
`vi /etc/my.cnf`
my.cnf 是MySQL 的配置文件，改前先备份`cp /etc/my.cnf /etc/my.cnf.bak`
在[mysqld]下面加上`default-character-set=utf8`后服务器启动不了，莫名其妙。后来这么改就好了：
```conf
[mysqld]
init_connect='SET collation_connection = utf8_unicode_ci' 
init_connect='SET NAMES utf8' 
character-set-server=utf8 
collation-server=utf8_unicode_ci 
skip-character-set-client-handshake
...
[client]
default-character-set=utf8
```
###9.MySQL5.7默认监听在IPv6的地址，可以改my.cnf监听IPv4地址族
修改 my.cnf，在[mysqld]下面添加一行配置：
`bind-address = 0.0.0.0`
###10. 配置防火墙
对于测试环境，往往需要本地连接测试环境数据库，需要配置防火墙，允许接收 3306 端口的连接请求。生产环境为了安全起见，不建议这么做。

添加端口：`firewall-cmd --zone=public --permanent --add-port=3306/tcp`
重启防火墙：`firewall-cmd --reload`
###11. 允许远程 root 用户登录
现在的数据库，远程登录时会报错“Host 'XX.XX.XX.XX' is not allowed to connect to this MySQL server”。
对于测试环境，往往需要本地连测试环境接数据库，需要配置MySQL，允许远程 root 用户登录。生产环境为了安全起见，不建议这么做。
登录  MySQL 服务器后，执行下面的命令，允许所有主机的所有用户远程登录：
```
GRANT ALL ON *.* to root@'%' IDENTIFIED BY '你的MySQL密码';
FLUSH PRIVILEGES;
```
其中，`*.*` 表示 `数据库名.表名`，星号表示所有数据库的所有表。`root@'%'` 表示 `用户名.允许连接的主机IP`，百分号表示允许所有远程主机连接。`FLUSH PRIVILEGES;` 表示刷新所有权限，替代了重启数据库这一步。
###12. 设置时区
[官方参考资料](https://dev.mysql.com/doc/refman/5.5/en/time-zone-support.html)
MySQL服务器维护 3 个时区设置：
#####1. 系统时区 `system_time_zone`。
MySQL 服务器启动时，会尝试确定 Linux 主机的时区，并用它来设置 `system_time_zone` 系统变量。

更改系统时区的 2 种方式：

- 在启动时使用 `mysqld_safe` 选项来设置MySQL服务器的系统时区 。
- 在启动 `mysqld` 之前，配置 `my.cnf` 文件，通过设置环境变量来设置它。
#####2. 服务器的当前时区 `time_zone`。
全局 `time_zone` 系统变量指示 MySQL 服务器当前正操作的时区。`time_zone` 的初始值是 `SYSTEM`，表示 MySQL 服务器的时区与系统时区 `system_time_zone` 一致。
```
注意：如果设置为 `SYSTEM`，则每个需要时区计算的MySQL函数调用都会调用系统库来确定当前的系统时区。这个调用可能受全局互斥的保护，导致争用。
```
初始全局服务器时区值可以在启动时通过命令行上的选项 ` --default-time-zone=timezone` 显式指定 ，也可以在 [mysqld] 之下加 `default-time-zone=timezone` 来修改时区。如：`default-time-zone = '+8:00'`，'+8:00' 表示与UTC的偏移量，就是中国时间。
如果您有SUPER 权限，则可以使用以下语句在登录 MySQL 后设置全局服务器时区值：
`mysql> SET GLOBAL time_zone = '+8:00';`
#####3. 每个连接时区。
连接的每个客户端都有自己的时区设置，由会话级 `time_zone` 变量给出 。会话级 `time_zone` 变量从全局 `time_zone` 系统变量获得初始值，但客户端可以使用以下语句更改自己的时区：
`mysql> SET time_zone = timezone;`

查看安装好的 MySQL 的时区。我的 CentOS 安装好后使用 CST 中国标准时间。
```
mysql> show variables like '%time_zone%';
+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone | CST    |
| time_zone        | SYSTEM |
+------------------+--------+
2 rows in set (0.00 sec)
```
这些数据表示，MySQL 用的是中国时间 CST。