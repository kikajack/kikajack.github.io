#1. 原理及特性
原理：通过 script 标签引入 JavaScript 代码不受同源策略限制。
```
<script src="http://xx.com/my_function.php?a=x&b=666"></script>
```
特性：JSONP 兼容所有的浏览器。但是只能用 GET 请求。
script 标签引入的文件内容是不能够被客户端的 JavaScript  获取到的，不会影响到被引用文件的安全，所以 script标签引入的文件不用遵循浏览器的同源策略。而通过ajax加载的文件内容是能够被客户端 JavaScript  获取到的，所以ajax必须遵循同源策略。
#2. 通过 JSONP 引入函数
首先在本地 JavaScript  代码中提前定义好回调函数，然后通过 script 标签引入服务器端封装好的 JavaScript 代码，这段代码就是一个函数调用，同时将数据作为函数的参数传入。
```script
<script>
  //提前定义好回调函数
    function JSONP_my_function (users){  
        console.dir(users);  
    }  
</script>  

<script src="http://xx.com/my_function.php"></script>  
```
```php
<?php
    //返回一个 JavaScript 函数的调用
    echo 'JSONP_my_function ({"name":"jack", "age":18})';
?>  
```
#3. 通过 JSONP 引入变量
直接在服务器端把数据组装为 JavaScript  的格式返回到页面。页面在 script 节点加载完成后就可以取出数据使用。
```script
<script>
  //判断script节点是否加载完毕
    js.onload = js.onreadystatechange = function() {  
      if (!this.readyState || this.readyState === 'loaded' || this.readyState === 'complete') {  
          console.log(users);//此处取出其他域的数据  
          js.onload = js.onreadystatechange = null;  
      }  
  };  
</script>  

<script src="http://xx.com/my_function.php"></script>  
```
```php
<?php
    //返回一个 JavaScript 函数的调用
    echo 'var obj = {"name":"jack", "age":18})';
?>  
```