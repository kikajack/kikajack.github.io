参考：
http://www.awaimai.com/371.html
http://www.php-internals.com/book/?p=chapt02/02-02-03-fastcgi

##0. 总结
Web Server只是内容分发者。对于静态资源，Web Server在文件系统找到资源后返回给浏览器。而对于动态请求（XX.php等），Web Server需要在配置文件中寻找对应的解析引擎，然后把请求对应的文件交给解析引擎去解析，得到解析结果后返回给浏览器。CGI协议或Fast-CGI协议，就是Web Serve和解析引擎之间数据交互的协议。
##1.CGI协议
CGI:通用网关接口(Common Gateway Interface)，它是一个**协议**。CGI是Web应用程序（PHP,Java等）与Web服务器（Nginx,Aphche等）之间的接口标准。
CGI应用程序可以由多种编程语言编写，如Perl、C\C++、Shell等。
对于每一个Web请求，CGI都要创建一个进程，然后解析配置文件，初始化执行环境并执行。因为性能低下，基本上不用了。

**特点：**
- 采用fork-and-execute模式，为每个请求创建进程，性能低下。
- 功能十分有限：CGI只能收到一个请求，输出一个响应。很难在CGI体系去对Web请求的控制，例如：用户认证等。
##2.Fast-CGI协议
Fast-CGI:通用网关接口(Common Gateway Interface)，它是一个**协议**。FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，将CGI解释器进程保持在内存中，用TCP或Socket方式跟远程主机或本地的进程建立连接。
Fast-CGI会先创建一个主进程（master），解析配置文件（php.ini等），初始化执行环境，然后再创建多个工作进程（worker）。请求由master接收，转给worker处理，处理完后进程仍然保留，可以接受下一个请求。
常用的Fast-CGI程序有2类：
- Web Server内置：微软IIS的 ISAPI，Apache的php5_module模块。
- 独立程序或集成于编程语言：php-fpm，spawn-fcgi。
工作流程：
1. Web Server启动。注意WebServer的配置文件中需指明Fast-CGI的调用方式，比如Nginx的配置中需要这一行`fastcgi_pass   127.0.0.1:9000;`。
2. Fast-CGI程序启动（比如php-fpm）并初始化，启动多个CGI解释器进程(多个php-cgi)，等待来自Web Server的连接。
3. 当客户端请求到达Web Server时，Web Server反手就把请求发给Fast-CGI进程管理器，由Fast-CGI进程管理器选择并连接一个CGI解释器（Fast-CGI子进程）。
4. Fast-CGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当Fast-CGI子进程关闭连接时，请求便处理完成。Fast-CGI子进程接着等待下一个连接。 

**特点：**
- 语言无关性，只要能进行标准输出（stdout）的语言都可以作为Fast-CGI标准的Web后端。
- 服务器架构无关性。
- 多进程，常驻内存，进程数可以动态调整。
- 隔离性：Fast-CGI后端和Web Server运行在不同的进程中，通过Socket进行通信，后端的任何故障不会导致Web Server挂掉。
##3. CLI
CLI:命令行接口(Command Line Interface)
在Linux下的终端输入`php 参数`，即可执行php命令或脚本。比如`php -m`查看PHP安装了那些扩展。
##4. php-cgi
php-cgi是解释PHP脚本的**程序**。
php-cgi是PHP官方的FastCGI 实现，但没有进程管理。解析脚本时，启动php-cgi进程即可。
在CGI 模式时，当Web Server 收到动态请求时，会启动php-cgi进程，php-cgi进程会解析php.ini 文件，初始化环境并执行，最后返回结果，结束该php-cgi进程。

特点：
- 修改了php.ini配置文件后，无法平滑重启，只能重启php-cgi才能生效。
- 没有进程管理功能。
- 每个php-cgi进程大约占用20M内存（7-25MB）。
##5. php-fpm
php-fpm（FastCGI Process Manager）是实现了Fast-CGI协议的**程序**。参与进程管理，可以调度php-cgi进程。
php-fpm包含了php-cgi，附带进程管理，以C/S方式运行。
在PHP内核中集成了php-fpm，源码安装时使用--enalbe-fpm这个编译参数就可以安装。
**php-fpm进程启动后，解析php.ini配置文件，然后启动多个工作进程（worker），每个工作进程内部都嵌入了一个 PHP 解释器，PHP 解释器是 PHP 代码真正执行的地方。**修改php.ini配置文件后，需要重启php-fpm才可生效，此时已经存在的worker会继续处理完手上的活。
php-fpm进程自带了Fast-CGI Server，监听9000端口。
**PHP-fpm仅仅是个进程管理器, 它仍会调用PHP解释器来处理请求。**
PHP解释器(在Windows下)就是php-cgi.exe。

特点：
- php-fpm的master主进程常驻内存。worker工作进程数量可以动态调整。
- 修改了php.ini配置文件后，可以平滑重启，后面启动的worker会用新的配置，已经存在的worker处理完请求后结束进程。
- 有进程管理功能。
- 需要PHP解释器php-cgi的配合来处理请求。