序列化 (Serialization)：将对象的状态信息转换为可以**存储**或**传输**的形式的过程。
警告：各种语言所采用的序列化机制往往不一样，不兼容。
##1 序列化serialize()
`string serialize ( mixed $value )`

serialize() 返回字符串，此字符串包含了表示 value 的字节流，同时不丢失其类型和结构。
serialize() 可处理除了 resource 之外的任何类型。甚至可以 serialize() 那些包含了指向其自身引用的数组。
当序列化对象时，PHP 将试图在序列动作之前调用该对象的成员函数 __sleep()。这样就允许对象在被序列化之前做任何清除操作。
##2 反序列化unserialize()
`mixed unserialize ( string $str )`

想要将已序列化的字符串变回 PHP 的值，可使用 unserialize()。
当使用 unserialize() 恢复对象时， 将调用 __wakeup() 成员函数。
**如果要想在另外一个文件中反序列化一个对象，必须要在反序列化之前定义这个对象的类。**
```
// 未定义对象的类时，如果反序列化，会得到下面的__PHP_Incomplete_Class对象，不含方法。
__PHP_Incomplete_Class Object
(
    [__PHP_Incomplete_Class_Name] => Person
    [name:Person:private] => jack
    [age:Person:private] => 18
    [sex:Person:private] => M
)
```

```PHP
<?php

class Person {
  private $name;
  private $age;
  private $sex;
  
  function __construct($name, $age, $sex) {
    $this->name = $name;
    $this->age = $age;
    $this->sex = $sex;
  }

  function __sleep() {
    $arr = array('name', 'age');
    return $arr;
  }

  function __wakeup() {
    $this->sex = 'F';
    $this->age = 20;
  }

  function say() {
    echo 'My name is: ' . $this->name . ',My age is: ' . $this->age . ',My sex is: ' . $this->sex . PHP_EOL;
  }
}

$p = new Person('jack', 18, 'M');
$p->say(); // My name is: jack,My age is: 18,My sex is: M
$s = serialize($p); //得到（O:6:"Person":3:{s:12:"Personname";s:4:"jack";s:11:"Personage";i:18;s:11:"Personsex";s:1:"M";}）字符串，有特殊字符
echo $s . PHP_EOL;
$o = unserialize($s); // 反序列化之前必须要定义这个对象的类
$o->say(); // My name is: jack,My age is: 20,My sex is: F
```
