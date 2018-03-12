##1.PHP数据类型
字符串、整数、浮点数、逻辑、数组、对象、NULL。
##2.类型判断
`is_array()、is_float()、is_int()、is_integer()、is_string() 和 is_object()。 `
类型匹配时，以上方法会返回TRUE，否则返回FALSE。
##3.类型转换
PHP数据类型有三种转换方式：
1.  在要转换的变量之前加上用括号括起来的目标类型 
```
•（int）、（integer）：转换成整形 
•（float）、（double）、（real）：转换成浮点型 
•（string）：转换成字符串 
•（bool）、（boolean）：转换成布尔类型 
•（array）：转换成数组 
•（object）：转换成对象 
```
2.  使用3个具体类型的转换函数，intval()、floatval()、strval() 
3.  使用通用类型转换函数settype(mixed var,string type) 
```
type 的可能值为： 
“boolean” （或为“bool”，从 PHP 4.2.0 起） 
“integer” （或为“int”，从 PHP 4.2.0 起） 
“float” （只在 PHP 4.2.0 之后可以使用，对于旧版本中使用的“double”现已停用） 
“string” 
“array” 
“object” 
“null” （从 PHP 4.2.0 起） 
```
