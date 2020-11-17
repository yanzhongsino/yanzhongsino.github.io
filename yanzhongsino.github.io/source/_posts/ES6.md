     

# ES6

发表于 2018-06-12 | 分类于 [javascript](/categories/javascript/)

### [](#常量 "常量")常量

const pi = 3.1415926  
常量是只读的，不可修改。

### [](#作用域 "作用域")作用域

ES6中有块级作用域  
const、let，在每次循环内都有自己的闭包和作用域。  
\{\}内就是块级作用域。

### [](#箭头函数 "箭头函数")箭头函数

1.  语法：\(参数\)=>\{函数体\}  
    当只有一个参数时，可省略\(\)；当函数体只有return something时，可以省略\{\}。eg：i=>i+1
2.  箭头函数的this是定义该名称时this的指向

### [](#默认参数 "默认参数")默认参数

function func\(x=1,y=5,z=20\)\{  
return x+y+z  
\}

#### [](#可变参数 "可变参数")可变参数

不知数量参数的求和  
function func\(…a\)\{  
a.forEach\(item=>\{  
sum+=item\*1  
\}\);  
return sum  
\}

#### [](#利用扩展运算符合并数组 "利用扩展运算符合并数组")利用扩展运算符合并数组

var params = \[‘hello’,true,1,23\]  
var concatlist = \[1,2,’world’,…params\]  
相当于concatlist为\[1,2,’world’,’hello’,true,1,23\]

### [](#对象代理 "对象代理")对象代理

私有属性  
let Person = \{  
name: ‘karen’,  
sex: ‘female’,  
age: 18  
\}

let person = new Proxy\(Person,\{  
get\(target,key\)\{  
return target\[key\]  
\},  
set\(target,key,value\)\{  
if\(key\!==’sex’\)\{  
target\[key\]=value;  
\}  
\}  
\}\)  
通过Proxy对象代理实现保护Person的sex属性，使其不能修改。

[\# html](/tags/html/) [\# css](/tags/css/) [\# javascript](/tags/javascript/) [\# ES6](/tags/ES6/)

[bootstrap](/2018/06/11/bootstrap/ "bootstrap")

[gulp](/2018/06/12/gulp/ "gulp")

* 文章目录
* 站点概览

yanzhong

在学习前端的路上的一些笔记和学习心得。

[11 日志](/archives/)

[7 分类](/categories/index.html)

[9 标签](/tags/index.html)

<!--noindex-->

1.  [1. 常量](#常量)
2.  [2. 作用域](#作用域)
3.  [3. 箭头函数](#箭头函数)
4.  [4. 默认参数](#默认参数)
    1.  [4.1. 可变参数](#可变参数)
    2.  [4.2. 利用扩展运算符合并数组](#利用扩展运算符合并数组)
5.  [5. 对象代理](#对象代理)

<!--/noindex-->
