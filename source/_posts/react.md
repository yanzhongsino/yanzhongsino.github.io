     

# react

发表于 2018-06-14 | 分类于 [react](/categories/react/)

## [](#JSX "JSX")JSX

react使用JSX代替javascript，JSX是有点像XML的javascript语法扩展。

### [](#基本语法 "基本语法")基本语法

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br></pre></td><td class="code"><pre><span class="line">ReactDOM.render(</span><br><span class="line">    &lt;div&gt;</span><br><span class="line">    &lt;h1&gt;react教程&lt;/h1&gt;</span><br><span class="line">    &lt;h2&gt;欢迎学习 React&lt;/h2&gt;</span><br><span class="line">    &lt;p&gt;这是一个很不错的 JavaScript 库!&lt;/p&gt;</span><br><span class="line">    &lt;/div&gt;,</span><br><span class="line">    document.getElementById('example')</span><br><span class="line">);</span><br></pre></td></tr></tbody></table>

多个html标签需要全包含在一个div中，以上代码表示在id=example的html标签位置渲染一个div，里面有两个标题和一个段落。

### [](#嵌入javascript表达式 "嵌入javascript表达式")嵌入javascript表达式

* JSX中可以嵌入javascript表达式，但需要写在花括号\{\}内。
* 注释也需要写在花括号中。
* 花括号中的数组会自动展开所有成员。
* 表达式不能使用`if else`语句，但可以用`conditional`三元运算表达式。  
  `<h1>{i == 1 ? 'True!' : 'False'}</h1>`

### [](#内联样式 "内联样式")内联样式

使用驼峰法设置内联样式

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br></pre></td><td class="code"><pre><span class="line">var myStyle = {</span><br><span class="line">    fontSize: 100,</span><br><span class="line">    color: '#FF0000'</span><br><span class="line">};</span><br><span class="line">ReactDOM.render(</span><br><span class="line">    &lt;h1 style = {myStyle}&gt;菜鸟教程&lt;/h1&gt;,</span><br><span class="line">    document.getElementById('example')</span><br><span class="line">);</span><br></pre></td></tr></tbody></table>

### [](#渲染HTML标签（strings）和React组件（classes） "渲染HTML标签（strings）和React组件（classes）")渲染HTML标签（strings）和React组件（classes）

React 可以渲染 HTML 标签 \(strings\) 或 React 组件 \(classes\)。

要渲染 HTML 标签，只需在 JSX 里使用小写字母的标签名。

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">var myDivElement = &lt;div className="foo" /&gt;;</span><br><span class="line">ReactDOM.render(myDivElement, document.getElementById('example'));</span><br></pre></td></tr></tbody></table>

要渲染 React 组件，只需创建一个大写字母开头的本地变量。

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br></pre></td><td class="code"><pre><span class="line">var MyComponent = React.createClass({/*...*/});</span><br><span class="line">var myElement = &lt;MyComponent someProperty={true} /&gt;;</span><br><span class="line">ReactDOM.render(myElement, document.getElementById('example'));</span><br></pre></td></tr></tbody></table>

### [](#tips "tips")tips

由于 JSX 就是 JavaScript，一些标识符像 class 和 for 不建议作为 XML 属性名。作为替代，React DOM 使用 className 和 htmlFor 来做对应的属性。

## [](#组件 "组件")组件

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

## [](#生命周期 "生命周期")生命周期

#### [](#组件的生命周期可分成三个状态： "组件的生命周期可分成三个状态：")组件的生命周期可分成三个状态：

* Mounting：已插入真实 DOM
* Updating：正在被重新渲染
* Unmounting：已移出真实 DOM

#### [](#生命周期的方法有： "生命周期的方法有：")生命周期的方法有：

* componentWillMount 在渲染前调用,在客户端也在服务端。
* componentDidMount : 在第一次渲染后调用，只在客户端。之后组件已经生成了对应的DOM结构，可以通过this.getDOMNode\(\)来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作\(防止异部操作阻塞UI\)。
* componentWillReceiveProps 在组件接收到一个新的 prop \(更新后\)时被调用。这个方法在初始化render时不会被调用。
* shouldComponentUpdate 返回一个布尔值。在组件接收到新的props或者state时被调用。在初始化时或者使用forceUpdate时不被调用。
* 可以在你确认不需要更新组件时使用。
* componentWillUpdate在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。
* componentDidUpdate 在组件完成更新后立即调用。在初始化时不会被调用。
* componentWillUnmount在组件从 DOM 中移除的时候立刻被调用。

