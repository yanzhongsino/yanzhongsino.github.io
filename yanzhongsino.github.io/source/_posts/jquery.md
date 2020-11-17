

# jQuery基础知识

发表于 2018-06-05 | 分类于 [javascript](/categories/javascript/)

## [](#jquery的安装 "jquery的安装")jquery的安装

只需要在html的script标签中引用jquery，两种方式：

* 通过CDN（内容分发网络）引用jquery，多个CDN可以使用。比如百度CDN，在html中按如下语法引用。

  `<script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"></script>`

* 在[jquery.com](http://jquery.com/download/)中下载jquery文本，它是一个js文件，在script标签中引用这个文件即可。`<script type="text/javascript" src="flexible.js"></script>`

## [](#jquery语法 "jquery语法")jquery语法

### [](#基础语法 "基础语法")基础语法

`$(selector).action()`

* 美元符号`$`定义 jQuery
* 选择符`(selector）`“查询”和”查找” HTML 元素，选择器类似css选择器。
* jQuery 的 `.action()`执行对元素的操作，主要通过方法的实现来达到jquery的目的。
* jQuery允许我们在相同元素上运行多条jQuery命令，一条接一条，这称为链接技术（chaining）。

## [](#jquery基础知识 "jquery基础知识")jquery基础知识

`$.each(array/object,callback)`可以遍历数组或者对象

`$.trim(string)`用于去除字符串两端的空白字符

`$(selector).get([index])`get方法通过索引在获取的dom对象中查找元素，从0开始索引对象。

`$(selector).index(selector/element)`index方法通过已知的元素搜索对应的索引值。

## [](#jquery动画 "jquery动画")jquery动画

### [](#三种隐藏元素的方式 "三种隐藏元素的方式")三种隐藏元素的方式

#### [](#基础隐藏显示动画-display-none） "基础隐藏显示动画 (display:none）")基础隐藏显示动画 \(display:none）

1.  `$(selector).hide([ duration ], [ easing ], [ complete ])`  
    隐藏，只有元素的`display`不为`none`时才有效，options可以为隐藏的时长，就产生了动画效果。
2.  `$(selector).show([ duration ], [ easing ], [ complete ])`  
    显示，只有元素的`display：none`时才有效，options可以为显示的时长。
3.  `$(selector).toggle([ duration ], [ easing ], [ complete ])`  
    hide和show的切换，动画效果是从右至左，横向动作。

    以上三种方法修改的是元素的`display`属性，会影响元素的`width`、`height`和`opacity`三个属性。

#### [](#上卷下拉效果-height-0 "上卷下拉效果 (height:0)")上卷下拉效果 \(height:0\)

1.  `$(selector).slideDown([ duration ], [ easing ], [ complete ])`  
    下拉效果
2.  `$(selector).slideUp([ duration ], [ easing ], [ complete ])`  
    上卷效果
3.  `$(selector).slideToggle([ duration ], [ easing ], [ complete ])`  
    slideDown和slideUp的切换，动画效果是从下至上，竖向动作。

    以上三种方法修改的是元素的`height`属性。

#### [](#淡入淡出效果-opacity-0 "淡入淡出效果 (opacity:0)")淡入淡出效果 \(opacity:0\)

1.  `$(selector).fadeOut([ duration ], [ easing ], [ complete ])`  
    淡出效果
2.  `$(selector).fadeIn([ duration ], [ easing ], [ complete ])`  
    淡入效果
3.  `$(selector).fadeToggle([ duration ], [ easing ], [ complete ])`  
    fadeOut和fadeIn的切换。
4.  `$(selector).fadeTo(duration,opacity,callback)`

    以上四种方法修改的是元素的`opacity`属性，不会影响元素的高度和宽度。

### [](#自定义动画 "自定义动画")自定义动画

`$(selector).animate( properties ,[ duration ], [ easing ], [ complete ] )`

其中，`properties`是不小于一个css属性的键值对构成的object对象，所有用于动画的属性必须是**数字**的。`duration`设置动画执行的时间。`easing`规定每个动画的每一步完成后要执行的函数。`progress`规定每一次动画调用时会执行这个回调，是一个进度的概念。`complete`动画完成的回调函数。

`$(selector).stop( [clearQueue ], [ jumpToEnd ] ) .stop( [queue ], [ clearQueue ] ,[ jumpToEnd ] )`

* .stop\(\); 停止当前动画，点击在暂停处继续开始
* .stop\(true\); 如果同一元素调用多个动画方法，尚未被执行的动画被放置在元素的效果队列中。这些动画不会开始，直到第一个完成。当调用.stop\(\)的时候，队列中的下一个动画立即开始。如果clearQueue参数提供true值,那么在队列中的动画其余被删除并永远不会运行
* .stop\(true,true\); 当前动画将停止，但该元素上的 CSS 属性会被立刻修改成动画的目标值。

## [](#jquery-操作html的DOM "jquery 操作html的DOM")jquery 操作html的DOM

### [](#捕获 "捕获")捕获

1.  .text\(\) - 设置或返回所选元素的文本内容
2.  .html\(\) - 设置或返回所选元素的内容（包括 HTML 标记）
3.  .val\(\) - 设置或返回表单字段的值
4.  .attr\(\) -设置或返回属性值  
    以上四种方法，如果没有参数，就返回相应值；如果有参数，参数为设置的值。

### [](#添加内容 "添加内容")添加内容

1.  .append\(“追加内容”\) 在被选元素的结尾插入内容（仍然在该元素内部）
2.  .prepend\(“追加内容”\) 在被选元素开头插入内容（仍然在该元素内部）
3.  .after\(“追加内容”\) 在被选元素的后面插入内容（不在该元素内部）
4.  .before\(“追加内容”\) 在被选元素的前面插入内容（不在该元素内部）  
    以上四种方法都可以有多个参数，实现批量追加内容。

### [](#删除内容 "删除内容")删除内容

1.  .remove\(\) 删除被选元素及其子元素；接受一个jquery选择器语法的参数，用于过滤被删元素。
2.  .empty\(\) 从被选元素中删除子元素

### [](#操作CSS "操作CSS")操作CSS

1.  addClass\(“classname”\) - 向被选元素添加一个或多个类
2.  removeClass\(“classname”\) - 从被选元素删除一个或多个类
3.  toggleClass\(“classname”\) - 对被选元素进行添加/删除类的切换操作
4.  css\(“propertyname”，“value”\) /css\(\{“propertyname”:”value”,”propertyname”:”value”,…\}\)- 若无设置value值，则返回指定样式属性；若设置value值，就设置样式属性。

### [](#控制尺寸 "控制尺寸")控制尺寸

1.  width\(\) 方法设置或返回元素的宽度（不包括内边距、边框或外边距）。
2.  height\(\) 方法设置或返回元素的高度（不包括内边距、边框或外边距）。
3.  innerWidth\(\) 方法返回元素的宽度（包括内边距）。
4.  innerHeight\(\) 方法返回元素的高度（包括内边距）。
5.  outerWidth\(\) 方法返回元素的宽度（包括内边距和边框）。
6.  outerHeight\(\) 方法返回元素的高度（包括内边距和边框）。

## [](#DOM遍历 "DOM遍历")DOM遍历

1.  祖先
    * parent\(\) 方法返回被选元素的直接父元素。
    * parents\(\) 方法返回被选元素的所有祖先元素，它一路向上直到文档的根元素 \(\)。
    * parentsUntil\(\) 方法返回介于两个给定元素之间的所有祖先元素。
2.  后代
    * children\(\) 方法返回被选元素的所有直接子元素。
    * find\(\) 方法返回被选元素的后代元素，一路向下直到最后一个后代。
3.  同胞
    * siblings\(\) 方法返回被选元素的所有同胞元素。
    * next\(\) 方法返回被选元素的下一个同胞元素。
    * nextAll\(\) 方法返回被选元素的所有跟随的同胞元素。
    * nextUntil\(\) 方法返回介于两个给定参数之间的所有跟随的同胞元素。
    * prev\(\), prevAll\(\) 以及 prevUntil\(\) 方法的工作方式与上面的方法类似，只不过方向相反而已：它们返回的是前面的同胞元素（在 DOM 树中沿着同胞之前元素遍历，而不是之后元素遍历）。
4.  过滤
    * first\(\) 方法返回被选元素的首个元素。
    * last\(\) 方法返回被选元素的最后一个元素。
    * eq\(\) 方法返回被选元素中带有指定索引号的元素。
    * filter\(\) 方法允许您规定一个标准。不匹配这个标准的元素会被从集合中删除，匹配的元素会被返回。
    * not\(\) 方法返回不匹配标准的所有元素，与filter\(\)方法相反。

## [](#tips "tips")tips

使用jquery实现相同样式容器的批量导入。  
从json文件中取数据并遍历数据，然后通过每条数据与重复项字符串的手动拼接，获得字符串格式的html结构，在把这个结果字符串通过`$("selector").append(结果字符串)`到指定位置，即可实现导入相同样式容器到html中。

[\# html](/tags/html/) [\# css](/tags/css/) [\# javascript](/tags/javascript/) [\# jQuery](/tags/jQuery/)

[javascript基础知识](/2018/06/05/javascript基础知识/ "javascript基础知识")

[GitHube Pages+Hexo搭建独立静态博客](</2018/06/05/GitHube Pages+Hexo搭建独立静态博客 /> "GitHube Pages+Hexo搭建独立静态博客")

* 文章目录
* 站点概览

yanzhong

在学习前端的路上的一些笔记和学习心得。

[11 日志](/archives/)

[7 分类](/categories/index.html)

[9 标签](/tags/index.html)

<!--noindex-->

1.  [1. jquery的安装](#jquery的安装)
2.  [2. jquery语法](#jquery语法)
    1.  [2.1. 基础语法](#基础语法)
3.  [3. jquery基础知识](#jquery基础知识)
4.  [4. jquery动画](#jquery动画)
    1.  [4.1. 三种隐藏元素的方式](#三种隐藏元素的方式)
     1.  [4.1.1. 基础隐藏显示动画 \(display:none）](#基础隐藏显示动画-display-none）)
     2.  [4.1.2. 上卷下拉效果 \(height:0\)](#上卷下拉效果-height-0)
     3.  [4.1.3. 淡入淡出效果 \(opacity:0\)](#淡入淡出效果-opacity-0)
    2.  [4.2. 自定义动画](#自定义动画)
5.  [5. jquery 操作html的DOM](#jquery-操作html的DOM)
    1.  [5.1. 捕获](#捕获)
    2.  [5.2. 添加内容](#添加内容)
    3.  [5.3. 删除内容](#删除内容)
    4.  [5.4. 操作CSS](#操作CSS)
    5.  [5.5. 控制尺寸](#控制尺寸)
6.  [6. DOM遍历](#DOM遍历)
7.  [7. tips](#tips)

<!--/noindex-->

