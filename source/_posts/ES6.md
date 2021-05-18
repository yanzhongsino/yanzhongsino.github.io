---
title: ES6
date: 2018-06-12 14:53:00
categories: 
- computer
- web
tags: 
- ES6
description: ES6基础知识
---  

## 常量

const pi = 3.1415926  
常量是只读的，不可修改。

## 作用域

ES6中有块级作用域  
const、let，在每次循环内都有自己的闭包和作用域。  
{}内就是块级作用域。

## 箭头函数

1.  语法：(参数)=>{函数体}  
    当只有一个参数时，可省略()；当函数体只有return something时，可以省略{}。eg：i=>i+1
2.  箭头函数的this是定义该名称时this的指向

## 默认参数

function func(x=1,y=5,z=20){  
return x+y+z  
}

### 可变参数

不知数量参数的求和  
function func(…a){  
a.forEach(item=>{  
sum+=item*1  
});  
return sum  
}

### 利用扩展运算符合并数组

var params = [‘hello’,true,1,23]  
var concatlist = [1,2,’world’,…params]  
相当于concatlist为[1,2,’world’,’hello’,true,1,23]

## 对象代理

私有属性  
let Person = {  
name: ‘karen’,  
sex: ‘female’,  
age: 18  
}

let person = new Proxy(Person,{  
get(target,key){  
return target[key]  
},  
set(target,key,value){  
if(key!==’sex’){  
target[key]=value;  
}  
}  
})  
通过Proxy对象代理实现保护Person的sex属性，使其不能修改。