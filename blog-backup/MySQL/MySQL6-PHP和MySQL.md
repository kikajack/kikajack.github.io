#1.PHP的MySQL扩展
参考：http://php.net/manual/zh/mysqli.overview.php
PHP本身不能连接数据库，需要通过扩展才可以连。常用的MySQL数据库扩展有3种：
- mysql扩展（不建议）：纯面向过程。老旧，已不再开发只维护。
- mysqli扩展（首选）：部分面向对象（推荐），部分面向过程。支持事务，支持多语句执行，支持prepared语句。
- PDO扩展（第二选）：PHP Database Object，是PHP应用中的一个数据库抽象层规范。PDO提供了一个统一的API接口可以使得你的PHP应用不去关心具体要连接的数据库类型，可以在任何需要的时候无缝切换数据库服务器。类似Java应用中的JDBC。**缺点是会限制让你不能使用MySQL服务端提供的数据库高级特性，如多语句执行。**
#2.连接MySQL（用mysqli）
##1.面向过程风格（不推荐）
基本上所有函数以‘mysqli_’开头。
###1.连接数据库：mysqli_connect()函数
语法：`mysqli_connect(host,username,password,dbname,port,socket);`
返回值：  返回一个代表到 MySQL 服务器的连接的对象。
例如：`$mysqli=mysqli_connect("localhost","my_user","my_password","my_db");`
##2.面向对象风格
常用的类有：mysqli类（代表PHP和Mysql数据库之间的一个连接），mysqli_result类（代表从一个数据库查询中获取的结果集），mysqli_stmt类（代表一个prepared语句），mysqli_driver类，mysqli_warning类。
基本流程：连接数据库得到mysqli对象---->组装SQL语句得到mysqli_stmt对象----->执行SQL语句得到mysqli_result对象------>取记录或字段。
###1.连接数据库：mysqli::construct()
语法：mysqli::__construct (host,username,password,dbname,port,socket)
所有参数都可以不填，用php.ini文件中的配置`ini_get('mysqli.default_host')`等等（不建议）。host前加“p:”可以产生持久连接。从连接池中打开连接时会自动调用 mysqli_change_user() 方法。
示例：`$mysqli = new mysqli("localhost", "my_user", "my_password", "my_db");`
###2.判断连接是否成功：mysqli_connect_error()函数（5.3版本之前）或connect_error()方法（5.3版本后才可以用）
报错信息：mysqli_connect_error()或connect_error()
报错序号：mysqli_connect_errno()或mysqli_connect_errno()，5.3之后才可以用  
```php
if (mysqli_connect_error()) {//5.3之前用
  die('Connect Error (' . mysqli_connect_errno() . ') '
      . mysqli_connect_error());
}
if ($mysqli->connect_error) {//connect_errorno属性5.3之后才可以用 
  die('Connect error'.$mysqli->connect_errorno.$mysqli->connect_error);
}
```
###3.设置字符集：set_charset()方法
$mysqli->set_charset('utf8');
###4.将字符串转为合法的SQL语句：real_escape_string ()方法
语法：
`string mysqli::real_escape_string ( string $escapestr )`
使用前必须先成功连接数据库且设置好字符编码（set_charset()）。
将会被编码的字符是：NUL (ASCII 0), \n, \r, \, ', ", 和 Control-Z。