[\# html](/tags/html/) [\# css](/tags/css/) [\# javascript](/tags/javascript/) [\# react](/tags/react/)

[gulp](/2018/06/12/gulp/ "gulp")

* 文章目录
* 站点概览

yanzhong

在学习前端的路上的一些笔记和学习心得。

[11 日志](/archives/)

[7 分类](/categories/index.html)

[9 标签](/tags/index.html)

<!--noindex-->

1.  [1. JSX](#JSX)
    1.  [1.1. 基本语法](#基本语法)
    2.  [1.2. 嵌入javascript表达式](#嵌入javascript表达式)
    3.  [1.3. 内联样式](#内联样式)
    4.  [1.4. 渲染HTML标签（strings）和React组件（classes）](#渲染HTML标签（strings）和React组件（classes）)
    5.  [1.5. tips](#tips)
2.  [2. 组件](#组件)
3.  [3. 生命周期](#生命周期)
    1.  [3.0.1. 组件的生命周期可分成三个状态：](#组件的生命周期可分成三个状态：)
    2.  [3.0.2. 生命周期的方法有：](#生命周期的方法有：)

<!--/noindex-->

© 2018 yanzhong

由 [Hexo](https://hexo.io) 强力驱动

|

主题 — [NexT.Pisces](https://github.com/iissnan/hexo-theme-next) v5.1.4

if \(Object.prototype.toString.call\(window.Promise\) \!== '\[object Function\]'\) \{ window.Promise = null; \}                            var NexT = window.NexT || \{\}; var CONFIG = \{ root: '/', scheme: 'Pisces', version: '5.1.4', sidebar: \{"position":"left","display":"post","offset":12,"b2t":false,"scrollpercent":false,"onmobile":false\}, fancybox: true, tabs: true, motion: \{"enable":true,"async":false,"transition":\{"post\_block":"fadeIn","post\_header":"slideDownIn","post\_body":"slideDownIn","coll\_header":"slideLeftIn","sidebar":"slideUpIn"\}\}, duoshuo: \{ userId: '0', author: '博主' \}, algolia: \{ applicationID: '', apiKey: '', indexName: '', hits: \{"per\_page":10\}, labels: \{"input\_placeholder":"Search for Posts","hits\_empty":"We didn't find any results for the search: \$\{query\}","hits\_stats":"\$\{hits\} results found in \$\{time\} ms"\} \} \};   react | 钟燕的技术博客

[钟燕的技术博客](/)

技术菜鸟的成长之路

* [  
  首页](/)
* [  
  标签](/tags/)
* [  
  分类](/categories/)
* [  
  归档](/archives/)

       

# react

发表于 2018-06-14 | 分类于 [react](/categories/react/)

## [](#JSX "JSX")JSX

react使用JSX代替javascript，JSX是有点像XML的javascript语法扩展。

### [](#基本语法 "基本语法")基本语法

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br></pre></td><td class="code"><pre><span class="line">ReactDOM.render(</span><br><span class="line">    &lt;div&gt;</span><br><span class="line">    &lt;h1&gt;react教程&lt;/h1&gt;</span><br><span class="line">    &lt;h2&gt;欢迎学习 React&lt;/h2&gt;</span><br><span class="line">    &lt;p&gt;这是一个很不错的 JavaScript 库!&lt;/p&gt;</span><br><span class="line">    &lt;/div&gt;,</span><br><span class="line">    document.getElementById('example')</span><br><span class="line">);</span><br></pre></td></tr></tbody></table>

多个html标签需要全包含在一个div中，以上代码表示在id=example的html标签位置渲染一个div，里面有两个标题和一个段落。

### [](#嵌入javascript表达式 "嵌入javascript表达式")嵌入javascript表达式

* JSX中可以嵌入javascript表达式，但需要写在花括号\{\}内。
* 注释也需要写在花括号中。
* 花括号中的数组会自动展开所有成员。
* 表达式不能使用`if else`语句，但可以用`conditional`三元运算表达式。  
  `<h1>{i == 1 ? 'True!' : 'False'}</h1>`

### [](#内联样式 "内联样式")内联样式

使用驼峰法设置内联样式

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br></pre></td><td class="code"><pre><span class="line">var myStyle = {</span><br><span class="line">    fontSize: 100,</span><br><span class="line">    color: '#FF0000'</span><br><span class="line">};</span><br><span class="line">ReactDOM.render(</span><br><span class="line">    &lt;h1 style = {myStyle}&gt;菜鸟教程&lt;/h1&gt;,</span><br><span class="line">    document.getElementById('example')</span><br><span class="line">);</span><br></pre></td></tr></tbody></table>

### [](#渲染HTML标签（strings）和React组件（classes） "渲染HTML标签（strings）和React组件（classes）")渲染HTML标签（strings）和React组件（classes）

React 可以渲染 HTML 标签 \(strings\) 或 React 组件 \(classes\)。

要渲染 HTML 标签，只需在 JSX 里使用小写字母的标签名。

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">var myDivElement = &lt;div className="foo" /&gt;;</span><br><span class="line">ReactDOM.render(myDivElement, document.getElementById('example'));</span><br></pre></td></tr></tbody></table>

要渲染 React 组件，只需创建一个大写字母开头的本地变量。

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br></pre></td><td class="code"><pre><span class="line">var MyComponent = React.createClass({/*...*/});</span><br><span class="line">var myElement = &lt;MyComponent someProperty={true} /&gt;;</span><br><span class="line">ReactDOM.render(myElement, document.getElementById('example'));</span><br></pre></td></tr></tbody></table>

### [](#tips "tips")tips

由于 JSX 就是 JavaScript，一些标识符像 class 和 for 不建议作为 XML 属性名。作为替代，React DOM 使用 className 和 htmlFor 来做对应的属性。

## [](#组件 "组件")组件

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

## [](#生命周期 "生命周期")生命周期

#### [](#组件的生命周期可分成三个状态： "组件的生命周期可分成三个状态：")组件的生命周期可分成三个状态：

* Mounting：已插入真实 DOM
* Updating：正在被重新渲染
* Unmounting：已移出真实 DOM

#### [](#生命周期的方法有： "生命周期的方法有：")生命周期的方法有：

* componentWillMount 在渲染前调用,在客户端也在服务端。
* componentDidMount : 在第一次渲染后调用，只在客户端。之后组件已经生成了对应的DOM结构，可以通过this.getDOMNode\(\)来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作\(防止异部操作阻塞UI\)。
* componentWillReceiveProps 在组件接收到一个新的 prop \(更新后\)时被调用。这个方法在初始化render时不会被调用。
* shouldComponentUpdate 返回一个布尔值。在组件接收到新的props或者state时被调用。在初始化时或者使用forceUpdate时不被调用。
* 可以在你确认不需要更新组件时使用。
* componentWillUpdate在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。
* componentDidUpdate 在组件完成更新后立即调用。在初始化时不会被调用。
* componentWillUnmount在组件从 DOM 中移除的时候立刻被调用。

[\# html](/tags/html/) [\# css](/tags/css/) [\# javascript](/tags/javascript/) [\# react](/tags/react/)

[gulp](/2018/06/12/gulp/ "gulp")

* 文章目录
* 站点概览

yanzhong

在学习前端的路上的一些笔记和学习心得。

[11 日志](/archives/)

[7 分类](/categories/index.html)

[9 标签](/tags/index.html)

<!--noindex-->

1.  [1. JSX](#JSX)
    1.  [1.1. 基本语法](#基本语法)
    2.  [1.2. 嵌入javascript表达式](#嵌入javascript表达式)
    3.  [1.3. 内联样式](#内联样式)
    4.  [1.4. 渲染HTML标签（strings）和React组件（classes）](#渲染HTML标签（strings）和React组件（classes）)
    5.  [1.5. tips](#tips)
2.  [2. 组件](#组件)
3.  [3. 生命周期](#生命周期)
    1.  [3.0.1. 组件的生命周期可分成三个状态：](#组件的生命周期可分成三个状态：)
    2.  [3.0.2. 生命周期的方法有：](#生命周期的方法有：)

<!--/noindex-->

© 2018 yanzhong

由 [Hexo](https://hexo.io) 强力驱动

|

主题 — [NexT.Pisces](https://github.com/iissnan/hexo-theme-next) v5.1.4

if \(Object.prototype.toString.call\(window.Promise\) \!== '\[object Function\]'\) \{ window.Promise = null; \}                            var NexT = window.NexT || \{\}; var CONFIG = \{ root: '/', scheme: 'Pisces', version: '5.1.4', sidebar: \{"position":"left","display":"post","offset":12,"b2t":false,"scrollpercent":false,"onmobile":false\}, fancybox: true, tabs: true, motion: \{"enable":true,"async":false,"transition":\{"post\_block":"fadeIn","post\_header":"slideDownIn","post\_body":"slideDownIn","coll\_header":"slideLeftIn","sidebar":"slideUpIn"\}\}, duoshuo: \{ userId: '0', author: '博主' \}, algolia: \{ applicationID: '', apiKey: '', indexName: '', hits: \{"per\_page":10\}, labels: \{"input\_placeholder":"Search for Posts","hits\_empty":"We didn't find any results for the search: \$\{query\}","hits\_stats":"\$\{hits\} results found in \$\{time\} ms"\} \} \};   react | 钟燕的技术博客

[钟燕的技术博客](/)

技术菜鸟的成长之路

* [  
  首页](/)
* [  
  标签](/tags/)
* [  
  分类](/categories/)
* [  
  归档](/archives/)

       

# react

发表于 2018-06-14 | 分类于 [react](/categories/react/)

## [](#JSX "JSX")JSX

react使用JSX代替javascript，JSX是有点像XML的javascript语法扩展。

### [](#基本语法 "基本语法")基本语法

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br></pre></td><td class="code"><pre><span class="line">ReactDOM.render(</span><br><span class="line">    &lt;div&gt;</span><br><span class="line">    &lt;h1&gt;react教程&lt;/h1&gt;</span><br><span class="line">    &lt;h2&gt;欢迎学习 React&lt;/h2&gt;</span><br><span class="line">    &lt;p&gt;这是一个很不错的 JavaScript 库!&lt;/p&gt;</span><br><span class="line">    &lt;/div&gt;,</span><br><span class="line">    document.getElementById('example')</span><br><span class="line">);</span><br></pre></td></tr></tbody></table>

多个html标签需要全包含在一个div中，以上代码表示在id=example的html标签位置渲染一个div，里面有两个标题和一个段落。

### [](#嵌入javascript表达式 "嵌入javascript表达式")嵌入javascript表达式

* JSX中可以嵌入javascript表达式，但需要写在花括号\{\}内。
* 注释也需要写在花括号中。
* 花括号中的数组会自动展开所有成员。
* 表达式不能使用`if else`语句，但可以用`conditional`三元运算表达式。  
  `<h1>{i == 1 ? 'True!' : 'False'}</h1>`

### [](#内联样式 "内联样式")内联样式

使用驼峰法设置内联样式

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br></pre></td><td class="code"><pre><span class="line">var myStyle = {</span><br><span class="line">    fontSize: 100,</span><br><span class="line">    color: '#FF0000'</span><br><span class="line">};</span><br><span class="line">ReactDOM.render(</span><br><span class="line">    &lt;h1 style = {myStyle}&gt;菜鸟教程&lt;/h1&gt;,</span><br><span class="line">    document.getElementById('example')</span><br><span class="line">);</span><br></pre></td></tr></tbody></table>

### [](#渲染HTML标签（strings）和React组件（classes） "渲染HTML标签（strings）和React组件（classes）")渲染HTML标签（strings）和React组件（classes）

React 可以渲染 HTML 标签 \(strings\) 或 React 组件 \(classes\)。

要渲染 HTML 标签，只需在 JSX 里使用小写字母的标签名。

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">var myDivElement = &lt;div className="foo" /&gt;;</span><br><span class="line">ReactDOM.render(myDivElement, document.getElementById('example'));</span><br></pre></td></tr></tbody></table>

要渲染 React 组件，只需创建一个大写字母开头的本地变量。

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br></pre></td><td class="code"><pre><span class="line">var MyComponent = React.createClass({/*...*/});</span><br><span class="line">var myElement = &lt;MyComponent someProperty={true} /&gt;;</span><br><span class="line">ReactDOM.render(myElement, document.getElementById('example'));</span><br></pre></td></tr></tbody></table>

### [](#tips "tips")tips

由于 JSX 就是 JavaScript，一些标识符像 class 和 for 不建议作为 XML 属性名。作为替代，React DOM 使用 className 和 htmlFor 来做对应的属性。

## [](#组件 "组件")组件

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

## [](#生命周期 "生命周期")生命周期

#### [](#组件的生命周期可分成三个状态： "组件的生命周期可分成三个状态：")组件的生命周期可分成三个状态：

* Mounting：已插入真实 DOM
* Updating：正在被重新渲染
* Unmounting：已移出真实 DOM

#### [](#生命周期的方法有： "生命周期的方法有：")生命周期的方法有：

* componentWillMount 在渲染前调用,在客户端也在服务端。
* componentDidMount : 在第一次渲染后调用，只在客户端。之后组件已经生成了对应的DOM结构，可以通过this.getDOMNode\(\)来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作\(防止异部操作阻塞UI\)。
* componentWillReceiveProps 在组件接收到一个新的 prop \(更新后\)时被调用。这个方法在初始化render时不会被调用。
* shouldComponentUpdate 返回一个布尔值。在组件接收到新的props或者state时被调用。在初始化时或者使用forceUpdate时不被调用。
* 可以在你确认不需要更新组件时使用。
* componentWillUpdate在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。
* componentDidUpdate 在组件完成更新后立即调用。在初始化时不会被调用。
* componentWillUnmount在组件从 DOM 中移除的时候立刻被调用。

[\# html](/tags/html/) [\# css](/tags/css/) [\# javascript](/tags/javascript/) [\# react](/tags/react/)

[gulp](/2018/06/12/gulp/ "gulp")

* 文章目录
* 站点概览

yanzhong

在学习前端的路上的一些笔记和学习心得。

[11 日志](/archives/)

[7 分类](/categories/index.html)

[9 标签](/tags/index.html)

<!--noindex-->

1.  [1. JSX](#JSX)
    1.  [1.1. 基本语法](#基本语法)
    2.  [1.2. 嵌入javascript表达式](#嵌入javascript表达式)
    3.  [1.3. 内联样式](#内联样式)
    4.  [1.4. 渲染HTML标签（strings）和React组件（classes）](#渲染HTML标签（strings）和React组件（classes）)
    5.  [1.5. tips](#tips)
2.  [2. 组件](#组件)
3.  [3. 生命周期](#生命周期)
    1.  [3.0.1. 组件的生命周期可分成三个状态：](#组件的生命周期可分成三个状态：)
    2.  [3.0.2. 生命周期的方法有：](#生命周期的方法有：)

<!--/noindex-->

© 2018 yanzhong

由 [Hexo](https://hexo.io) 强力驱动

|

主题 — [NexT.Pisces](https://github.com/iissnan/hexo-theme-next) v5.1.4

if \(Object.prototype.toString.call\(window.Promise\) \!== '\[object Function\]'\) \{ window.Promise = null; \}                            var NexT = window.NexT || \{\}; var CONFIG = \{ root: '/', scheme: 'Pisces', version: '5.1.4', sidebar: \{"position":"left","display":"post","offset":12,"b2t":false,"scrollpercent":false,"onmobile":false\}, fancybox: true, tabs: true, motion: \{"enable":true,"async":false,"transition":\{"post\_block":"fadeIn","post\_header":"slideDownIn","post\_body":"slideDownIn","coll\_header":"slideLeftIn","sidebar":"slideUpIn"\}\}, duoshuo: \{ userId: '0', author: '博主' \}, algolia: \{ applicationID: '', apiKey: '', indexName: '', hits: \{"per\_page":10\}, labels: \{"input\_placeholder":"Search for Posts","hits\_empty":"We didn't find any results for the search: \$\{query\}","hits\_stats":"\$\{hits\} results found in \$\{time\} ms"\} \} \};   react | 钟燕的技术博客

[钟燕的技术博客](/)

技术菜鸟的成长之路

* [  
  首页](/)
* [  
  标签](/tags/)
* [  
  分类](/categories/)
* [  
  归档](/archives/)

       

# react

发表于 2018-06-14 | 分类于 [react](/categories/react/)

## [](#JSX "JSX")JSX

react使用JSX代替javascript，JSX是有点像XML的javascript语法扩展。

### [](#基本语法 "基本语法")基本语法

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br></pre></td><td class="code"><pre><span class="line">ReactDOM.render(</span><br><span class="line">    &lt;div&gt;</span><br><span class="line">    &lt;h1&gt;react教程&lt;/h1&gt;</span><br><span class="line">    &lt;h2&gt;欢迎学习 React&lt;/h2&gt;</span><br><span class="line">    &lt;p&gt;这是一个很不错的 JavaScript 库!&lt;/p&gt;</span><br><span class="line">    &lt;/div&gt;,</span><br><span class="line">    document.getElementById('example')</span><br><span class="line">);</span><br></pre></td></tr></tbody></table>

多个html标签需要全包含在一个div中，以上代码表示在id=example的html标签位置渲染一个div，里面有两个标题和一个段落。

### [](#嵌入javascript表达式 "嵌入javascript表达式")嵌入javascript表达式

* JSX中可以嵌入javascript表达式，但需要写在花括号\{\}内。
* 注释也需要写在花括号中。
* 花括号中的数组会自动展开所有成员。
* 表达式不能使用`if else`语句，但可以用`conditional`三元运算表达式。  
  `<h1>{i == 1 ? 'True!' : 'False'}</h1>`

### [](#内联样式 "内联样式")内联样式

使用驼峰法设置内联样式

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br></pre></td><td class="code"><pre><span class="line">var myStyle = {</span><br><span class="line">    fontSize: 100,</span><br><span class="line">    color: '#FF0000'</span><br><span class="line">};</span><br><span class="line">ReactDOM.render(</span><br><span class="line">    &lt;h1 style = {myStyle}&gt;菜鸟教程&lt;/h1&gt;,</span><br><span class="line">    document.getElementById('example')</span><br><span class="line">);</span><br></pre></td></tr></tbody></table>

### [](#渲染HTML标签（strings）和React组件（classes） "渲染HTML标签（strings）和React组件（classes）")渲染HTML标签（strings）和React组件（classes）

React 可以渲染 HTML 标签 \(strings\) 或 React 组件 \(classes\)。

要渲染 HTML 标签，只需在 JSX 里使用小写字母的标签名。

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">var myDivElement = &lt;div className="foo" /&gt;;</span><br><span class="line">ReactDOM.render(myDivElement, document.getElementById('example'));</span><br></pre></td></tr></tbody></table>

要渲染 React 组件，只需创建一个大写字母开头的本地变量。

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br></pre></td><td class="code"><pre><span class="line">var MyComponent = React.createClass({/*...*/});</span><br><span class="line">var myElement = &lt;MyComponent someProperty={true} /&gt;;</span><br><span class="line">ReactDOM.render(myElement, document.getElementById('example'));</span><br></pre></td></tr></tbody></table>

### [](#tips "tips")tips

由于 JSX 就是 JavaScript，一些标识符像 class 和 for 不建议作为 XML 属性名。作为替代，React DOM 使用 className 和 htmlFor 来做对应的属性。

## [](#组件 "组件")组件

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

## [](#生命周期 "生命周期")生命周期

#### [](#组件的生命周期可分成三个状态： "组件的生命周期可分成三个状态：")组件的生命周期可分成三个状态：

* Mounting：已插入真实 DOM
* Updating：正在被重新渲染
* Unmounting：已移出真实 DOM

#### [](#生命周期的方法有： "生命周期的方法有：")生命周期的方法有：

* componentWillMount 在渲染前调用,在客户端也在服务端。
* componentDidMount : 在第一次渲染后调用，只在客户端。之后组件已经生成了对应的DOM结构，可以通过this.getDOMNode\(\)来进行访问。 如果你想和其他JavaScript框架一起使用，可以在这个方法中调用setTimeout, setInterval或者发送AJAX请求等操作\(防止异部操作阻塞UI\)。
* componentWillReceiveProps 在组件接收到一个新的 prop \(更新后\)时被调用。这个方法在初始化render时不会被调用。
* shouldComponentUpdate 返回一个布尔值。在组件接收到新的props或者state时被调用。在初始化时或者使用forceUpdate时不被调用。
* 可以在你确认不需要更新组件时使用。
* componentWillUpdate在组件接收到新的props或者state但还没有render时被调用。在初始化时不会被调用。
* componentDidUpdate 在组件完成更新后立即调用。在初始化时不会被调用。
* componentWillUnmount在组件从 DOM 中移除的时候立刻被调用。

[\# html](/tags/html/) [\# css](/tags/css/) [\# javascript](/tags/javascript/) [\# react](/tags/react/)

[gulp](/2018/06/12/gulp/ "gulp")

* 文章目录
* 站点概览

yanzhong

在学习前端的路上的一些笔记和学习心得。

[11 日志](/archives/)

[7 分类](/categories/index.html)

[9 标签](/tags/index.html)

<!--noindex-->

1.  [1. JSX](#JSX)
    1.  [1.1. 基本语法](#基本语法)
    2.  [1.2. 嵌入javascript表达式](#嵌入javascript表达式)
    3.  [1.3. 内联样式](#内联样式)
    4.  [1.4. 渲染HTML标签（strings）和React组件（classes）](#渲染HTML标签（strings）和React组件（classes）)
    5.  [1.5. tips](#tips)
2.  [2. 组件](#组件)
3.  [3. 生命周期](#生命周期)
    1.  [3.0.1. 组件的生命周期可分成三个状态：](#组件的生命周期可分成三个状态：)
    2.  [3.0.2. 生命周期的方法有：](#生命周期的方法有：)

<!--/noindex-->
