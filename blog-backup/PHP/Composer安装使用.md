参考：
Composer简介：http://docs.phpcomposer.com/00-intro.html
Composer 的结构：http://phpernotes.com/php/composer_schema
安装Laravel：https://docs.golaravel.com/docs/5.0/installation/

#0. 总结
Composer安装之前，首先需要安装php。Composer以phar包的PHP可执行文件形式存在，安装好后用php执行即可（局部安装先切换到composer安装目录后用`php composer.phar 参数`执行，全局安装用`composer 参数`执行）。
Composer最经常用的，就是用来安装一个完整的项目（比如Laravel框架），或为当前项目安装一个扩展（比如为用CodeIgniter框架开发的项目安装monolog日志记录工具）。
###1. 用Composer安装项目
1. 首先进入到要放项目的目录，比如`/home/kika`。
2. 安装，用`create-project`命令。
正常命令（国外）：`composer create-project --prefer-dist laravel/laravel my-project "5.5.*"`
国内安装，首先需要切换国内的安装源，才可以安装：
```
composer config -g repo.packagist composer https://packagist.phpcomposer.com
composer create-project --prefer-dist laravel/laravel my-project "5.5.*"
```
命令执行后，会创建目录`/home/kika/my-project`。
例如，安装 CodeIgniter，执行命令：`composer create-project codeigniter/framework test `，即可在当前目录下创建 test 目录，项目安装到这个目录下。
3. 配置服务器，访问当前项目即可。
###2. 用Composer安装扩展
1. 进入项目的根目录，比如`/home/kika/ci-project`。
2. 安装，**用`require`命令**。注意，不要用`update`。
```
composer require monolog/monolog
```
命令执行后，会自动更新`composer.json`文件，为`require`属性添加一行`"monolog/monolog": "^1.23"`。对于`composer.lock`文件，如果没有则创建该文件，有则更新。同时，会在当前目录下创建或更新vendor目录，所有下载的依赖都放在vendor目录。
**注意：不要手工改动`composer.json`文件，执行命令更加可靠。**
###3. 移除扩展
```
composer remove monolog/monolog
```
#1. 安装
###1. Linux
1.  全局安装
安装后，将phar包拷贝到新建的`/usr/local/bin/composer`目录即可。此时可以直接通过命令`composer`执行。
```
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
```
2.  局部安装
可以通过命令`php composer.phar`执行。

- 切换路径到需要安装Composer的项目根目录，执行
``` Linux
curl -sS https://getcomposer.org/installer | php
```
- 也可以在安装的时候，指定目录（绝对或相对路径）
``` Linux
curl -sS https://getcomposer.org/installer | php -- --install-dir=bin
```
###2. Window
####1. 官网安装
在官网下载exe文件，直接安装即可。注意安装过程中会寻找已经安装的php可执行文件的位置。
地址：https://getcomposer.org/Composer-Setup.exe
####2. 国内镜像安装
参考：https://pkg.phpcomposer.com/#how-to-install-composer
打开命令行并依次执行下列命令安装最新版本的 Composer：
```
1. 下载安装脚本到当前目录
php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');"
2. 执行安装过程
php composer-setup.php
3.删除安装脚本
php -r "unlink('composer-setup.php');"
```
###3. 切换仓库源为国内镜像
默认仓库太卡，可以通过修改配置，切换为国内镜像。
参考：https://pkg.phpcomposer.com/
```
修改 composer 的全局配置文件：
composer config -g repo.packagist composer https://packagist.phpcomposer.com
```
#2. 基本使用
在项目的根目录下，需要配置文件`composer.json`。
###1. 指定依赖：require告诉 Composer 你的项目需要依赖哪些包。require 需要一个 包名称 （例如 monolog/monolog） 映射到 包版本 （例如 1.0.*） 的对象。
- 包名称：由供应商名称和其项目名称构成。
- 包版本：版本约束可以用几个不同的方法来指定。

