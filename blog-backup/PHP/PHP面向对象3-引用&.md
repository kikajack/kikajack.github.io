PHP中，可以引用的有变量，函数，对象。引用符号统一用&。
引用，实际上就是用不同的名字访问同一个变量内容。引用不是指针，不指向实际的内存。
PHP中的引用并不是通常意义的引用，并不是指针，就行PHP中的重载并不是真正的重载一样！！！
链接：http://php.net/manual/zh/language.references.php
##1.变量引用
指多个变量名指向同一个变量，每次改动都影响其他指向此位置的变量。
```php
$a = 3;
$b = & $a; //此时$b和$a指向同一块内存
$b = 4; //对$b的重新赋值，也会影响$a
echo $a; //$a值变为4
```
变量引用的一个常见用法是引用传递，将实参的引用传到方法中，从而实现在方法中改变原数据。
注意在函数调用时没有引用符号——只有函数定义中有。
```php
$a = 3;
f($a); //此时只能传变量名，如果传整形或字符串等数据，会报错
function f(& $param) { //此处取了实参的引用
  $param++;
}
echo $a; //$a值变为4
```
##2.函数的引用返回
当函数有返回值时，可以通过函数引用的形式，在函数外部更改函数的返回值。可以认为是**将函数返回值所占用的内存引用传递给函数外部的变量，使两个变量表示同一块内存**。
函数的引用返回，就是将函数的返回值当做引用。
```php
function & f() {  //方法定义时，方法名前加&表示该方法可以被引用
    static $a = 3;
    echo $a++;
    return $a;
}

$b = &f();  //引用该函数，并将函数返回值和$b指向同一个变量
echo $b;
$b = $b + 100;  //此时同时改变函数返回值$a
f();    //本行打印104
上例打印出：34104
```
`$a = f();`改变`$a`并不会影响函数`f()`的返回值，但是`$a = &f();`在改变`$a`时同时影响函数`f()`的返回值。
##3.对象引用
PHP中，对象的复制默认就是通过引用传递来实现，`$o2 = $o1`执行时，实际就是将`$o1`的引用复制给`$o2`，两个对象实际上是同一个。
####对象复制
参考：http://php.net/manual/zh/language.oop5.cloning.php

如果我们想创建对象的副本，需要使用clone关键字（这将调用对象的 `__clone()` 方法）。对象中的 `__clone()` 方法不能被直接调用。
```
//两种写法都可以
$copy_of_object = clone $object;
$copy_of_object = clone($object);
```
当用 clone 复制对象后，PHP 5 会对对象的所有属性执行一个浅复制（shallow copy）。所有的基本属性会复制到新的存储空间，但是引用属性仍然会是一个指向原来的变量的引用。可以在要复制的对象中自定义 `__clone()` 方法，复制对象完成后会自动调用 `__clone()` 方法，可用于修改属性的值。
```
<?php
class SubObject
{
    static $instances = 0;
    public $instance;

    public function __construct() {
        $this->instance = ++self::$instances;
    }

    public function __clone() {
        $this->instance = ++self::$instances;
    }
}

class MyCloneable
{
    public $object1;
    public $object2;

    function __clone()
    {
      
        // 强制复制一份this->object， 否则仍然指向同一个对象
        $this->object1 = clone $this->object1;
    }
}

$obj = new MyCloneable();

$obj->object1 = new SubObject();
$obj->object2 = new SubObject();

$obj2 = clone $obj;


print("Original Object:\n");
print_r($obj);

print("Cloned Object:\n");
print_r($obj2);

?>
```