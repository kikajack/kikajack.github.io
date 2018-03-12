##1. $this 伪变量
当一个方法在类定义的内部被调用时，有一个可用的伪变量 `$this`。`$this` 代表当前对象（但如果是从第二个对象静态调用时也可能是另一个对象）。
##2. 继承
一个类可以在声明中用 extends 关键字继承另一个类，子类就会继承父类所有**公有的和受保护的**方法。除非子类覆盖了父类的方法，被继承的方法都会保留其原有功能。PHP不支持多重继承，一个类只能继承一个基类。
除非使用了自动加载，否则一个类必须在使用之前被定义。如果一个类扩展了另一个，则父类必须在子类之前被声明。
##3. 构造函数
具有构造函数的类会在每次创建新对象时先调用此方法，所以非常适合在使用对象之前做一些初始化工作。
如果子类中定义了构造函数则不会隐式调用其父类的构造函数。要执行父类的构造函数，需要在子类的构造函数中调用 parent::__construct()。如果子类没有定义构造函数则会如同一个普通的类方法一样从父类继承（假如没有被定义为 private 的话）。
##4. 析构函数
析构函数会在到某个对象的所有引用都被删除或者当对象被显式销毁时执行。
和构造函数一样，父类的析构函数不会被引擎暗中调用。要执行父类的析构函数，必须在子类的析构函数体中显式调用 parent::__destruct()。此外也和构造函数一样，子类如果自己没有定义析构函数则会继承父类的。

析构函数即使在使用 exit() 终止脚本运行时也会被调用。在析构函数中调用 exit() 将会中止其余关闭操作的运行。
##5. 属性和方法的访问控制（可见性）
参考：http://php.net/manual/zh/language.oop5.visibility.php

访问控制的关键字有 public（公有），protected（受保护）和 private（私有）。

- public：公有的类成员，可以在任何地方被访问。
- protected：受保护的类成员，可以被其自身以及其子类和父类的内部访问。
- private：私有的类成员，只能被其定义所在的类内部访问。

####（1）属性的访问控制
类属性必须定义为公有，受保护，私有之一。如果用 var 定义，则被视为公有。
```
class MyClass {
    public $public = 'Public';
    protected $protected = 'Protected';
    private $private = 'Private';
}

$obj = new MyClass();
echo $obj->public; // 这行能被正常执行
echo $obj->protected; // 这行会产生一个致命错误
echo $obj->private; // 这行也会产生一个致命错误

class MyClass2 {
  function fun() {
    echo $public; // 这行能被正常执行
    echo $protected; // 这行能被正常执行
    echo $private; // 这行会产生一个致命错误
  }
}
$obj2 = new MyClass2();
$obj2->fun();
```
类属性定义为公有后，可以在类外部随意更新（值、类型），可能会在使用属性的时候导致错误。因此，**最好将属性封装为私有或受保护的，然后留接口（方法）供外部读取或设置属性值，在赋值前进行合法性判断等处理操作。**
####（2）方法的访问控制
类中的方法可以被定义为公有，私有或受保护。没有设置关键字时默认为公有。
类中只有声明为 public  的方法，可以在类的外部随意调用。
父类中声明为 public 和 protected 的方法，子类会自动继承。
####（3）其他对象的访问控制
同一个类的对象即使不是同一个实例也可以互相访问对方的私有与受保护成员。
```
class Father {
  private $name;

  public function __construct($name) {
    $this->name = $name;
  }
  
  public function f(Father $o) {
    echo $o->name;
  }
}

class Son1 extends Father {

}

class Son2 extends Father {

}

$s1 = new Son1("jack");
$s1->f(new Son2("rose"));
```