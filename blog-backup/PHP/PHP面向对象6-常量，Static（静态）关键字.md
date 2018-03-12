在访问PHP类中的成员属性或方法时，如果属性或者方法被声明成 const（定义常量）或者static（声明静态），就必须使用类名或 `self` 配合`::`来访问。否则用实例对象或 `$this` 配合 `->`来访问。
```
class A {
  const NAME = 'jack';
  static $age = 20;
  $sex = 'male';

  public function f() {
    echo self::NAME; // 用 self 访问常量成员属性时，不加美元符号
    echo self::$age; // 用 self 访问静态成员属性时，加美元符号
    echo $this->sex; // 用 $this 伪变量访问成员属性时，不加美元符号
  }

  public static function f_static() {
    echo self::NAME;
    echo self::$age;
    // echo $this->$sex; // 静态方法可以不用实例化就调用，所以不能用 $this 伪变量
  }
}

echo A::NAME;
echo A::$age;
A::f_static();

$a = new A();
echo $a->f();
```
#一. 常量
###1. 常量特性
- 常量默认大小写敏感（用 define 函数设置常量时，第三个参数设为 TRUE 可以改为大小写不敏感）。通常常量标识符总是大写的。
- 常量只能包含标量数据（boolean、integer、float和string）。
- 常量可以分为2种：类常量，普通常量。
- 常量的定义有2种方法：const 关键字，define()方法。
- 常量可以不用理会变量的作用域，在任何地方定义和访问（使用命名空间时，const 和 define 会有区别）。
- 常量前面没有美元符号（$）。
###2. 类常量
可以把在类中始终保持不变的值定义为常量。在定义和使用常量的时候不需要使用 `$` 符号。
常量的值必须是一个定值，不能是变量，类属性，数学运算的结果或函数调用。
###3. 普通常量
可以用 `define()` 函数在运行时定义一个常量。也可以用 `const` 关键字。
```
class Father {
  const NAME = 'jack';

  //这里用 define 会报错，因为这里不能出现函数调用。属性中的变量可以初始化，但是初始化的值必须是常数，PHP 脚本在编译阶段时就可以得到其值，而不依赖于运行时的信息才能求值。
  //define("AGE", 20);

  public function __construct() {
    // 类内部调用的属性是常量时，必须用`self::`来访问
    echo self::NAME;
  }
  
  public function f(Father $o) {
    echo $o->NAME;
  }
}

$o = new Father();

// 类外部调用类的常量时，必须用`类名::`来访问
echo Father::NAME;
echo $o::NAME; // 自 5.3.0 起

// 常量定义时不需要`$`符号
const A = 'a';
echo A;
define(B, 'b');
echo B;

$className = 'Father';
echo $className::NAME; // 自 5.3.0 起
```
###4. const 关键字，define()方法的区别
- const 是一个语言结构，而 define 是一个函数。const 在编译时比 define 快。
- const 可用于声明类的成员常量，而 define 不可以（以为声明类的成员常量的地方不可以调用函数）。
- const 不能在条件语句中定义常量，而 define 函数可以在条件语句中使用。
```
if (true) { 
  const NAME = 'jack'; //这里用 const 会报错
  define("AGE", 20);
}
```
- const 声明的常量的名字必须是定值，不能是变量，而 define 可以采用表达式作为常量的名字。
- const 声明的常量的值：在 PHP 5 中，必须是一个定值，不能是变量，类属性，数学运算的结果或函数调用，在 PHP 7 中支持数学运算。而 define 的常量的值：在 PHP 5 中，value 必须是标量( integer、 float、string、boolean、NULL），在 PHP 7 中还允许是个 array。
```
const NAME= 1 + 2 + 3; // PHP 5 中会报错，PHP 7 没问题
define("MAN", array('jack', 'kali')); // PHP 5 中会报错，PHP 7 没问题
```
- const 定义的常量名字对大小写敏感，而define可以通过第三个参数（为true表示大小写不敏感，大小写不敏感的常量以小写的方式储存）来指定是否对大小写敏感。没必要啊。
- PHP 7 中还允许是个 array 的值。
- 在命名空间里，const 作用于当前空间，而 define 的作用是全局的。
###5. 检测常量是否存在： defined 函数
```
if (defined('TEST')) {
    echo TEST;
}
```
#二. static关键字
static 关键字定义静态方法，静态属性，静态变量以及后期静态绑定。
**静态声明是在编译时解析的。**
###1. 静态方法
静态方法可以不实例化类而直接访问，也**可以通过一个类已实例化的对象来访问**。
由于静态方法不需要通过对象即可调用，所以**伪变量 $this 在静态方法中不可用**。
用静态方式调用一个非静态方法会导致一个 E_STRICT 级别的错误。
###2. 静态属性
静态属性可以不实例化类而直接访问，但是**不能通过一个类已实例化的对象来访问**。
静态属性不可以由对象通过 -> 操作符来访问。
静态属性只能被初始化为文字或常量，不能使用表达式。所以可以把静态属性初始化为整数或数组，但不能初始化为另一个变量或函数返回值，也不能指向一个对象。
```
class A {
    static $a = 1;
    
    public static function f() {
        echo self::$a;
        // echo $this->$a; // 静态方法内无法用 $this 伪变量
        echo "f()";
    }
    
    public function f2() {
        echo self::$a; // 非静态方法可以调用静态属性
        self::f();
        $this->f(); // 非静态方法可以调用静态方法
    }
}

echo A::$a;
A::f();

$b = new A();
// echo $b->$a; // 这里会报错，静态属性不能用已实例化的对象来访问。
$b->f(); // 静态方法可以用已实例化的对象来访问。
$b->f2();
```
###3. 静态变量
####1) 在函数作用域中存在时，当程序执行离开此作用域时，静态变量值并不丢失
```
function test() {
    static $a = 0; // 不管函数执行多少次，只初始化一次
    echo $a;
    $a++;
}
```
####2) 静态变量不可以存放引用值
可以把静态属性初始化为整数或数组，但不能初始化为另一个变量或函数返回值，也不能指向一个对象。
```
<?php
function &get_instance_ref() {
    static $obj;

    echo 'Static object: ';
    var_dump($obj);
    if (!isset($obj)) {
        // 将一个引用赋值给静态变量
        $obj = &new stdclass;
    }
    $obj->property++;
    return $obj;
}

function &get_instance_noref() {
    static $obj;

    echo 'Static object: ';
    var_dump($obj);
    if (!isset($obj)) {
        // 将一个对象赋值给静态变量
        $obj = new stdclass;
    }
    $obj->property++;
    return $obj;
}

$obj1 = get_instance_ref();
$still_obj1 = get_instance_ref();
echo "\n";
$obj2 = get_instance_noref();
$still_obj2 = get_instance_noref();
?>

//以上例程会输出：
Static object: NULL
Static object: NULL

Static object: NULL
Static object: object(stdClass)(1) {
["property"]=>
int(1)
}
```
当在函数中把一个引用赋值给一个静态变量时，再次调用函数时，静态变量并没有被记住之前的值。**可以把静态变量当做数组，每个数组元素就可以存放引用了。**
```
function config_item($item) {
  static $_config;

  if (empty($_config)) {
    // 引用不能直接分配给静态变量，所以我们使用一个数组
    $_config[0] =& get_config();
  }

  return isset($_config[0][$item]) ? $_config[0][$item] : NULL;
}
```
###4. 后期静态绑定
“后期绑定”的意思是说，static:: 不再被解析为定义当前方法所在的类，而是在实际运行时计算的。也可以称之为“静态绑定”，因为它可以用于（但不限于）静态方法的调用。
使用 self:: 或者 `__CLASS__` 对当前类的静态引用，取决于定义当前方法所在的类，和最初的调用者无关。而 static 则表示最初调用的那个类。
static只能调用静态属性。

实例一：
```
class A {
    public static function foo() {
        static::who(); // 后期静态绑定从这里开始
    }
 
    public static function who() {
        echo __CLASS__."\n";
    }
}
 
class B extends A {
    public static function test() {
        A::foo(); // 正常的静态方法调用，调用者是 A
        parent::foo(); // 调用 B 类的父类 A 的 foo 方法，并告诉 foo 方法最原始的调用者是 C
        self::foo(); // self 指定义该方法的类，即 B，但是 B 没有定义 foo 方法，访问父类的 foo 方法，它将原始的调用者 C 向上传递，
    }
 
    public static function who() {
        echo __CLASS__."\n";
    }
}
class C extends B {
    public static function who() {
        echo __CLASS__."\n";
    }
}
 
C::test(); // 类 C 的 test 方法从类 B 继承，原始调用者是 C
// 结果是ACC。因为类 B 和 C 都没有重写方法 foo()，所以最终执行的是类 A 的已经设置为后期静态绑定的方法。
```