判断设备类型的地方有很多，比如在服务端通过 Nginx 或 Apache 等判断，在项目中通过 UserAgent 判断。[这个网站上有各种开源的解决方案](http://detectmobilebrowsers.com/)
现在的移动设备类型比较一致了，只需要判断安卓、苹果两种类型即可。
#1. 通过 JavaScript 判断设备类型
对于静态页面构成的项目（比如 GitHub 的个人首页，只能上传静态页面），如果需要判断设备类型，那就得在终端设备上通过 JavaScript 实现。

###Navigator 对象 [详细文档参考这里](https://developer.mozilla.org/zh-CN/docs/Web/API/Navigator)：
Navigator 对象表示浏览器的状态和标识，允许 JavaScript 脚本查询其中的信息。其中的 userAgent 属性是一个声明了浏览器用于 HTTP 请求的用户代理头的字符串。下图是 Chrome 浏览器的 console 打印出来的信息：
![这里写图片描述](http://img.blog.csdn.net/20180128165152936?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
### 对比 UserAgent
**可以通过判断 navigator.userAgent 里面是否有某些值来判断设备类型**。比如苹果手机有 `iPhone` 字段，Windows 系统的 PC 有 `Windows` 字段。

- PC 版的 UserAgent 属性：`userAgent:"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/63.0.3239.132 Safari/537.36"`
- 模拟手机版的 UserAgent 属性：`serAgent:"Mozilla/5.0 (iPhone; CPU iPhone OS 9_1 like Mac OS X) AppleWebKit/601.1.46 (KHTML, like Gecko) Version/9.0 Mobile/13B143 Safari/601.1"`

### 具体代码
通过正则表达式，可以把各种想识别的移动设备用 UserAgent 匹配出来。下面的例子（正则的 i 表示忽略大小写）匹配了安卓设备、苹果手机和平板、黑莓设备，如果终端确实是这些设备中的一种，则 is_pc 变量为 true，否则为 false。
```
var is_mobile = /Android|iPhone|iPad|iPod|BlackBerry/i.test(navigator.userAgent);
```
也可以利用字符串的 indexOf 方法去判断。
#2. 通过 Nginx 判断设备类型
如果不同版本的项目是不同的团队做的，那在服务器进行跳转就很合适了。
下面的例子中，在 Nginx 配置文件中增加一个变量 `$is_mobile` 标志设备类型，通过正则表达式匹配 UserAgent 实现设备类型的判断。
```
server {    
    listen 80;  
    set $is_mobile no; # 默认电脑版
      
    if ($http_user_agent ~* '(Android|iPhone|iPod|iPad)') {
        set $is_mobile yes;
    }
    
    location / {
        if ($is_mobile = yes) {
          root /html/mobile;
        } else {
          root /html/mobile;
    }
    }  
}  
```
#3. 项目代码判断设备类型（以 PHP 为例）
PHP 通过 `$_SERVER` 储存各类客户设备的信息。其中的 `$_SERVER['HTTP_USER_AGENT']` 是客户设备的 UserAgent。
##3.1 通过 UserAgent 判断
PHP 中通过 UserAgent 判断设备类型的方法和 JavaScript 类似，可以用正则表达式简单实现，设备类型匹配到后 `$is_mobile` 变量为 true：
```
$is_mobile = preg_match("/Android|iPhone|iPad|iPod|BlackBerry/i", $userAgent);
```
##3.2 通过 HTTP_X_WAP_PROFILE 判断
如果 `$_SERVER` 中有 HTTP_X_WAP_PROFILE 则一定是移动设备。
```
$is_mobile = isset ($_SERVER['HTTP_X_WAP_PROFILE']);
```
##3.3 生产环境中往往综合以上方法来判断
```
function isMobile() { 
    // 如果有 HTTP_X_WAP_PROFILE 则一定是移动设备
    if (isset ($_SERVER['HTTP_X_WAP_PROFILE'])) {
        return true;
    }
    // 判断手机发送的客户端标志 UserAgent
    if (isset ($_SERVER['HTTP_USER_AGENT'])) {
        return preg_match("/(" . implode('|', $clientkeywords) . ")/i", $_SERVER['HTTP_USER_AGENT'])) ? true : false;
    }
} 
```
#4. 判断是否在微信中
微信的 UserAgent 中有一个标志 `MicroMessenger`，通过这个标识可以识别微信。

PHP 示例，在微信中时 `$is_wx` 为 true：
```
$is_wx = preg_match("/MicroMessenger/i", $_SERVER['HTTP_USER_AGENT']);
```
JavaScript 示例，在微信中时 `is_wx` 为 true：
```
var is_wx = /MicroMessenger/i.test(navigator.userAgent);
```