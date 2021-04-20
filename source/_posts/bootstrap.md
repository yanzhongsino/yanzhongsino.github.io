---
title: bootstrap
date: 2018-06-11 15:53:00
categories: 
- computer
    - web
tags: 
- bootstrap
description: bootstrap基础知识
---   

# bootstrap

## bootstrap简介

bootstrap是一个用于快速开发web应用和网站的前端框架。  
它能用于开发响应式布局、移动设备优先的web项目。

## bootstrap的安装

需要在html的link标签引入bootstrap的css插件bootstrap.css，并在script标签中引用bootstrap的js插件bootstrap.js(bootstrap的js插件使用需要先引入jQuery)，有两种方式：

#### 通过CDN（内容分发网络）引用bootstrap，多个CDN可以使用。比如[bootCDN](http://www.bootcdn.cn/bootstrap/)。

#### 在[bootstrap中文网](http://v3.bootcss.com/)下载bootstrap，是一个文件夹，下有css、js、fonts三个目录。

在link标签中引入文件夹的css目录下的bootstrap.min.css文件

`<link href="css/bootstrap.min.css" rel="stylesheet">`

在script标签中引用文件夹的js目录下的bootstrap.min.js文件(bootstrap的js插件使用需要先引入jQuery)

## bootstrap的css

1.  使用`<img src="" class="img-responsive">`使图像对响应式布局更友好地支持。

2.  Bootstrap 排版、链接样式设置了基本的全局样式。分别是：

    * 为 body 元素设置 `background-color: #fff`;
    * 使用 `@font-family-base`、`@font-size-base` 和 `@line-height-base a`变量作为排版的基本参数
    * 为所有链接设置了基本颜色 `@link-color` ，并且当链接处于 :hover 状态时才添加下划线
    * 这些样式都能在 scaffolding.less 文件中找到对应的源码。

3.  使用**normalize.css**来建立跨浏览器的一致性。

4.  .container 类用于固定宽度并支持响应式布局的容器。.container-fluid 类用于 100\% 宽度，占据全部视口（viewport）的容器。

## 网格系统
