##1.数学函数
数学 (Math) 函数能处理 integer 和 float 范围内的值（对应 C 类型中的 long 和 double）。数学 (Math) 函数是 PHP 核心的组成部分，无需安装。
| 函数名 | 描述 | 示例 |
| ---| --- | --|
|abs() | 绝对值 |
|ceil()|  向上舍入为最接近的整数|
|decbin()|  把十进制转换为二进制|
|dechex()|  把十进制转换为十六进制|
|decoct()|  把十进制转换为八进制|
|floor()| 向下舍入为最接近的整数|
|is_nan()|  判断是否为合法数值|
|rand()|  返回随机整数|
|getrandmax()|  显示随机数最大的可能值|
|mt_rand(min,max)|  使用 Mersenne Twister 算法返回随机整数|想要 5 到 15（包括 5 和 15）之间的随机数，用 mt_rand(5, 15)|
|mt_getrandmax()| 显示随机数的最大可能值|
|pi()|  返回圆周率的值|