##1.基本知识
####1.MySQL的配置文件，调试方式
在Windows中，名为my.ini，而在Linux中则为my.cnf，路径默认为/etc/my.cnf。可以用`#locate my.cnf `命令找出所有配置文件。
MySQL无法启动时，可以通过`#mysqld --console` 方式在启动时显示详细信息。所有的异常会列出。
####2.启动，停止，重启（默认情况下）
  #service mysqld start
  #service mysqld stop
  #service mysqld restart
  #/etc/init.d/mysqld start
####3.mysqld与mysql
mysqld是用来启动mysql数据库的命令
mysql是打开并执行sql语句的命令
这两个都在mysql安装文件夹的bin目录下
####4.MySQL四种启动方式
1. mysqld
启动mysql服务器:./mysqld --defaults-file=/etc/my.cnf --user=root
客户端连接:
mysql --defaults-file=/etc/my.cnf
or
mysql -S /tmp/mysql.sock

2. mysqld_safe
启动mysql服务器:./mysqld_safe --defaults-file=/etc/my.cnf --user=root &
客户端连接:
mysql --defaults-file=/etc/my.cnf
or
mysql -S /tm/mysql.sock

3. mysql.server
cp -v /usr/local/mysql/support-files/mysql.server /etc/init.d/
chkconfig --add mysql.server
启动mysql服务器:service mysql.server {start|stop|restart|reload|force-reload|status}
客户端连接:同1、2

4. mysqld_multi
mkdir $MYSQL_BASE/data2
cat <<-EOF>> /etc/my.cnf
[mysqld_multi]
mysqld  = /usr/local/mysql/bin/mysqld_safe
mysqladmin = /user/local/mysql/bin/mysqladmin
user = mysqladmin
password = mysqladmin
[mysqld3306]
port            = 3306
socket          = /tmp/mysql3306.sock
pid-file  = /tmp/mysql3306.pid
skip-external-locking
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
basedir   = /usr/local/mysql
datadir   = /usr/local/mysql/data
[mysqld3307]
port            = 3307
socket          = /tmp/mysql3307.sock
pid-file  = /tmp/mysql3307.pid
skip-external-locking
key_buffer_size = 16M
max_allowed_packet = 1M
table_open_cache = 64
sort_buffer_size = 512K
net_buffer_length = 8K
read_buffer_size = 256K
read_rnd_buffer_size = 512K
myisam_sort_buffer_size = 8M
basedir   = /usr/local/mysql
datadir   = /usr/local/mysql/data2
EOF

`#mysql -S /tmp/mysql3306.sock
mysql>GRANT SHUTDOWN ON *.* TO 'mysqladmin'@'localhost' identified by 'mysqladmin' with grant option;`

`#mysql -S /tmp/mysql3307.sock
mysql>GRANT SHUTDOWN ON *.* TO 'mysqladmin'@'localhost' identified by 'mysqladmin' with grant option;`

启动mysql服务器:./mysqld_multi --defaults-file=/etc/my.cnf start 3306-3307
关闭mysql服务器:mysqladmin shutdown
  

##2.异常记录

####1.`[Warning] TIMESTAMP with implicit DEFAULT value is deprecated.Please use --explicit_defaults_for_timestamp server option (see documentation for more details).`
原因：此问题出现在MySQL5.6之后。从 5.6开始，timestamp 的默认行为是 deprecated 了，在MySQL启动时会出现以此警告。需要在启动服务前开启系统变量：

  #[mysqld]
  explicit_defaults_for_timestamp=true
在MySQL 5.6.6之前，TIMESTAMP的默认行为是比较诡异的，会造成一些隐含的问题：

1. TIMESTAMP字段未被显式声明为not null时，将会允许null值。设置为null就是null，而不是当前TIMESTAMP。

2. TIMESTAMP字段不会被自动赋值为DEFAULT CURRENT_TIMESTAMP 和 ON UPDATE CURRENT_TIMESTAMP属性。此时要显式去赋值。

3. TIMESTAMP字段被声明为not null并且没有显式声明一个默认值将被认定为没有默认值。向这样的字段插入记录将完全看SQL模式的心情。如果是strict严格模式，就报错；
如果不是，将被赋值为'00-00-00 00:00:00'并给一个警告。（这和MySQL处理其他时间类型数据一样，如DATETIME）。