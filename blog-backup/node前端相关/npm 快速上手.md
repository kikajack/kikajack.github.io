参考链接：
npm 官网文档：https://docs.npmjs.com

#一. npm 是什么
npm（node package manager，node 包管理器）：顾名思义，就是用于安装、卸载、更新、查看和搜索 node 包的工具。
#二. npm 生态
类似于 Linux 下的 `apt-get`，`yum`，PHP 中的 `Composer`，通过 npm 可以方便的管理各种 node 包，而不用再去管麻烦的依赖关系了。
npm 最大的优点：安装 node 包很方便，忽略依赖关系。
#三. npm 安装
node 安装好之后，npm 就顺带安装了，可以查看二者的版本号：
`node -v`
`npm -v`
#四. npm 使用
所有的 npm 命令，都可以通过 `--help` 参数获得帮助。例如：
`npm install --help`
##1. 安装 npm 包：install
[官方 install 命令文档](https://docs.npmjs.com/cli/install)
###1）本地安装和全局安装
####① 本地安装
`npm install package`

本地安装时，npm 会在当前目录下创建 node_modules 目录，并把所有安装包下载至此。
####② 全局安装
`npm install -g package`

全局安装会把包下载到系统目录（Windows 下一般是`C:\Users\用户名\AppData\Roaming\npm\node_modules`，Linux 下一般是 `/usr/local/lib/node_modules/`）。
###2） 安装指定版本
`npm install package@"1.2.3"` 安装版本指定为 1.2.3
`npm install package@">=0.1.0 <0.2.0"` 安装版本在 0.1.0 和 0.2.0之间即可
###3） 通过 package.json 进行安装
**通常在项目发布的时候，因为项目下的 node_modules 目录体积太大，不会跟项目一同发布。**但是项目所有的依赖关系已经在 package.json 这个文件里声明了，可以在项目根目录执行一个命令安装所有依赖包：

`npm install`
##2. 卸载 npm 包：uninstall
`npm uninstall package`
`npm uninstall package@"1.2.3"`
##3. 查看已经安装的 npm 包：ls
`npm ls`
##4. 查看指定包的信息：info
`npm info package`
##5. 更新指定包：update
`npm update package`
##6. 搜索包：search
`npm search package`
#五. npm 配置
通过npm config命令，可以配置 npm。
###1. 查看所有配置
`npm config list`
###2. 切换国内源

- 国内用户因为各种原因，用 npm 默认仓库会很慢，可以切换仓库为国内的 [淘宝镜像官网](https://npm.taobao.org/)：
`npm config set registry https://registry.npm.taobao.org --global` 注册模块镜像
`npm set disturl https://npm.taobao.org/dist` # node-gyp 编译依赖的 node 源码镜像
- 或者安装淘宝定制的 cnpm (gzip 压缩支持) 命令行工具代替默认的 npm:
`npm install -g cnpm --registry=https://registry.npm.taobao.org`
###3. 直接修改配置文件
`npm config edit`
#六. package.json 包描述文件
每个基于 node 的项目根目录下，会有这个文件，用于描述依赖信息。

- name：包名（如果发布这个包，name 字段会成为url的一部分）。
- version：package的版本，当package发生变化时，version也应该跟着一起变化，同时，你声明的版本需要通过 [semver](https://docs.npmjs.com/misc/semver) 的校验。[semver官网](https://semver.org/lang/zh-CN/)
- scripts：可执行的 npm 脚本。将一段命令用一个简短的名字代替。比如在 scripts 对象中添加这么一句话：`"build": "node build.js"`，然后用这个命令即可执行：`npm run build`，等同于执行 `node build.js`。可以[参考这里](http://www.ruanyifeng.com/blog/2016/10/npm_scripts.html)。
- dependencies：这里面的模块，是生产环境中需要的依赖（比如 `vue`，`vue-loader`，`vuex`）。在使用 npm install 安装 npm 包时，`--save` （默认就是这个参数）会把依赖包名称添加到 package.json 的 dependencies 下。
- devDependencies：这里面的模块，是我们开发时用的（比如 `css-loader`，`webpack`），不会被部署到生产环境。在使用 npm install 安装 npm 包时，`--save-dev` 则会添加到 devDependencies 下。