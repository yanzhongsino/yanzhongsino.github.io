---
title: 进化树处理和可视化工具Newick Utilities
date: 2023-10-18
categories: 
- bioinfo
- phylogeny
tags:
- bioinfo
- biosoft
- phylogeny
- tools
- Newick Utilities
description: 使用Newick Utilities处理进化树和可视化。Newick Utilities是一套用于处理系统发育树的unix shell工具，其功能包括重置根（re-root）、提取子树、修剪、合并枝以及可视化。
---

<div align="middle"></div>

# Newick Utilities简介
2010年，來自瑞士生物信息学研究所（Swiss Institute of Bioinformatics）的Thomas Junier 和Evgeny Zdobnov共同开发了一套名为Newick Utilities的Linux Shell环境下对newick格式树文件的处理和编辑工具。

是一套用于处理系统发育树的unix shell工具，其功能包括重置根（re-root）、提取子树、修剪、合并枝以及可视化。

## Newick Utilities工具主要特点：
1. 无需交互模式
2. 一次运行可以批量处理多个树文件
3. 能够处理大数据集的树文件
4. 既可以读取也可以输出

## Newick Utilities所有工具的功能：
1. nw_clade： 提取由节点标签指定的子树
2. nw_condense：简化树
3. nw_display：显示树
4. nw_duration：将节点分化时间转换为持续时间
5. nw_distance：打印节点间的遗传距离
6. nw_ed：流编辑器（类似于sed、awk）
7. nw_gen：随机树生成器
8. nw_indent：以缩行式显示树
9. nw_labels：打印节点标签
10. nw_luaed：类似于nw_ed
11. nw_match: 在另一棵树中查找树的匹配项
12. nw_order：排序
13. nw_prune：根据标签删除分支
14. nw_reroot：重置根
15. nw_rename：标签重命名
16. nw_sched
17. nw_stats：统计
18. nw_support：计算给定复制树的树的引导支持
19. nw_topology：改变分支属性，保持拓扑结构
20. nw_trim：将树的边缘设置在指定深度

# Newick Utilities工具的使用

## nw_prune：根据标签删除分支
从newick格式的树文件中提取部分支的子树时，使用`nw_clade`只能把需要提取的节点标签直接作为参数来提取，`nw_prune`提供了把节点标签保存到文件中，使用文件作为参数提取的功能。`nw_prune`是根据节点标签删除分支，加上`-v`参数则是根据节点标签保留分支，所以用`nw_prune`来根据文件提取指定分支的子树更为便捷。

需要提取的分支标签保存在文件subtree.txt中，每个节点标签一行，从tree.nw中提取subtree.txt指定的子节点组成的树，保存为subtree.nw。

`nw_prune -v tree.nw -f subtree.txt >subtree.nw`



# 4. reference
1. https://github.com/tjunier/newick_utils
2. https://www.twblogs.net/a/5d663e01bd9eee541c3323c3
3. https://www.jianshu.com/p/919e0cd34a0d

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>