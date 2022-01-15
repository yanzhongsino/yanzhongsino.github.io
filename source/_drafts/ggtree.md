---
title: 绘制进化树
date: 2022-01-15 17:00:00
categories: 
- bio
- bioinfo
tags:
- tutorial
- biosoft
- phylogeny
- evolutionary tree
- itol
- ggtree
description: 记录进化树的绘制，软件的使用。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

# 进化树
进化树的相关知识可以参考[博文进化树相关知识](https://yanzhongsino.github.io/2021/11/20/bio_evolution.tree/)。

# 进化树的绘制
进化树的绘制是指把newick格式的树文件转换成树图形，有许多软件可以使用，在线网页版（itol，EvolView），也有R包（ggtree），或者软件figtree。
如果是简单的树绘制，推荐在线网页**itol**；如果需要精细的注释，推荐R包**ggtree**。

## 在线绘制进化树
1. [itol](https://itol.embl.de/)
2. [EvolView](https://www.evolgenius.info/evolview/#login)

## R包ggtree【推荐】
R包treeio用于解析各种格式的进化树文件，ggtree用于绘制进化树和注释。
### 树文件的导入
#### 树文件
1. 树文件格式
树文件格式包括Newick树格式，NEXUS树格式，New Hampshire eXtended(NHX)树格式，Jplace树格式。其中Newick树格式最常见。
2. 建树软件
有许多建树软件可以生成树文件。包括RAxML，iqtree，BEAST，MrBayes，PAML(BASEML,CODEML)，r8s，MEGA，HyPhy，Phylip，以及生成Jplace格式树的EPA和PPLACER。

#### 导入树文件
1. 文本输入
- 如果进化树的物种数量不多，内容比较简单，可以直接把进化树编辑成Newick格式，文本输入R，用read.newick读取。
```R
library(treeio)
treetext<-"((((Mc:0.418672,Eg:0.201388):0.148853,((Pt:0.219587,(Cs:0.209405,(Gr:0.224188,At:0.492461):0.027774):0.012564):0.019343,(Pp:0.194515,(Mt:0.354580,Cl:0.338644:0.022063):0.019111):0.011710):0.029914,Vv:0.169100):0.241611,Mg:0.241611);"
tree <- read.newick(textConnection(treetext))
```

- 也可以编辑成NHX格式，中括号内[]添加注释信息&&NHX:，用read.nhx读取。

```R
treetext<-"((((Mc:0.418672[&&NHX:I=+5728:D=-1170:G=Malvids],Eg:0.201388[&&NHX:I=+1420:D=-2519:G=Malvids])[&&NHX:C=1]:0.148853,((Pt:0.219587[&&NHX:I=+5355:D=-1003:G=Fabids],(Cs:0.209405[&&NHX:I=+1026:D=-3516:G=Malvids],(Gr:0.224188[&&NHX:I=+4875:D=-998:G=Malvids],At:0.492461[&&NHX:I=+1776:D=-2891:G=Malvids]):0.027774[&&NHX:C=2]):0.012564[&&NHX:C=3]):0.019343[&&NHX:C=4],(Pp:0.194515[&&NHX:I=+1159:D=-2949:G=Fabids],(Mt:0.354580[&&NHX:I=+3310:D=-1984:G=Fabids],Cl:0.338644[&&NHX:I=+794:D=-3735:G=Fabids]):0.022063[&&NHX:C=5]):0.019111[&&NHX:C=6]):0.011710[&&NHX:C=7]):0.029914[&&NHX:C=8],Vv:0.169100[&&NHX:I=+1522:D=-3045:G=Vitales]):0.241611[&&NHX:C=9],Mg:0.241611[&&NHX:I=+2278:D=-1909:G=Asterids])[&&NHX:C=10];"
tree <- read.nhx(textConnection(treetext))
```

2. treeio导入树文件
treeio支持解析大部分软件输出的树文件，包括RAxML(read.raxml)，iqtree，BEAST(read.beast)，MrBayes(read.mrbayes)，PAML(BASEML,CODEML)(read.paml_rst,read.codeml,read.codeml_mlc)，r8s(read.r8s)，MEGA(read.mega)，HyPhy(read.hyphy,read.hyphy.seq)，Phylip(read.phylip)，以及Newick树格式(read.newick)，NHX格式树(read.nhx)，Jplace格式树（EPA，PPLACER,read.jplace），jtree格式(read.jtree)等。

```R
library(treeio)
file <- system.file("extdata/BEAST", "beast_mcc.tree", package="treeio")
beast <- read.beast(file)
```

```R
library(treeio)
tree <- read.newick("sample.tre") #读取newick格式的树文件
```

### 画树
1. 基础画树
```R
library(ggtree)
ggtree(tree) #默认参数画树
```

2. 添加注释美化的常用设置
ggplot可以添加的图层都可以用在ggtree上。

```R
ggtree(tree)
+ geom_tiplab(color="black",size = 4,hjust = -0.2) \ #添加物种名
+ geom_point(color="#6FE1F8", size=5, alpha=0.7) \ #在节点处添加圆点
+ geom_text(aes(label=node),size=3) \ #在节点处添加数字标签，也可以用这个来确定每个节点是几号节点
+ geom_highlight(node=15,fill="firebrick",alpha=0.6) #高亮15号节点
```

3. 添加注释美化的其他设置

```R
ggtree(mtree, size=0.8, branch.length='none', mrsd="2021-10-01") \ #画树，size指定枝的粗细，branch.length指定枝长，mrsd指定采样日期用于画时间尺度轴
+ geom_tiplab(color="black", size = 4, hjust = -0.1, align=T, linesize=0.7) \ # 末端节点标上物种，size文字大小，hjust是离进化枝线条的距离，align=T文字间对齐，linesize设置虚线尺寸。
+ geom_point(color="#6FE1F8", size=5, alpha=0.7) \ #节点标记圆点，配色和透明度
+ geom_text(aes(label=node),size=3) \ #节点标记上数字，尺寸
+ geom_highlight(node=15,fill="firebrick",alpha=0.6) #高亮指定分支（节点node所在分支）
+ geom_label2(aes(x=0.79,label=D),color="black",vjust=-0.2,fill="#7FFF00",alpha=0.6) #做标记,aes里的x指定位置，label指定内容
+ geom_strip(9,10,label="Cercidoideae",offset = 0.06,offset.text = 0.012,color = "#FF6100",barsize = 1.5,fontsize = 5,angle = 90,hjust = 0.5) #显示条带和文字，用于指定clade
+ geom_range("length_0.95_HPD", color='red', size=2, alpha=.5) #在节点上画长条形的时间95%置信范围
+ xlim(NA,1) #如果内容超出边界，就拓展显示
```

画成有枝长的树，在ggtree里指定branch.length参数。
画时间标尺，在ggtree里指定mrsd参数为最近的采样日期，用于画时间尺度轴。

4. 一个生成树的例子
```R
treetext<-"((((Mc:0.418672[&&NHX:I=+5728:D=-1170:G=Malvids],Eg:0.201388[&&NHX:I=+1420:D=-2519:G=Malvids])[&&NHX:C=1]:0.148853,((Pt:0.219587[&&NHX:I=+5355:D=-1003:G=Fabids],(Cs:0.209405[&&NHX:I=+1026:D=-3516:G=Malvids],(Gr:0.224188[&&NHX:I=+4875:D=-998:G=Malvids],At:0.492461[&&NHX:I=+1776:D=-2891:G=Malvids]):0.027774[&&NHX:C=2]):0.012564[&&NHX:C=3]):0.019343[&&NHX:C=4],(Pp:0.194515[&&NHX:I=+1159:D=-2949:G=Fabids],(Mt:0.354580[&&NHX:I=+3310:D=-1984:G=Fabids],Cl:0.338644[&&NHX:I=+794:D=-3735:G=Fabids]):0.022063[&&NHX:C=5]):0.019111[&&NHX:C=6]):0.011710[&&NHX:C=7]):0.029914[&&NHX:C=8],Vv:0.169100[&&NHX:I=+1522:D=-3045:G=Vitales]):0.241611[&&NHX:C=9],Mg:0.241611[&&NHX:I=+2278:D=-1909:G=Asterids])[&&NHX:C=10];"
tree <- read.nhx(textConnection(treetext))

p1 <- ggtree(tree,size=0.8) #画树，size指定线条粗细
+ geom_tiplab(color="black",size = 4.5,hjust = -0.1, align=TRUE, linesize=0.7) #添加tiplab-物种名
+ geom_nodepoint(color="#DCDCDC", size=6, alpha=0.9) #节点处添加圆点，指定颜色和尺寸
+ geom_nodelab(aes(label=C), size=3.5, hjust = 1) #在注释了标签C的节点处添加C的值，这里的标签C用于注释节点序号
+ geom_label2(aes(x=0.79,label=D),color="black",vjust=-0.2,fill="#7FFF00",alpha=0.6) # 在注释了D的位置添加D的值，这里的标签D用于注释收缩基因的数量，x的值用于指定添加位置。
+ geom_label2(aes(x=0.73,label=I),color="black",vjust=-0.2,fill="#FF7D40",alpha=0.6) # 在注释了I的位置添加I的值，这里的标签I用于注释扩张基因的数量，x的值用于指定添加位置。
+ xlim(NA,1.1) # 如果物种名或者其他标签超出显示范围，用xlim增大x轴显示。

p2<- ggtree::rotate(p1,13) #旋转节点13
p2<- p1 %>% rotate(node=14) %>% rotate(node=16) # 旋转节点14和节点16

p2+geom_strip(10,10,label="Vitales",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5)+geom_strip(3,7,label="Fabids",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5)+geom_strip(1,4,label="Fabids",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5)
```

# reference
https://yulab-smu.top/treedata-book/index.html