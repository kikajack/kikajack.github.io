#1. form 表单的 `Content-Type` 类型
###1.1 通过 HTTP 的 POST 方式提交数据时，常见的数据类型有四种：

- `application/x-www-form-urlencoded`（`<form>` 表单默认数据类型）
- `multipart/form-data`（`<form>` 表单可以通过添加 enctype 属性实现：`<form method="post" action="xx.php" enctype="multipart/form-data>"`）
- `application/json` （AJAX 中常用。PHP 无法通过 `$_POST` 对象从这种请求中获得内容，需要从 `php://input` 里获得原始输入流，或者把 JSON 字符串作为 val，仍然放在键值对里，以 `x-www-form-urlencoded` 方式提交）
- `text/xml`

###1.2 form 表单的 `Content-Type` 类型：

- `Content-Type: application/x-www-form-urlencoded` ：**默认编码方式**。提交的数据按照 `key1=val1&key2=val2` 的方式进行编码，key 和 val 都进行了 URL 转码。例如 `title=AAA&val=123` ，在PHP 中用 `$_POST['title']` 可以获取到 name 为 title 的值 AAA。
- `Content-Type: multipart/form-data` ：可以通过 `enctype` 参数设置：`<form method="post" action="xx.php" enctype="multipart/form-data>"`。这种情况下的表单提交时，会自动在 `Content-Type: multipart/form-data;` 后面添加 `boundary` 字段，每一组字段之间用 `boundary` 字段分割。如果传输的是文件，还要包含文件名和文件类型信息。下面是一个示例：
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
#2. form 表单中实现同名字段的提交（radio 除外）
场景：有时候需要用表单填写**批量信息**，比如需要收集用户的两个以上的联系人姓名、照片、手机号等信息。一般会有一个模板，一个添加联系人的按钮，按一下复制一份模板，这时就会出现相同 name 的 input。当然可以在提交之前在页面上通过 JavaScript 处理数据。

form 表单不带 method 属性时，默认用 GET 方式提交数据。

对于同名的 input，只需要在 name 后面加方括号 `[]`（是 PHP 的性质而不是 HTML 规范）。PHP 会自动处理，把同名且含有方括号的 input，放在以这个名字命名的数组中。用 $_POST[‘name’] 就可以得到数据。示例：
```
<form method="POST" action="xx.php">
  <input type="text" name="code[]">
  <input type="text" name="code[]">
  <input type="checkbox" name="check[]" checked>
  <input type="checkbox" name="check[]" checked>
  <select name="select[]">
        <option value="v1"></option>
        <option value="v2"></option>
    </select>
    <select name="select[]">
        <option value="v3"></option>
        <option value="v4"></option>
    </select>
    <input type="file" name="file[]">
    <input type="file" name="file[]">
    <input type="submit">
</form>

// PHP 的 $_POST 中的数据（要确保文件上传表单的属性是 enctype="multipart/form-data"，否则文件上传不了。）
Array
(
    [code] => Array
        (
            [0] => 第一个 input 中输入的数据
            [1] => 第二个 input 中输入的数据
        )
  [check] => Array
        (
            [0] => on
            [1] => on
        )
    [select] => Array
        (
            [0] => 第一个 select 中选择的数据
            [1] => 第二个 select 中选择的数据
        )
    [file] => Array # 注意，表单属性是 enctype="application/x-www-form-urlencoded"时，这里可以看到 file，但是没法用。。。
        (
            [0] => 第一个文件
            [1] => 第二个文件
        )
)
// PHP 的 $_FILES 中的数据，file_get_contents($_FILES['file']['tmp_name'][0] 可以读内容
Array
(
    [file] => Array
        (
            [name] => Array
                (
                    [0] => id.txt
                    [1] => API.txt
                )

            [type] => Array
                (
                    [0] => text/plain
                    [1] => text/plain
                )

            [tmp_name] => Array
                (
                    [0] => C:\Users\THINK\AppData\Local\Temp\phpDC1F.tmp
                    [1] => C:\Users\THINK\AppData\Local\Temp\phpDC20.tmp
                )

            [error] => Array
                (
                    [0] => 0
                    [1] => 0
                )

            [size] => Array
                (
                    [0] => 185
                    [1] => 1820
                )

        )

)
```