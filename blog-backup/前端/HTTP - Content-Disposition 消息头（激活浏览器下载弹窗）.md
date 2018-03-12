[MDN 资料](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition)
# 可用位置
Content-Disposition 消息头最初是在 MIME 标准中定义的，**HTTP 表单**及 **POST 请求**只用到了其所有参数的一个子集。

- 在常规的 HTTP 应答中，Content-Disposition 消息头指示回复的内容该以何种形式展示，是以内联的形式（即网页或者页面的一部分），还是以附件的形式下载并保存到本地。
- 在multipart/form-data类型的应答消息体中， Content-Disposition 消息头可以被用在multipart消息体的子部分中，用来给出其对应字段的相关信息。各个子部分由在Content-Type 中定义的分隔符分隔。用在消息体自身则无实际意义。
#语法
##1. 作为消息主体中的消息头
在HTTP场景中，第一个参数或者是inline（默认值，表示回复中的消息体会以页面的一部分或者整个页面的形式展示），或者是attachment（意味着消息体应该被下载到本地；大多数浏览器会呈现一个“保存为”的对话框，将filename的值预填为下载后的文件名，假如它存在的话）。
```
Content-Disposition: inline
Content-Disposition: attachment
Content-Disposition: attachment; filename="filename.jpg"
```
##2. 作为表单 multipart 中的 body 中的消息头
第一个参数总是固定不变的 `form-data;`。附加的参数为分号(;)分隔的名值对，值必须用双引号。
```
Content-Disposition: form-data
Content-Disposition: form-data; name="fieldName"
Content-Disposition: form-data;name="fieldName";filename="filename.jpg"
```
`name`：表单字段名，每一个字段名会对应一个子部分。在同一个字段名对应多个文件的情况下（例如，带有multiple 属性的 `<input type="file">` 元素），则多个子部分共用同一个字段名。如果 name 参数的值为 `'_charset_'`，意味着这个子部分表示的不是一个 HTML 字段，而是在未明确指定字符集信息的情况下各部分使用的默认字符集。
`filename`：文件的初始名，可选，同时要进行一定的转换以符合服务器文件系统规则。这个参数主要用来提供展示性信息。当与 `Content-Disposition: attachment` 一同使用的时候，它被用作"保存为"对话框中呈现给用户的默认文件名。
# 激活浏览器下载弹窗
只有响应头中有 `Content-Disposition: attachment; filename="xx"` 即可下载而不是在浏览器中展示。大多数浏览器默认会将 filename 作为文件名。。
```
200 OK
Content-Type: text/html; charset=utf-8
Content-Disposition: attachment; filename="cool.html"
Content-Length: 22

<HTML>Save me!</HTML>
```