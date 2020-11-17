

# GitHub Pages Hexo搭建独立静态博客

发表于 2018-06-05 | 分类于 [GitHube](/categories/GitHube/)

参考[csdn教程](https://blog.csdn.net/sunshine940326/article/details/52552283)、[简书教程](https://www.jianshu.com/p/6fdb19aa4558)、[阮一峰日志](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)、[jekyllrb](https://jekyllrb.com/)和[hexo官方文档](https://hexo.io/zh-cn/docs/)等资料。

githube pages是githube的一个功能，专门用于管理用户自行编写的静态网页，提供静态网页存储空间，许多人把它用于搭建独立博客。

githube pages需要相应的博客引擎驱动，主流的是[jekyllrb](https://jekyllrb.com/)和[hexo](https://hexo.io)。

hexo是不仅是博客引擎驱动，还是一个快速、简洁高效的博客框架，可生成静态网页。

# [](#通过githube-pages和hexo搭建独立博客 "通过githube pages和hexo搭建独立博客")通过githube pages和hexo搭建独立博客

### [](#搭建前的准备 "搭建前的准备")搭建前的准备

1.  github账号
2.  git安装

## [](#github-pages部分 "github pages部分")github pages部分

在github账户上创建一个名为`youraccount.github.io`的新仓库，这个特殊的名称直接关联了gitpages，gitpages服务器会自动把上传到这个仓库的文档进行部署发布，并可以通过分配的域名`https://youraccount.github.io/`实现博客的访问。

## [](#hexo部分 "hexo部分")hexo部分

_安装hexo前需要安装node.js和git。_

### [](#hexo安装 "hexo安装")hexo安装

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br></pre></td><td class="code"><pre><span class="line">npm install hexo-cli -g #安装hexo</span><br><span class="line">hexo init &lt;folder&gt; #在选定位置新建一个网站，并初始化folder文件夹，会获得博客网站的基本目录和一个示意的hello world博客网站。</span><br><span class="line">cd &lt;folder&gt; #进入文件夹</span><br><span class="line">npm install #安装依赖</span><br><span class="line">npm install hexo-server --save  #安装hexo-server以能够使用服务器，--save是把hexo-server保存到package.json的依赖列表中。</span><br><span class="line">hexo server #启动服务器，默认网址为&lt;http://localhost:4000/&gt;,这时会打开一个网页，即为你的博客网站。</span><br></pre></td></tr></tbody></table>

### [](#hexo配置 "hexo配置")hexo配置

在创建的博客网站文件夹下的`_config.yml`文件中修改大部分配置。参考[hexo配置](https://hexo.io/zh-cn/docs/configuration.html)

#### [](#部署部分的配置 "部署部分的配置")部署部分的配置

1.  安装hexo-deployer-git  
    `$ npm install hexo-deployer-git --save` #安装hexo-deployer-git，从而实现通过`hexo g`部署网页到gitpages。
2.  修改配置文件的deploy部署配置  
    `_config.yml`文件的`deploy`部分，修改成

    <table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br></pre></td><td class="code"><pre><span class="line">deploy:</span><br><span class="line">  type: git</span><br><span class="line">  repo: git@github.com:yanzhongsino/yanzhongsino.github.io.git</span><br><span class="line">  branch: master</span><br></pre></td></tr></tbody></table>

    repo部分填写的是在githube上创建的`youraccount.github.io`仓库的ssh地址。

### [](#hexo使用 "hexo使用")hexo使用

1.  在hexo建的博客网站文件夹下的source>\_posts目录下新建和编辑markdown文档，这就是写文章的地方。

2.  `hexo clean` #清除缓存文件（db.json\)和已生成的静态文件（public），如果需要重新生成静态网页（`hexo g`）和部署网页（`hexo d`），之前需要执行这个清除命令。

3.  markdown写完，可以用hexo生成静态网页，通过执行`hexo generate`\(可以简写成`hexo g`\)命令实现。静态网页会根据source>\_posts目录下所有markdown文件编译成静态网页（html），生成的静态文件会在博客网站文件夹下生成的public文件夹中。

hexo g –watch可以监视文件变动，并重新生成静态文件，只有变动的文件才会写入。

4.  生成静态文件后，可以使用`hexo serve`命令启动本地服务器来查看博客网站。

5.  生成静态文件后，也可以使用`hexo deploy`\(可以简写成`hexo d`\)命令来部署网站到gitpages。部署之后，可以在githube的page项目中查看到变化。我的是<https://github.com/yanzhongsino/yanzhongsino.github.io>。很快也可以在<https://yanzhongsino.github.io/>线上看到生成的博客。

[\# html](/tags/html/) [\# git](/tags/git/) [\# markdown](/tags/markdown/)

[jQuery基础知识](/2018/06/05/jquery基础知识/ "jQuery基础知识")

[bootstrap](/2018/06/11/bootstrap/ "bootstrap")

* 文章目录
* 站点概览

yanzhong

在学习前端的路上的一些笔记和学习心得。

[11 日志](/archives/)

[7 分类](/categories/index.html)

[9 标签](/tags/index.html)

<!--noindex-->

- [GitHub Pages Hexo搭建独立静态博客](#github-pages-hexo搭建独立静态博客)
- [通过githube pages和hexo搭建独立博客](#通过githube-pages和hexo搭建独立博客)
    - [搭建前的准备](#搭建前的准备)
  - [github pages部分](#github-pages部分)
  - [hexo部分](#hexo部分)
    - [hexo安装](#hexo安装)
    - [hexo配置](#hexo配置)
      - [部署部分的配置](#部署部分的配置)
    - [hexo使用](#hexo使用)

<!--/noindex-->
