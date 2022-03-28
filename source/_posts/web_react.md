---
title: react
date: 2018-06-14 14:53:00
categories: 
- computer
- web
tags: react
description: react基础知识
---     

# 1. react

## 1.1. JSX

react使用JSX代替javascript，JSX是有点像XML的javascript语法扩展。

### 1.1.1. 基本语法

ReactDOM.render(
  div 
    h1 react教程 /h1
    h2 欢迎学习 React /h2
    p 这是一个很不错的 JavaScript 库! /p
  /div
document.getElementById('example'))

多个html标签需要全包含在一个div中，以上代码表示在id=example的html标签位置渲染一个div，里面有两个标题和一个段落。

### 1.1.2. 嵌入javascript表达式

* JSX中可以嵌入javascript表达式，但需要写在花括号{}内。
* 注释也需要写在花括号中。
* 花括号中的数组会自动展开所有成员。
* 表达式不能使用`if else`语句，但可以用`conditional`三元运算表达式。  
  `<h1>{i == 1 ? 'True!' : 'False'}</h1>`

### 1.1.3. 内联样式

使用驼峰法设置内联样式

ReactDOM.render(
  h1 style = {myStyle} 菜鸟教程 /h1
  document.getElementById('example'))

### 1.1.4. 渲染HTML标签（strings）和React组件（classes）

React 可以渲染 HTML 标签 (strings) 或 React 组件 (classes)。

要渲染 HTML 标签，只需在 JSX 里使用小写字母的标签名。


要渲染 React 组件，只需创建一个大写字母开头的本地变量。


### 1.1.5. tips

由于 JSX 就是 JavaScript，一些标识符像 class 和 for 不建议作为 XML 属性名。作为替代，React DOM 使用 className 和 htmlFor 来做对应的属性。

## 1.2. 组件

组件API的7个方法：

* 设置状态：setState
* 替换状态：replaceState
* 设置属性：setProps
* 替换属性：replaceProps
* 强制更新：forceUpdate
* 获取DOM节点：findDOMNode
* 判断组件挂载状态：isMounted

react元素是DOM标签或者用户自定义组件，当是自定义组件时，会将JSX属性作为单个对象传递给该组件，这个对象是“props”。  
组件名称必须以大写字母开头。  
组件的返回值只能有一个根元素。  
所有的React组件必须像纯函数（即不能改变输入值的函数）那样使用它们的props。  
React应用中，按钮、表单、对话框、整个屏幕的内容等，这些通常都被表示为组件。

## 1.3. 生命周期

1. 组件的生命周期可分成三个状态：

* Mounting：已插入真实 DOM
* Updating：正在被重新渲染
* Unmounting：已移出真实 DOM

2. 生命周期的方法有：

* componentWillMount 在渲染前调用,在客户端也在服务端。
* componentDidMount : 在第一次渲染后调用，只在客户端。之后组件已经生成了对应的DOM结构，可以通过this.getDOMNode()来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作(防止异部操作阻塞UI)。
* componentWillReceiveProps 在组件接收到一个新的 prop (更新后)时被调用。这个方法在初始化render时不会被调用。
* shouldComponentUpdate 返回一个布尔值。在组件接收到新的props或者state时被调用。在初始化时或者使用forceUpdate时不被调用。
* 可以在你确认不需要更新组件时使用。
* componentWillUpdate在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。
* componentDidUpdate 在组件完成更新后立即调用。在初始化时不会被调用。
* componentWillUnmount在组件从 DOM 中移除的时候立刻被调用。