###5.操作数据库：
####1.prepare()和bind_param()，bind_result()
可以有效防止SQL注入，推荐。
- `mysqli_stmt mysqli::prepare ( string $query )`
处理SQL语句得到mysqli_stmt对象，只能写一条SQL语句且不加分号或‘\g’结束。
例如：`$stmt = $mysqli->prepare("INSERT INTO CountryLanguage VALUES (?, ?, ?, ?)");`
- `bool mysqli_stmt::bind_param ( string $types , mixed &$var1 [, mixed &$... ] )`
绑定参数到prepare()方法的SQL语句中预留的问号占位符。可以用四种类型：`$types="i"整型，$types="s"字符串，$types="d"浮点型，$types="b"blob型。`
例如：`$stmt->bind_param('sssd', $code, $language, $official, $percent);`
- `bool mysqli_stmt::execute ( void )`
执行SQL语句。
- `bool mysqli_stmt::bind_result ( mixed &$var1 [, mixed &$... ] )`
将执行SQL语句得到的结果字段按顺序放入参数列表。
例如：
```php
$stmt = $mysqli->prepare("SELECT Code, Name FROM Country ORDER BY Name LIMIT 5")) {
$stmt->execute();
$stmt->bind_result($Code, $Name);
```
- `mysqli_result mysqli_stmt::get_result ( void )`
取SQL语句执行后的结果集（可以是多条记录），可以用fetch_assoc()在循环中逐条取出。
- `array mysqli_result::fetch_assoc ( void )`
从结果集中取一行。
例如：
```php
$results = $stmt->get_result();
while ($row = $results->fetch_assoc()) {
  print_r($row);
}
```
```php
//取记录中所有字段或不清楚记录中有几个字段时，用get_result()配合fetch_assoc()遍历。
$i = 1;
if ($stmt = $mysqli->prepare('SELECT * FROM student WHERE StudentID=?')) {
  $stmt->bind_param("i", $i);
  $stmt->execute(); 
  $results = $stmt->get_result();
  while ($myrow = $results->fetch_assoc()) {
    print_r($myrow);
  }
  $stmt->close();
}
$mysqli->close();
```
```php
//取记录中的指定字段时，用bind_result(字段名1[，字段名2....])
$i = 1;
if ($stmt = $mysqli->prepare('SELECT * FROM student WHERE StudentID=?')) {
  $stmt->bind_param("i", $i);
  $stmt->execute();
  $stmt->bind_result($ID, $Name);
  $stmt->fetch();
  $stmt->close();
  echo $ID,$Name;
}
$mysqli->close();
```
####2.query()和multi_query()方法
**难以解决SQL注入问题，建议prepare()和bind_param()执行SQL语句，或改用同样有Prepared Statement机制的PDO。**
query()：执行单条SQL语句对数据库执行一次查询。
multi_query()：执行多条SQL语句，用分号分隔。需配合next_result()方法循环遍历。
语法：`mixed mysqli::query ( string $query [, int $resultmode = `MYSQLI_STORE_RESULT ] )
参数：
`$query`：SQL语句，必须被转义为合法的语句（string mysqli::escape_string ( string $escapestr )）。
`$resultmode`：返回值类型，默认是MYSQLI_STORE_RESULT ，也可以用MYSQLI_USE_RESULT 。区别是MYSQLI_STORE_RESULT模式下自动保存查询结果，可以立刻再次执行SQL命令。但MYSQLI_USE_RESULT模式下的查询结果必须全部读取完后（mysqli::fetch_all()）或主动释放结果集所占空间（mysqli_result::free(void)或mysqli_result::free_result(void)或mysqli_result::close(void)）才能进行下一次SQL操作。
返回值：
失败时返回 FALSE，通过mysqli_query() 成功执行SELECT, SHOW, DESCRIBE或 EXPLAIN查询会返回一个mysqli_result 对象，其他查询则返回TRUE。
`$result = $mysqli->query("SELECT Name FROM City LIMIT 10");`
`$result = $mysqli->fetch_all();`
###6.切换用户或数据库：change_user()方法
语法：`bool mysqli::change_user (string $user, string $password, string $database)`
相当于重新连接当前主机。
###7.连接或改变已连接的数据库：select_db()方法
`bool mysqli::select_db ( string $dbname )`
实例化mysqli对象时如果没指定数据库，可以用select_db()方法指定。select_db()也可切换数据库。
###8.关闭连接：close()方法
`$mysqli->close();`
