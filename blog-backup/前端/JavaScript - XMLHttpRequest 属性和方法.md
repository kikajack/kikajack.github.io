[MDN XMLHttpRequest 定义](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest)
[MDN XMLHttpRequest 使用](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/Using_XMLHttpRequest)

XMLHttpRequest 支持的数据类型：所有类型的数据资源，并不局限于 XML。
XMLHttpRequest 支持的协议：HTTP，file，FTP。

通过构造函数 `XMLHttpRequest()` 可以构造一个 XMLHttpRequest 对象。通过 open() 方法初始化，send() 方法发送请求，onload 属性接受响应。
#1. 简单示例
[XMLHttpRequest 请求并处理二进制流（图片）](https://segmentfault.com/q/1010000000443286)，通过 POST 方式，数据格式 Blob：
```
var btn = document.getElementById("btn") ,
    addr = document.getElementById("v") ;

btn.addEventListener("click", function(evt){
  var xhr = new XMLHttpRequest() ,
    val = addr.textContent ;
  xhr.onload = function() {
    var objectURL = URL.createObjectURL(this.response),
      img = document.createElement("img");
    img.src = objectURL;
    img.height = 60;
    img.onload = function(e) {
      window.URL.revokeObjectURL(this.src);
    };
    document.body.appendChild(img) ;
  } ;
  xhr.open("POST", val);
  xhr.responseType = "blob";
  xhr.send(null);
}, false) ;
```
#2. 属性
XMLHttpRequest 接口继承了 XMLHttpRequestEventTarget 和 EventTarget 的属性。

从 XMLHttpRequestEventTarget  继承来的属性有：
|属性|  类型| 描述|
|-|-|-|
|onabort| nsIDOMEventListener|当用户终止操作时回调的 JavaScript 函数对象|
|**onerror**| nsIDOMEventListener|发生错误导致操作无法完成时回调的 JavaScript 函数对象|
|**onload**|  nsIDOMEventListener|任务成功完成时回调的 JavaScript 函数对象|
|onloadend| nsIDOMEventListener|任务结束（成功、失败、用户终止）时回调的 JavaScript 函数对象，跟在 abort、error 或 load 任意一个事件之后|
|onloadstart| nsIDOMEventListener|刚好在任务开始时回调的 JavaScript 函数对象|
|onprogress|  nsIDOMEventListener|在 loadstart 事件之后，abort、error 或 load 任意一个事件之前，回调 0 次或多次的 JavaScript 函数对象|

自身的属性有：
|属性|  类型| 描述|
|-|-|-|
|onreadystatechange|Function?|当 readyState 属性改变时会调用这个 JavaScript 函数对象。<br/>回调函数会在user interface线程中调用。|
|**readyState**|  unsigned short|请求的五种状态<br/>0：UNSENT(未打开)，open()方法还未被调用<br/>1：OPENED(未发送)，open()方法已经被调用<br/>2：HEADERS_RECEIVED(已获取响应头)，send()方法已经被调用, 响应头和响应状态已经返回<br/>3：LOADING(正在下载响应体)，响应体下载中，responseText中已经获取了部分数据<br/>4：DONE(请求完成)，整个请求过程已经完毕|
|**response**|  varies|二进制响应数据。响应实体的类型由 responseType 来指定，浏览器自动处理数据为这个类型。可以是 ArrayBuffer， Blob， Document， JavaScript 对象 (即 "json")， 或者是字符串。如果请求未完成或失败，则该值为 null。|
|**responseType**|  XMLHttpRequestResponseType|告诉服务器你期望的响应格式。<br/>"" (空字符串)：字符串(默认值)<br/>"arraybuffer"：ArrayBuffer<br/>"blob"：Blob<br/>"document"：Document<br/>"json"：JavaScript 对象，解析服务器传递回来的JSON 字符串。<br/>"text"：字符串|
|**responseText**|  DOMString|  文本响应数据，或是当请求未成功或还未发送时为 null。只读。|
|**status**|  unsigned short| 该请求的响应状态码 (例如, 状态码200 表示一个成功的请求)。只读。|
|**statusText**|  DOMString|  该请求的响应状态信息,包含一个状态码和原因短语 (例如 "200 OK")。只读。|
|**upload**|  XMLHttpRequestUpload| 可以在 upload 上添加一个事件监听来跟踪上传过程，监测上传进度|
|**withCredentials**| boolean |表明在进行跨站(cross-site)的访问控制(Access-Control)请求时，是否使用认证信息(例如cookie或授权的header)。 默认为 false。注意: 这不会影响同站(same-site)请求。|

#3. 方法

|方法|  描述|
|-|-|
|**open()**|初始化一个请求|
|setRequestHeader()|给指定的HTTP请求头赋值。在这之前，你必须确认已经调用 open() 方法打开了一个url。|
|abort()|如果请求已经被发送，则立刻中止请求|
|getAllResponseHeaders()|返回所有响应头信息, 如果响应头还没接受，则返回null。注意:对于 multipart 多部分请求，会返回当前部分请求的 headers，而不是整个报文的 headers。|
|overrideMimeType()|重写服务器返回的 MIME type。例如，强制把一个响应流当作“text/xml”来处理和解析，即使服务器没有指明数据是这个类型。注意，这个方法必须在send()之前被调用。|
|**send()**|发送请求。如果该请求是异步模式(默认)，该方法会立刻返回. 相反，如果请求是同步模式，则直到请求的响应完全接受以后,该方法才会返回。所有相关的事件绑定必须在调用send()方法之前进行。|
##3.1 open()
初始化一个请求. 该方法用于JavaScript代码中。如果是本地代码, 使用 openRequest() 方法代替。

注意: **XHR 请求只能初始化一次，再次初始化会中止这个请求。**在一个已经激活的 request 下（已经调用 open() 或 openRequest() 方法的 request），再次调用这个方法相当于调用 abort（）方法。
```
void open(
   DOMString method,
   DOMString url,
   optional boolean async,
   optional DOMString user,
   optional DOMString password
);
```
参数：

- method：请求所使用的HTTP方法，如 "GET", "POST", "PUT", "DELETE"等，只对 HTTP 或 HTTPS 请求有效。如果下个参数是非 HTTP(S) 的 URL，则忽略该参数。
- url：该请求所要访问的 URL。
- async：是否执行异步操作，可选，布尔值，默认为 true 执行异步操作。如果值为 false，则 send() 方法不会返回任何东西，直到接受到了服务器的返回数据。如果为值为true，一个对开发者透明的通知会发送到相关的事件监听者，异步监听请求的结果。
- user：用户名，可选参数，为授权使用。默认参数为空string。
- password：密码，可选参数，为授权使用。默认参数为空string。

##3.2 setRequestHeader()
给指定的 HTTP 请求头赋值。在这之前，你必须确认已经调用 open() 方法打开了一个url。
```
void setRequestHeader(
   DOMString header,
   DOMString value
);
```
参数：

- header：将要被赋值的请求头名称
- value：给指定的请求头赋的值。