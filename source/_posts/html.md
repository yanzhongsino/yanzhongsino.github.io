---
title: html
date: 2018-06-04 15:53:00
categories: web
tags: html
description: html基础知识
---
# HTML基础知识


### 格式

html lang="en"
!--设置语言--!
!--head部分开始--!
head eta charset="UTF-8"
!--设置编码方式--!
meta name="viewport" content="width=device-width, initial-scale=1.0"
!--设置网页内容宽度为显示设备的宽度，初始比例为1.0--!
meta http-equiv="X-UA-Compatible" content="ie=edge"
!--兼容IE浏览器--
link rel="stylesheet" type="text/css" href="reset.css"
!--链接重置样式的reset.css--!
link rel="stylesheet" type="text/css" href="index.css"
!--链接应用于网页的样式--!
script src="index.js" /script
!--链接应用于网页的js文件--!
titleDocument/title
!--title会显示在浏览器的顶部标签页--!
/head
!--head部分结束--!
body
!--网页的主体内容--!
/body
/html


## 表单form

使用label来联系，

## tips

* 设置多个`<input type=radio>`时，需要把多个input的name设置成一样，才会实现在多个radio中单选的效果。