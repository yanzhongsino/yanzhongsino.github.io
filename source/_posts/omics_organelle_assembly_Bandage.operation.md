---
title: 细胞器基因组组装过程中常用软件Bandage的操作
date: 2024-12-10
categories:
- omics
- organelle
- assembly
tags:
- organelle
- mitogenome
- Bandage

description: 记录细胞器基因组组装过程中常用软件Bandage的操作。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=261602&auto=1&height=32"></iframe></div>

# 1. Bandage介绍
Bandage全称是a Bioinformatics Application for Navigating De novo Assembly Graphs Easily，是一个用于从头组装基因组的交互式可视化的软件。

从头组装基因组的组装草图包含已组装的contigs（Bandage里称为nodes），但没有很好的办法获取这些contigs之间的连接关系（Bandage里称为edges），Bandage就是为了可视化contigs之间的连接关系而开发的。 

# 2. Bandage的功能
1. 使用Bandage，用户可以放大图中的特定区域，并通过移动节点、添加标签、改变颜色和提取序列等方式与组装图互动。 
2. 用户可以在 Bandage 图形用户界面上进行 BLAST 搜索，搜索结果会以高亮显示在图中。通过显示contigs之间的联系，Bandage 为分析全新的组装提供了新的可能性，而这是单独研究contigs所无法实现的。
3. Bandage常用来组装细胞器基因组（尤其是复杂的植物线粒体基因组）时，查看不同contigs之间的可能连接方式，以辅助组装细胞器基因组。

# 3. Bandage常用操作
## 3.1. 输入输出
1. 输入
- File-load graph：可以输入fastg组装图文件
- File-load CSV data：可以输入CSV数据
2. 输出
- File-Save image(current view)：保存目前视图到png图片文件
- File-Save image(entire scene)：保存整个视图到png图片文件

## 3.2. 鼠标操作
可以直接尝试使用，就知道分别是什么功能了。
1. 鼠标左键：原形状移动
2. 鼠标右键：变形
3. ctrl+鼠标滚动：zoom in/zoom out 放大/缩小试图。
4. ctrl+鼠标左键：移动Node的一端。会出现手掌图标，拖动Node的一端，另一端的位置不受影响。
5. ctrl+鼠标右键：拖动则旋转视图。


# 4. references
1. Bandage paper：https://academic.oup.com/bioinformatics/article/31/20/3350/196114
2. Bandage github：https://github.com/rrwick/Bandage
3. Bandage介绍页面：http://rrwick.github.io/Bandage

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>