|名称 |实例 |描述|
|---------|-----------|--------|
|确切的版本号 |`1.0.2`| 你可以指定包的确切版本。
|范围 |`>=1.0` `>=1.0,<2.0` `>=1.0,<1.1\|>=1.2`|  通过使用比较操作符可以指定有效的版本范围。 <br/>有效的运算符：\|>、>=、<、<=、!=。 <br/>你可以定义多个范围，用逗号隔开，这将被视为一个逻辑AND处理。一个管道符号\|将作为逻辑OR处理。 AND 的优先级高于 OR。
|通配符  |`1.0.*`| 你可以使用通配符*来指定一种模式。1.0.*与>=1.0,<1.1是等效的。
|赋值运算符  |`~1.2`|  下一个重要版本。这对于遵循语义化版本号的项目非常有用。~1.2相当于>=1.2,<2.0，而 ~1.2.3 相当于 >=1.2.3,<1.3。使用 ~ 指定最低版本，但允许版本号的最后一位数字上升。

```json
{
    "require": {
        "monolog/monolog": "1.0.*"
    }
}
```
###2. 安装依赖
install 命令首先检查锁文件`composer.lock`是否存在，如果存在，它将下载锁文件指定的版本（忽略 composer.json 文件中的定义）。如果不存在，Composer 将读取 composer.json 并创建锁文件。
```
php composer.phar install
```
这将会找到 monolog/monolog 的最新版本，并将它下载到 vendor/monolog/monolog  目录。 把第三方的代码到一个指定的目录 vendor是一个惯例。
如果你正在使用Git来管理你的项目， 你可能要添加 vendor 到你的 .gitignore 文件中。 你不会希望将所有的代码都添加到你的版本库中。
install 命令还会创建一个 composer.lock 文件到你项目的根目录中。
###3. composer.lock - 锁文件
**目的：使指定项目的每一个环境，每一个开发者，每一时刻都下载与指定版本完全相同的依赖。持续集成服务器、生产环境、你团队中的其他开发人员都使用相同的依赖。即使你的依赖已经发布了许多新的版本。**

在安装依赖后，Composer 将把安装时确切的版本号列表写入 composer.lock 文件。这将锁定改项目的特定版本。
需要提交composer.lock 和 composer.json这两个文件到版本库中。
###4. 更新依赖的版本update
因为锁文件的存在，你的依赖更新了新的版本时你将不会获得任何更新。要更新你的依赖版本需要使用 update 命令。这将获取最新匹配的版本（根据你的 composer.json 文件）并将新版本更新进锁文件。
- 更新所有依赖
```
php composer.phar update
```
- 安装或更新一个依赖，可以白名单它们：
```
php composer.phar update monolog/monolog [...]
```
###5. Packagist
**如果你要发布一个Composer资源，就用用到Packagist了。**

Packagist是 Composer 的主要资源库（https://packagist.org/）。 一个 Composer 的库基本上是一个包的源：记录了可以得到包的地方。Packagist 的目标是成为大家使用库资源的中央存储平台。这意味着你可以 require 那里的任何包。
 任何支持 Composer 的开源项目应该发布自己的包在 packagist 上。虽然并不一定要发布在 packagist 上来使用 Composer，但它使我们的编程生活更加轻松。
 
###6. 自动加载
 vendor目录下有个autoload.php 文件。引入这个文件，就可以实现自动加载。
```
require 'vendor/autoload.php';
```
如果你的项目依赖 monolog，你就可以像这样开始使用这个类库，并且他们将被自动加载。
```
$log = new Monolog\Logger('name');
$log->pushHandler(new Monolog\Handler\StreamHandler('app.log', Monolog\Logger::WARNING));

$log->addWarning('Foo');
```
#3. 用 Composer 安装库（资源包）
###1.  每一个项目都是一个包
只要项目根目录中有 composer.json 文件，那么这个项目就是一个包。当你添加一个 require 到项目中，你就是在创建一个依赖于其它库的包。项目的composer.json 文件添加了name属性后，项目就是一个有名字的包。

