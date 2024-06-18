---
title: 博客日常撰写和备份
date: 2021-04-20 16:22:52
categories: 
- blog
tags: 
- blog
- hexo
- github
- sync
- website maintenance
- markdown

description: 这篇博客是记录博客日常撰写和备份。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=20707476&auto=1&height=32"></iframe><music URL></div>

在根据博客[hexo建站，github.io发布，多终端同步](https://yanzhongsino.github.io/2018/06/05/blog_hexo.github/)配置了hexo网站（使用next主题）的基础上，记录了博客日常撰写、备份。

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
	- bioinfo
	- experiment
	- theory
	- knowledge

- biosoft

- omics
	- genome
	- transcriptome
	- plastome
	- mitochondrion

- plot
	- R

- computer
	- system
		- windows
		- linux
	- program language
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

- linux
	- basics
	- shell
	- text processing
	- operation and maintenance

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


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>