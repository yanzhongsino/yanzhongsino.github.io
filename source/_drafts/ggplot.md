---
title: ggplot
date: 2021-11-22 19:50:00
categories: 
- bio
- bioinfo
tags:
- tutorial
- biosoft
- phylogeny
- evolutionary tree
description: 
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

# ggplot


## plot


## 多个密度图放一起
可以把数据放在一起，添加一个分类标签列，颜色指定这个分类标签列，则可以实现一张图上展示多个密度图。
ggplot(da)+geom_density(aes(x=V3,colour=V5),adjust=2)+xlim(0,0.5)+theme_classic()


## theme
ggplot2提供一些主题，包括默认的theme_grey()，白色背景的theme_bw()，和经典主题theme_classic()。可以直接在图后面加上` + theme_classic()`使用。

推荐`theme_classic()`主题，白色背景，没有网格。

另外，ggthemes包提供了一些主题，可以`require(ggthemes)`载入ggthemes包后使用：
- theme_economist()
- theme_economist_white()
- theme_wsj()
- theme_excel()
- theme_few()
- theme_foundation()
- theme_igray()
- theme_solarized()
- theme_stata()
- theme_tufte()



# references