mcrypt 已经废弃，对称/非对称加解密统一在 OpenSSL 中实现。

#1. 安装
phpize 命令是用来准备 PHP 扩展库的编译环境的。[详情参考这里](http://php.net/manual/zh/install.pecl.phpize.php)，简单步骤如下：
```
$ cd extname
$ phpize
$ ./configure
$ make && make install
```
make install 安装成功后将创建 extname.so 库（类似于 Windows 下的 dll 文件）并放置于 PHP 的扩展库目录中。需要在 php.ini 配置文件中加入 extension=extname.so 这一行之后才能使用此扩展库。
如果系统中没有 phpize 命令并且使用 yum 或 apt 等方式安装 PHP，那要安装 PHP 包相应的开发版本，此版本通常包含了 phpize 命令以及相应的用于编译 PHP 及其扩展库的头文件。

##1.1 Linux
查看 PHP 扩展的安装目录用这个命令：`php -i | grep extension_dir`：
```
[root@VM_120_242_centos ~]# php -i | grep extension_dir
extension_dir => /usr/lib64/php/modules => /usr/lib64/php/modules
sqlite3.extension_dir => no value => no value
```
查看 PHP 已经安装的扩展用这个命令：`php -m`：
```
[root@VM_120_242_centos openssl]# php -m
[PHP Modules]
bz2
calendar
Core
curl
date
fileinfo
filter
gd
hash
iconv
json
libxml
mcrypt
mysqli
openssl
pcntl
pcre
PDO
pdo_mysql
Phar
readline
Reflection
session
SimpleXML
sockets
SPL
standard
zip
zlib
```
CentOS7 通过 yum 命令安装后的扩展模块位置及详情：
```
[root@VM_120_242_centos ~]# cd /usr/lib64/php/modules/
[root@VM_120_242_centos modules]# ll
总用量 5972
-rwxr-xr-x 1 root root   24696 6月  10 2017 bz2.so
-rwxr-xr-x 1 root root   33808 6月  10 2017 calendar.so
-rwxr-xr-x 1 root root   82808 6月  10 2017 curl.so
-rwxr-xr-x 1 root root 3164288 6月  10 2017 fileinfo.so
-rwxr-xr-x 1 root root  378328 6月  10 2017 gd.so
-rwxr-xr-x 1 root root   70360 6月  10 2017 gmp.so
-rwxr-xr-x 1 root root   45112 6月  10 2017 iconv.so
-rwxr-xr-x 1 root root   40688 6月  10 2017 json.so
-rwxr-xr-x 1 root root   45232 6月  10 2017 mcrypt.so
-rwxr-xr-x 1 root root  133208 6月  10 2017 mysqlnd_mysqli.so
-rwxr-xr-x 1 root root  282000 6月  10 2017 mysqlnd.so
-rwxr-xr-x 1 root root   28792 6月  10 2017 pdo_mysqlnd.so
-rwxr-xr-x 1 root root  112368 6月  10 2017 pdo.so
-rwxr-xr-x 1 root root  272152 6月  10 2017 phar.so
-rwxr-xr-x 1 root root   58368 6月  10 2017 simplexml.so
-rwxr-xr-x 1 root root   87392 6月  10 2017 sockets.so
-rwxr-xr-x 1 root root   32904 6月  10 2017 xmlreader.so
-rwxr-xr-x 1 root root   54168 6月  10 2017 xml.so
-rwxr-xr-x 1 root root   49160 6月  10 2017 xmlwriter.so
-rwxr-xr-x 1 root root  138704 6月  10 2017 zip.so
```
###1.1.1 编译安装
#### 编译的同时安装 OpenSSL 模块
如果本地编译安装 PHP7，可以通过参数 `--with-openssl[=DIR]` 安装 OpenSSL 模块，例如 `--with-openssl=/usr/local`。
####单独安装 OpenSSL 模块
安装后可以通过单独编译 openssl.so 模块来安装。注意，编译安装和其他方式（yum，apt 等）后的目录结构不一样，需要通过 `whereis php` 命令自行判断。**PHP 源码包括了 OpenSSL，可以通过源码创建 openssl.so 文件。**
如果 PHP 源码的下载目录是 `/data/php-7.0.12`，那么 OpenSSL 的目录为 `/data/php-7.0.12/ext/openssl`，进入  OpenSSL 的目录后，执行以下操作：
```
mv config0.m4 config.m4 # 只识别 config.m4 文件，需要重命名
/data/php/bin/phpize # 初始化 PHP 扩展库的编译环境
./configure --with-openssl --with-php-config=/data/php-7.0.12/bin/php-config # 配置
make && make install # 编译
```
安装成功会生成一个包含 openssl.so 文件的目录，把 openssl.so 文件拷贝到 php.ini 中指定的 extension_dir 目录下，修改 `php.ini` 配置文件，设置**扩展组件的存放目录**和具体的**扩展组件**即可。

如果执行 `./configure` 时报错 `Cannot find OpenSSL...`，则说明 Linux 上没有安装 OpenSSL，需要先安装（我的 CentOS 安装命令为：`yum install openssl openssl-devel`）。
```
extension_dir = "/usr/lib64/php/modules/"
extension=openssl.so
```
**安装任何扩展模块后，都需要重启服务器（Nginx 或 Apache）和 PHP（一般就是 php-fpm）。**
##1.2 Windows 
直接下载最新版本的 PHPStudy，集成了 OpenSSL。需要在 `php.ini` 配置文件中开启 OpenSSL 扩展：`extension=php_openssl.dll`。
#2. 常用函数
##openssl_encrypt [加密数据](http://php.net/manual/zh/function.openssl-encrypt.php)，对应 aes，des，3des，rsa 等算法
(PHP 5 >= 5.3.0, PHP 7)

`string openssl_encrypt ( string $data , string $method , string $key [, int $options = 0 [, string $iv = "" [, string &$tag = NULL [, string $aad = "" [, int $tag_length = 16 ]]]]] )`
以指定的方式和 key 加密数据，返回原始或 base64 编码后的字符串。

- 参数：
data：待加密的明文信息数据。
method：加密方式。通过函数 `openssl_get_cipher_methods()` 可获取有效的加密方式列表。常用的有：`AES-128-CBC`，`AES-128-ECB`，`DES-CBC`，`DES-ECB`，
key：密码，注意**这个密码经常会再进行一次哈希，而不是直接使用**。
options：以下标记的按位或： `OPENSSL_RAW_DATA` 原生数据，对应数字1，不进行 base64 编码。`OPENSSL_ZERO_PADDING` 数据进行 base64 编码再返回，对应数字0。
iv：非 NULL 的初始化向量。对于 ECB 模式可以省略，但 CBC 模式不可省略。
tag：使用 AEAD 密码模式（GCM 或 CCM）时传引用的验证标签。
aad：附加的验证数据。
tag_length：验证 tag 的长度。GCM 模式时，它的范围是 4 到 16。
- 返回值：
成功时返回加密后的字符串， 或者在失败时返回 FALSE。
## openssl_decrypt [解密数据](http://php.net/manual/zh/function.openssl-decrypt.php)
(PHP 5 >= 5.3.0, PHP 7)

`string openssl_decrypt ( string $data , string $method , string $key [, int $options = 0 [, string $iv = "" [, string $tag = "" [, string $aad = "" ]]]] )`
用指定的方法和密码，解密原始的或经过 base64 编码的字符串。

- 参数
data：The encrypted message to be decrypted.
method：The cipher method. For a list of available cipher methods, use openssl_get_cipher_methods().
key：The key.
options：options can be one of OPENSSL_RAW_DATA, OPENSSL_ZERO_PADDING.
iv：A non-NULL Initialization Vector.
tag：The authentication tag in AEAD cipher mode. If it is incorrect, the authentication fails and the function returns FALSE.
aad：Additional authentication data.
- 返回值
成功后返回解密后的字符串，在失败时返回 FALSE。
## openssl_digest [摘要，哈希](http://php.net/manual/zh/function.openssl-digest.php)，对应 md5，sha1 等算法
(PHP 5 >= 5.3.0, PHP 7)

`string openssl_digest ( string $data , string $method [, bool $raw_output = FALSE ] )`
通过指定的方式来计算给定数据的摘要哈希信息。返回原始数据或经过 binhex 编码的数据。

- 参数
data：待哈希的数据。
method：哈希算法。例如"sha256"。通过 `openssl_get_md_methods()`获取所有可用的哈希算法。
raw_output：Setting to TRUE will return as raw output data, otherwise the return value is binhex encoded.
- 返回值
成功时返回摘要哈希值，在失败时返回 FALSE。
#3. 常用的辅助函数
[ord 函数和 chr 函数参考](http://www.hishenyi.com/archives/178)
[php实现java的byte数组转换](http://heaven--18.iteye.com/blog/1129735)
[PHP对接java的AES/ECB/PKCS5Padding加密方式](http://blog.51cto.com/laok8/1909479)
## 返回字符串的 ASCII 码值（扩展了 ord 函数，可以处理 UTF8 编码的字符串）[参考文档](http://php.net/manual/zh/function.ord.php)
```
function ordutf8($string, &$offset) {
    $code = ord(substr($string, $offset,1)); 
    if ($code >= 128) {        //otherwise 0xxxxxxx
        if ($code < 224) $bytesnumber = 2;                //110xxxxx
        else if ($code < 240) $bytesnumber = 3;        //1110xxxx
        else if ($code < 248) $bytesnumber = 4;    //11110xxx
        $codetemp = $code - 192 - ($bytesnumber > 2 ? 32 : 0) - ($bytesnumber > 3 ? 16 : 0);
        for ($i = 2; $i <= $bytesnumber; $i++) {
            $offset ++;
            $code2 = ord(substr($string, $offset, 1)) - 128;        //10xxxxxx
            $codetemp = $codetemp*64 + $code2;
        }
        $code = $codetemp;
    }
    $offset += 1;
    if ($offset >= strlen($string)) $offset = -1;
    return $code;
}

// 使用
$offset = 0;
while ($offset >= 0) {
  echo $offset.": ".ordutf8($str, $offset)."\n";
}
```
## 获取字符串对应的的字节数组（对应 JAVA 的字节数组 byte array）
```
function getBytes($string) {  
    $bytes = array();  
    for ($i = 0; $i < strlen($string); $i++){    //遍历每一个字符，用 ord 函数转为 ASCII 码后，拼接成数组
      // Java 中的 byte 类型只有一个字节，存储范围 -128~127，ord 函数生成的 ASCII 码范围是0~255，需要处理
    if (ord($str[$i]) >= 128) {
      $byte = ord($str[$i]) - 256;
    } else {
      $byte = ord($str[$i]);
    }
    }
    $bytes[] = $byte;  
    return $bytes;  
}
```
## 获取 数字 对应的的字节数组（对应 JAVA 的字节数组 byte array）
Java 中强制类型转换时，其他类型的数据不管有多少个字节，只保留对应二进制数据的低 8 位（最后一个字节）。
```
// 只保留整型数据对应 32 bit 中的低 8 位
function intToByte($num) {
  $num = decbin($num);  // 把十进制数字转换为二进制
  $num = substr($num,-8); // 取后8位
  
     //第一位是 1 代表是负数，需要减去 256 
  if (substr($num, 0, 1) == 1) {
    return bindec($num) - 256; //bindec 也是php自带的函数，可以把二进制数转为十进制
  }
  return bindec($num);
}
```
## 将字符串的每个字节用对应 ASCII 码的十六进制表示
```
// 转为 16 进制表示，其中 ord 函数将字符转为对应的 ASCII 码，dechex 函数将数字从十进制转为十六进制
function strToHex($string)
{
    $hex = "";
    for ($i = 0; $i < strlen($string); $i++) {
        $tmp = dechex(ord($string[$i]));
        if (strlen($tmp) == 1) $tmp = '0'.$tmp; // 一个字节必须占 2 个十六进制位，不够则补 0
        $hex .= $tmp;
    }
    $hex = strtoupper($hex);
    return $hex;
}
```