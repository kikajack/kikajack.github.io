[MDN - Document API文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Document)
[MDN - FormData API文档](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)
[MDN - Blob API文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)
[MDN - File API文档](https://developer.mozilla.org/zh-CN/docs/Web/API/File)
[MDN - ArrayBuffer API文档](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)
[张鑫旭博客的相关文章](http://www.zhangxinxu.com/wordpress/2013/10/understand-domstring-document-formdata-blob-file-arraybuffer/)

window.URL，FileReader 对象，ArrayBuffer 对象都牵扯到很多知识。

XMLHttpRequest 指定了 responseType 属性后，浏览器会自动把接受到的数据进行解析，生成 responseType 指定的数据类型。
#1. DOMString
DOMString 是一个 UTF-16 字符串。由于 JavaScript 默认编码格式就是 UTF-16，所以 **DOMString 就是 JavaScript 中的 String 类型**。
XMLHttpRequest 中的属性 responseText 类型是 DOMString，可以直接通过 JavaScript 当做字符串处理（MIME 指明数据类型是文本类型时）。
#2. Document
XMLHttpRequest 可以传输 HTML、XML 类型的 DOM 文档数据。
Document 接口继承自 Node 及 EventTarget 接口。
Document 接口描述了任何类型的文档的公共属性和方法。根据文档的具体类型 (例如 HTML、XML、SVG,  ... )，可以使用对应类型的 API：HTML 文档实现了 HTMLDocument 接口，而 SVG 文档实现了 SVGDocument 接口。
支持的属性、事件、方法很多，直接参考 MDN 文档。
#3. FormData
借助 FormData 对象，可以通过 XMLHttpRequest 实现 form 表单提交（采用的 enctype 是“multipart/form-data”）。[参考这一篇](http://blog.csdn.net/kikajack/article/details/79298363)。
##3.1 FormData 语法：
###3.1.1 构造函数
`new FormData (form? : HTMLFormElement)`
参数：form (可选)，HTML 表单元素，可以包含任何形式的表单控件，包括文件输入框。

###3.1.2 方法 append()
FormData 对象只有一个方法：append()，用于给当前 FormData 对象添加一个键/值对（可以添加二进制数据）。
```
void append(DOMString name, Blob value, optional DOMString filename);
void append(DOMString name, DOMString value);
```
参数值：
name：字段名称。
value：字段值。可以是 Blob 对象或 File 对象，或是字符串，如果都不是，则该值会被自动转换成字符串。
filename：(可选) 指定文件的文件名。当 value 参数被指定为 Blob 对象或 File 对象时，该文件名会被发送到服务器上。对于 Blob 对象来说，这个值默认为"blob"。
```
var formData = new FormData(); // 用 JavaScript 动态创建 FormData 对象
var formData = new FormData(form); // 用 form 元素创建 FormData 对象
formData.append('secret_token', '1234567890'); // 动态添加数据到 FormData 对象
```
##3.2 示例
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
#4. Blob
BLOB (binary large object，二进制大对象) 是计算机界通用术语。Blob对象中的数据，JavaScript 中可能并没有对应的类型，比如各种类型的图片。
Blob 对象最常见的用途，就是图片二进制形式的上传与下载。
##4.1 Blob 语法：
###4.1.1 构造函数
```
Blob Blob(
  Array blobParts,
  [可选] BlobPropertyBag properties
);
```
参数：

- blobParts：包含任意多个元素的数组，内容串联后添加到 Blob 对象中。数组元素可以是的ArrayBuffer，ArrayBufferView(typed array)，Blob，或者 DOMString 对象。
- properties：对象，设置Blob对象的一些属性。目前仅支持 type 属性，表示 Blob 的类型：`{type : 'application/json'}`。

示例：
```
var debug = {hello: "world"};
var blob = new Blob([JSON.stringify(debug, null, 2)],
  {type : 'application/json'});
```
###4.1.2 属性
|属性|类型|解释|
|-|-|-|
|Blob.size| unsigned long|只读，Blob 对象中所包含数据的大小（字节）。|
|Blob.type| DOSString|只读，字符串，表明该 Blob 对象所包含数据的 MIME 类型。如果类型未知，则该值为空字符串。|
###4.1.3 方法
`Blob.slice([start,[ end ,[contentType]]])`
返回一个新的 Blob对象，包含了源 Blob对象中指定范围内的数据。跟 JavaScript 中数组和字符串的 slice 函数用法一致。
参数：

- start：开始索引，可以为负数，语法类似于数组的slice方法。默认值为0。
- end：结束索引，可以为负数，语法类似于数组的slice方法。默认值为最后一个索引。
- contentType：新的 Blob 对象的 MIME 类型，这个值将会成为新的 Blob 对象的 type 属性的值，默认为一个空字符串。
##4.2 示例
###4.2.1 [XMLHttpRequest 请求并处理二进制流（图片）](https://segmentfault.com/q/1010000000443286)
POST 方式，数据格式 Blob，此时图片的 URL 地址既不是 HTTP，也不是 Base64 URL，而是 Blob 形式：
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
###4.2.2 从 Blob 中提取数据
**从 Blob 中读取内容的唯一方法是使用 FileReader**。通过使用 FileReader 的其它方法可以把 Blob 读取为字符串或者 data URL。以下代码将 Blob 的内容作为类型数组读取：
```
var reader = new FileReader();
reader.addEventListener("loadend", function() {
   // reader.result 就是 blob 中的数据转化为的类型数组
});
reader.readAsArrayBuffer(blob);
```
###4.2.3 使用 Blob 创建一个指向类型数组的 URL
```
var typedArray = GetTheTypedArraySomehow();
var blob = new Blob([typedArray], {type: "application/octet-binary"});// 传入一个合适的MIME类型
var url = URL.createObjectURL(blob);
// 会产生一个类似blob:d3958f5c-0777-0845-9dcf-2cb28783acaf 这样的URL字符串
// 你可以像使用一个普通URL那样使用它，比如用在 img.src 上。
```
#5. File
通常情况下， File 对象是来自用户在一个 `<input>` 元素上选择文件后返回的 [FileList](https://developer.mozilla.org/en-US/docs/Web/API/FileList)  对象，也可以是来自由拖放操作生成的 [DataTransfer](https://developer.mozilla.org/en-US/docs/Web/API/DataTransfer) 对象。

File 接口基于 Blob，继承并扩展了 Blob 的功能使其支持用户系统上的文件，且可以用在任意的 Blob 类型的 context 中。比如说， FileReader，URL.createObjectURL()，createImageBitmap()，及 XMLHttpRequest.send() 都能处理 Blob  和 File。
[从Web应用程序使用文件可以参考这里](https://developer.mozilla.org/en-US/docs/Using_files_from_web_applications)。
##5.1 属性和方法
除了从 Blob 对象继承的属性与方法，只有两个可用的方法：
`File.lastModifiedDate`：返回当前 File 对象所引用文件最后修改时间， 自 1970年1月1日0:00 以来的毫秒数。
`File.name`：返回当前 File 对象所引用文件的名字。
#6. ArrayBuffer
>The ArrayBuffer object is used to represent a generic, fixed-length raw binary data buffer. You cannot directly manipulate the contents of an ArrayBuffer; instead, you create one of the typed array objects or a DataView object which represents the buffer in a specific format, and use that to read and write the contents of the buffer.

ArrayBuffer 对象表示**通用的固定长度的**二进制数据缓冲区，**用于把数据源提前写入在内存中，长度固定**。
**ArrayBuffer 的内容不能直接操纵**，但是可以通过类型化数组 Typed Arrays 或 DataView 对象来操纵。
Blob 对象中也是二进制数据，Blob 对象可以 append 追加 ArrayBuffer 类型的数据。

JavaScript 类型数组 Typed Arrays 的资料[看这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Typed_arrays)。
##6.1 创建 ArrayBuffer
可以通过构造函数创建指定字节数的 ArrayBuffer，或 FileReader 对象的 readAsArrayBuffer 方法从 Blob 创建 ArrayBuffer。
```
new ArrayBuffer(length) // 创建指定字节数的 ArrayBuffer
FileReader.readAsArrayBuffer(blob) // 从 Blob 创建 ArrayBuffer
```

##6.2 DataView 对象
[MDN - DataView API 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/DataView)

DataView 视图是一个可以从 ArrayBuffer 对象中读写多种数值类型的底层接口，在读写时不用考虑平台字节序问题。

###6.2.1 构造函数
```
new DataView(buffer [, byteOffset [, byteLength]])
```
参数：

- buffer：DataView 对象的数据源，ArrayBuffer 或 SharedArrayBuffer  对象。
- byteOffset：可选，此 DataView 对象的第一个字节在 buffer 中的偏移。如果不指定则默认从第一个字节开始。
- byteLength：可选，此 DataView 对象的字节长度。如果不指定则默认与 buffer 的长度相同。

示例：
```
var view1 = new DataView(buffer);
var view2 = new DataView(buffer,12,4); // 从第 12 字节开始，取 4 字节
```
###6.2.2 属性
所有 DataView 实例都继承自 DataView.prototype，我们也可以向 DataView 对象中添加其他属性。
|属性|解释|
|-|-|-|
|DataView.prototype.constructor|指定用来生成原型的构造函数.初始化值是标准内置DataView构造器|
|DataView.prototype.buffer| 只读，被视图引入的ArrayBuffer.创建实例的时候已固化因此是只读的|
|DataView.prototype.byteLength| 只读，从 ArrayBuffer中读取的字节长度. 创建实例的时候已固化因此是只读的|
|DataView.prototype.byteOffset| 只读，从 ArrayBuffer读取时的偏移字节长度. 创建实例的时候已固化因此是只读的|
###6.2.3 方法
####1. 读：
|方法名|解释|
|-|-|
|DataView.prototype.getInt8()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处获取一个8-bit数(一个字节)|
|DataView.prototype.getUint8()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处获取一个8-bit数(无符号字节)|
|DataView.prototype.getInt16()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处获取一个16-bit数(短整型)|
|DataView.prototype.getUint16()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处获取一个16-bit数(无符号短整型)|
|DataView.prototype.getInt32()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处获取一个32-bit数(长整型)|
|DataView.prototype.getUint32()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处获取一个32-bit数(无符号长整型)|
|DataView.prototype.getFloat32()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处获取一个32-bit数(浮点型)|
|DataView.prototype.getFloat64()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处获取一个64-bit数(双精度浮点型)|

####2. 写：
|方法名|解释|
|-|-|
|DataView.prototype.setInt8()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处储存一个8-bit数(一个字节)|
|DataView.prototype.setUint8()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处储存一个8-bit数(无符号字节)|
|DataView.prototype.setInt16()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处储存一个16-bit数(短整型)|
|DataView.prototype.setUint16()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处储存一个16-bit数(无符号短整型)|
|DataView.prototype.setInt32()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处储存一个32-bit数(长整型)|
|DataView.prototype.setUint32()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处储存一个32-bit数(无符号长整型)|
|DataView.prototype.setFloat32()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处储存一个32-bit数(浮点型)|
|DataView.prototype.setFloat64()|从DataView起始位置以byte为计数的指定偏移量(byteOffset)处储存一个64-bit数(双精度浮点型)|

###6.2.4 示例
```
var buffer = new ArrayBuffer(16);
var dv = new DataView(buffer, 0);

dv.setInt16(1, 42);
dv.getInt16(1); //42
```
##6.3 类型化数组
[MDN - Typed_arrays API 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Typed_arrays)

>JavaScript 类型化数组是一种类似数组的对象，并提供了一种用于访问原始二进制数据的机制。 正如你可能已经知道，Array 存储的对象能动态增多和减少，并且可以存储任何 JavaScript 值。JavaScript 引擎会做一些内部优化，以便对数组的操作可以很快。然而，随着 Web 应用程序变得越来越强大，尤其一些新增加的功能例如：音频视频编辑，访问 WebSockets 的原始数据等，很明显有些时候如果使用 JavaScript 代码可以快速方便地通过类型化数组来操作原始的二进制数据将会非常有帮助。
但是，不要把类型化数组与正常数组混淆，因为在类型数组上调用  Array.isArray()  会返回false。此外，并不是所有可用于正常数组的方法都能被类型化数组所支持（如 push 和 pop）。

JavaScript 类型数组（Typed Arrays）将实现拆分为缓冲和视图两部分。缓冲（由 ArrayBuffer 对象实现）描述的是一个数据块。缓冲没有格式可言，并且不提供机制访问其内容。为了访问在缓冲对象中包含的内存，你需要使用视图。视图提供了上下文 — 即数据类型、起始偏移量和元素数 — 将数据转换为实际有类型的数组。

类型化数组视图具有自描述性的名字和所有常用的数值类型像 Int8，Uint32，Float64 等等。有一种特殊类型的数组 Uint8ClampedArray，仅操作0到255之间的数值，这对于 Canvas 数据处理非常有用。
###6.3.1 类型化数组常用的数值类型
|Type|  Value Range|  Size in bytes|  Description|  Web IDL type| Equivalent C type|
|-|
|Int8Array| -128 to 127|  1|  8-bit two's complement signed integer|  byte| int8_t|
|Uint8Array|  0 to 255| 1|  8-bit unsigned integer| octet|  uint8_t
|Uint8ClampedArray| 0 to 255| 1|  8-bit unsigned integer (clamped)| octet|  uint8_t
|Int16Array|  -32768 to 32767|  2|  16-bit two's complement signed integer| short|  int16_t
|Uint16Array| 0 to 65535| 2|  16-bit unsigned integer|  unsigned short| uint16_t
|Int32Array|  -2147483648 to 2147483647|  4|  32-bit two's complement signed integer| long| int32_t
|Uint32Array| 0 to 4294967295|  4|  32-bit unsigned integer |unsigned long| uint32_t
|Float32Array|  1.2x10-38 to 3.4x1038|  4|  32-bit IEEE floating point number ( 7 significant digits e.g. 1.1234567)| unrestricted float| float
|Float64Array|  5.0x10-324 to 1.8x10308|  8|  64-bit IEEE floating point number (16 significant digits e.g. 1.123...15)|  unrestricted double|  double
###6.3.2 类型数组的 API
|方法|解释|
|-|
|FileReader.prototype.readAsArrayBuffer()|读取对应的 Blob 或 File的内容|
|XMLHttpRequest.prototype.send()|XMLHttpRequest 实例的 send() 方法参数支持类型化数组和 ArrayBuffer 对象。|
|ImageData.data|是 Uint8ClampedArray 对象，用来描述包含按照 RGBA 序列的颜色数据的一维数组，其值的范围 [0, 255]。|
##6.4 示例
###6.4.1 使用视图和缓冲
```
// 创建一个16字节固定长度的缓冲，初始化为0
var buffer = new ArrayBuffer(16);

// 确认一下数据的字节长度
if (buffer.byteLength === 16) {
  console.log("Yes, it's 16 bytes.");
} else {
  console.log("Oh no, it's the wrong size!");
}

// 需要创建视图来操作缓冲。缓冲内的数据被格式化为32位的有符号整数数组
var int32View = new Int32Array(buffer);
// 可以像普通数组一样访问该数组中的元素：
for (var i = 0; i < int32View.length; i++) {
  int32View[i] = i * 2;
}
```
该代码会将数组以0, 2, 4和6填充 （一共4个4字节元素，所以总长度为16字节）。
###6.4.2 同一数据的多个视图
在同一数据上可以创建多个视图。
```
// 创建一个16字节固定长度的缓冲，初始化为0
var buffer = new ArrayBuffer(16);

// 需要创建视图来操作缓冲。缓冲内的数据被格式化为32位的有符号整数数组
var int32View = new Int32Array(buffer);
// 可以像普通数组一样访问该数组中的元素：
for (var i = 0; i < int32View.length; i++) {
  int32View[i] = i * 2;
}
var int16View = new Int16Array(buffer);

for (var i = 0; i < int16View.length; i++) {
  console.log("Entry " + i + ": " + int16View[i]);
}
```
创建的2字节整数视图共享上文的4字节整数视图的缓冲，然后以2字节整数打印出缓冲里的数据，得到0, 0, 2, 0, 4, 0, 6, 0这样的输出。
###6.4.3 转换为普通数组
在处理完一个类型化数组后，有时需要把它转为普通数组，以便使用数组的方法。可以调用 Array.from 实现这种转换，如果不支持 Array.from 的话，也可以通过如下代码实现：
```
var typedArray = new Uint8Array([1, 2, 3, 4]),
    normalArray = Array.prototype.slice.call(typedArray);
normalArray.length === 4;
normalArray.constructor === Array;
```