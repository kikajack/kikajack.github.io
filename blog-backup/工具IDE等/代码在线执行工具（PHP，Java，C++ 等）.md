
# http://www.it1352.com/Onlinetools
支持**几十种语言**的在线运行。
缺点：对**请求频率限制太严格**了，一分钟不到十次吧。。。可以清理浏览器 Cookie 之后重新访问。必须用示例中的 Rextester 类作为类名。
可以**嵌入到自己网站**，比如 Java 嵌入语句如下：
```
<iframe src="http://www.it1352.com/Onlinetools/OnlineCompileCommon/4?c_height=100&r_height=100&code=&autoExecute=true" style="width:520px;height:450px;"></iframe>
```
参数： c_height—>【源码框高度】 r_height—>【结果框高度】 code—>【代码片段(需URL编码)】 autoExecute—>【是否自动执行】

# http://demo.php.cn/
专门在线执行 PHP 代码的网站，快，代码窗口可以自动换行，但预览窗口不会自动换行，好在可以全屏预览。**PHP 版本是 7.0.8。**

# http://anycodes.cn/zh/
可以在线执行各种语言，但是 PHP 的版本较低，低于 5.4 。
优点：速度快，稳定，支持多个文件（目前是两个）。

# http://www.dooccn.com/php/
可以在线执行**多种语言**，可以选择 PHP 的多个版本。php5.3 php5.4 php5.5 php5.6 php7。
缺点：速度比较慢。

# https://ideone.com
在线执行代码，然后生成分享链接。可以把链接发给别人，也可以嵌入网页。
优点：支持的语言多，版本新。PHP 版本是 7.1.0。
比如，下面这个链接是我执行代码后生成的，
`https://ideone.com/uIdTu3`，对应的分享 JavaScript 代码为 `<script src="https://ideone.com/e.js/uIdTu3" type="text/javascript" ></script>`。

# https://tool.lu/coderunner/
各种语言都可以在线执行，**PHP 的版本是 5.4.16。**
登录后可以保存在线代码，可以嵌入博客。首页还有各种其他工具。
缺点：不是很稳定，sandbox 经常挂掉。出现这一句 `sandbox> exited with status 0` 就表示已经挂掉了，需要刷新重连。

[这个应该是作者的博客](https://type.so/)
[tool.lu技术架构参考这里](https://type.so/linux/tool-lu-architecture.html)，摘抄如下：
####背景
一个字，穷！在小流量的情况下，这个应该算是比较经济的解决方案了吧(各种容灾都没有，监控没有，服务的吞吐测试没有)。哈哈哈...
####后端的业务处理和服务
整个的网站都放在aliyun的VPS上。
由于工具网站的后端处理比较耗资源，于是将业务处理服务部署到了两台VPS上。（aliyun +1 & 美国 +1）
Redis只是做了少量的缓存作用，所以图中并未给出。

Untitled.png

####面对前端的一些优化
cdn 现在全部都放在aliyun的VPS上。

- 使用nginx的 nginx-http-concat 扩展合并多个文件请求。
- http_image_filter_module 进行一些图片的实时压缩计算

域名分别为 s1.tool.lu, s2.tool.lu, s3.tool.lu
####爬虫
现在所有的爬虫均基于Scrapy编写，全部部署在 美国的vps上;数据储存在MariaDB。
####虚拟化
主要用于一些不可信任代码的执行。

选型Docker，可限制CPU和Mem，不能限制Disk，但是Docker在CentOS6.x下的问题较多，各种坑；最近使用CentOS7搭建之后貌似很Happy。