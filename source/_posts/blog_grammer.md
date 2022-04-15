---
title: 博客撰写的语法和技巧
date: 2022-04-15
categories: 
- blog
tags: 
- blog
- hexo
- github
- markdown
- next
- image
- music
- quote

description: 这篇博客是记录日常博客撰写过程中的语法和技巧，markdown的使用，hexo的使用，next主题的使用等。目前记录了在博客中插入图片，音乐和引用的方式。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=5284273&auto=1&height=32"></iframe><music URL></div>

# 1. 撰写博客
## 1.1. 插入图片
若是本地图片，就放在特定位置，然后在blog中引用；若是网络图片，就直接复制图片的url地址引用；

### 1.1.1. 推荐用法
若是自己的图片，可以统一保存在sources/images/目录下，在github上同步后，复制github上图片地址，就可以像引用网络图片一样引用图片url地址。

```
<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/R_plot_venn_2.png?raw=true" width=50% title="venn_2" align="center" />

**<p align="center">Figure 1. Venn diagram with 2 groups**
from [github: yanzhongsino](https://github.com/yanzhongsino/yanzhongsino.github.io)</p>
```

`<img />`用于图片显示，src填写图片网址，width指图片占屏幕的比例（同时还有类似的height参数，但建议不设置height以保持图片长宽比例），title是图片名称(如果网址失效会显示title值的文字)，align设置对齐方式。
`<p align="center">Figure 1. text</p>`用于图的标题居中显示，from指示图源(引用其他来源的图时)，**用于加粗显示。

效果如下：
<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/R_plot_venn_2.png?raw=true" width=50% title="venn_2" align="center" />