为了使它成为一个可安装的包，你需要给它一个名称。你可以通过 composer.json 中的 name 来定义：
```
{
    "name": "acme/hello-world",
    "require": {
        "monolog/monolog": "1.0.*"
    }
}
```
上例中项目的名称（name）为 acme/hello-world，其中 acme 是供应商的名称。供应商的名称是必须填写的。

注意： 如果你不知道拿什么作为供应商的名称， 那么使用你 github 上的用户名通常是不错的选择。 虽然包名不区分大小写，但惯例是使用小写字母，并用连字符作为单词的分隔。
###2. 平台软件包
Composer 将那些已经安装在系统上，但并不是由 Composer 安装的包视为一个虚拟的平台软件包。这包括PHP本身，PHP扩展和一些系统库。

- php 表示用户的 PHP 版本要求，你可以对其做出限制。例如 >=5.4.0。如果需要64位版本的 PHP，你可以使用 php-64bit 进行限制。
- hhvm 代表的是 HHVM（也就是 HipHop Virtual Machine） 运行环境的版本，并且允许你设置一个版本限制，例如，'>=2.3.3'。
- ext-<name> 可以帮你指定需要的 PHP 扩展（包括核心扩展）。通常 PHP 拓展的版本可以是不一致的，将它们的版本约束为 * 是一个不错的主意。一个 PHP 扩展包的例子：包名可以写成 ext-gd。
- lib-<name> 允许对 PHP 库的版本进行限制。
以下是可供使用的名称：curl、iconv、icu、libxml、openssl、pcre、uuid、xsl。

可以使用 composer show --platform 命令来获取可用的平台软件包的列表。
###3. 指明自己开发的包的版本
**尽量避免手动设置版本号，因为标签的值必须与标签名相匹配。**
你需要一些方法来指明自己开发的包的版本，当你在 Packagist 上发布自己的包，它能够从 VCS (git, svn, hg) 的信息推断出包的版本，因此你不必手动指明版本号，并且也不建议这样做。
如果你想要手动创建并且真的要明确指定它，你只需要添加一个 version 字段：`"version": "1.0.0"`

####1. 标签
对于每一个看起来像版本号的标签，都会相应的创建一个包的版本。它应该符合 'X.Y.Z' 或者 'vX.Y.Z' 的形式，-patch、-alpha、-beta 或 -RC 这些后缀是可选的。在后缀之后也可以再跟上一个数字。
下面是有效的标签名称的几个例子：

- 1.0.0
- v1.0.0
- 1.10.5-RC1
- v4.4.4beta2
- v2.0.0-alpha
- v2.0.4-p1

注意： 即使你的标签带有前缀 v， 由于在需要 require 一个版本的约束时是不允许这种前缀的， 因此 v 将被省略（例如标签 V1.0.0 将创建 1.0.0 版本）。
####2. 分支
对于每一个分支，都会相应的创建一个包的开发版本。如果分支名看起来像一个版本号，那么将创建一个如同 {分支名}-dev 的包版本号。例如一个分支 2.0 将产生一个 2.0.x-dev 包版本（加入了 .x 是出于技术的原因，以确保它被识别为一个分支，而 2.0.x 的分支名称也是允许的，它同样会被转换为 2.0.x-dev）。如果分支名看起来不像一个版本号，它将会创建 dev-{分支名} 形式的版本号。例如 master 将产生一个 dev-master 的版本号。

下面是版本分支名称的一些示例：

- 1.x
- 1.0 (equals 1.0.x)
- 1.1.x
####3. 别名
它表示一个包版本的别名。例如，你可以为 dev-master 设置别名 1.0.x-dev，这样就可以通过 require 1.0.x-dev 来得到 dev-master 版本的包。
###4.锁文件
在项目中提交 composer.lock 文件后，整个项目将始终针对同一个依赖版本进行测试。
###5. 安装
```
{
    "name": "acme/blog",
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/username/hello-world"
        }
    ],
    "require": {
        "acme/hello-world": "dev-master"
    }
}
```
注意：

- 如果不想将它发布为一个库，`name`可以不写。
- 如果库已经发布在packagist仓库，`repositories`可以不写。

