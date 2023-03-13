---
title: ggplot
date: 2022-10-17
categories: 
- biosoft
- R
- plot
tags:
- R package
- biosoft
- ggplot
- phylogeny
- evolutionary tree
description: 
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

# ggplot


## plot

## 密度图

ks分布密度图
```
library(ggplot2)
library(ggpmisc)
data <- read.table("all.results",header=T)
p <- ggplot(data, aes(Ks)) + geom_density(size=1,color="black")+xlab("Synonymous substitution rate(Ks)")+ylab("Percent of gene pairs
")+theme(panel.grid=element_blank())+theme(panel.border = element_blank())+theme(axis.line = element_line(size=0.8, colour = "black"))+scale_y_continuous(breaks=seq(0, 2.5, 0.2))+xlim(0,3)
pb <- ggplot_build(p)
pic <- p + stat_peaks(data = pb[['data']][[1]], aes(x=x, y=density), geom= 'text', color="darkblue", ,vjust=-0.5)
ggsave(file="Ks.pdf",plot=pic,width=10,height=5)
```


library(ggplot2)
library(ggpmisc)
data <- read.table("all.results",header=T)
ggplot(data,aes(Ks))+geom_density(size=1,color="white",fill="lightblue",alpha=0.8,adjust=2)+xlab("Synonymous substitution rate(Ks)")+ylab("Percent of gene pairs")
+geom_vline(xintercept = 0.25,linetype="dashed",color="darkblue")+annotate(geom="text",x=0.45,y=2,label="0.25",color="darkblue",size=5)
+geom_vline(xintercept = 1.74,linetype="dashed",color="darkblue")+annotate(geom="text",x=2,y=2,label="1.74",color="darkblue",size=5)
+xlim(0,3)+scale_y_continuous(breaks=seq(0, 2.5, 0.2))+ theme_classic()+theme(axis.title.x = element_text(size = 15, face = "bold"),axis.title.y = element_text(size = 15, face = "bold"),axis.text.x = element_text(size = 13, face = "bold"),axis.text.y = element_text(size = 13, face = "bold"))

# adjust用于调整曲线平滑度

pic <- ggplot(data,aes(Ks))+geom_histogram(size=0.5,color="black",fill="lightblue",alpha=0.8,bins=100)+xlab("Synonymous substitution rate(Ks)")+ylab("Number of gene pairs")
+geom_vline(xintercept = 0.25,linetype="dashed",color="darkblue")+annotate(geom="text",x=0.45,y=2,label="0.25",color="darkblue",size=5)
+geom_vline(xintercept = 1.74,linetype="dashed",color="darkblue")+annotate(geom="text",x=2,y=2,label="1.74",color="darkblue",size=5)
+xlim(0,3)+scale_y_continuous(breaks=seq(0, 2.5, 0.2))+ theme_classic()+theme(axis.title.x = element_text(size = 15, face = "bold"),axis.title.y = element_text(size = 15, face = "bold"),axis.text.x = element_text(size = 13, face = "bold"),axis.text.y = element_text(size = 13, face = "bold"))
ggsave(file="Ks.pdf",plot=pic,width=6,height=6)
保存成6*6inches的图片，label的字体大小size=5，xy轴的字体大小size=15比较合适。

   
# bins用于调整直方图数量


+geom_vline(xintercept = 0.25,linetype="dashed",color="darkblue") #添加垂直x轴的线条，xintercept定义x轴值。垂直y轴的用geom_hline()

+annotate(geom="text",x=0.45,y=2,label="0.25",color="darkblue",size=5) # 添加注释


## 多个密度图放一起
可以把数据放在一起，添加一个分类标签列，颜色指定这个分类标签列，则可以实现一张图上展示多个密度图，每个密度图纵坐标（即密度值）是单独计算的。
ggplot(da)+geom_density(aes(x=V3,colour=V5),adjust=2)+xlim(0,0.5)+theme_classic()

## 分组直方图和密度图
ggplot(data,aes(x=Ks)) + geom_histogram(data = subset(data, group == "mcmd"), aes(y = ..density..),fill = "red", alpha = 0.2,binwidth = 0.002,size=0.5)+geom_density(data = subset(data, group == "mcmd"), aes(y = ..density..), col = "red", size=1.2)+ geom_histogram(data = subset(data, group == "mdmd"), aes(y = ..density..),fill = "blue", alpha = 0.2,binwidth = 0.002,size=0.5)+geom_density(data = subset(data, group == "mdmd"), aes(y = ..density..), col = "blue", size=1.2)+ geom_histogram(data = subset(data, group == "mcmc"), aes(y = ..density..),fill = "green", alpha = 0.2,binwidth = 0.002,size=0.5)+geom_density(data = subset(data, group == "mcmc"), aes(y = ..density..), col = "green", size=1.2) +xlab("Synonymous substitution rate(Ks)")+ylab("Percent of Total Paralogs")+theme_classic()+scale_y_continuous(breaks=seq(0, 21, 3))+xlim(0,0.8)+theme(legend.position = c(0.4,0.5), axis.title.x = element_text(size = 16, face = "bold"),axis.title.y = element_text(size = 16, face = "bold"),axis.text.x = element_text(size = 15, face = "bold"),axis.text.y = element_text(size = 15, face = "bold")) + annotate(geom="text",x=0.06,y=20,label="0.03",color="#F8766D",size=6) + annotate(geom="text",x=0.17,y=6,label="0.20",color="#00BA38",size=6) + annotate(geom="text",x=0.72,y=3,label="0.70",color="#00BA38",size=6) + annotate(geom="text",x=0.23,y=6,label="0.21",color="#4169E1",size=6) + annotate(geom="text",x=0.66,y=3,label="0.68",color="#4169E1",size=6)


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

图例的位置
theme(legend.position = c(0.8,0.6)) #数字代表图例在图中的横坐标比例，在0-1之间，如果超出图的范围可能消失


## 坐标轴
+theme(axis.title.x = element_text(size = 13, face = "bold", family = "myFont", color = "black", vjust = 0.5, hjust = 0.5, angle = 45), #设置x轴标签格式，大小size，加粗/斜体face(plain普通,bold加粗,italic斜体,bold.italic斜体加粗)，位置调整vjust&hjust，角度angle
axis.title.y = element_text(size = 12, face = "bold"), #设置y轴标签格式
axis.text.x = element_text(size = 10, face = "bold"), #设置x轴刻度标签格式
axis.text.y = element_text(size = 10, face = "bold")) #设置y轴刻度标签格式


## 对数变化

- scale_x_log10() #把x轴值转换成对数比例

## 分面分组
facet_wrap(~group)


# references

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>
