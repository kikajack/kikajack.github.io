[官方文档](http://php.net/manual/zh/features.commandline.webserver.php)
##1. 简介
PHP 5.4.0起， CLI SAPI 提供了一个内置的Web服务器，可用于本地开发使用。
##2. 使用
###1）参数：
**-S**：启动 PHP 自带的 Web Server，后面跟网址及监听的端口号。默认会把 URI 请求发送到 PHP 所在的的工作目录（Working Directory）进行处理。
```
$ php -S localhost:8080
```
**-t**：自定义网站的根目录。
```
$ php -S localhost:8080 -t /your/project/path
```
**路由（Router）脚本**：在命令行启动 Web Server 时，如果指定了一个PHP文件，则这个文件会作为一个“路由”脚本，每次请求都会先执行这个脚本。如果这个脚本返回 FALSE ，那么直接返回请求的文件（例如请求静态文件不作任何处理）。否则会把输出返回到浏览器。如果请求未指定执行哪个PHP文件，则默认执行目录内的 index.php 或者 index.html。如果这两个文件都不存在，服务器会返回404错误。
```
$ php -S localhost:8080 -t /your/project/path  router.php
```
**-c**：指定 PHP 配置文件。
```
$ php -S localhost:8000 -c /your/php/path/php.ini
```
**-d**：指定 PHP 配置文件中的某一项的配置。
```
$ php -S localhost:8000 -doutput_buffering=32 //输出缓存设置为32字节
```
##2. PHP 自带 Web Server 的特点
1. 获取到的 sapi_name 是 cli-server。**php_sapi_name() === 'cli-server'**
2. 不支持.htaccess文件。
