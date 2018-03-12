#1. link 用法
语法：
```
<link href="url路径" rel="stylesheet" type="text/css" media="媒体查询列表"/>
```
示例：
```
<link href="common.css" rel="stylesheet" type="text/css" media="print,tv"/>
```

link 是 HTML 文件中的标签，在 `<head>` 标签中使用。如果指定了媒体查询类型，则只有在满足条件的情况下才会引入指定的样式文件。
#2. @import 
[MDN 中的资料](https://developer.mozilla.org/zh-CN/docs/Web/CSS/@import)

语法：
```
@import url;
@import url 媒体查询列表; /*一个逗号分隔的 媒体查询 条件列表，满足条件时才引入 CSS。*/
```
示例：
```
@import url("fineprint.css") print;
@import url("bluish.css") projection, tv;
@import 'custom.css';
@import url("chrome://communicator/skin/");
@import "common.css" screen, projection;
@import url('mobile.css') (max-width: 680px);
@import url('landscape.css') screen and (orientation:landscape);
```
`@import` 是 CSS 中的一个[@规则](https://developer.mozilla.org/en-US/docs/Web/CSS/At-rule)，必须先于所有其他类型的规则（`@charset` 规则除外），结尾需要加分号。`@import` 用于**从其他样式表导入样式规则**。因为必须要用 CSS 引擎来解析，所以只能出现在 CSS 文件中或 HTML文件的 `<style>` 标签中。
#3. link 和 @import 异同点
1. link 是 HTML 文件中的标签，在 `<head>` 标签中引入 CSS 文件。
2. `@import ` 是 CSS 中的一个 @规则，只能出现在 CSS 文件中或 HTML文件的 `<style>` 标签中。
3. @import 和 link 一样，都可以定义媒体查询(media queries)

#4. 非模块化开发时尽量不要用 @import
##1. 非模块化开发
**`@import` 引入的样式在所在的 CSS 文件加载完成后再加载，不推荐使用。**

正常开发时，所有的 CSS 文件都需要引入。如果在某个 CSS 文件中使用了 `@import` ，浏览器要先下载使用了 `@import` 的 CSS，解析完后发现有另外的 CSS 文件需要下载，再去下载、解析，增加了用户的等待时间。
如果 CSS 内容不多, 可以合并到一个文件里, 减少请求次数。对于多个独立的 CSS 文件, 最好直接用 link 标签加载。

##2. 模块化开发
在用 webpack 等工具开发时，会合并 CSS 文件。如果 CSS 文件相互之间有依赖，可以直接用也只能用 `@import` 引入，最后构建好的文件会合并文件，不会出现 `@import`。