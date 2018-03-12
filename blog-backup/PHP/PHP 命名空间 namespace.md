参考：
http://php.net/manual/zh/language.namespaces.rationale.php

#一. 概述
###1）命名空间定义和用途
命名空间是一种封装事物的方法，类似于文件系统。可以**解决命名冲突**，**提高代码可读性**。
在PHP中，命名空间用来解决在编写类库或应用程序时创建**可重用的代码**如类或函数时碰到的两类问题：

- 用户编写的代码与PHP内部的类/函数/常量或第三方类/函数/常量之间的名字冲突。
- 为很长的标识符名称(通常是为了缓解第一类问题而定义的)创建一个别名（或简短）的名称，提高源代码的可读性。
#二. 特性
###1. PHP保留的命名空间
名为PHP或php的命名空间，以及以这些名字开头的命名空间（例如PHP\Classes）被保留用作语言内核使用，而不应该在用户空间的代码中使用。
###2. 全局命名空间
所有的类与函数的定义默认都是在全局空间，除非明确使用命名空间。在名称前加上**前缀 `\` 表示该名称是全局空间中的名称**，即使该名称位于其它的命名空间中时也是如此。
###3. 受命名空间影响的代码
虽然任意合法的PHP代码都可以包含在命名空间中，但只有以下类型的代码受命名空间的影响，它们是：**类（包括抽象类和[traits](http://php.net/manual/zh/language.oop5.traits.php)）、接口、函数和常量**。
###4. 命名空间可以嵌套，形成子命名空间。
`namespace MyProject\Sub\Level;`
###5. PHP 遇到非限定的类、函数或常量名称时，使用不同的优先策略
参考[后备全局函数/常量](http://php.net/manual/zh/language.namespaces.fallback.php)。
类名称总是解析到当前命名空间中的名称。因此在访问系统内部或不包含在命名空间中的类名称时（比如 Exception），必须使用完全限定名称。
对于函数和常量来说，如果当前命名空间中不存在该函数或常量，PHP 会退而使用全局空间中的函数或常量。
###6. 引入文件时，文件中的代码不受所在命名空间影响
```
Included files will default to the global namespace. 
<?php 
//test.php 
namespace test { 
  include 'test1.inc'; 
  echo '-',__NAMESPACE__,'-<br />'; 
} 
?> 

<?php 
//test1.inc 
  echo '-',__NAMESPACE__,'-<br />'; 
?> 

Results of test.php: 

-- 
-test-
```
###7. 导入操作是在编译执行的，但动态的类名称、函数名称或常量名称则不是。参考[名称解析规则](http://php.net/manual/zh/language.namespaces.rules.php)
```
<?php
use My\Full\Classname as Another, My\Full\NSname;

$obj = new Another; // 导入时（编译阶段）实例化一个 My\Full\Classname 对象
$a = 'Another';
$obj = new $a;      // 运行时实例化一个 Another 对象
?>
```
#三. 用法
命名空间通过关键字namespace 来声明。如果一个文件中包含命名空间，它必须在其它所有代码之前声明命名空间，除了一个以外：declare关键字。所有非 PHP 代码包括空白符都不能出现在命名空间的声明之前：
##1）定义命名空间
###1. 定义单个命名空间
```
<?php         //尖括号之前不能有任何字符
declare(encoding='UTF-8');
namespace MyProject;  //必须是代码的最开始，除了 declare 关键字

const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }

?>
```
###2. 定义子命名空间
```
<?php
namespace MyProject\Sub\Level;
const CONNECT_OK = 1;
class Connection { /* ... */ }
function connect() { /* ... */  }
?>
```
上面的例子创建了常量MyProject\Sub\Level\CONNECT_OK，类 MyProject\Sub\Level\Connection和函数 MyProject\Sub\Level\connect。
###3. 在同一个文件中定义多个命名空间（不推荐）
大括号语法
```
<?php
namespace MyProject {
  const CONNECT_OK = 1;
  class Connection { /* ... */ }
  function connect() { /* ... */  }
}

namespace AnotherProject {
  const CONNECT_OK = 1;
  class Connection { /* ... */ }
  function connect() { /* ... */  }
}
?>
```
将全局的非命名空间中的代码与命名空间中的代码组合在一起，只能使用大括号形式的语法。全局代码必须用一个不带名称的 namespace 语句加上大括号括起来，例如：
```
<?php
namespace MyProject {
  const CONNECT_OK = 1;
  class Connection { /* ... */ }
  function connect() { /* ... */  }
}

