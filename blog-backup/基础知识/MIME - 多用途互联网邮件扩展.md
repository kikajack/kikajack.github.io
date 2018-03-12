[维基百科对 MIME 的定义](https://zh.wikipedia.org/wiki/%E5%A4%9A%E7%94%A8%E9%80%94%E4%BA%92%E8%81%AF%E7%B6%B2%E9%83%B5%E4%BB%B6%E6%93%B4%E5%B1%95)
[RFC 文档](https://tools.ietf.org/html/rfc6838)
完整的 MIME 类型列表可以 [参考 MDN](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types)  、 [参考 W3school](http://www.w3school.com.cn/media/media_mimeref.asp) 或 [IANA 官方机构的列表](https://www.iana.org/assignments/media-types/media-types.xhtml)
[MIME 语法及结构](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)
#1. MIME 概述
**HTTP 协议使用 MIME 类型（而不是文件扩展名）来确定文档类型。**

MIME（Multipurpose Internet Mail Extensions）是一个互联网标准，用来**表示文档的性质和格式**。它扩展了电子邮件标准，使其能够支援：

非ASCII字符文本；
非文本格式附件（二进制、声音、图像等）；
由多部分（multiple parts）组成的消息体；
包含非ASCII字符的头信息（Header information）。这个标准被定义在RFC 2045、RFC 2046、RFC 2047、RFC 2048、RFC 2049等RFC中。
#2. MIME 语法
```
type/subtype
```
MIME 由类型与子类型两个字符串组成，不允许空格。type 表示可以被分为复数子类的独立类型。subtype 表示细分后的每个类型。MIME 类型对大小写不敏感，通常都是小写。
使用时，用 Content-Type 指定类型，例如： 
```
Content-Type: [type]/[subtype]; parameter
```
`Content-Type:text/css; charset=utf-8`
parameter 用来指定附加信息，通常是用于指定 text 文本的编码方式的 charset 参数。MIME 每个 type 都有默认的 subtype，不能确定消息的 subtype 的情况下，消息被看作默认的 subtype 进行处理。text 默认是 text/plain，application 默认是 application/octet-stream，multipart 默认是 multipart/mixed。
##2.1 MIME 的独立类型
独立类型对文件分类，可以是如下之一：

|类型 |描述 |典型示例|
|-|-|
|text |普通文本，可读  |text/plain, text/html, text/css, text/javascript|
|image  |图像，动态图（比如 gif） |image/gif, image/png, image/jpeg, image/bmp, image/webp
|audio  |音频文件 |audio/mpeg, audio/webm, audio/ogg, audio/wav
|video  |视频文件 |video/webm, video/ogg
|application  |二进制数据  |application/octet-stream, application/vnd.mspowerpoint, <br/>application/xml,  application/pdf,application/json|
|multipart|多部分消息，可以是不同类型|multipart/form-data,multipart/byteranges|
|message|E-mail消息||
##2.2 Multipart 类型
```
multipart/form-data
multipart/byteranges
```
Multipart 类型表示文档中含有多个部分。multipart/form-data 可以使用 HTML Forms 和 POST 方法，此外 multipart/byteranges使用状态码206 Partial Content来发送整个文件的子集，而HTTP不能处理的复合文件使用一个特殊的方式：将信息直接传送给浏览器（这时可能会建立一个“另存为”窗口，但是却不知道如何去显示内联文件。）
###2.2.1 multipart/form-data （常用）
`multipart/form-data` 可用于 HTML Form 表单提交。**如果要上传文件 file，则必须在表单中指定 `enctype:"multipart/form-data"`**。作为多部分文档格式，`multipart/form-data` 可以同时上传多个不同类型的文件，用边界线（一个由'----'开始的字符串）划分出不同部分。每一部分有自己的实体及 HTTP 请求头（Content-Disposition 指定当前部分的文件名和表单中的字段名等信息，Content-Type 指定当前部分的文件类型）。
```
POST http://xx.com HTTP/1.1
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryMTqSEAwmA5xk7oKK

------WebKitFormBoundaryMTqSEAwmA5xk7oKK
Content-Disposition: form-data; name="text"

title
------WebKitFormBoundaryMTqSEAwmA5xk7oKK
Content-Disposition: form-data; name="userImage"; filename="chrome.png"
Content-Type: image/png

PNG ... 此处的图片内容省略 ...

------WebKitFormBoundaryMTqSEAwmA5xk7oKK
Content-Disposition: form-data; name="riskName"

中文
------WebKitFormBoundaryMTqSEAwmA5xk7oKK
Content-Disposition: form-data; name="riskAmounts"

{"1":{"name":"10万","premium":"5","amount":"100000"},"2":{"name":"20万","premium":"8","amount":"200000"}}
------WebKitFormBoundaryMTqSEAwmA5xk7oKK
```
###2.2.2 multipart/byteranges （没用过）
`multipart/byteranges` 表示把由几部分组成的响应报文发送回浏览器（需要浏览器发出这个请求，且服务器理解并成功处理请求）。当服务器发送状态码 `206 Partial Content ` 时，这个 MIME 类型用于指出这个响应报文由几部分组成，每一个都有其请求范围。每一个不同的部分都有 Content-Type 这样的HTTP头来说明文件的实际类型，以及 Content-Range 来说明当前部分所属的范围。
```
HTTP/1.1 206 Partial Content
Accept-Ranges: bytes
Content-Type: multipart/byteranges; boundary=3d6b6a416f9b5
Content-Length: 385

--3d6b6a416f9b5
Content-Type: text/html
Content-Range: bytes 100-200/1270

eta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="vieport" content
--3d6b6a416f9b5
Content-Type: text/html
Content-Range: bytes 300-400/1270

-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: "Open Sans", "Helvetica
--3d6b6a416f9b5--
```
#3. MIME 类型
**对于 text 文件类型若没有特定的 subtype，就使用 `text/plain`。**
**对于二进制文件类型若没有特定或已知的 subtype，就使用 `application/octet-stream`。**

- `application/octet-stream`：应用程序文件的默认值。对于未知类型的应用程序文件浏览器一般不会自动执行或询问执行，会当做附件一样来对待（类似在 HTTP 头中设置 Content-Disposition 时，会激活‘文件下载’对话框）。
- `text/plain`：文本文件默认值。对于未知类型的文本文件 ，浏览器认为是可以直接展示的。
- `text/css`：CSS 文件想要在网页上被解释执行就必须把 MIME 设置为 text/css 。
- `text/html`：HTML 内容。
- 图片类型
图片类型是在网页中使用的，唯一被广泛识别以及考虑过web安全的类型：
|MIME| 类型|  图片类型|
|-|-|
|image/gif| GIF| 图片 (无损耗压缩方面被PNG所替代)
|image/jpeg|  JPEG| 图片
|image/png| PNG| 图片
|image/svg+xml| SVG图片| (矢量图)

另外的一些图片种类可以在Web文档中找到。比如很多浏览器支持 icon 类型的图标作为 favicons或者类似的图标，并且浏览器在MIME类型中的 image/x-icon 支持ICO图像。


