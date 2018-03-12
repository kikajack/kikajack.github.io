json_decode要求的字符串比较严格，不满足以下情况就报错：

1. 使用 UTF-8 编码
2. 必须符合 JSON 格式，不能在最后出现逗号，不能使用单引号
3. 不能有控制字符（换行，tab 等）
` $result =  preg_replace('/[\x00-\x1F\x80-\x9F]/u', '', trim($result));`

#1. json_last_error
如果编码或解码时报错，返回 JSON 编码解码时最后发生的错误。[参考官网](http://php.net/manual/zh/function.json-last-error.php)
`int json_last_error ( void )`

返回数字对应的含义：
0 = JSON_ERROR_NONE，没有错误
1 = JSON_ERROR_DEPTH，堆栈超过指定的最大深度
2 = JSON_ERROR_STATE_MISMATCH，模式不匹配
3 = JSON_ERROR_CTRL_CHAR，**非法的 UTF8 序列（控制字符），比如换行符**
4 = JSON_ERROR_SYNTAX，**语法错误，比如单引号**
5 = JSON_ERROR_UTF8，**不是 UTF8 编码**
6 = JSON_ERROR_RECURSION
7 = JSON_ERROR_INF_OR_NAN
8 = JSON_ERROR_UNSUPPORTED_TYPE
#2. json_encode
对变量进行 JSON 编码。
`string json_encode ( mixed $value [, int $options = 0 [, int $depth = 512 ]] )`
```
参数：

$value：待编码的 value ，除了resource 类型之外，可以为任何数据类型。所有字符串数据的编码必须是 UTF-8。
$options:由以下常量组成的二进制掩码： JSON_HEX_QUOT, JSON_HEX_TAG, JSON_HEX_AMP, JSON_HEX_APOS, JSON_NUMERIC_CHECK, JSON_PRETTY_PRINT, JSON_UNESCAPED_SLASHES, JSON_FORCE_OBJECT, JSON_PRESERVE_ZERO_FRACTION, JSON_UNESCAPED_UNICODE, JSON_PARTIAL_OUTPUT_ON_ERROR。 关于 JSON 常量详情参考JSON 常量页面。
$depth:设置最大深度。 必须大于0。
```
#3. json_decode
对 JSON 格式的字符串进行解码。
```
参数:

$json:待解码的 json string 格式的字符串。这个函数仅能处理 UTF-8 编码的数据。
$assoc:当该参数为 TRUE 时，将返回 array 而非 object 。
$depth:指定递归深度。
$options:JSON解码的掩码选项。 现在有两个支持的选项。 第一个是JSON_BIGINT_AS_STRING， 用于将大整数转为字符串而非默认的float类型。第二个是 JSON_OBJECT_AS_ARRAY， 与将assoc设置为 TRUE 有相同的效果。

返回值:
正常返回对象或数组，无法解析 或 数据递归深度超过限制时，返回 NULL。
```