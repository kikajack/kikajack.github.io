##1.变量
PHP中，变量支持的数据类型有：int整数，float（等同于double，双精度）实数，string字符串，bool布尔值，array数组，object对象。此外有2个特殊类型：NULL（空，未赋值、已重置或赋值为NULL的变量就是NULL类型）和resource（资源，保存了到外部资源的一个引用，特定的内置函数如数据库函数，返回resource类型的变量）。
**PHP变量名对大小写敏感。**
###1.变量类型获取、判断及转换
####1.类型获取
string gettype(\$var)：官方不建议使用，速度慢，可能以后会变。
####2.类型判断
bool is_integer(\$var)— is_int() 的别名：检测变量是否是整数，是则返回true。
bool is_numeric(\$var)：检测变量是否为数字或数字字符串。
bool is_float(\$var)：检测变量是否是浮点型。
bool is_string(\$var)：检测变量是否是字符串。
bool is_bool(\$var)：检测变量是否是布尔型。
bool is_scalar ( mixed $var )：检测变量是否是一个标量，标量变量是指那些包含了 integer、float、string 或 boolean的变量，而 array、object 和 resource 则不是标量。
bool is_array(\$var)：检测变量是否是数组。
bool is_object(\$var)：检测变量是否是对象。
bool is_null(\$var)：检测变量是否是NULL。
bool is_resource(\$var)：检测变量是否是资源类型。
####3.类型转换
- 直接在变量前用(int),(float)...转换。
- 用settype()转换类型
bool settype ( mixed &\$var , string \$type )
- 也可以用每个类型都有的转换方法。
int intval ( mixed \$var [, int \$base = 10 ] )通过使用指定的进制 base 转换（默认是十进制）获取变量的整数值。最大的值取决于操作系统。
float floatval ( mixed \$var )返回变量 var 的 float 数值。
string strval ( mixed \$var )获取变量的字符串值。


###2.变量作用域（global作用域和local作用域）
内置超级全局变量在脚本的任何地方都可见。
static（静态）：函数内创建并被声明为静态的变量无法在函数外部可见。
global（全局）：在一个脚本中声明的全局变量（函数外），整个脚本可见，但函数内不可见。
local（局部）：在函数内声明，仅在函数内可见。函数终止时变量回收。
函数内声明的变量为全局变量时，变量名需要与全局变量一样。

###3.global关键字
需要在函数内访问全局变量时，需要global关键字。
```PHP
<?php

$g = 5;
f();

function f() {
  global $g; //引入全局变量$g
  echo $g;
}
```
PHP 同时在名为 $GLOBALS[index] 的数组中存储了所有的全局变量。下标存有变量名。这个数组在函数内也可以访问，并能够用于直接更新全局变量。
```PHP
<?php

$g = 5;
f();

function f() {
  echo $GLOBALS['g'];
}
```
###4.static关键字
静态全局变量，无意义。

通常，当函数完成/执行后，会删除函数内的所有局部变量。首次声明变量时使用 static 关键词，则不会删除该局部变量。可以在函数的多次执行过程中保持该值（普通变量每次都会初始化）。每当函数被调用时，这个变量所存储的信息都是函数最后一次被调用时所包含的信息。
```PHP
<?php

$g = 5;
f();
f();
f();

function f() {
    static $x = 5;
    echo $x; //依次打印567。如果是普通变量，则打印555。
    $x ++;
}
```
###5.可变变量
变量名不能在编程时确定时，需要可变变量。
\$name = 'varname';
\$\$name = 5; //等价于\$varname = 5;
###6.变量赋值
####1.传值赋值
当一个变量的值赋予另外一个变量时，开辟新的存储空间存放变量。改变其中一个变量的值，将不会影响到另外一个变量。
变量默认总是传值赋值。
####2.引用赋值
新变量引用了原始变量。改动新的变量将影响到原始变量，反之亦然。
用法：将一个 & 符号加到将要赋值的变量前（源变量），比如`$a = &$b;`
注意：只有有名字的变量才可以引用赋值。比如`$a = &(7 + 2)`
##2.常量
常量的范围是全局的。不用管作用区域就可以在脚本的任何地方访问常量。
###1.自定义常量
define("FOO",     "something"); //只能在类外面用
###2.类常量
常量的值必须是一个定值，不能是变量，类属性，数学运算的结果或函数调用。
const VALUE = 0.0;   //类内或类外都可以用
```PHP
<?php
class MyClass
{
    const constant = 'constant value';

    function showConstant() {
        echo  self::constant . "\n";
    }
}

echo MyClass::constant . "\n";

$classname = "MyClass";
echo $classname::constant . "\n"; // 自 5.3.0 起

$class = new MyClass();
$class->showConstant();

echo $class::constant."\n"; // 自 PHP 5.3.0 起
?>
```
###3.魔术常量
|名称|意思|
|:-----:|-----|
|\__LINE__  |文件中的当前行号。|
|\__FILE__  |文件的完整路径和文件名。如果用在被包含文件中，则返回被包含的文件名。自 PHP 4.0.2 起，__FILE__ 总是包含一个绝对路径（如果是符号连接，则是解析后的绝对路径），而在此之前的版本有时会包含一个相对路径。|
|\__DIR__ |文件所在的目录。如果用在被包括文件中，则返回被包括的文件所在的目录。它等价于 dirname(__FILE__)。除非是根目录，否则目录中名不包括末尾的斜杠。（PHP 5.3.0中新增） =|
|\__FUNCTION__  |函数名称（PHP 4.3.0 新加）。自 PHP 5 起本常量返回该函数被定义时的名字（区分大小写）。在 PHP 4 中该值总是小写字母的。|
|\__CLASS__ |类的名称（PHP 4.3.0 新加）。自 PHP 5 起本常量返回该类被定义时的名字（区分大小写）。在 PHP 4 中该值总是小写字母的。类名包括其被声明的作用区域（例如 Foo\Bar）。注意自 PHP 5.4 起 __CLASS__ 对 trait 也起作用。当用在 trait 方法中时，__CLASS__ 是调用 trait 方法的类的名字。|
|\__TRAIT__ |Trait 的名字（PHP 5.4.0 新加）。自 PHP 5.4 起此常量返回 trait 被定义时的名字（区分大小写）。Trait 名包括其被声明的作用区域（例如 Foo\Bar）。|
|\__METHOD__  |类的方法名（PHP 5.0.0 新加）。返回该方法被定义时的名字（区分大小写）。|
|\__NAMESPACE__ |当前命名空间的名称（区分大小写）。此常量是在编译时定义的（PHP 5.3.0 新增）。|