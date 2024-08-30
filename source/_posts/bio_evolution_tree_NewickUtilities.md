---
title: 进化树处理和可视化工具Newick Utilities
date: 2024-08-30
categories: 
- bio
- evolution
- tree
tags:
- bioinfo
- biosoft
- phylogeny
- tree
- tools
- Newick Utilities
description: 使用Newick Utilities处理进化树和可视化。Newick Utilities是一套用于处理系统发育树的unix shell工具，其功能包括重置根（re-root）、提取子树、修剪、合并枝以及可视化和修饰树。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=32288422&auto=1&height=32"></iframe></div>

# 1. Newick Utilities简介
2010年，來自瑞士生物信息学研究所（Swiss Institute of Bioinformatics）的Thomas Junier 和Evgeny Zdobnov共同开发了一套Linux Shell环境下对newick格式树文件的处理和编辑工具，名为Newick Utilities。

这是一套用于处理系统发育树的unix shell工具，其功能包括重置根（re-root）、提取子树、修剪、合并枝以及可视化。

## 1.1. Newick Utilities工具主要特点：
1. 无需交互模式
2. 一次运行可以批量处理多个树文件
3. 能够处理大数据集的树文件
4. 既可以读取也可以输出

## 1.2. Newick Utilities所有工具的功能：
1. nw_clade： 提取由节点标签指定的子树
2. nw_condense：简化树
3. nw_display：显示树，作图
4. nw_duration：将节点分化时间转换为持续时间
5. nw_distance：展示节点间的遗传距离
6. nw_gen：随机树生成器
7. nw_indent：以缩行式显示树
8. nw_labels：打印节点标签
9. nw_match: 鉴定两棵树相同的拓扑结构
10. nw_order：排序树
11. nw_prune：根据标签删除分支
12. nw_reroot：重置根
13. nw_rename：标签重命名
14. nw_stats：统计和显示树的属性
15. nw_support：计算给定复制树的树的bootstrap支持率
16. nw_topology：展示树的拓扑结构，忽略枝的信息
17. nw_trim：将树的边缘设置在指定深度
18. nw_ed：流编辑器（类似于sed、awk）
19. nw_luaed：流编辑器（类似nw_ed），用Lua
20. nw_sched：流编辑器（类似nw_luaed），用Scheme

# 2. Newick Utilities工具的使用

## 2.1. Newick Utilities工具的安装
`conda install bioconda::newick_utils`

## 2.2. 输入文件
输入的树文件必须是Newick格式，可以是多棵树的树文件。
并且在所有程序的命令中，输入树文件必须是第一个参数。

支持多种输入形式

`nw_display tree.nw`或`cat tree.nw | nw_display -`或`nw_display - < tree.nw`

# 3. Newick Utilities工具的具体操作
## 3.1. nw_clade：提取子树
- `nw_clade tree.nw Ipsea >subtree.nw`：根据节点标签Ipsea提取子树
- `nw_clade -m tree.nw Melastoma Faurea Styloceras`：根据末端的叶标签鉴定是否形成单系

## 3.2. nw_prune：根据标签删除分支，也可用于提取子树
提取部分支的子树时，使用`nw_clade`只能把需要提取的节点标签直接作为参数来提取，`nw_prune`提供了把节点标签保存到文件中，使用文件作为参数提取的功能。

`nw_prune`是根据节点标签删除分支，加上`-v`参数则是根据节点标签保留分支，所以用`nw_prune`来根据文件提取指定分支的子树更为便捷。

需要提取的分支标签保存在文件subtree.txt中，每个节点标签一行，从tree.nw中提取subtree.txt指定的子节点组成的树，保存为subtree.nw。

`nw_prune -v tree.nw -f subtree.txt >subtree.nw`

## 3.3. nw_reroot：重定根

`nw_reroot tree.nw Cymbidium Ipsea | nw_display -s -`：重定根，把Cymbidium和Ipsea共同作为外类群，并画出这棵树。


# 4. reference
1. https://github.com/tjunier/newick_utils
2. https://gensoft.pasteur.fr/docs/newick-utils/1.6/nwutils_tutorial.pdf
4. https://www.jianshu.com/p/919e0cd34a0d

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>