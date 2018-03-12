###JSON格式
JSON(JavaScript Object Notation, JS 对象标记) 是一种轻量级的数据交换格式。JSON用完全独立于编程语言的文本格式来存储和表示数据。实际上JSON就是字符串，描述了一个对象（花括号）或数组（方括号）。例如：`'{"name":"jack","age":25}'`或`'[{"name":"jack","age":25},  {"name":"rose","age":13}]'`。
JSON格式特别严格，必须遵守：
1. 键名必须加双引号。
2. 属性值只能是数值（10进制），字符串（加双引号），布尔值，null，数组或符合JSON要求的对象，不可以是函数，NaN，Infinity，undefined。
3. 最后一个属性后面不能有逗号
4. 小数点后必须有数字
###JSON在线校验网站
http://www.bejson.com/
###JavaScript处理JSON的三大方法
 1.  JSON.stringify(value[, replacer[, space]])：将JavaScript对象序列化成字符串。
 参数说明：
- value:必需， JavaScript对象。
- replacer:可选。函数或数组。
如果 replacer 为函数，则 JSON.stringify 将调用该函数，并传入每个成员的键和值，这个函数必须对每一项都有返回。使用返回值而不是原始值。函数必须针对每一个原来的属性值都要有新属性值的返回。如果是数组形式，那么key是索引，而value是这个数组项。
如果 replacer 是一个数组，只有在数组中出现的属性才会被序列化进结果字符串。成员的转换顺序与键在数组中的顺序一样。当 value 参数也为数组时，将忽略 replacer 数组。
- space:可选，文本添加缩进、空格和换行符，不要用。

 因为大部分JavaScript对象的语法并不严格，所以JSON.stringify会对参数值进行处理：
- 键名自动加双引号。
- 非数组对象的属性，可能会乱序。
- 非数组对象中的undefined属性的元素、函数，序列化过程中会忽略。但是数组对象中的undefined属性的元素、函数会变为null。
- 不论在数组还是非数组的对象中，NaN、Infinity和-Infinity都被转化为null
```javascript
var manObj={  
    "firstName": "Kika",
    "lastName": "Jack",
    "age":18
};

var manStr1=JSON.stringify(manObj,function(k,v) {
  if (k === "age") {
    return "000" + v;
  } else {
    return v;
  }
});
var manStr2=JSON.stringify(manObj,["firstName","age","address"]);
```

 2.JSON.parse(text[, reviver])  :解析字符串为JavaScript对象
 对字符串格式要求严格，要是字符串格式有误，会报错。
 参数说明：
- text:必需， 一个有效的 JSON 字符串。
- reviver: 可选，一个转换结果的函数， 处理解析后的每一个属性并返回

```JavaScript
JSON.parse(str,function(k,v){  
    console.log(k);
    console.log(v);
});
```

 3.toJSON()：将 JavaScript对象转换为字符串，并格式化为 JSON 数据格式。如果一个对象上实现了toJSON方法，调用JSON.stringify去序列化这个对象时，JSON.stringify会序列化这个对象的toJSON方法返回的值。Date类型可以直接传给JSON.stringify做参数，就是因为Date类型内置了toJSON方法。

```javascript
var info={  
    "msg":"data...",
    "toJSON":function(){
        var msg=new Object();
        msg["msg"]="hehe";
        return msg;
    }
};
//返回{"msg":"hehe"}
JSON.stringify(info); 

var d=new Date();
var n=d.toJSON();
```
