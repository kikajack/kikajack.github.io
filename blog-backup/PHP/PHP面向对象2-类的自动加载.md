当项目比较大时，有些页面需要引入很多类，有些类是全局使用的，你需要不停的require文件。这时自动加载机制可以很有用。类的命名需要有一定的规律。
参考链接：
http://php.net/manual/zh/function.autoload.php#refsect1-function.autoload-returnvalues
http://php.net/manual/zh/function.spl-autoload-register.php
##1.常规自动加载__autoload()（注意：PHP7.2废弃）
##2.自定义自动加载spl_autoload_register()
注册一个或多个给定的函数作为 __autoload 的实现。注意此方法会使__autoload()方法失效。
```php
<?php

//不推荐使用__autoload
// function __autoload($class) {
//     include 'classes/' . $class . '.class.php';
// }

function my_autoloader1($class) {
    include 'my_classes/' . $class . '.class.php';
}

function my_autoloader2($class) {
    include 'your_classes/' . $class . '.class.php';
}

//将自定义的2个函数注册为自动加载用的函数
spl_autoload_register('my_autoloader1');
spl_autoload_register('my_autoloader2');

// 或者，自 PHP 5.3.0 起可以使用一个匿名函数
spl_autoload_register(function ($class) {
    include 'classes/' . $class . '.class.php';
});
```
```
<?php

class Loader {
    public static function loadClass($class) {
        $file = $class . '.php';
        if (is_file($file)) {
            require_once($file);
        }
    }
}

//注册用于自动加载的静态方法
spl_autoload_register(array('Loader', 'loadClass'));

//实例化对象时，会去自动加载类文件'A.php'
$a = new A();
```
```
<?php

//在类的构造方法中注册用于自动加载的方法，实例化后就可以用
class ClassAutoloader {
    public function __construct() {
        spl_autoload_register(array($this, 'loader'));
    }
    private function loader($className) {
        echo 'Trying to load ', $className, ' via ', __METHOD__, "()\n";
        include $className . '.php';
    }
}

$autoloader = new ClassAutoloader();

//实例化对象时，会去自动加载类文件'Class1.php'和'Class2.php'
$obj = new Class1();
$obj = new Class2();

```