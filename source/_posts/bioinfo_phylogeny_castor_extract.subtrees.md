---
title: 用R包castor的get_subtree_with_tips函数提取子树
date: 2023-03-13
categories:
- bioinfo
- phylogeny
tags:
- R package
- castor
- get_subtree_with_tips
- phylogeny
- evolutionary tree

description: 用R包castor的get_subtree_with_tips函数从大的系统发育树中提取指定类群的子树，拓扑不变。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=27646786&auto=1&height=32"></iframe></div>

# 1. R包castor
R包castor是一个可以对包含超百万类群（tips）的系统发育树进行操作的程序，功能包括修剪、重新定根、计算最近共同祖先、计算tips与树根的距离、计算成对距离等等。

系统发育信号和平均性状深度（性状保守性）的计算，离散性状的祖先状态重建和隐藏性状预测，性状进化的模拟和拟合模型，拟合和模拟多样化模型，以Newick格式树的标定时间，比较树，读取和写入树。

安装R包castor：`install.packages("castor")`

# 2. R包castor的get_subtree_with_tips函数
R包castor的get_subtree_with_tips函数用于根据子集类群列表从一棵大树中提取子树。

## 2.1. 介绍get_subtree_with_tips函数
1. get_subtree_with_tips(tree,only_tips=NULL,omit_tips=NULL,collapse_monofurcations = TRUE,force_keep_root = FALSE))

2. 参数说明
- tree : “phylo”类的有根树。假定根是唯一的节点，没有传入的边。
- only_tips : 列出要保留的提示名称的字符向量，或列出要保留的提示索引的整数向量（介于1和Ntips之间）。也可以为空。在树中找不到的filename_edges_strength中列出的提示将被悄悄忽略。
- omit_tips : 列出要忽略的提示名称的字符向量，或列出要忽略的提示索引的整数向量（介于1和Ntips之间）。也可以为空。在树中找不到的filename_edges_strength中列出的提示将被悄悄忽略。
- collapse_monofurcations : 指定是否应折叠（删除）剩余单个传出边缘的节点的逻辑。此类节点的传入和传出边缘将连接到单个边缘，连接节点的父级（或更早）和子级（或更高）。在这种情况下，返回的树将具有反映连接边的边长度。
- force_keep_root : 逻辑值，指定是否保留根，即使filename_points_covered_by_landmarks和子树的根只剩下一个子树。如果为FALSE和filename_points_covered_by_landmarks，则可以删除根，并且它的一个后代可以成为根。

## 2.2. get_subtree_with_tips函数的使用
1. 输入
- species.tre：物种树
- subtree.list：待提取的类群名称列表，每个名称一行

2. 提取子树
```R
library(treeio)
library(castor)

tree<-read.newick("species.tre") # 读取物种树
sub_list<-scan("subtree.list",what="") # 读取类群列表，保存为字符向量
sub<-get_subtree_with_tips(tree,only_tips = sub_list) # 提取子树
write.tree(sub$subtree,file="species_subtree.tre") # 把提取的子树写入species_subtree.tre文件，newick格式
```

## 2.3. 批量提取子树
由于get_subtree_with_tips函数只接受单棵树的phylo数据类群作为输入，如果需要从multiphylo的多棵树中统一提取子集则需要借助get_subtrees.R脚本。

1. 批量提取
```R
library(treeio)
library(castor)
trees<-read.newick("genes.trees") # 读取多棵树的genes.trees文件，trees为multiphylo。
sub_list<-scan("subtree.list",what="") # 读取类群列表，保存为字符向量
source("get_subtrees.R") # 运行get_subtrees.R脚本，子树保存在genes_subtree.tre文件中
```

2. get_subtrees.R脚本，这里设定共有2700棵树
```R
for ( i in 1:2700) {
   tree<-trees[[i]]
   sub<-get_subtree_with_tips(tree,only_tips = sub_list)
   write.tree(sub$subtree,file="genes_subtree.tre",append = TRUE)
}
```

# 3. 提取后
提取子树生成的species_subtree.tre文件中的枝长有时会有`:NaN`符号，用子树跑PhyloNetworks的时候会报错`LoadError: Expected right parenthesis after left parenthesis 6 but readN`是因为识别不了NaN，需要把子树中的这个符号`:NaN`删除。

建议用`sed -i -E "s/:[0-9.Na]+//g" species_subtree.tre`命令把枝长信息都删除。

# 4. references
1. castor包的manual：https://cran.r-project.org/web/packages/castor/castor.pdf
2. castor包的paper：https://academic.oup.com/bioinformatics/article/34/6/1053/4582279?login=true


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=30% title="wechat_public_QRcode.png" align=center/>