namespace { // global code
  session_start();
  $a = MyProject\connect();
  echo MyProject\Connection::start();
}
?>
```
##2）使用命名空间
####文件系统有 2 种访问方式（3种表现形式）：

1. 相对路径：所有操作都基于当前目录。例如：直接访问当前目录下的文件 `a.txt` 或 访问当前目录下的嵌套目录中的文件 `dir/a.txt`，会找当前目录下的 `a.txt`文件或当前目录下的 dir 目录下的 `a.txt` 文件。
2. 绝对路径：所有操作基于根目录。例如：`/home/dir/a.txt`，先找到根目录下的 home 目录，再找到 home 目录下的 dir 目录，最后在 dir 目录下找 `a.txt` 文件。

####PHP 命名空间中的元素使用同样的原理：

1. 非限定名称（名称中不包含命名空间分隔符的标识符，例如 Foo），所有操作基于当前命名空间（使用命名空间的话）或全局命名空间（全局代码）。注意[后备全局函数/常量](http://php.net/manual/zh/language.namespaces.fallback.php)。
例如 `$a=new foo();` 或 `foo::staticmethod();`。如果当前命名空间是 currentnamespace，foo 将被解析为 currentnamespace\foo。如果使用 foo 的代码是全局的，不包含在任何命名空间中的代码，则 foo 会被解析为 foo。 
2. 限定名称（名称中含有命名空间分隔符的标识符，例如 Foo\Bar），所有操作基于当前命名空间（使用命名空间的话）或全局命名空间（全局代码）。
例如 `$a = new subnamespace\foo();` 或 `subnamespace\foo::staticmethod();`。如果当前的命名空间是 currentnamespace，则 foo 会被解析为 currentnamespace\subnamespace\foo。如果使用 foo 的代码是全局的，不包含在任何命名空间中的代码，foo 会被解析为subnamespace\foo。
3. 完全限定名称（用 `\` 或 namespace 开头），或包含了全局前缀操作符的名称，所有操作基于根命名空间。例如， `$a = new \currentnamespace\foo();` 或 `\currentnamespace\foo::staticmethod();`。在这种情况下，foo 总是被解析为代码中的文字名(literal name)currentnamespace\foo。
访问任意全局类、函数或常量，都可以使用完全限定名称，例如 \strlen() 或 \Exception 或 \INI_ALL。
```
<?php
namespace foo;
use blah\blah as foo;

$a = new my\name();   // "foo\my\name" 类的实例
foo\bar::name();    // 调用 "blah\blah\bar" 类中的静态方法 name
my\bar();         // 调用方法 "foo\my\bar"
$a = my\BAR;      // 将 $a 设置为常量 "foo\my\BAR" 的值
$b = strlen('hi');    // 调用全局函数 "strlen" ，因为 "foo\strlen" 不存在
$c = INI_ALL;       // 将 $b 设置为全局常量 "INI_ALL"，因为 "foo\INI_ALL" 不存在
?>
```
##3）用别名引入外部命名空间
允许通过别名引用或导入外部的**完全限定名称**，是命名空间的一个重要特征。这有点类似于在类 unix 文件系统中可以创建对其它的文件或目录的符号连接。
所有支持命名空间的PHP版本支持三种别名或导入方式：为类名称使用别名、为接口使用别名或为命名空间名称使用别名。PHP 5.6开始允许导入函数或常量或者为它们设置别名。
导入外部命名空间时，前导的反斜杠是不必要的也不推荐的，因为导入的名称必须是完全限定的，不会根据当前的命名空间作相对解析。
```
<?php
namespace foo;
use blah\blah as foo; //
use My\Full\Classname as Another, My\Full\NSname; //可以一次导入多个命名空间

// aliasing a function (PHP 5.6+)
use function My\Full\functionName as func;

// importing a constant (PHP 5.6+)
use const My\Full\CONSTANT;
...
...
```
#四. `namespace` 关键字和 `__NAMESPACE__` 常量
###1. namespace 关键字
关键字 namespace 可用来显式访问当前命名空间或子命名空间中的元素。它等价于类中的 self 操作符。
```
<?php
namespace MyProject;

use blah\blah as mine; // see "Using namespaces: importing/aliasing"

blah\mine(); // calls function MyProject\blah\mine()
namespace\blah\mine(); // calls function MyProject\blah\mine()

namespace\func(); // calls function MyProject\func()
namespace\sub\func(); // calls function MyProject\sub\func()
namespace\cname::method(); // calls static method "method" of class MyProject\cname
$a = new namespace\sub\cname(); // instantiates object of class MyProject\sub\cname
$b = namespace\CONSTANT; // assigns value of constant MyProject\CONSTANT to $b
?>
```
###2. `__NAMESPACE__` 常量
常量 `__NAMESPACE__` 的值是包含当前命名空间名称的字符串。在全局的，不包括在任何命名空间中的代码，它包含一个空的字符串。
常量 `__NAMESPACE__` 在动态创建名称时很有用。
```
<?php
namespace MyProject;

echo '"', __NAMESPACE__, '"'; // 输出 "MyProject"
?>


<?php

echo '"', __NAMESPACE__, '"'; // 输出 ""
?>

Example #3 使用__NAMESPACE__动态创建名称

<?php
namespace MyProject;

function get($classname)
{
    $a = __NAMESPACE__ . '\\' . $classname;
    return new $a;
}
?>
```