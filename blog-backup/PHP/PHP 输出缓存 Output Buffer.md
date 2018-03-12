参考：
http://gywbd.github.io/posts/2015/1/php-output-buffer-in-deep.html
http://php.net/manual/zh/book.outcontrol.php

当 PHP 脚本有输出时，输出控制函数可以缓存输出（包括消息头和消息体），而不是立刻输出。
这在多种不同情况中非常有用，尤其是用来在脚本开始输出数据后，发送http头信息到浏览器。输出控制函数不影响由 header() 或 setcookie()发送的文件头信息，仅影响象 echo 这样的函数和PHP代码块间的数据。
#1. 缓冲区意义及用途
###1）消息头和消息体
**任何协议都必须在发送消息体之前发送消息头。**
如果使用输出缓冲区层，任何跟消息头的输出有关的PHP函数（header()，setcookie()，session_start()）都使用了内部的sapi_header_op()函数，把内容写入到消息头缓冲区中。输出内容时，例如使用printf()，这些内容会写入到输出缓冲区。发送这个输出缓冲区中的内容时，PHP会先发送消息头，再发送消息体。
###2）CLI 和 CGI 模式的区别
1. CLI 模式下，PHP 脚本直接输出到 STDOUT （打印到计算机屏幕，写入磁盘等），不需要 PHP 的缓冲区。
2. CGI 模式下，PHP 脚本的输出会通过网络传输到用户浏览器，通过缓冲区可以减少网络传输次数，提高性能。
###3）输出缓冲区用途
1. 如果需要将准备输出到浏览器的数据一直保存在服务器端，直到确认后才发送，会造成比较大的资源开销。用输出缓冲来节省资源。
2.  header() 发送的信息必须在输出流的最前端。如果需要先用 echo 之类的输出函数，再用 header，就需要用到输出缓存。
###4）PHP 中的输出缓冲区
####1. 默认 PHP 输出缓冲区
如果想使用默认 PHP 输出缓冲区层的话，你不能使用 CLI，因为 CLI 禁用了这个层。
如果使用的 SAPI（一般是 PHP-FPM）是 CGI 模式，在 php.ini 配置文件中，可以设置3个选项：

- output_buffering
- implicit_flush
- output_handler

**输出缓存控制的这3个选项，必须通过编辑 php.ini 配置文件或者是在执行 PHP 程序的时候使用-d选项（`php -doutput_buffering=32 -dimplicit_flush=1 -S127.0.0.1:8080 -t/var/www`）才能改变它们的值。**

output_buffering 选项设置为 On 时，将在所有的脚本中使用输出控制，输出缓冲区的大小将是16kB。也可以将该选项设定为指定的最大字节数（例如 output_buffering=4096，表示可以先写入4096个字节，然后再通过 CLI 或 CGI 跟下面的 SAPI 层通信）。从PHP 4.3.5 版开始，该选项在 PHP-CLI 下总是为 Off。output_buffering 的默认设置是 4096kB。如果不使用任何php.ini文件（或者也不会在启动PHP的时候使用-d选项），它的默认值将为0，禁用输出缓冲区。

implicit_flush 选项默认为 FALSE。如将该选项改为 TRUE，PHP 将使输出层，在每段信息块输出后，自动刷新。这等同于在每次使用 print、echo 等函数或每个 HTML 块之后，调用 PHP 中的 flush() 函数。
不在web环境中使用 PHP 时，打开这个选项对程序执行的性能有严重的影响，通常只推荐在调试时使用。在 CLI SAPI 的执行模式下，该标记默认为 TRUE。

output_handler是一个回调函数，它可以在缓冲区刷新之前修改缓冲区中的内容。只有内置函数可以使用此指令。对于用户定义的函数，使用 ob_start()。
####2. 用户输出缓冲区
调用 ob_start() 可以创建用户输出缓冲区。用户输出缓冲区可以嵌套，每个新建缓冲区都会堆叠到之前的缓冲区上，每当新缓冲区被填满或者溢出，都会执行刷新操作，然后把其中的数据传递给下一个缓冲区。
```
<?php

ob_start();
echo "Hello\n"; // 一直被保存在输出缓冲区中直到调用 ob_end_flush() 
setcookie("cookiename", "cookiedata"); // 对setcookie()的调用也成功存储了一个cookie，而不会引起错误。（正常情况下，在数据被发送到浏览器后，就不能再发送http头信息了。）
ob_end_flush();

?>
```
#2. Output Control 输出控制函数
参考：http://php.net/manual/zh/ref.outcontrol.php
##1）ob_start：打开输出控制缓冲
`bool ob_start ([ callback $output_callback [, int $chunk_size [, bool $erase ]]] )`
当输出缓冲激活后，脚本将不会输出内容（除http标头外），需要输出的内容被存储在内部缓冲区中。
参数：
callback：回调函数，把一个输出缓冲区内容字符串当作参数，并返回一个字符串送到浏览器。 当输出缓冲区被( ob_flush(), ob_clean() 或者相似的函数)冲刷（送出）或者被清洗的时候；或者在请求结束之际输出缓冲区内容被冲刷到浏览器的时候该函数将会被调用。 当调用 output_callback 时，它将收到输出缓冲区的内容作为参数 并预期返回一个新的输出缓冲区作为结果，这个新返回的输出缓冲区内容将被送到浏览器。 
```
<?php

function callback($buffer)
{
  // replace all the apples with oranges
  return (str_replace("apples", "oranges", $buffer));
}

ob_start("callback");

?>
<html>
<body>
<p>It's like comparing apples to oranges.</p>
</body>
</html>
<?php

ob_end_flush();

?>
以上例程会输出：

<html>
<body>
<p>It's like comparing oranges to oranges.</p>
</body>
</html>
```
##2）ob_clean：丢弃输出缓冲区中的内容
不会销毁输出缓冲区。
##3）ob_flush：冲刷出（送出）输出缓冲区中的内容
在调用ob_flush()之后缓冲区内容将被丢弃。
##4）ob_get_contents：返回输出缓冲区的内容
只是得到输出缓冲区的内容，但不清除它。
##5）ob_end_clean：清空（擦除）缓冲区并关闭输出缓冲
丢弃最顶层输出缓冲区的内容并关闭这个缓冲区。
##6）ob_end_flush：冲刷出（送出）输出缓冲区内容并关闭缓冲
送出最顶层缓冲区的内容（如果里边有内容的话），并关闭缓冲区。
##7）ob_get_clean： 得到当前缓冲区的内容并删除当前输出缓冲
实质上是一起执行了 ob_get_contents() 和 ob_end_clean()。