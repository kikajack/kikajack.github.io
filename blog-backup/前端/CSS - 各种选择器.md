#1. 通配符选择器 `*`
通配符选择器 `*` 可以选中所有的元素。作用范围太大，可能会导致性能比较差。
```
* {
  margin: 0px;
}
```
#2. ID 选择器 `#`
ID 选择器每次只选中一个具有指定 ID 的元素。
ID 在每个 HTML 页面上具有唯一性，稀缺性。
`<div id="nav"></div>`
```
#nav {
  position: relative;
}
```
#3. 类选择器 `.`
类选择器每次选中一系列具有相同 class 的元素。
`<div class="good"></div>`
```
.good {
  color: red;
}
```
#4. 交集选择器
交集选择器只选中同时满足所有这些选择器的元素。
两个或多个选择器相邻，且选择器之间没有任何的连接符号，构成交集选择器。选择器可以是标签名、id 或 class 类名。
```
<p class="good">this is p1</p>
```
```
p.good {
  color: red;
}
```
#5. 并集选择器 `,`
并集选择器选中所有的含有并集选择器中的至少一个选择器的元素。
选择器之间利用逗号 `,` 连接。选择器可以是标签名、id 或 class 类名。
```
<h1 class="h">this is h</h1>
<p class="p">this is p </p>
```
```
.h, .p {    /* class 为 p 或 h 的元素都会选中*/
  color: red;
}
```
#6. 后代选择器 空格 ` `
后代选择器选中具有指定**后代关系（包括但不限于父子，爷孙等）**的子元素。
选择器之间利用空格 ` ` 连接。
```
<div class="d1">
  <div class="d2">
    <p class="p">this is p1 </p>
  </div>
  <p class="p">this is p2 </p>
</div>
```
```
.d1 .p { /* 不管父子还是爷孙，都是后代 */
  color: red;
}
```
#7. 子元素选择器 `>`
子元素选择器选中具有指定**父子关系**的子元素。
选择器之间利用 `>` 连接。
```
<div class="d1">
  <div class="d2">
    <p class="p">this is p1 </p>
  </div>
  <p class="p">this is p2 </p>
</div>
```
```
.d1 > .p { /* 只选择父子关系 */
  color: red;
}
```
#8. 相邻兄弟选择器 `+`
相邻兄弟选择器：可选择紧接在第一个元素后的元素，且二者有相同父元素。
```
<div class="d1">--content--</div>
<p>this is p</p>
<ul>
  <li>List item 1</li>
  <li>List item 2</li>
  <li>List item 3</li>
</ul>
```
```
li + li { /* 第二个和第三个列表项变为红色。第一个列表项不受影响。 */
  color: red;
}
d1 + p {
  margin-top:50px;
}
```
#9. 通用兄弟选择器 `~`
通用兄弟选择器：第二个元素必须跟（不一定是紧跟）在第一个元素之后，且他们都有一个共同的父元素。
```
<span>This is not red.</span>
<p>Here is a paragraph.</p>
<code>Here is some code.</code>
<span>And here is a span.</span>
```
```
p ~ span { /* 只影响 p 元素后面的所有的 span 元素 */
  color: red;
}
```
#10. 属性选择器 `[]`
属性选择器选择带有特殊属性的标签。
常用于区分 input 标签的属性。
|用法|含义|
|-|-|
|E[attr]|存在 attr 属性的元素|
|E[attr=val]|存在 attr 属性且值完全等于 val的元素|
|E[attr*=val]|存在 attr 属性且值的任意位置包含 val 的元素|
|E[attr^=val]|存在 attr 属性且值的开始位置包含 val 的元素|
|E[attr$=val]|存在 attr 属性且值的结束位置包含 val 的元素|
```
<p class="good">this is p1</p>
<input type="text" value="this is input">
<input type="password" id="password">
```
```
p[class]{
  color: red;
}
input[type=password]{
  color: orange;
}
```
#11. 伪元素选择器
为了和伪类区分，伪元素选择器用了两个冒号开头，`::first-letter`
|语法|含义|
|-|-|
|::before|在元素前添加内容，必须含有 content 属性|
|::after|在元素后添加内容，必须含有 content 属性|
|::first-letter|文本的第一个字|
|::first-line|文本的第一行|
|::selection|用户选中的文本|

```
<div class="d1">--content--</div>
```
```
.d1::before {
  content: "before";
}
.d1::after {
  content: "after";
}
```