参考：
https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS
https://www.w3.org/TR/cors/
http://www.ruanyifeng.com/blog/2016/04/cors.html

#一. 概述
CORS（Cross-Origin Resource Sharing，跨域资源访问）：需要**访问**跨域资源或需要发起跨域AJAX时，可以在 API 容器中（例如 XMLHttpRequest 或 Fetch ）使用。跨域访问控制在 Web 应用服务端进行，安全。

CORS需要**浏览器**和**服务器**同时支持。IE10以下浏览器不支持CORS。

服务器通过 HTTP 首部字段声明哪些源站有权限访问哪些资源。
#二. 简单请求和非简单请求
浏览器将CORS请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。
##1. 简单请求
简单请求：不会触发 CORS 预检请求的请求。对于简单请求，浏览器直接发出CORS请求。
简单请求必须满足：
```
（1) 请求方法是以下三种方法之一：

- HEAD
- GET
- POST

（2）HTTP的头信息不超出以下几种字段：

- Accept
- Accept-Language
- Content-Language
- Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain
```
浏览器发现请求是跨域AJAX请求时，自动在头部加上 Origin 字段，表明请求来源。`Origin: http://xx.com`
服务器收到请求后：

- 如果 Origin 指定的源，不在许可范围内，服务器会返回一个正常的HTTP响应（状态码可能是200），但这个响应的头信息没有包含`Access-Control-Allow-Origin`字段，浏览器因此知道出错了，从而抛出一个错误，被XMLHttpRequest的onerror回调函数捕获。
- 如果 Origin 指定的源，在许可范围内，需要返回 HTTP 头信息：`Access-Control-Allow-Origin`，指明服务器允许哪些源访问。如果要允许多个源访问，需要配置Nginx（）。
 - `Access-Control-Allow-Origin: null` 不允许所有源跨源访问
 - `Access-Control-Allow-Origin: *` 允许所有源访问
 - `Access-Control-Allow-Origin: http://yy.com` 只允许`http://yy.com`访问
##2. 非简单请求
####1. 需预检的请求必须首先使用 OPTIONS   方法发起一个预检请求到服务器，以获知服务器是否允许该实际请求：
```
OPTIONS /resources/ HTTP/1.1
Origin: http://foo.example
Access-Control-Request-Method: POST
Access-Control-Request-Headers: X-PINGOTHER, Content-Type


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:39 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://foo.example
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400
```
####2. 预检请求完成之后，发送实际请求：
```
POST /resources/post-here/ HTTP/1.1
X-PINGOTHER: pingpong
Content-Type: text/xml; charset=UTF-8
[data...]


HTTP/1.1 200 OK
Date: Mon, 01 Dec 2008 01:15:40 GMT
Server: Apache/2.0.61 (Unix)
Access-Control-Allow-Origin: http://foo.example
```
##3. 附带身份凭证的请求
CORS 可以基于  HTTP cookies 和 HTTP 认证信息发送身份凭证。
**默认不发送。**必须手动将 XMLHttpRequest 的 withCredentials 标志设置为 true，从而向服务器发送 Cookies。但是，如果服务器端的响应中未携带 Access-Control-Allow-Credentials: true ，**浏览器将不会把响应内容返回给请求的发送者**。
对于附带身份凭证的请求，服务器不得设置 Access-Control-Allow-Origin 的值为“*”。

##4. CORS常用的HTTP首部字段解释：
| 字段 | 由谁设置 | 作用 |
|------|------|--|
|**Origin**|浏览器|指明简单请求或预检请求从哪个源发起，不管是否跨域都会设置|
|Access-Control-Request-Method|浏览器|在预检请求中出现，用于通知服务器在真正的请求中会采用哪种<br/> HTTP 方法。因为预检请求所使用的方法总是 OPTIONS ，与实际请求<br/>所使用的方法不一样，所以这个首部是必要的。|
|Access-Control-Request-Headers|浏览器|在预检请求中出现，通知服务器在真正的请求中会采用哪些请求首部。|
|**Access-Control-Allow-Origin**|服务器|必传，指明资源可以被哪些跨域的源访问。它的值要么是请求时Origin字段<br/>的值，要么是一个*，表示接受任意域名的请求。|
|Access-Control-Allow-Credentials|服务器|是否允许浏览器读取response的内容。当用在对preflight预检请求的响应中<br/>时，它指定了实际的请求是否可以使用credentials。请注意：简单 GET 请求<br/>不会被预检；如果对此类请求的响应中不包含该字段，这个响应将被忽略掉，<br/>并且浏览器也不会将相应内容返回给网页。|
|Access-Control-Expose-Headers|服务器|默认只有6种简单响应首部可以暴露给外部：Cache-Control、<br/>Content-Language、Content-Type、Expires、Last-Modified、Pragma。<br/>如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定。<br/>可指定多个响应首部<br/>`Access-Control-Expose-Headers: Content-Length, X-Rev`|
|Access-Control-Max-Age|服务器|表示预检请求的返回结果（即 Access-Control-Allow-Methods 和<br/>Access-Control-Allow-Headers 提供的信息） 可以被缓存多久。单位是秒。<br/>不同浏览器的上限和默认值不一样。如果值为 -1，则表示禁用缓存|
|Access-Control-Allow-Methods|服务器|在预检请求中，其指明了实际请求所允许使用的 HTTP 方法|
|Access-Control-Allow-Headers|服务器|在预检请求中，指明了实际请求中允许携带的首部字段|