---
title: blog
date: 2021-04-20 16:22:52
categories: 
- blog
tags: 
- hexo
- github
- sync
- website maintenance
- markdown
- tutorial
description: 这篇博客是记录日常博客撰写过程中的一些技巧，markdown的使用，github.io搭建网站的维护，hexo的使用等。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=20707476&auto=1&height=32"></iframe><music URL></div>

# 1. blog的categories和tags
categories和tags的记录

```
---
title: blog
date: 2021-04-20 16:50:00
categories: 

- bio
	- concept
	- taxon
	- biosoftware
	- experiment
	- theory
	- knowledge

- omics
	- genome
	- transcriptome
	- plastome
	- mitochondrion



- computer
	- system
		- Windows
		- Linux
	- programming language
		- python
		- R
		- perl
		- java
		- C
	- IDE
		- vim
		- VScode
		- git
	- script
	- web

- blog


tags: 
- genome assemble
- genome annotation
- phylogeny
- divergence time
- WGD
- HGT
- molecular experiment
- homolog
- ortholog
- paralog
- xenolog
- analog
- orthology
- orthogroup
- gene family
---  
```

# 2. 日常blog撰写和备份操作
在做好blog搭建后，blog撰写和日常管理可参考这部分内容。

## 2.1. blog同步
  
养成习惯，每次开始撰写blog前都通过git bash进入工作区，进行`git pull`命令把github端的hexo分支的更新（更新可能是其他终端上提交的）同步到本地，实现多终端的内容完全同步。
但如果本地有未提交的更新，则千万不要用`git pull`，否则会覆盖本地更新；直接进入下一步；直到使用`git add .`，`git commit -m "submit"`，`git push origin hexo`提交备份本地更新到github端的hexo分支后才可以使用`git pull`(一般是在其他终端，把github的hexo分支更新拉到其他终端设备使用)。

## 2.2. blog撰写
    
在本地source/_posts下添加和修改md文档实现blog的日常撰写和修改。

使用命令`hexo new "newpostname"`可以在hexo/source/_posts下新建一个newpostname.md的文件，这个文件以scaffolds/post.md为模板，修改scaffolds/post.md文件可以修改hexo new命令生成的新blog文件样式。

## 2.3. blog备份
   
只要blog有更改或者新增，或者配置文件有修改，即工作区（即本地的hexo目录或github.io目录）有文件修改，则建议对文件进行备份到GitHub端的hexo分支。
用三条命令`git add .`，`git commit -m "submit"`，`git push origin hexo`备份工作区，包括md博客源文件和hexo部署到github端的hexo分支。三条命令执行前建议通过`hexo clean`清除缓存和public目录，以免备份不需要的文件。

## 2.4. blog发布
    
可根据自身需求决定是否发布blog到github.io网站，一般写的blog完整程度比较高时可以发布。使用`hexo clean & hexo g -d`命令，根据source/_posts下的博客源文件生成public目录（网站html并同步到github端的master分支，即发布blog到github.io网站。


总结一下，在配置好写作环境后的任意一台终端的日常工作流应该是：
1. `git pull`同步远程github库的hexo更新到本地
2. `hexo new "newblog"`在source/_posts/下添加md格式的blog，或者修改已有的blog
3. `git add .`,`git commit -m "commit notes"`,`git push`把修改备份到github端
4. 下次写作重复以上三个步骤
5. 直至blog完善成熟后，用命令`hexo clean & hexo g -d`生成网站并部署到github.io


# 3. 攥写blog
## 3.1. 插入图片
若是本地图片，就放在特定位置，然后在blog中引用；若是网络图片，就直接复制图片的url地址引用；
### 3.1.1. 绝对路径本地引用
当Hexo项目中只用到少量图片时，可以将图片统一放在source/images文件夹中，在blog中通过markdown语法访问它们，即`![](/images/image.jpg)`。

图片既可以在首页内容中访问到，也可以在文章正文中访问到。

### 3.1.2. 相对路径本地引用
图片除了可以放在统一的images文件夹中，还可以放在文章自己的目录中。文章的目录可以通过配置_config.yml来生成。
- 将_config.yml文件中的配置项post_asset_folder设为true`post_asset_folder: true`;
- 执行命令`hexo new postname`，在source/_posts中会生成文章postname.md和同名文件夹postname;
- 将图片资源放在post_name中，文章就可以使用相对路径引用图片资源了。
- 在文章中添加代码`![](image.jpg)`，markdown的引用方式。

图片只能在文章中显示，但无法在首页中正常显示。

### 3.1.3. CDN引用
1. 除了在本地存储图片，还可以将图片上传到一些免费的CDN服务中。比如Cloudinary提供的图片CDN服务，在Cloudinary中上传图片后，会生成对应的url地址，将地址直接拿来引用即可。
2. 把代码`<div align="middle">这里粘贴生成的url地址</div>`粘贴到文章中即可；align为了美观设置成居中；

### 3.1.4. 标签插件语法引用
如果希望图片在文章和首页中同时显示，可以使用标签插件语法，本地和网络图片都适用。
- 本地图片资源，不限制图片尺寸，使用 `{% asset_img image.jpg This is an image %}`；
- 网络图片资源，限制图片显示尺寸，`{% img http://www.something.gif 200 400 vi-vim-cheat-sheet %}` 

### 3.1.5. HTML语法引用
`<img src="SpellCheck.png" width="50%" height="50%" title="拼写检查工具Grammarly." alt="拼写检查工具Grammarly."/>`
启用fancybox：点击查看图片大图
我使用的是Hexo的NexT主题，NexT主题中提供了fancybox的方便接口。
Usage：https://github.com/theme-next/theme-next-fancybox3
markdown用法：`{% img http://www.viemu.com/vi-vim-cheat-sheet.gif 600 600 "点击查看大图:vi/vim-cheat-sheet" %}`

## 3.2. 插入背景音乐
1. 打开网易云网页版，找到想听的歌曲，然后点击**生成外链播放器**，然后复制网易云音乐的插件页面的HTML代码；
2. 把代码`<div align="middle">这里粘贴刚刚复制的代码</div>`粘贴到文章中即可；align为了美观设置成居中；
