##1. 概述
前端工程化后，完全手工搭建一个项目框架复杂且易错。通过各个框架（React，Vue）提供的脚手架工具，可以简单执行几行命令，就把整个项目的框架搭建完毕，简单高效统一，并且自动整合各种工具（Webpack，ESLint等）。
##2. 前提
在使用 Vue 提供的脚手架 Vue-cli 之前，需要首先安装 node.js 环境（需要用到 node 的 npm 包管理工具）。另外因为墙的原因，记得将 npm 的源切换为国内的镜像 [淘宝镜像](https://npm.taobao.org/)：
```
npm set registry https://registry.npm.taobao.org # 注册模块镜像
npm set disturl https://npm.taobao.org/dist # node-gyp 编译依赖的 node 源码镜像
```
或者安装 cnpm 命令：
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
##3. 安装vue-cli脚手架构建工具
运行以下命令：
```
npm install -g vue-cli
```
##4. 用 vue-cli 构建项目
```
 vue init webpack projectName
```
`vue init webpack projectName` 命令执行后，会在当前目录中，创建一个新目录 `projectName`。
`webpack` 选项表示，当前项目使用的构建工具是 `webpack`。
初始化过程中，需要用户输入一部分信息，比如 项目名称，项目描述，作者，是否使用 `ESLint` 代码语法检查工具，可以一路回车，使用默认配置。
##5. 安装依赖
在目录 `projectName` 中，有一个文件 `package.json`，其中的 `dependencies` 和 `devDependencies` 中的内容，分别是项目生产环境和测试环境所需要的依赖，必须安装。
```
cd projectName
// 在项目根目录中，执行 npm install 命令，安装依赖
npm install 
```
![package.json 文件](http://img.blog.csdn.net/20180105114919726?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
依赖安装完成后，项目根目录中会出现 node_modules 文件夹，项目需要的依赖包资源存放于此。
##6. 测试
```
npm run dev
```
命令执行后，会提示你项目运行的地址和端口，打开浏览器访问即可。默认是 localhost:8080，如果端口被占用，可能会用 8081,8082等等。
![Vue 提示信息](http://img.blog.csdn.net/20180105115810142?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQva2lrYWphY2s=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
根据提示，打开浏览器，访问 localhost:8081，如果出现 Vue 页面，表示项目构建正常。

对 `npm run dev` 的解释：
`package.json` 文件中，还有一个 `scripts` 属性，这里面放的是一些脚本，简化执行的命令，比如在项目根目录中运行命令 `npm run dev`，npm 会在 `package.json` 文件中找 `scripts` 属性中的 `dev` 属性，并运行。最终运行的命令就是 `webpack-dev-server --inline --progress --config build/webpack.dev.conf.js`
##7. 框架概述
[参考这里](https://github.com/cy0707/Learn_Vue/issues/6)
 rc 结尾的文件代表运行时自动加载的文件，配置等。
```
.babelrc ------bebel的相关配置
.editorconfig-----文本
.eslintignore----eslint代码检测忽略的文件
.eslintrc.js-----eslint配置文件
.gitignore-----git提交的忽略的文件
.postcssrc.js-----postcss的配置文件
index.html-----项目的入口文件
package.json------整个项目配置信息
README.md----项目说明
build/----构建相关的文件
config/------项目的配置文件
node_modules/-----项目的依赖
src/------项目的源文件
static/----没用到过
```
####1. babel
用于解决浏览器兼容性问题。Babel6.x 版本之后，所有的插件都是可插拔的。这也意味着你安装了Babel之后，是不能工作的，需要配置对应的.babelrc文件才能发挥完整的作用。
####2. EditorConfig
EditorConfig帮助开发人员定义和维护一致的编码风格在不同的编辑器和IDE。EditorConfig项目包含一个文件格式定义编码风格和文本编辑器插件的集合。EditorConfig文件易于阅读并且他们与版本控制器很好地合作。
####3.ESlint
ESLint是一个QA工具，用来避免低级错误和统一代码的风格。ESLint被设计为完全可配置的。
ESlint支持三种配置文件：Javascript，JSON，YAML。
####4. eslintignore 和 gitignore
eslintignore 中放不进行eslint的检测的文件或目录。

在Git工作区的根目录下创建一个特殊的.gitignore文件，然后把要忽略的文件名填进去，Git就会自动忽略这些文件。
####5. postcss
处理css的的插件。