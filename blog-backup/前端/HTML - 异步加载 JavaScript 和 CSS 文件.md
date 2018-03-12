浏览器在解析 HTML 文件时，对于每一个 link 标签和带 src 属性的 script 标签，都会发起请求获取对应的文件。为了优化页面的加载速度，通常会把稍后才使用到的文件（比如字体文件，某个页面才会用的文件）异步加载。
#1. 异步加载 JavaScript
只要在 script 标签中添加 async 属性，即可实现异步加载。下面两种写法都可以：
```
<script async="async" src="https://js-sec.indexww.com/ht/p/185901-159836282584097.js"></script>
<script async src="https://js-sec.indexww.com/ht/p/185901-159836282584097.js"></script>
```
#2. 异步加载 CSS
async 属性对 link 标签是无效的。
##2.1 特定媒体类型的 CSS 文件
对于特定媒体类型的 CSS 文件，只要指定 media 即可按需加载。比如只有在电视上显示时才加载的样式：
```
<link rel="stylesheet" media="tv" href="tv.css" />
```
media_type: all | aural | braille | handheld | print | projection | screen | tty | tv | embossed
媒体类型更多资料，可以 [参考 MDN](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Media_queries)
##2.2 一般 CSS 文件
一般 CSS 文件的异步加载有两种实现方法：hack 方法和借助 JavaScript。
###2.2.1 hack 方法
[详情参考这里](https://codepen.io/tigt/post/async-css-without-javascript)
借助媒体类型 `media="bogus"`，然后再在 body 标签内的最后面添加引入 CSS 文件的 link 标签即可实现指定 CSS 文件的异步加载。
```
<head>
    <!-- unimportant nonsense -->
    <link rel="stylesheet" href="style.css" media="bogus">
</head>
<body>
    <!-- other unimportant nonsense, such as content -->
    <link rel="stylesheet" href="style.css">
</body>
```
###2.2.2 借助 JavaScript
[loadCSS 项目](https://github.com/filamentgroup/loadCSS) 实现了一套方法。

在页面的 head 区域通过粘贴代码或者引用文件 `loadCSS.js` 的方式完成初始化，暴露全局函数 `loadCSS`。后面需要异步加载 CSS 文件时，调用该函数并传入 URL 即可。
```
<head>
  <script type="text/javascript">
    !function(e){"use strict";var n=function(n,t,o){var l,r=e.document,i=r.createElement("link");if(t)l=t;else{var a=(r.body||r.getElementsByTagName("head")[0]).childNodes;l=a[a.length-1]}var d=r.styleSheets;i.rel="stylesheet",i.href=n,i.media="only x",l.parentNode.insertBefore(i,t?l:l.nextSibling);var f=function(e){for(var n=i.href,t=d.length;t--;)if(d[t].href===n)return e();setTimeout(function(){f(e)})};return i.onloadcssdefined=f,f(function(){i.media=o||"all"}),i};"undefined"!=typeof module?module.exports=n:e.loadCSS=n}("undefined"!=typeof global?global:this);
  </script>
</head>

<script type="text/javascript">
  loadCSS("//fonts.lug.ustc.edu.cn/css?family=Source+Code+Pro:300,600");
</script>
```