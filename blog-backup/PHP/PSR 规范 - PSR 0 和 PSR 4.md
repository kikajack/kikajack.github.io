[PSR-0 规范](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md)
[PSR-4 自动加载规范 - 中文](https://laravel-china.org/topics/2081/psr-specification-psr-4-automatic-loading-specification)
[PHP FIG 组织](https://psr.phphub.org/)
[PSR-0 和 PSR-4 对比，及测试用例](https://segmentfault.com/a/1190000000380008)

#1. PSR 规范及 FIG 组织
PSR 是 PHP Standard Recommendations 的简写，由 PHP FIG 组织制定的 PHP 规范，是 PHP 开发的实践标准。

PHP FIG（Framework Interoperability Group，框架可互用性小组）由几位开源框架的开发者成立于 2009 年，包括但不限于 Laravel, Joomla, Drupal, Composer, Phalcon, Slim, Symfony, Zend Framework，代表了大部分的 PHP 社区。

项目的目的在于：通过框架代表之间讨论，制定一个协作标准，各个框架遵循统一的编码规范，避免各家自行发展的风格阻碍了 PHP 的发展。

通过的标准，按照 PSR-0，PSR-1 等编号。
常用的规范：
|编号 | 名称|
|-|-|
|PSR-3| 日志接口规范|
|PSR-4| 自动加载规范|
#2. PSR-0
已废弃。
**在 PSR-0 中目录结构要与命名空间严格对应**，无法插入一个单独的目录。Vendor\Package\Class 在 psr-0 会里被直接转换成同样的路径，而 PSR-4 则没有这样的强制要求。
Composer 使用 PSR-0 风格时，目录会出现嵌套：
```
vendor/
    vendor_name/
        package_name/
            src/
                Vendor_Name/
                    Package_Name/
                        ClassName.php       # Vendor_Name\Package_Name\ClassName
            tests/
                Vendor_Name/
                    Package_Name/
                        ClassNameTest.php   # Vendor_Name\Package_Name\ClassName
```
Composer 使用 PSR-4 风格，目录简洁：
```
vendor/
    vendor_name/
        package_name/
            src/
                ClassName.php       # Vendor_Name\Package_Name\ClassName
            tests/
                ClassNameTest.php   # Vendor_Name\Package_Name\ClassNameTest
```
**PSR-0 中下划线会转换成目录分隔符**，以兼容 PHP 5.3 以前版本，而 PSR-4 不处理下划线。

**PSR-4 要求在 autoloader 中不允许抛异常 exceptions，也不允许触发任何级别的错误 errors，也不应该有返回值。**

PSR-4 更简洁更灵活了，但这使得它相对更复杂了。例如通过完全符合 PSR-0 标准的 class name，通常可以明确的知道这个 class 的路径，而 PSR-4 可能就不是这样了。
#3. PSR-4
PHP 命名空间仅仅是一个符号而已，和文件系统没有关系。 autoload 怎么去引入文件要看 autoload 函数是怎么写的。
但**在 PSR-4 规范中，命名空间和文件系统有对应关系。**通过遵守 PSR-4 自动加载规范，可以直接使用 Composer，免去了自己写 autoload 函数的麻烦。
##3.1 详细说明
一个完整的类名需具有以下结构:
```
\<命名空间>(\<子命名空间>)*\<类名>
```
- 完整的类名 必须 要有一个顶级命名空间，被称为 "vendor namespace"；
- 完整的类名 可以 有一个或多个子命名空间；
- 完整的类名 必须 有一个最终的类名。

**自动加载器（autoloader）中 一定不可 抛出异常、一定不可 触发任一级别的错误信息以及 不应该 有返回值。**这是因为在注册多个 autoloaders 时，如果一个 autoloader 没有找到对应的类文件，应该交给下一个 autoloader 来处理，而不能阻断这个处理通道。
##3.2 例子
下表展示了符合规范完整类名、命名空间前缀和文件基目录所对应的文件路径。

|完整类名|  命名空间前缀| 文件基目录|  文件路径|
|-|-|-|-|
|\Acme\Log\Writer\File_Writer|  Acme\Log\Writer|./acme-log-writer/lib/| ./acme-log-writer/lib/File_Writer.php|
|\Aura\Web\Response\Status| Aura\Web| /path/to/aura-web/src/| /path/to/aura-web/src/Response/Status.php|
|\Symfony\Core\Request| Symfony\Core| ./vendor/Symfony/Core/| ./vendor/Symfony/Core/Request.php|
|\Zend\Acl| Zend| /usr/includes/Zend/|  /usr/includes/Zend/Acl.php|
比如下面的例子，命名空间的前缀就是 `GraphQL\XinYou`，对应这个命名空间的基目录是当前目录下的 `application/controllers/graphql/`。对于完整类名 `GraphQL\XinYou\Type\UserType`，实际上会在这个命名空间对应的基目录下寻找 `/type/UserType.php`。组合起来就是 `application/controllers/graphql/type/UserType.php`
```
"autoload": {
  "psr-4": {
    "GraphQL\\XinYou\\": "application/controllers/graphql/"
  }
}
```
