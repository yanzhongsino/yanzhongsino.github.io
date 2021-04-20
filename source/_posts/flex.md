---
title: flex
date: 2018-06-12 14:53:00
categories: 
- computer
    - web
tags: 
- flex
description: flex基础知识
---

# flex

以下是学习[阮一峰老师教程](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)后的笔记。  
传统布局解决方案基于**盒模型**，常使用`display`、`position`和`float`属性来实现，但对某些特殊布局非常不方便，比如垂直居中。  
使用**`display:flex`**可以方便地实现多个元素的优雅排版，居中对齐也很方便。

## flex基本概念

1.  任何元素都可以指定为flex布局。

    * 块级元素 `display:flex;`
    * 行内元素 `display:inline-flex;`

    _注意：设置flex布局后，子元素的`float`、`clear`和`vertical-align`属性就会失效。_

2.  对元素采用flex布局后，此元素被称为**flex容器**，它的所有子元素成为容器成员，被称为**flex项目**。  
    容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做`main start`，结束位置叫做`main end`；交叉轴开始位置叫做`cross start`，结束位置叫做`cross end`。  
    项目默认沿主轴排列。单个项目占据的主轴空间叫`main size`，占据的交叉轴空间叫`cross size`。

### 容器的属性

#### flex-flow

`flex-flow`属性是`flex-direction`和`flex-wrap`两个属性的简写，默认值为`row nowrap`。

1.  flex-direction

    `flex-direction`属性决定主轴的方向，它有四个可选的值。

    * row（default）：主轴为水平方向，起点在左端。
    * row-reverse：主轴为水平方向，起点在右端。
    * column：主轴为垂直方向，起点在上沿。
    * column-reverse：主轴为垂直方向，起点在下沿。

2.  flex-wrap

    `flex-wrap`属性决定，如果一条轴线排不下所有项目的情况下如何换行，有三个可选的值。

    * nowrap\(default\)：不换行。
    * wrap：换行，从`main start`方向往`main end`方向排列，第一行在`main start`方向（上方）。
    * wrap-reverse：换行，从`main end`方向往`main start`方向排列，第一行在`main end`方向（下方）。

#### 对齐方式

1.  justify-content  
    `justify-centent`属性定义了项目在主轴上的对齐方式，与定义的主轴方向`flex-direction`有关，下面假设主轴从左到右，有四个可选的值。

    * flex-start（default）：左对齐
    * flex-end：右对齐
    * center： 居中
    * space-between：两端对齐，项目之间的间隔都相等。
    * space-around：每个项目两侧的间隔相等。所以，项目之间的间隔比项目与边框的间隔大一倍。

2.  align-items  
    `align-items`属性定义了项目在交叉轴上的对齐方式，与定义的主轴方向`flex-direction`有关，下面假设交叉轴从上到下，有五个可选的值。

    * stretch（default）：如果项目未设置高度或设为auto，将占满整个容器的高度。
    * flex-start：交叉轴的起点对齐。
    * flex-end：交叉轴的终点对齐。
    * center：交叉轴的中点对齐。
    * baseline: 项目的第一行文字的基线对齐。

3.  align-content  
    `align-content`属性定义了多根轴线的对齐方式。如果项目只有一根轴线，该属性不起作用。有六个可选的值。

    * stretch（default）：轴线占满整个交叉轴。
    * flex-start：与交叉轴的起点对齐。
    * flex-end：与交叉轴的终点对齐。
    * center：与交叉轴的中点对齐。
    * space-between：与交叉轴两端对齐，轴线之间的间隔平均分布。
    * space-around：每根轴线两侧的间隔都相等。所以，轴线之间的间隔比轴线与边框的间隔大一倍。

### 项目属性

#### order

`order`属性定义项目的排列顺序，数值越小，排列越靠前，默认为0。

#### align-self

`align-selff`属性允许单个项目有与其他项目不一样的对齐方式，可覆盖`align-items`属性，默认为auto，表示继承父元素的`align-items`属性，如果没有父元素，则等同于`stretch`。有六个可选的值，除了auto，其他都与align-items属性完全一致。

* auto(default)
* flex-start
* flex-end
* center
* baseline
* stretch

#### flex

`flex`属性是`flex-grow`、`flex-shrink`、`flex-basis`三个属性的简写，默认值为`0 1 auto`，后两个属性可选。  
flex属性有两个快捷键：`auto(1 1 auto)`和`none(0 0 auto)`。

1.  flex-grow  
    `flex-grow`属性定义如果存在剩余空间，项目的放大比例，默认为0，即不放大。  
    如果容器存在多余空间，多余空间分配给各个项目的比例按各个项目的flex-grow的值占容器内所有项目的flex-grow值的和的比例来确定。
2.  flex-shrink  
    `flex-shrink`属性定义了如果容器空间不足，项目的缩小比例，默认为1，负值无效。  
    如果容器的空间不足，需要缩小的空间会分配给各个项目，并按各个项目的flex-shrink值占容器内所有项目的flex-shrink值和的比例来确定。
3.  flex-basis  
    `flex-basis`属性定义了在放大和缩小之前，项目占据的主轴空间（main size），值为长度值，默认值为`auto`，即项目本来大小。s
