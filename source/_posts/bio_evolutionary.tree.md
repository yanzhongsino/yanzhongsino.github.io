---
title: 进化树相关知识
date: 2021-11-20 11:50:00
categories: 
- bio
- evolution
tags:
- phylogeny
- phylogenetic tree
- evolutionary tree
- bifurcated tree
- cladogram
- phylogram
- chronogram
description: 介绍了进化树和进化树的分类等知识
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=26657608&auto=1&height=32"></iframe></div>

# 1. 进化树
系统树(phylogenetic tree/phylogeny)，又叫进化树(evolutionary tree)，是基于形态或遗传特征差异展示物种或其他实体间的进化关系的分支图或树图。

# 2. 进化树分类
## 2.1. 根据有无祖先根分类
1. 有根树(rooted tree)
    根节点是所有其他节点的父节点，每个带有后代的节点代表这些后代的最近共同祖先。在有根树中，只有根节点的度数（指传入和传出边的总数）是2，其他节点的最小度数都为3。使树生根的常见方式是指定无争议的外类群。
2. 无根树(unrooted tree)
    仅说明叶节点的相关性，无需推断祖先。

## 2.2. 根据分歧数量分类
有根树和无根树都可以是二叉的，也可以是多叉的。
1. 二叉树(bifurcated tree)
    有根二叉树每个内部节点都有两个后代，无根二叉树每个内部节点有三个邻近的自由树。
2. 多叉树(multifurcated tree)
    有根多叉树在某些内部节点可能有两个以上后代，无根多叉树在某些内部节点可能有三个以上邻近自由树。

## 2.3. 根据有无标签
1. 标签树(labeled tree)
    标签树在叶节点有特定的值。
2. 非标签树(unlabeled tree)
    只有树型。

## 2.4. 特殊的树的类型
1. 树状图(dendrogram)
    树状图的总称，无论有无系统发生关系。
2. 分支树(cladogram)
   只有分支模式，枝长不包含变化量信息，中间节点也不代表祖先。
3. 系统树(phylogram)
   枝长与变化量/碱基替换数成比例。
4. 超度量树/时序树(chronogram)
    枝长与时长成比例。
5. 速率树(ratogram)
    枝长表示替换速率。
6. 达尔格伦树(dahlgrenogram)
    系统发育树的部分样品的图。
7. 系统发育网(phylogenetic network)
    有向无环图。
8. 罗马树(romerogram)/纺锤树(spindle diagram)/气泡树(bubble diagram)
    横向宽度代表生物的分类学多样性，纵坐标是地质时间。为了反映多个类群的丰度随着时间的变化。由于不适合处理并系类群，被提议不再使用。
9. 生命珊瑚(coral of life)
    达尔文提出珊瑚可能比树更适合描绘生命的进化。

# 3. 读进化树
## 3.1. 根据树型
- 当树的所有叶节点没有对齐的时候，枝长可能代表变化量/碱基替换数(phylogram)/替换速率(ratogram)；
- 当树的所有叶节点对齐的时候，枝长如果一致则无信息(cladogram)，枝长如果不一致可能代表进化时长(chronogram)。


# 4. reference
1. [wiki:Phylogenetic_tree](https://en.wikipedia.org/wiki/Phylogenetic_tree)
2. https://www.nature.com/scitable/topicpage/reading-a-phylogenetic-tree-the-meaning-of-41956/


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>