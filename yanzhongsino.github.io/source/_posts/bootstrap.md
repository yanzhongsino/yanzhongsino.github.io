   

# bootstrap

发表于 2018-06-11 | 分类于 [bootstrap](/categories/bootstrap/)

## [](#bootstrap简介 "bootstrap简介")bootstrap简介

bootstrap是一个用于快速开发web应用和网站的前端框架。  
它能用于开发响应式布局、移动设备优先的web项目。

## [](#bootstrap的安装 "bootstrap的安装")bootstrap的安装

需要在html的link标签引入bootstrap的css插件bootstrap.css，并在script标签中引用bootstrap的js插件bootstrap.js\(bootstrap的js插件使用需要先引入jQuery\)，有两种方式：

#### [](#通过CDN（内容分发网络）引用bootstrap，多个CDN可以使用。比如bootCDN。 "通过CDN（内容分发网络）引用bootstrap，多个CDN可以使用。比如bootCDN。")通过CDN（内容分发网络）引用bootstrap，多个CDN可以使用。比如[bootCDN](http://www.bootcdn.cn/bootstrap/)。

在html中按如下语法引用。

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br><span class="line">7</span><br><span class="line">8</span><br><span class="line">9</span><br><span class="line">10</span><br><span class="line">11</span><br></pre></td><td class="code"><pre><span class="line">&lt;!-- 最新版本的 Bootstrap 核心 CSS 文件 --&gt;</span><br><span class="line">&lt;link rel="stylesheet" href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap.min.css" integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u" crossorigin="anonymous"&gt;</span><br><span class="line">	</span><br><span class="line">&lt;!-- 可选的 Bootstrap 主题文件（一般不用引入） --&gt;</span><br><span class="line">&lt;link rel="stylesheet" href="https://cdn.bootcss.com/bootstrap/3.3.7/css/bootstrap-theme.min.css" integrity="sha384-rHyoN1iRsVXV4nD0JutlnGaslCJuC7uwjduW9SVrLvRYooPp2bWYgmgJQIXwl/Sp" crossorigin="anonymous"&gt;</span><br><span class="line">	</span><br><span class="line">&lt;!--引入jquery--&gt;</span><br><span class="line">&lt;script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"&gt;&lt;/script&gt;</span><br><span class="line">	</span><br><span class="line">&lt;!-- 最新的 Bootstrap 核心 JavaScript 文件 --&gt;</span><br><span class="line">&lt;script src="https://cdn.bootcss.com/bootstrap/3.3.7/js/bootstrap.min.js" integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa" crossorigin="anonymous"&gt;&lt;/script&gt;</span><br></pre></td></tr></tbody></table>

#### [](#在bootstrap中文网下载bootstrap，是一个文件夹，下有css、js、fonts三个目录。 "在bootstrap中文网下载bootstrap，是一个文件夹，下有css、js、fonts三个目录。")在[bootstrap中文网](http://v3.bootcss.com/)下载bootstrap，是一个文件夹，下有css、js、fonts三个目录。

在link标签中引入文件夹的css目录下的bootstrap.min.css文件

`<link href="css/bootstrap.min.css" rel="stylesheet">`

在script标签中引用文件夹的js目录下的bootstrap.min.js文件\(bootstrap的js插件使用需要先引入jQuery\)

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br></pre></td><td class="code"><pre><span class="line">&lt;!--引入jquery--&gt;</span><br><span class="line">&lt;script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js"&gt;&lt;/script&gt;</span><br><span class="line">&lt;!--引入js--&gt;</span><br><span class="line">&lt;script src="js/bootstrap.min.js"&gt;&lt;/script&gt;</span><br></pre></td></tr></tbody></table>

## [](#bootstrap的css "bootstrap的css")bootstrap的css

1.  使用`<img src="" class="img-responsive">`使图像对响应式布局更友好地支持。

    <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br></pre></td><td class="code"><pre><span class="line">.img-responsive{</span><br><span class="line">	display:inline-block;</span><br><span class="line">	height:auto;</span><br><span class="line">	max-width:100%;</span><br><span class="line">}</span><br></pre></td></tr></tbody></table>

2.  Bootstrap 排版、链接样式设置了基本的全局样式。分别是：

    * 为 body 元素设置 `background-color: #fff`;
    * 使用 `@font-family-base`、`@font-size-base` 和 `@line-height-base a`变量作为排版的基本参数
    * 为所有链接设置了基本颜色 `@link-color` ，并且当链接处于 :hover 状态时才添加下划线
    * 这些样式都能在 scaffolding.less 文件中找到对应的源码。

3.  使用**normalize.css**来建立跨浏览器的一致性。

4.  .container 类用于固定宽度并支持响应式布局的容器。.container-fluid 类用于 100\% 宽度，占据全部视口（viewport）的容器。

## [](#网格系统 "网格系统")网格系统

## [](#小麦官网项目xiaomai-v1 "小麦官网项目xiaomai-v1")小麦官网项目xiaomai-v1

使用bootstrap使移动端和pc端用一套代码  
使用gulp打包，生成dist文件夹后，可以在server.js中启动

[\# html](/tags/html/) [\# css](/tags/css/) [\# javascript](/tags/javascript/) [\# bootstrap](/tags/bootstrap/)

[GitHube Pages+Hexo搭建独立静态博客](</2018/06/05/GitHube Pages+Hexo搭建独立静态博客 /> "GitHube Pages+Hexo搭建独立静态博客")

[ES6](/2018/06/12/ES6/ "ES6")

* 文章目录
* 站点概览

yanzhong

在学习前端的路上的一些笔记和学习心得。

[11 日志](/archives/)

[7 分类](/categories/index.html)

[9 标签](/tags/index.html)

<!--noindex-->

1.  [1. bootstrap简介](#bootstrap简介)
2.  [2. bootstrap的安装](#bootstrap的安装)
    1.  [2.0.1. 通过CDN（内容分发网络）引用bootstrap，多个CDN可以使用。比如bootCDN。](#通过CDN（内容分发网络）引用bootstrap，多个CDN可以使用。比如bootCDN。)
    2.  [2.0.2. 在bootstrap中文网下载bootstrap，是一个文件夹，下有css、js、fonts三个目录。](#在bootstrap中文网下载bootstrap，是一个文件夹，下有css、js、fonts三个目录。)

* [3. bootstrap的css](#bootstrap的css)
* [4. 网格系统](#网格系统)
* [5. 小麦官网项目xiaomai-v1](#小麦官网项目xiaomai-v1)

<!--/noindex-->
