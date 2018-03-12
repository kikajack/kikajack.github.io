###1. CGI 模式
在服务器配置信息中，增加参数字段。例如，Nginx 中配置如下：
```
# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000

location ~ \.php(.*)$  {
   fastcgi_pass   127.0.0.1:9000;
   fastcgi_index  index.php;
   fastcgi_split_path_info  ^((?U).+\.php)(/?.+)$;
   fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
   fastcgi_param  PATH_INFO  $fastcgi_path_info;
   fastcgi_param  PATH_TRANSLATED  $document_root$fastcgi_path_info;
   fastcgi_param  CI_ENV  'development'; // 这里会传参数 $_SERVER['CI_ENV']='development'
   include        fastcgi_params;
}
```
###2. CLI 模式
在 CLI 模式下，通过命令行可以执行 PHP 脚本。
```
php -f test.php  //执行脚本文件
php -r 'echo 123' //在命令行直接运行 PHP 代码
```
**把要传递的参数写在调用 PHP 命令的语句最前面**，就可以传参数给 PHP 脚本或语句了。在脚本中用 `$_SERVER['param name']` 形式调用即可。

```
脚本文件cli-param -test

<?php

echo $_SERVER['ENVIRONMENT'];
```
```
# ENVIRONMENT="production" php cli-param-test
// 屏幕打印内容如下
production
```