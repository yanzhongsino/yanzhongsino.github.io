---
title: 集合可视化 —— 文氏图/韦恩图(Venn Diagram)
date: 2021-09-29 15:56:36
categories:
- computer language
- R
tags:
- plot
- R
- R package
- Venn
- gplots
- VennDiagram
description: 记录了制作韦恩图(Venn Diagram)展示多集合共享关系的方法。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

# 1. 文氏图/韦恩图(Venn Diagram) —— 5个集合内的可视化
当有多个群组（集合）的数据，想要展示在不同的事物群组（集合）之间的数学或逻辑联系时，可使用韦恩Venn图和UpSet图展示多集合直接的共享关系和各自的独享关系。

当集合数量少于等于5个时，多用韦恩图；当集合数量大于5个时，多用upset图。

韦恩图的绘制推荐使用R包gplots；upset图的绘制推荐使用R包UpSetR，参考博文[upset图](https://yanzhongsino.github.io/2021/09/29/R_plot_upset/)。

## 1.1. 应用场景
1. 利用orthofinder找到的不同物种的orthogroups的结果来绘制Venn Diagram韦恩图或者UpSet plot图，查看不同物种间共享的orthogroups的数量关系。

## 1.2. 数据准备
从orthofinder的结果文件Results_date/Orthogroups/Orthogroups.GeneCount.tsv（每个物种拥有每种orthogroups的基因数量的统计）中提取每个物种拥有的orthogroups信息；并合并作为venn的输入文件。

```sh
species=($(head -1 Orthogroups.GeneCount.tsv|sed "s/.pep//g")) #读取Orthogroups.GeneCount.tsv第一行表头写入species列表中
for i in $(seq 1 `expr ${#species[@]} - 2`); do awk -v awka=`expr $i + 1` '$awka != 0 {print $1}' Orthogroups.GeneCount.tsv  |sed "1s/.*/${species[$i]}/" >${species[$i]}.temp; done #第二到倒数第二列（物种数据所在列）不为0时，输出第一列内容（OG的ID号）。
paste *.temp >orthogroups.venn #把提取的所有物种横向拼接成一个文件，即为venn的输入。
rm *.temp #删除临时文件
```

## 1.3. 韦恩图绘制
### 1.3.1. 在线画韦恩图
1. http://bioinformatics.psb.ugent.be/webtools/Venn/
2. http://jvenn.toulouse.inra.fr/app/example.html

快和方便，但图不好看。

### 1.3.2. R包画韦恩图

#### 1.3.2.1. R包gplots
R包gplots画韦恩图方便快捷参数少，缺点是画出来的图黑白无填充色，适合初期查看数据。

```R
install.packages("gplots") #安装gplots包
library(gplots) #载入gplots包
setwd("D:/R/") #设置工作目录
data <- read.table("orthogroups.venn", header = T, sep = "\t") #读入数据文件
head(data) #查看数据文件
attach(data) #把数据用于全局
venn(data = list(Athaliana,Graimondii,Ppersica,Mtruncatula,Vvinifera)) #画韦恩图，需要几维就填入几列数据，黑白无填充图，根据data的表头，即第一行来填写。
```
<br>
```R
> head(data)
  Athaliana  Csativus Csinensis Graimondii Mguttatus Mtruncatula  Ppersica Ptrichocarpa Vvinifera
1 OG0000001 OG0000001 OG0000001  OG0000001 OG0000000   OG0000000 OG0000001    OG0000001 OG0000001
2 OG0000002 OG0000002 OG0000002  OG0000002 OG0000001   OG0000001 OG0000002    OG0000002 OG0000002
3 OG0000003 OG0000003 OG0000003  OG0000003 OG0000002   OG0000002 OG0000003    OG0000003 OG0000003
4 OG0000004 OG0000004 OG0000004  OG0000004 OG0000003   OG0000003 OG0000004    OG0000004 OG0000004
5 OG0000005 OG0000005 OG0000005  OG0000005 OG0000004   OG0000004 OG0000005    OG0000005 OG0000005
6 OG0000007 OG0000007 OG0000007  OG0000007 OG0000005   OG0000005 OG0000007    OG0000007 OG0000006
```

#### 1.3.2.2. R包VennDiagram
R包VennDiagram画韦恩图参数较多，适合做用于发表的韦恩图。

1. 准备工作
```R
install.packages("VennDiagram") #安装VennDiagram包
library(VennDiagram) #载入VennDiagram包
library(grid) #载入grid包
data <- read.table("orthogroups.venn", header = T, sep = "\t") #读入数据文件
head(data) #查看数据文件
attach(data) #把数据用于全局
```

2. 一维韦恩图
```R
grid.newpage(); #清除已有图形，开始新的空白页
venn.single.plot <- venn.diagram(
  x = list(Athaliana=Athaliana), #等号前面是图上显示的数据名称，等号后面是数据源。
  filename = NULL,
  col = "black", #图的边界颜色
  lwd = 0,
  fontface = "bold",
  fill = "grey", #填充色
  aplha = 0.75, #透明度
  cex = 4,
  cat.cex = 3,
  cat.fontface = "bold",
); #一维韦恩图
grid.draw(venn.single.plot); #用venn.plot绘图
```

3. 二维韦恩图
```R
grid.newpage(); #清除已有图形，开始新的空白页
venn.pairwise.plot <- venn.diagram(
  x = list(Graimondii=Graimondii, Ppersica=Ppersica), #等号前面是图上显示的数据名称，等号后面是数据源。
  filename = NULL,
  lwd = 4,
  fill = c("cornflowerblue", "darkorchid1"), #填充色
  aplha = 0.75, #透明度
  label.col = "white",
  cex = 4,
  fontfamily = "serif",
  fontface = "bold",
  cat.col =  c("cornflowerblue", "darkorchid1"),
  cat.cex = 3,
  cat.fontfamily = "serif",
  cat.fontface = "bold",
  cat.dist = c(0.03, 0.03),
  cat.pos = c(-20,14)
); #二维韦恩图
grid.draw(venn.pairwise.plot); #用venn.plot绘图
```

<img src="venn_2.png" width=80% height=80% title="venn_2" align=center/>

4. 三维韦恩图
```R
grid.newpage(); #清除已有图形，开始新的空白页
venn.triple.plot <- venn.diagram(
  x = list(Athaliana=Athaliana, Ppersica=Ppersica, Graimondii=Graimondii), #等号前面是图上显示的数据名称，等号后面是数据源。
  filename = NULL,
  col = "transparent",
  fill = c("red", "blue", "green"),
  alpha = 0.5,
  label.col = c("darkred", "white", "darkblue", "white",
                "white", "white", "darkgreen"),
  cex = 2.5,
  fontfamily = "serif",
  fontface = "bold",
  cat.default.pos = "text",
  cat.col = c("darkred", "darkblue", "darkgreen"),
  cat.cex = 2.5,
  cat.fontfamily = "serif",
  cat.dist = c(0.06, 0.06, 0.03),
  cat.pos = 0
);
grid.draw(venn.triple.plot); #用venn.plot绘图
```
<img src="venn_3.png" width=50% height=50% title="venn_3" align=center/>

5. 四维韦恩图
```R
grid.newpage(); #清除已有图形，开始新的空白页
venn.quad.plot <- venn.diagram(
  x = list(Mtruncatula=Mtruncatula, Vvinifera=Vvinifera, Graimondii=Graimondii, Ppersica=Ppersica), #等号前面是图上显示的数据名称，等号后面是数据源，数据顺序与图上顺序一致。
  filename = NULL,
  col = "black",
  lty = "dotted", #边框线类型
  lwd = 4,
  fill = c("cornflowerblue", "green", "yellow", "darkorchid1"),
  alpha = 0.50,
  label.col = c("orange", "white", "darkorchid4", "white", "white", "white",
                "white", "white", "darkblue", "white",
                "white", "white", "white", "darkgreen", "white"),
  cex = 2.5,
  fontfamily = "serif",
  fontface = "bold",
  cat.col = c("darkblue", "darkgreen", "orange", "darkorchid4"),
  cat.cex = 2.5, #分类名称字体大小
  cat.fontfamily = "serif"
); # 四维韦恩图
grid.draw(venn.quad.plot); #用venn.plot绘图
```

<img src="venn_4.png" width=50% height=50% title="venn_4" align=center/>

6. 五维韦恩图
```R
grid.newpage(); #清除已有图形，开始新的空白页
venn.quintuple.plot <- venn.diagram(
  x = list(Athaliana=Athaliana, Ppersica=Ppersica, Graimondii=Graimondii, Mtruncatula=Mtruncatula, Vvinifera=Vvinifera),
  filename = NULL, # 韦恩图结果文件保存路径和名称
  col = "black", #指定图形的圆周边缘颜色，transparent透明
  fill = c("dodgerblue", "goldenrod1", "darkorange1", "seagreen3", "orchid3"), # 填充颜色
  alpha = 0.50, #透明度
  cex = c(1.5, 1.5, 1.5, 1.5, 1.5, 1, 0.8, 1, 0.8, 1, 0.8, 1, 0.8,
          1, 0.8, 1, 0.55, 1, 0.55, 1, 0.55, 1, 0.55, 1, 0.55, 1, 1, 1, 1, 1, 1.5), #每个区域label名称的大小
  cat.col = c("dodgerblue", "goldenrod1", "darkorange1", "seagreen3", "orchid3"), #分类颜色
  cat.cex = 1.5, #每个分类名称大小
  cat.dist = 0.07, #分类名称距离边的距离
  cat.just = list(c(-1,-1),c(2,-1),c(2,1),c(0,4),c(-0.5,-1)), #分类名称的位置，圈内或圈外
  cat.fontface = "bold",
  cat.fontfamily = "serif", # 分类字体
  margin = 0.2 #图距离边缘的距离
); # 五维韦恩图
grid.draw(venn.quintuple.plot); #用venn.plot绘图
```

<img src="venn_5.png" width=50% height=50% title="venn_5" align=center/>

#### 1.3.2.3. 保存R生成的图
在RStudio里作图，选择Export-Save as PDF
一般参数：PDF Size(A4)+Orientation(Landscape)
保存图像为pdf即可。

# 2. references
https://www.jianshu.com/p/79fb263e41ff