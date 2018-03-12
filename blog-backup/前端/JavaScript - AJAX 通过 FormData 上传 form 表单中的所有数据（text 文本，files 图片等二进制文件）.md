[FormData 对象的使用](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData/Using_FormData_Objects)
[FormData 定义](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)

#1. FormData 对象
**FormData 对象是 ES6 中的对象，IE10 以下不兼容。**
利用FormData对象，可以通过 JavaScript 用键值对来模拟一系列表单控件，还可以使用 XMLHttpRequest 的 send() 方法来异步的提交这个"表单"。比起普通的 ajax，使用 FormData 的最大优点就是我们可以异步上传一个二进制文件。如果把表单的编码类型设置为 multipart/form-data ，则通过FormData 传输的数据格式和表单通过 submit() 方法传输的数据格式相同。
##1.1 FormData 对象创建方法
FormData 对象创建的两种方法：创建空 FormData 对象，然后调用 append() 方法添加字段；或通过HTML表单创建FormData对象。

- 创建空 FormData 对象，然后调用 append() 方法添加字段：
```javascript
var formData = new FormData();

formData.append("username", "Groucho");
formData.append("accountnum", 123456); // 数字 123456 会被立即转换成字符串 "123456"
formData.append("userfile", fileInputElement.files[0]); // HTML 文件类型input，由用户选择

// JavaScript file-like 对象
var content = '<a id="a"><b id="b">hey!</b></a>'; // 新文件的正文...
var blob = new Blob([content], { type: "text/xml"});

formData.append("webmasterfile", blob);

var request = new XMLHttpRequest();
request.open("POST", "submitform.php");
request.send(formData);
```
- 通过HTML表单创建FormData对象：
```
// HTML 
<form id="data" method="post" enctype="multipart/form-data">
    <input type="text" name="first" value="Bob" />
    <input name="image" type="file" />
    <button>Submit</button>
</form>

// JS，基于 jQuery
var formData = new FormData($("form#data"));

var request = new XMLHttpRequest();
request.open("POST", "submitform.php");
request.send(formData);
```
##1.2 FormData 对象支持的类型
FormData 对象支持 file 或 blob 类型的文件。
```
// 通过第三个可选参数设置发送请求的头 Content-Disposition 指定文件名。如果不指定文件名，将使用名字“blob”。
data.append("myfile", myBlob, "filename.txt");
```
##1.3 通过 jQuery 使用 FormData 对象
```
var fd = new FormData(document.querySelector("form"));
fd.append("CustomField", "This is some extra data");
$.ajax({
  url: "stash.php",
  type: "POST",
  data: fd,
  processData: false,  // 不处理数据
  contentType: false   // 不设置内容类型
});
```
##1.4 方法 append()
append() 方法用于给 FormData 对象添加一个键/值对。
```
void append(DOMString name, Blob value, optional DOMString filename);
void append(DOMString name, DOMString value);
```
参数值
name：字段名称。
value：字段值。非字符串时会被自动转换成字符串。
filename：(可选) 指定文件的文件名，当 value 参数被指定为一个 Blob 对象或者一个 File 对象时，该文件名会被发送到服务器上。对于Blob对象来说，这个值默认为"blob"。
注：如果将 Blob 对象作为字段值添加到 FormData 对象中，则在使用 Ajax 将这个 FormData 对象提交到服务器上时，提交数据中代表对应文件的文件名的"Content-Disposition"字段的值可能会因浏览器的不同而不同，默认为"blob"。
#2. 服务端取 form 数据（以 PHP 为例）
PHP 中，文本数据从 `$_POST` 中读取，二进制数据从 `$_FILES` 中读取。
```
<?php

print_r($_POST);
print_r($_FILES);
?>
```