**<p align="center">Figure 1. Venn diagram with 2 groups**
from [github: yanzhongsino](https://github.com/yanzhongsino/yanzhongsino.github.io)</p>

### 1.1.2. markdown语法
1. 统一放在images目录下
当Hexo项目中只用到少量图片时，可以将图片统一放在source/images文件夹中，在blog中通过markdown语法访问它们，即`![图片注释](/images/image.jpg "图片标题")`。

图片既可以在首页内容中访问到，也可以在文章正文中访问到。

2. 放在文章各自目录下
图片除了可以放在统一的images文件夹中，还可以放在文章自己的目录中。文章的目录可以通过配置_config.yml来生成。
- 将_config.yml文件中的配置项post_asset_folder设为true`post_asset_folder: true`;
- 执行命令`hexo new postname`，在source/_posts中会生成文章postname.md和同名文件夹postname;
- 将图片资源放在post_name中，文章就可以使用相对路径引用图片资源了。
- 在文章中添加markdown语法代码`![图片注释](image.jpg "图片标题")`即可。

图片只能在文章中显示，但无法在首页中正常显示。

### 1.1.3. HTML语法【推荐】
用markdown语法无法指定图片的尺寸和对齐方式，建议用HTML语法插入图片，以实现更好的控制。
`<img src="/images/image.png" width=80% height=80% title="picture" alt="picture" align=center/>`

`<img src="url" title="title" width="80%" height="80%" />`

### 1.1.4. 标签插件语法引用
如果希望图片在文章和首页中同时显示，可以使用标签插件语法，本地和网络图片都适用。
- 本地图片资源，不限制图片尺寸，使用 `{% asset_img image.jpg This is an image %}`；
- 网络图片资源，限制图片显示尺寸，`{% img http://www.something.gif 200 400 vi-vim-cheat-sheet %}` 

### 1.1.5. CDN引用
1. 除了在本地存储图片，还可以将图片上传到一些免费的CDN服务中。比如Cloudinary提供的图片CDN服务，在Cloudinary中上传图片后，会生成对应的url地址，将地址直接拿来引用即可。
2. 【实测这种方法不生效】把代码`<div align="middle" style="width:200px; margin:auto">这里粘贴生成的url地址</div>`粘贴到文章中即可；align为了美观设置成居中。

### 1.1.6. fancybox
1. 启用fancybox
启用fancybox：点击查看图片大图。

Hexo的NexT主题中提供了fancybox的方便接口。

Usage：https://github.com/theme-next/theme-next-fancybox3
markdown用法：`{% img http://www.viemu.com/vi-vim-cheat-sheet.gif 600 600 "点击查看大图:vi/vim-cheat-sheet" %}`

2. Hexo部分图片禁用fancybox

hexo在使用fancybox插件时，图片的效果还是很可观的，但是我们往往是不需要所有的图片都用fancybox；
例如：hexo next主题下，添加某些图片的时候，有些事不需要可点击的
修改theme\next\source\js\src\utils.js 红色字体部分；

```
diff --git a/source/js/src/utils.js b/source/js/src/utils.js
index 0f3704e..8516665 100644
--- a/source/js/src/utils.js
+++ b/source/js/src/utils.js
@@ -11,6 +11,7 @@ NexT.utils = NexT.$u = {
       .not('.group-picture img, .post-gallery img')
       .each(function() {
         var $image = $(this);
+        if ($(this).hasClass('nofancybox')) return;
         var imageTitle = $image.attr('title');
         var $imageWrapLink = $image.parent('a');
```

在img标签使用的时候加上class=”nofancybox”即可。`<img src="http://www.viemu.com/vi-vim-cheat-sheet.gif" class="nofancybox" />`

## 1.2. 插入背景音乐
1. 打开网易云网页版，找到想听的歌曲，然后点击**生成外链播放器**，然后复制网易云音乐的插件页面的HTML代码；
2. 把代码`<div align="middle">这里粘贴刚刚复制的代码</div>`粘贴到文章中即可；align为了美观设置成居中；

## 1.3. 插入引用
插入引用有多种方式
### 1.3.1. markdown方式【简单】
如果只是简单的标记引用，就选择行首添加`>`的方式，简单快捷。
1. 单行引用
- 在markdown中的行首添加`>`符号，就会把这段内容(和接下来无空格/空行分隔的段落)标记为引用内容(效果等于加上<blockquuote>标签)。
- 例子：
```
test 引用
>cite quote
test > cite content
```
- 效果：

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/blog_grammer_quote.png?raw=true" width=50% title=">引用效果" align="center" />

**<p align="center">Figure 2. >引用效果**

2. 多行引用
- 在每一行的行首都添加`>`符号。
3. 嵌套引用
- 第一行行首添加`>`符号，第二行行首添加`>>`符号，这样就会有第二行嵌套在第一行的效果。
- 例子：
```
test 嵌套引用
>cite quote 1
>>cite content 2
>>>cite content 3
```

- 效果：

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/blog_grammer_multiquote.png?raw=true" width=50% title=">嵌套引用效果" align="center" />

**<p align="center">Figure 3. >嵌套引用效果**

### 1.3.2. HTML方式【居中】
如果需要居中显示引用内容，可以选择HTML的blockquote来实现。
- 直接在markdown中写HTML格式来调用引用块blockquote
- 使用例子：`<blockquote class="blockquote-center">blockquote cite test</br>test2</br>test1 引用test test2 引用test test3 引用test test4 引用test test5 引用test test6 引用test test7 引用test test8 引用test test9 引用test test10 引用test test11 引用test test12 引用test test13 引用test test14 引用test test15 引用test test16 引用test test17 引用test test18 引用test test19 引用test test20</blockquote>`。其中`class="blockquote-center"`是为了使引用内容居中显示。
- 效果：

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/blog_grammer_blockquote.png?raw=true" width=80% title="blockquote HTML引用效果" align="center" />

**<p align="center">Figure 4. blockquote HTML引用效果**

### 1.3.3. 标签方式【全面】
hexo内置许多标签使得引用的功能更全面。
1. blockquote标签（hexo标签）
blockquote标签可插入包含作者、来源和标题的引用。
- 格式：
```
{% blockquote [author[, source]] [link] [source_link_title] %}
content
{% endblockquote %}
```

- 例子1：引用书中内容
```
{% blockquote David Levithan, Wide Awake %}
Do not just seek happiness for yourself. Seek happiness for all. Through kindness. Through mercy.
{% endblockquote %}
```

效果如下：

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/blog_grammer_blockquote1.png?raw=true" width=80% title="blockquote标签引用效果" align="center" />

**<p align="center">Figure 5. blockquote标签引用效果1**

- 例子2：引用Twitter
```
{% blockquote @DevDocs https://twitter.com/devdocs/status/356095192085962752 %}
NEW: DevDocs now comes with syntax highlighting. http://devdocs.io
{% endblockquote %}
```

效果如下：

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/blog_grammer_blockquote2.png?raw=true" width=80% title="blockquote标签引用效果" align="center" />

**<p align="center">Figure 6. blockquote标签引用效果2**

- 例子3：引用网页文章
```
{% blockquote Seth Godin http://sethgodin.typepad.com/seths_blog/2009/07/welcome-to-island-marketing.html Welcome to Island Marketing %}
Every interaction is both precious and an opportunity to delight.
{% endblockquote %}
```

效果如下：

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/blog_grammer_blockquote3.png?raw=true" width=80% title="blockquote标签引用效果" align="center" />

**<p align="center">Figure 7. blockquote标签引用效果3**

2. centerquote标签（next（版本>=0.4.5）主题内置标签）
这个标签生成一个带上下分割线的引用，同时引用文本自动居中。
- 使用格式：`{%centerquote}引用内容{%endcenterquote%}`，其中centerquote可以简写成`cq`。
- 例子：
```markdown
{%cq%}
test cq cite format
test1
test1 引用test test2 引用test test3 引用test test4 引用test test5 引用test test6 引用test test7 引用test test8 引用test test9 引用test test10 引用test test11 引用test test12 引用test test13 引用test test14 引用test test15 引用test test16 引用test test17 引用test test18 引用test test19 引用test test20

引用
{%endcq%}
```
- 效果如下：

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/blog_grammer_centerquote.png?raw=true" width=80% title="centerquote标签引用效果" align="center" />

**<p align="center">Figure 8. centerquote标签引用效果**

# 2. references
1. [hexo 标签插件:blockquote](https://hexo.io/docs/tag-plugins.html)