如果项目只是发布到VCS（Git或SVN），也可以通过Composer安装：在项目根目录中添加 composer.json 文件，并声明`name`和`require`属性（要想发布为库，必须有`name`属性）。需要安装该库的项目根目录下的composer.json文件需要声明`repositories`属性。
###6. 发布到 packagist
Packagist 是 Composer 主要的一个包信息存储库，它默认是启用的。任何在 packagist 上发布的包都可以直接被 Composer 使用。就像 monolog 它被 发布在 packagist 上，我们可以直接使用它，而不必指定任何额外的来源信息。
注意： 当你安装一个新的版本时，将会自动从它 source 中拉取。 详细请查看 install 命令。
#4. 常用命令
为了从命令行获得帮助信息，请运行 composer 或者 composer list 命令，然后结合 --help 命令来获得更多的帮助信息。`composer --help [命令]`
|命令|用途|示例|解释|
|-----|-----|--------|----|
|create-project|创建项目|composer create-project|从现有的包中创建一个新的项目。这相当于执行了一个 git clone 或 svn checkout 命令后，将这个包的依赖安装到它自己的 vendor 目录。默认在 packagist.org 上查找包。可以用`--repository-url`参数指定要搜索包的库，代替默认库 packagist.org。|
|install|安装|composer install|在当前目录读取 composer.json 文件，处理了依赖关系，并把其安装到 vendor 目录下<br/>如果当前目录下存在 composer.lock 文件，它会从此文件读取依赖版本，而不是根据 composer.json 文件去获取依赖。这确保了该库的每个使用者都能得到相同的依赖版本。<br/>如果没有 composer.lock 文件，composer 将在处理完依赖关系后创建它。|
|update|更新|全部更新：composer update <br/>指定更新：composer update vendor/package vendor/package2 <br/>通配符更新：composer update vendor/* |获取依赖的最新版本，并且升级 composer.lock 文件|
|**require**|**申明并安装依赖**|composer require|增加新的依赖包到当前目录的 composer.json 文件中，同时下载到vender目录|
|remove|移除依赖|composer remove|移除已安装的依赖
|show|展示|composer show<br/>composer show monolog/monolog|列出所有可用的软件包，或看一个包的详细信息|
|config|更改配置|php composer.phar config --list|修改包来源，编辑 Composer 的一些基本设置（本地的 composer.json 或者全局的 config.json 文件)。|
#5. composer.json架构
- require:必须的软件包列表，除非这些依赖被满足，否则不会完成安装。
- require-dev (root-only只能在项目根目录中用):为开发或测试等目的，额外列出的依赖。“root 包”的 require-dev 默认是会被安装的。然而 install 或 update 支持使用 --no-dev 参数来跳过 require-dev 字段中列出的包。
- repositories (root-only):使用自定义的包资源库。默认情况下 composer 只使用 packagist 作为包的资源库。通过指定资源库，你可以从其他地方获取资源包。Repositories 并不是递归调用的，只能在“Root包”的 composer.json 中定义。附属包中的 composer.json 将被忽略。支持以下类型的包资源库：
 - composer: 一个 composer 类型的资源库，是一个简单的网络服务器（HTTP、FTP、SSH）上的 packages.json 文件，它包含一个 composer.json 对象的列表，有额外的 dist 和/或 source 信息。这个 packages.json 文件是用一个 PHP 流加载的。你可以使用 options 参数来设定额外的流信息。
 - vcs: 从 git、svn 和 hg 取得资源。
 - pear: 从 pear 获取资源。
 - package: 如果你依赖于一个项目，它不提供任何对 composer 的支持，你就可以使用这种类型。你基本上就只需要内联一个 composer.json 对象。
```
{
    "repositories": [
        {
            "type": "composer",
            "url": "http://packages.example.com"
        },
        {
            "type": "composer",
            "url": "https://packages.example.com",
            "options": {
                "ssl": {
                    "verify_peer": "true"
                }
            }
        },
        {
            "type": "vcs",
            "url": "https://github.com/Seldaek/monolog"
        },
        {
            "type": "pear",
            "url": "http://pear2.php.net"
        },
        {
            "type": "package",
            "package": {
                "name": "smarty/smarty",
                "version": "3.1.7",
                "dist": {
                    "url": "http://www.smarty.net/files/Smarty-3.1.7.zip",
                    "type": "zip"
                },
                "source": {
                    "url": "http://smarty-php.googlecode.com/svn/",
                    "type": "svn",
                    "reference": "tags/Smarty_3_1_7/distribution/"
                }
            }
        }
    ]
}
```
- conflict:此列表中的包与当前包的这个版本冲突。它们将不允许同时被安装。请注意，在 conflict 中指定类似于 <1.0, >= 1.1 的版本范围时，这表示它与小于1.0 并且 同时大等于1.1的版本冲突，这很可能不是你想要的。在这种情况下你可能想要表达的是 <1.0 | >= 1.1 。
- replace:这个列表中的包将被当前包取代。这使你可以 fork 一个包，以不同的名称和版本号发布，同时要求依赖于原包的其它包，在这之后依赖于你 fork 的这个包，因为它取代了原来的包。
这对于创建一个内部包含子包的主包也非常的有用。例如 symfony/symfony 这个主包，包含了所有 Symfony 的组件，而这些组件又可以作为单独的包进行发布。如果你 require 了主包，那么它就会自动完成其下各个组件的任务，因为主包取代了子包。
注意，在使用上述方法取代子包时，通常你应该只对子包使用 self.version 这一个版本约束，以确保主包仅替换掉子包的准确版本，而不是任何其他版本。
- provide:List of other packages that are provided by this package. This is mostly useful for common interfaces. A package could depend on some virtual logger package, any library that implements this logger interface would simply list it in provide.
- suggest:建议安装的包
- autoload:PHP autoloader 的自动加载映射。Currently PSR-0 autoloading, PSR-4 autoloading, classmap generation and files includes are supported. PSR-4 is the recommended way though since it offers greater ease of use (no need to regenerate the autoloader when you add classes).
 - PSR-4:定义了一个命名空间到实际路径的映射（相对于包的根目录）。When autoloading a class like Foo\\Bar\\Baz a namespace prefix Foo\\ pointing to a directory src/ means that the autoloader will look for a file named src/Bar/Baz.php and include it if present. Note that as opposed to the older PSR-0 style, the prefix (Foo\\) is not present in the file path.请注意，命名空间的申明应该以 \\ 结束，以确保 autoloader 能够准确响应，不然相同前缀会导致冲突。例： Foo 将会与 FooBar 匹配，然而以反斜杠结束就可以解决这样的问题， Foo\\ 和 FooBar\\ 将会被区分开来。
```
{
    "autoload": {
        "psr-4": {
            "Monolog\\": "src/",
            "Vendor\\Namespace\\": ""
        }
    }
}
```
在 install/update 过程中，PSR-4 引用都将被整合为一个单一的键值对数组，存储至 `vendor/composer/autoload_namespaces.php` 文件中。
 - classmap:用 classmap 生成支持自定义加载的不遵循 PSR-0/4 规范的类库。要配置它指向需要的目录，以便能够准确搜索到类文件。引用的所有组合，都会在 install/update 过程中生成，并存储到 `vendor/composer/autoload_classmap.php` 文件中。这个 map 是经过扫描指定目录（同样支持直接精确到文件）中所有的 .php 和 .inc 文件里内置的类而得到的。
```
{
    "autoload": {
        "classmap": ["src/", "lib/", "Something.php"]
    }
}
```
 - Files:明确指定在每次请求时要载入哪些文件。通常作为函数库的载入方式（而非类库）。
```
{
    "autoload": {
        "files": ["src/MyLibrary/functions.php"]
    }
}
```
 - autoload-dev (root-only):开发过程中，可以指定自动加载的规则。单元测试类不应包含在主自动加载规则中，以避免污染生产环境的或其他用您的软件包作为依赖项的自动加载器。因此，最好为单元测试使用专用路径并将其添加到autoload-dev部分中。
```
{
    "autoload": {
        "psr-4": { "MyLibrary\\": "src/" }
    },
    "autoload-dev": {
        "psr-4": { "MyLibrary\\Tests\\": "tests/" }
    }
}
```
- include-path:不建议用，这是目前唯一支持传统项目的做法，所有新的代码都建议使用自动加载。 这是一个过时的做法，但 Composer 将仍然保留这个功能。
- name:包名。包括供应商名称和项目名称，使用 / 分隔。对于需要发布的包（库），这是必须填写的。
- description:一个包的简短描述。通常这个最长只有一行。对于需要发布的包（库），这是必须填写的。
- version:建议忽略。通常能够从 VCS (git, svn, hg) 的信息推断出包的版本号。
- type:包的安装类型，默认为 library。composer 原生支持以下4种类型：
 - library: 这是默认类型，它会简单的将文件复制到 vendor 目录。
 - project: 这表示当前包是一个项目，而不是一个库。例：框架应用程序 Symfony standard edition，内容管理系统 SilverStripe installer 或者完全成熟的分布式应用程序。使用 IDE 创建一个新的工作区时，这可以为其提供项目列表的初始化。
 - metapackage: 当一个空的包，包含依赖并且需要触发依赖的安装，这将不会对系统写入额外的文件。因此这种安装类型并不需要一个 dist 或 source。
 - composer-plugin: 一个安装类型为 composer-plugin 的包，它有一个自定义安装类型，可以为其它包提供一个 installler。详细请查看 自定义安装类型。
- keywords:该包相关的关键词的数组。这些可用于搜索和过滤。
- homepage:项目主页的 URL 地址。
- time:版本发布时间。必须符合 YYYY-MM-DD 或 YYYY-MM-DD HH:MM:SS 格式。
- license:包的许可协议，它可以是一个字符串或者字符串数组。例如：`"license": "MIT"`
- authors:包的作者。这是一个对象数组。每个对象必须包含以下属性：
 - name: 作者的姓名，通常使用真名。
 - email: 作者的 email 地址。
 - homepage: 作者主页的 URL 地址。
 - role: 该作者在此项目中担任的角色（例：开发人员 或 翻译）。
- support:获取项目支持的向相关信息对象。这个对象可以包含以下属性：
 - email: 项目支持 email 地址。
 - issues: 跟踪问题的 URL 地址。
 - forum: 论坛地址。
 - wiki: Wiki 地址。
 - irc: IRC 聊天频道地址，类似于 irc://server/channel。
 - source: 网址浏览或下载源。
- minimum-stability (root-only)：定义了通过稳定性过滤包的默认行为。默认为 stable（稳定）。因此如果你依赖于一个 dev（开发）包，你应该明确的进行定义。对每个包的所有版本都会进行稳定性检查，而低于 minimum-stability 所设定的最低稳定性的版本，将在解决依赖关系时被忽略。对于个别包的特殊稳定性要求，可以在 require 或 require-dev 中设定。
- prefer-stable (root-only)：当此选项被激活时，Composer 将优先使用更稳定的包版本。使用 `"prefer-stable": true` 来激活它。
可用的稳定性标识（按字母排序）：dev、alpha、beta、RC、stable。
- config (root-only):可用的选项，仅用于项目。
- scripts (root-only)：Composer 允许你在安装过程中的各个阶段挂接脚本。

```
{
  "description": "The CodeIgniter framework",
  "name": "codeigniter/framework",
  "type": "project",
  "homepage": "https://codeigniter.com",
  "license": "MIT",
  "support": {
    "forum": "http://forum.codeigniter.com/",
    "wiki": "https://github.com/bcit-ci/CodeIgniter/wiki",
    "slack": "https://codeigniterchat.slack.com",
    "source": "https://github.com/bcit-ci/CodeIgniter"
  },
  "require": {
    "php": ">=5.3.7"
  },
  "suggest": {
    "paragonie/random_compat": "Provides better randomness in PHP 5.x"
  },
  "require-dev": {
    "mikey179/vfsStream": "1.1.*",
    "phpunit/phpunit": "4.* || 5.*"
  }
}
```