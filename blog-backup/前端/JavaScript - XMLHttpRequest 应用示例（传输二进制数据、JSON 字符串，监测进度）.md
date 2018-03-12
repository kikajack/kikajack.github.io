[XMLHttpRequest - Living Standard 一直更新的官方标准](https://xhr.spec.whatwg.org/)
[MDN XMLHttpRequest 使用](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest)
[描述 XMLHttpRequest-2 新功能的博客文章](https://www.html5rocks.com/zh/tutorials/file/xhr2/)

#1. 异步模式或同步模式
XMLHttpRequest 对象的 open() 方法的第三个参数 async 设为 false 时，该 XMLHttpRequest 请求以同步模式进行。**同步模式用户体验差，在主线程上的同步请求在部分浏览器上不可用。**
```
// open 函数完整参数列表
void open(
   DOMString method,
   DOMString url,
   optional boolean async,
   optional DOMString user,
   optional DOMString password
);

// 创建同步 XMLHttpRequest
var xhr = new XMLHttpRequest();
xhr.open("POST", "xx.com/xx", false);
```
#2. 处理响应
XMLHttpRequest 可以发送和接收**文本数据**，也可以发送和接受**二进制**内容。
##2.1 处理 XML （responseXML 属性）
通过 XMLHttpRequest 获取远程的 XML 文档的内容，会放在这个属性中。麻烦，基本不再使用了。
##2.2 处理文本数据（responseText 属性）
获取远程的文本数据。
通过 XMLHttpRequest 收发 JSON 数据可以 [参考这里](https://itbilu.com/javascript/js/NkzjoC4j.html)，示例如下：
```
var xhr = new XMLHttpRequest();
xhr.open("POST", "xx.com", true);
xhr.setRequestHeader('content-type', 'application/json'); // 设置 HTTP 头，数据指定为 JSON 格式
xhr.onreadystatechange = function() {
    if (xhr.readyState == 4) {
        if(xhr.getResponseHeader('content-type')==='application/json'){
      var result = JSON.parse(xhr.responseText); // 必须从 responseText 读文本数据
      /* ... */
    } else{
      console.log(xhr.responseText);
    }
  }
}
xhr.send(JSON.stringify({"text":"hello"}));
```
##2.3 处理二进制数据（response 属性）
###2.3.1 利用 responseType 属性
可以通过 **responseType 属性（XMLHttpRequest Level 2 规范中新添加的）** 告诉浏览器返回数据的期望类型（可以是 空字符串 (默认)、blob、text、arraybuffer、document）。response 属性的值会根据 responseType 属性的值的不同而不同, 可能会是一个 ArrayBuffer、Blob、Document、string、或者为 NULL (如果请求未完成或失败)。
```
var xhr = new XMLHttpRequest();

xhr.onload = function() {
  if (this.status == 200) {
    var blob = this.response; // 注意必须从 response 中读取二进制，而不是 responseText
    var img = document.createElement("img");
    img.onload = function(e) {
      window.URL.revokeObjectURL(img.src); 
    };
    img.src = window.URL.createObjectURL(blob);
    /* 填充到页面上 */
    $("#imgcontainer").html(img);
  }
}
xhr.open("GET", url, true);
xhr.responseType = "blob";
xhr.setRequestHeader("client_type", "DESKTOP_WEB");
xhr.send();
```
###2.3.2 利用 overrideMimeType() 方法
利用 XMLHttpRequest 的  overrideMimeType() 方法可以重写服务器返回的 MIME type，并指示浏览器按照这个类型来处理数据。这不是标准方法。
```
var xhr = new XMLHttpRequest();
xhr.open("GET", url, true);
// retrieve data unprocessed as a binary string
xhr.overrideMimeType("text/plain; charset=x-user-defined");
/* ... */
```
#3. 发送二进制数据
XMLHttpRequest 对象的 send 方法已被增强，可以直接传入 ArrayBuffer、Blob 或者 File 对象来发送二进制数据。
##3.1 发送 Blob 示例
```
// 用 POST 方法将 blob 发送到服务器
var xhr = new XMLHttpRequest();
xhr.open("POST", url, true);
xhr.onload = function (oEvent) {
  // ...
};

var debug = {hello: "world"};
var blob = new Blob([JSON.stringify(debug, null, 2)],
  {type : 'application/json'}); // 创建 blob 对象，同时指定类型

xhr.send(blob);
```
##3.2 发送 ArrayBuffer 示例
```
// 用 POST 方法将 ArrayBuffer 发送到服务器
var myArray = new ArrayBuffer(512);
var longInt8View = new Uint8Array(myArray);

for (var i=0; i< longInt8View.length; i++) {
  longInt8View[i] = i % 255;
}

var xhr = new XMLHttpRequest;
xhr.open("POST", url, false);
xhr.send(myArray);
```
#4. 监测传输进度
可以用于各种上传、下载过程中的进度条，实现进度指示功能。
XMLHttpRequest 提供了各种在**请求处理期间**发生的事件以供监听，包括定期进度通知、 错误通知等。**需要在请求调用 open() 之前添加事件监听。**
XMLHttpRequest 支持 DOM 的 progress 事件监测，遵循 Web API 进度事件规范。这些事件实现了 ProgressEvent 接口。
##4.1 progress 事件特性
- progress 事件对象的 **total 和 loaded 属性**分别表示**需要传输的总字节数和已经传输的字节数**。lengthComputable 属性的值是 false 时，总字节数是未知并且 total 的值为零。
- progress 事件同时存在于下载和上传过程。下载相关事件在 XMLHttpRequest 对象上被触发。上传相关事件在 XMLHttpRequest.upload 对象上被触发
- progress 事件在传输文件时（使用 `file:` 协议）无效。
##4.2 下载时监测进度示例
```
var xhr = new XMLHttpRequest();

// 为事件添加事件监听，这些事件在使用 XMLHttpRequest 执行数据传输时被发出。
xhr.addEventListener("progress", updateProgress, false);
xhr.addEventListener("load", transferComplete, false);
xhr.addEventListener("error", transferFailed, false);
xhr.addEventListener("abort", transferCanceled, false);

xhr.open();

...

// progress on transfers from the server to the client (downloads)
function updateProgress(evt) {
  // progress 事件的 lengthComputable 属性存在时，可以计算数据已经传输的比例(loaded 已传输大小，total 总大小）
  if (evt.lengthComputable) {
    var percentComplete = evt.loaded / evt.total;
    ...
  } else {
    ...
  }
}

function transferComplete(evt) {
  alert("The transfer is complete.");
}

function transferFailed(evt) {
  alert("An error occurred while transferring the file.");
}

function transferCanceled(evt) {
  alert("The transfer has been canceled by the user.");
}
```
##4.3 上传时监测进度示例
```
var xhr = new XMLHttpRequest();

xhr.upload.addEventListener("progress", updateProgress);
xhr.upload.addEventListener("load", transferComplete);
xhr.upload.addEventListener("error", transferFailed);
xhr.upload.addEventListener("abort", transferCanceled);

req.open();
```
#5. 提交表单
XMLHttpRequest 可以通过 FormData 提交表单，也可以不用 FormData。
使用 FormData 简单快捷，但是被收集的数据无法使用 JSON.stringify() 转换为一个 JSON 字符串。
不用 FormData 时，复杂但灵活。
##5.1 只使用 XMLHttpRequest 提交表单（不用 FormData）
**只使用 XMLHttpRequest 就可以完成表单提交（不包括文件）。上传一个或多个文件时需要额外的 FileReader API。**
使用 XMLHttpRequest 提交表单时，只要告诉浏览器如何设置 HTTP header，如何组装数据就可以了。具体的代码可以 [参考这里的 MDN 示例](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest#%E6%8F%90%E4%BA%A4%E8%A1%A8%E5%8D%95%E5%92%8C%E4%B8%8A%E4%BC%A0%E6%96%87%E4%BB%B6)
一个 form 表单可以用以下四种方式发送：

- 使用 POST 方法，并设置 enctype 属性为 `application/x-www-form-urlencoded` (默认)。header 和 body 部分内容如下：
```
Content-Type: application/x-www-form-urlencoded

foo=bar&baz=The+first+line.&#37;0D%0AThe+second+line.%0D%0A
```
- 使用 POST 方法，并设置 enctype 属性为 `text/plain`。header 和 body 部分内容如下：
```
Content-Type: text/plain

foo=bar
baz=The first line.
The second line.
```
- 使用 POST 方法，并设置 enctype 属性为 `multipart/form-data`。header 和 body 部分内容如下：
```
Content-Type: multipart/form-data; boundary=---------------------------314911788813839

-----------------------------314911788813839
Content-Disposition: form-data; name="foo"

bar
-----------------------------314911788813839
Content-Disposition: form-data; name="baz"

The first line.
The second line.

-----------------------------314911788813839--
```
- 使用 GET 方法（这种情况下 enctype 属性会被忽略），下面这样的字符串将被简单的附加到 URL 后面：
```
?foo=bar&baz=The%20first%20line.%0AThe%20second%20line.
```
##5.2 使用 FormData 提交表单
FormData 对象可以**用 JavaScript 动态创建**或**用 HTML 页面已有的 form 元素来创建**。
关于 FormData 对象，可以参考 [这里的 MDN 资料](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/Using_FormData_Objects)。
###5.2.1 用 JavaScript 动态创建 FormData 对象
```
function sendForm() {
  var formData = new FormData(); // 用 JavaScript 动态创建
  formData.append('username', 'johndoe');
  formData.append('id', 123456);

  var xhr = new XMLHttpRequest();
  xhr.open('POST', '/server', true);
  xhr.onload = function(e) { ... };

  xhr.send(formData);
}
```
###5.2.2 用 HTML 页面已有的 form 元素创建 FormData 对象
```
<form id="myform" name="myform" action="/server">
  <input type="number" name="id" value="123456">
  <input type="submit" onclick="return sendForm(this.form);">
</form>
<script>
  function sendForm(form) {
    var formData = new FormData(form); // 用 form 元素创建
    
    formData.append('secret_token', '1234567890'); // 动态添加数据
    
    var xhr = new XMLHttpRequest();
    xhr.open('POST', form.action, true);
    xhr.onload = function(e) { ... };
    xhr.send(formData);
    
    return false; // 防止页面提交表单
  }
</script>
```
###5.2.3 使用 FormData 对象传文件
只需通过 append() 方法添加文件，浏览器就会在调用 send() 时构建 `multipart/form-data` 请求。
```
function uploadFiles(url, files) {
  var formData =new FormData();
  
  for (var i = 0, file; file = files[i]; ++i) {
    formData.append(file.name, file);
  }
  
  var xhr = new XMLHttpRequest();
  xhr.open('POST', url, true);
  xhr.onload = function(e) { ... };
  
  xhr.send(formData);
}

document.querySelector('input[type="file"]').addEventListener('change', function(e) {
  uploadFiles('/server', this.files);
}, false);
```
#6. 跨源资源共享 (CORS)
服务器端允许 Web 应用发送请求时，才可以使用 XMLHttpRequest。否则，客户端会抛出一个 `INVALID_ACCESS_ERR` 异常。
CORS 允许一个域上的网络应用向另一个域提交跨域 AJAX 请求。启用此功能非常简单，只需由服务器发送指定的 HTTP header `Access-Control-Allow-Origin` 即可。
##6.1 启用 CORS
对于允许跨源的资源，服务器需要在返回的 HTTP header 中增加一个 `Access-Control-Allow-Origin`。

- 允许某个源访问：
```
Access-Control-Allow-Origin: http://example.com
```
- 允许所有源访问：
```
Access-Control-Allow-Origin: *
```
##6.2 访问跨域资源
服务器端启用 CORS 后，跨域 XMLHttpRequest 请求可以顺利发出，和普通的 XMLHttpRequest 请求一样使用。
#7. 绕过缓存
##7.1 给 URL 添加时间戳
浏览器的**本地缓存都是以 URL 作为索引**的，URL 添加时间戳后，使每个请求都是唯一的，从而绕开缓存。
```
http://foo.com/bar.html?12345 // 对于不带 GET 参数的，用 ? 连接时间戳
http://foo.com/bar.html?foobar=baz&12345  // 对于带 GET 参数的，用 & 连接时间戳
```
```
// 使用正则表达式自动添加时间戳
var req = new XMLHttpRequest();
req.open("GET", url += ((/\?/).test(url) ? "&" : "?") + (new Date()).getTime(), false);
req.send(null); 
```
##7.2 通过 XMLHttpRequest 对象的 channel 属性实现
```
var xhr = new XMLHttpRequest();
xhr.open('GET', url, false);
xhr.channel.loadFlags |= Components.interfaces.nsIRequest.LOAD_BYPASS_CACHE;
xhr.send(null);
```