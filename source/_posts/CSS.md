---
title: CSS
date: 2018-06-04 15:53:00
categories: 
- computer
- web
tags: 
- CSS
description: CSS基础知识
---   


# CSS基础知识

## 动画的CSS实现

1.  transform

    用于元素的2D或3D变化，默认值为none，有六种种变化方式。

    * translate(x,y)、translate3d(x,y,z)位移变化，分别为2d和3d变化方式，衍生出translateX(x)、translateY(y)、translateZ(Z)
    * scale(x,y)、scale3d(x,y,z)缩放变化，同样衍生三轴的单独变化方式。
    * rotate(angle)、rotate3d(x,y,z,angle)旋转变化，同样衍生三轴的单独变化方式。rotate旋转中心点默认是元素中心，可以通过设置`tranform-origin：x-axis y-axis z-axis`来改变旋转中心点的位置。
    * skew(x-angle,y-angle)沿着x和y轴的2d倾斜变化，衍生两轴单独变化方式。
    * matrix(n,n,n,n,n,n)、matrix3d(n,n,n,n,n,n,n,n,n,n,n,n,n,n,n,n)分别定义2d和3d转换，使用6个值或16个值（4*4）的矩阵。
    * perspective(n)为3d元素定义透视视图。

    `transform-style:flat(default)/preserve-3d`指定嵌套元素怎样在三维空间呈现，flat表示所有子元素在2d平面呈现，preserve-3d表示所有子元素在3d空间呈现。

2.  transition

    `transition`是`transition-property`、`transition-duration`、`transition-timing-function`和`transition-delay`四个属性的简写，默认值`all 0 ease 0`，是指定元素属性发生渐变的一种效果，一定要事件触发。

    * `transition-property:width`指定需要渐变变化的css属性，比如宽度的渐变。
    * `transition-duration:xs`指定渐变效果的时长为x秒，必须指定，否则为0，等同于没有渐变效果。
    * `transition-timing-function`指定transition效果的转速曲线，可取以下值：
          * linear 规定以相同速度开始至结束的过渡效果（等于 cubic-bezier(0,0,1,1)）。
          * ease 规定慢速开始，然后变快，然后慢速结束的过渡效果（cubic-bezier(0.25,0.1,0.25,1)）。
          * ease-in 规定以慢速开始的过渡效果（等于 cubic-bezier(0.42,0,1,1)）。
          * ease-out 规定以慢速结束的过渡效果（等于 cubic-bezier(0,0,0.58,1)）。
          * ease-in-out 规定以慢速开始和结束的过渡效果（等于 cubic-bezier(0.42,0,0.58,1)）。
          * cubic-bezier(n,n,n,n) 在 cubic-bezier 函数中定义自己的值。可能的值是 0 至 1 之间的数值。
    * `transition-delay:xs`指定transition效果需要等待x秒才开始。

3.  animation

    通过@keyframes定义动画的每一帧，然后使用animation属性调用动画，animation默认值是`none 0 ease 0 1 normal`，是多个属性的简写，属性依次如下：

    * `animation-name`在@keyframes中定义的动画名。
    * `animation-duration`指定动画的时长。
    * `animation-timing-function`指定动画的转速曲线，同`transition`。
    * `animation-delay`指定动画在启动前的延迟间隔，可为负值，负值时直接跳过动画的制定时间开始播放。
    * `animation-iteration-count`指定动画播放次数，值为`infinite`时无限多次播放。
    * `animation-direction`指定是否轮流反向播放动画。
    * `animation-fill-mode`规定当动画不播放时（当动画完成时，或当动画有一个延迟未开始播放时），要应用到元素的样式。
    * `animation-play-state`指定动画是否正在运行或已暂停。

**@keyframes语法**

@keyframes myanimation{}

1.  tips

配合hover等事件来实现动画效果。

### 几个要点

#### class属性

* 元素在设置样式时，首先考虑是否有多个相同元素具有同一样式和是否复用的问题，如果单个元素使用的样式或者不会复用，则优先使用内联样式，在html中就实现。
* 如果需要在css文件中实现的样式，则尽量使用class属性，使得具有同样样式的元素使用同一class属性，减少css的冗余。
* 对class属性的命明尽量语意化。

#### 初始化

* 使用**reset.css**或者**normalize.css**来初始化，使得css设置更轻松。或者在css文档开头，简单地用`*{margin:0;padding:0}`来初始化所有元素的边距，因为某些元素（例如h）自带一些边距。
* 使用**felxible.js**来在css中可以使用**rem**（为显示设备宽度的十分之一）来作为长度单位，更多地应用在移动端。

#### border-width

参考[简书](https://www.jianshu.com/p/63fdbd7fdc9b)的这个总结和[慕课网](https://www.imooc.com/video/6858)的这个使用border-width制作折角效果的视频。border-width实现三角形等图形很方便。

当content区为空（width和height都为0）时，四个方向的border分别为三角形，当四个方向的border-width都不为0时，组成一个矩形。

没有定义基本border时，未被定义的边那个方向的矩形的一半不会显示，并影响相邻边的显示。所以，若只定义了对边的边框，则相当于四个边框都未被定义。  
当定义了基本border，即使是白色或透明，在此基础上再定义边框时，则会安装定义的边框的三角形的组合来显示。

## 问题笔记

1.  任意数量图片从左到右按顺序排列，每行预期排五个，图片间有相同间距的样式的实现。  
    Q:解决分散排列，并适用于不同数量图片的问题。  
    如果直接定义父容器的`display:flex`和`justify-content:space-between`可以实现图片分散排列，但是当图片数量不是5时（比如为3时），也会展示分散排列的效果，而非占据左边2/5的位置。  
    A：定义父容器的`display:flex`和子元素图片`width:20%`即可实现。  
    Q:解决图片排列紧凑的问题。  
    A:定义子元素图片`box-sizing:border-box;padding:1%;`来实现图片间隙。
2.  实现任意元素单行排列，不换行，超出省略并显示省略号。  
    Q：强制不换行  
    A：`white-space:nowrap;`  
    Q:超出省略  
    A：`overflow: hidden;`  
    Q：超出显示省略号  
    A：`text-overflow: ellipsis;`

## tips

* 公司项目少有使用**响应式**CSS，而是移动端和web端各一套CSS，以使得网页在不同设备的展示更和谐，主要是长度设置、移动端键盘弹出遮盖内容区、移动端触摸屏更多操作等问题。
* 尽量不要设定元素的`height`，使其保持auto。
* 一些icon、line和矩形可以使用`div`或者`:before :after`来实现。
* 使用`border-randius`属性来控制矩形或者圆形。
* `white-space`属性指定元素内的空白怎么处理，文本元素取值`nowrap`时，文本强制不换行，直到遇到  
  元素。
* 使用图片作为背景时，一般设置`background-position:center`来使图片居中展示，当图片与背景尺寸不匹配时，图片仍然居中。
