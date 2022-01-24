---
title: 绘制进化树
date: 2022-01-24 17:00:00
categories: 
- bio
- bioinfo
tags:
- tutorial
- R
- biosoft
- phylogeny
- evolutionary tree
- itol
- treeio
- ggtree
- facet_plot
- ggstance
- 系统发育网
- geom_taxalink
- multiPhylo
- ggdensitree
description: 记录进化树的绘制，软件的使用，详细介绍了ggtree绘制树和注释树。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=283100&auto=1&height=32"></iframe></div>

# 1. 进化树
进化树的相关知识可以参考[博文进化树相关知识](https://yanzhongsino.github.io/2021/11/20/bio_evolution.tree/)。

# 2. 进化树的绘制
进化树的绘制是指把newick格式的树文件转换成树图形，有许多软件可以使用，在线网页版（itol，EvolView），也有R包（ggtree），或者软件figtree。
如果是简单的树绘制，推荐在线网页**itol**；如果需要精细的注释，推荐R包**ggtree**。

## 2.1. 在线绘制进化树
1. [itol](https://itol.embl.de/)
2. [EvolView](https://www.evolgenius.info/evolview/#login)

## 2.2. R包ggtree【推荐】
R包treeio用于解析各种格式的进化树文件，ggtree用于绘制进化树和注释。
### 2.2.1. 树文件的导入
#### 2.2.1.1. 树文件
1. 树文件格式
树文件格式包括Newick树格式，NEXUS树格式，New Hampshire eXtended(NHX)树格式，Jplace树格式。其中Newick树格式最常见。
2. 建树软件
有许多建树软件可以生成树文件。包括RAxML，iqtree，BEAST，MrBayes，PAML(BASEML,CODEML)，r8s，MEGA，HyPhy，Phylip，以及生成Jplace格式树的EPA和PPLACER。

#### 2.2.1.2. treeio导入树文件
1. 文本输入
- 如果进化树的物种数量不多，内容比较简单，可以直接把进化树编辑成Newick格式，文本输入R，用read.newick读取。
```R
library(treeio)
treetext<-"((((Mc:0.418672,Eg:0.201388):0.148853,((Pt:0.219587,(Cs:0.209405,(Gr:0.224188,At:0.492461):0.027774):0.012564):0.019343,(Pp:0.194515,(Mt:0.354580,Cl:0.338644:0.022063):0.019111):0.011710):0.029914,Vv:0.169100):0.241611,Mg:0.241611);"
tree <- read.newick(textConnection(treetext))
```

- 也可以编辑成NHX格式，中括号内[]添加注释信息&&NHX:，用read.nhx读取。

```R
treetext<-"((((Mc:0.418672[&&NHX:I=+5728:D=-1170:G=Malvids],Eg:0.201388[&&NHX:I=+1420:D=-2519:G=Malvids])[&&NHX:C=1]:0
.148853,((Pt:0.219587[&&NHX:I=+5355:D=-1003:G=Fabids],(Cs:0.209405[&&NHX:I=+1026:D=-3516:G=Malvids],(Gr:0.224188[&&NHX:I=+4875:D=-998:G=Malvids],At:0.492461[&&NHX:I=+1776:D=-2891:G=Malvids]):0.027774[&&NHX:C=2]):0.012564[&&NHX:C=3]):0.019343[&&NHX:C=4],(Pp:0.194515[&&NHX:I=+1159:D=-2949:G=Fabids],(Mt:0.354580[&&NHX:I=+3310:D=-1984:G=Fabids],Cl:0.338644[&&NHX:I=+794:D=-3735:G=Fabids]):0.022063[&&NHX:C=5]):0.019111[&&NHX:C=6]):0.011710[&&NHX:C=7]):0.029914[&&NHX:C=8],Vv:0.169100[&&NHX:I=+1522:D=-3045:G=Vitales]):0.241611[&&NHX:C=9],Mg:0.241611[&&NHX:I=+2278:D=-1909:G=Asterids])[&&NHX:C=10];"
tree <- read.nhx(textConnection(treetext))
```

2. treeio导入树文件
treeio支持解析大部分软件输出的树文件，包括RAxML(read.raxml)，iqtree，BEAST(read.beast)，MrBayes(read.mrbayes)，PAML(BASEML,CODEML)(read.paml_rst,read.codeml,read.codeml_mlc)，r8s(read.r8s)，MEGA(read.mega)，HyPhy(read.hyphy,read.hyphy.seq)，Phylip(read.phylip)，以及Newick树格式(read.newick)，NHX格式树(read.nhx)，Jplace格式树（EPA，PPLACER,read.jplace），jtree格式(read.jtree)等。

```R
library(treeio)
file <- system.file("extdata/BEAST", "beast_mcc.tree", package="treeio")
tree <- read.beast(file)
```

```R
library(treeio)
tree <- read.newick("sample.tre") #读取newick格式的树文件
```

### 2.2.2. ggtree画树
ggtree函数是ggplot()的扩展，ggtree()相当于`ggplot() + geom_tree() + xlab(NA) + ylab(NA) + theme_tree()`的简单组合。ggplot2可添加的图层都可以直接应用于ggtree。
#### 2.2.2.1. 基础画树

```R
library(ggtree)
ggtree(tree) #默认参数画树
```

#### 2.2.2.2. 添加注释和美化
##### 2.2.2.2.1. 基础注释
1. 简单设置

```R
ggtree(tree)
+ geom_tiplab(color="black",size = 4,hjust = -0.2) \ #添加物种名
+ geom_point(color="#6FE1F8", size=5, alpha=0.7) \ #在节点处添加圆点
+ geom_text(aes(label=node),size=3) \ #在节点处添加数字标签，也可以用这个来确定每个节点是几号节点(节点的node号)
+ geom_highlight(node=15,fill="firebrick",alpha=0.6) #高亮15号节点
```

2. 常用设置

```R
ggtree(tree, size=0.8, branch.length='none', mrsd="2021-10-01") \ #画树，size指定枝的粗细，branch.length指定有无枝长信息，mrsd指定采样日期用于画时间尺度轴
+ geom_tiplab(color="black", size = 4, hjust = -0.1, align=T, linesize=0.7) \ # 末端节点标上物种，size文字大小，hjust是离进化枝线条的距离，align=T文字间左对齐，linesize设置虚线尺寸。
+ geom_point(color="#6FE1F8", size=5, alpha=0.7) \ #节点标记圆点，配色和透明度
+ geom_text(aes(label=node),size=3) \ #节点标记上数字，尺寸
+ geom_highlight(node=15,fill="firebrick",alpha=0.6) #高亮指定分支（节点node所在分支）
+ geom_label2(aes(x=0.79,label=D),color="black",vjust=-0.2,fill="#7FFF00",alpha=0.6) #做标记,aes里的x指定位置，label指定内容
+ geom_strip(9,10,label="Cercidoideae",offset = 0.06,offset.text = 0.012,color = "#FF6100",barsize = 1.5,fontsize = 5,angle = 90,hjust = 0.5) #用条带指定node9和node10之间的分支clade，barsize指定条带尺寸，label指定文字内容，fontsize指定文字尺寸，hjust指定文字位置
+ geom_range("length_0.95_HPD", color='red', size=2, alpha=.5) #在节点上画长条形的分歧时间的95%置信范围
+ xlim(NA,1) #如果内容超出边界，就用xlim拓展显示x轴的范围
```

3. 其他设置

```R
ggtree(tree) + scale_x_reverse() #反转x轴
ggtree(tree) + coord_flip() #置换x轴和y轴
ggtree(tree) + 
```

##### 2.2.2.2.2. 可视化数据并与系统发育树关联 —— facet_plot
可能有不同的数据类型，并希望将它们可视化并与树关联对齐，可用facet_plot函数来处理。

可视化图的类型：点图dotplot/直方图barplot/堆叠图stacked barplot/热图/箱线图boxplot。

ggtree中定义了操作符%<+%，来添加数据。添加之后，用户的数据对ggplot是可见的。可以用于树的注释。

1. 可视化点图dotplot —— geom_point

```R
tr <- rtree(30) # 用treeio的rtree函数生成30个tip的随机树
p <- ggtree(tr)
d1 <- data.frame(id=tr$tip.label, location=sample(c("GZ", "HK", "CZ"), 30, replace=TRUE)) #从("GZ", "HK", "CZ")中随机生成30个采样地点location数据
p1<- p %<+% d1 + geom_tippoint(aes(color=location)) #给树的端点根据location着色

d2 <- data.frame(id=tr$tip.label, val=rnorm(30, sd=3)) #生成两列数据，分别是tip.label和随机生成的30个sd为3的数据值，均值mean默认为0
p2 <- facet_plot(p1, panel="dot", data=d2, geom=geom_point, aes(x=val), color='firebrick') + theme_tree2() #用ggtree的facet_plot的geom_point函数画点图
```

**p2 dotplot**
<img src="[url](https://guangchuangyu.github.io/blog_images/Bioconductor/ggtree/facet_plot_dot2.png)" title="facet_plot_p2.png" width="80%" height="80%" />

2. 可视化堆叠直方图stacked barplot —— geom_barh
大多ggplot2的geom绘制垂直版本的图形对象，通过ggstance包可以制作水平版本的数据图geom，方便画直方图/堆叠直方图/热图/箱线图与系统发育树对应。
包括geom_barh()，geom_histogramh()，geom_linerangeh()，geom_pointrangeh()，geom_errorbarh()，geom_crossbarh()，geom_boxploth()，geom_violinh()。

```R
library(ggstance)
d3 <- data.frame(id = rep(tr$tip.label, each=2),
					value = abs(rnorm(60, mean=100, sd=50)),
					category = rep(LETTERS[1:2], 30)) #随机生成3列数据，分别是tip.label,随机值,代表category的A/B
p3 <- facet_plot(p2, panel = 'Stacked Barplot', data = d3, 
				geom = geom_barh, #用geom_barh画水平版本的堆叠直方图
				mapping = aes(x = value, fill = as.factor(category)), 
				stat='identity' ) 
```

**p3 barplot**
<img src="[url](https://guangchuangyu.github.io/blog_images/Bioconductor/ggtree/facet_plot_bar2.png)" title="facet_plot_p3.png" width="80%" height="80%" />

3. 可视化箱线图boxplot —— geom_boxploth
```R
d4 = data.frame(id=rep(tr$tip.label, each=20), 
				val=as.vector(sapply(1:30, function(i) 
								rnorm(20, mean=i)))
				)				
p4 <- facet_plot(p3, panel="Boxplot", data=d4, geom_boxploth, 
			mapping = aes(x=val, group=label, color=location)) #用geom_boxploth画水平版本的箱线图
```

**p4 boxplot**
<img src="[url](https://guangchuangyu.github.io/blog_images/Bioconductor/ggtree/facet_plot_boxplot2.png)" title="facet_plot_p4.png" width="80%" height="80%" />

#### 2.2.2.3. 各种类型的树
##### 2.2.2.3.1. 时序树(chronogram)
画时间标尺，在ggtree里指定mrsd参数为最近的采样日期，用于画时间尺度轴。

```R
ggtree(tree, mrsd="2022-01-01")+ theme_tree2() #mrsd代表most recent sampling date,指定采样时间，指定mrsd参数后会添加时间标尺
 
tree <- read.mega(system.file("extdata/MEGA7", "mtCDNA_timetree.nex", package = "treeio")) #读取系统自带的树
ggtree(tree) + geom_range('reltime_0.95_CI', color='red', size=3, alpha=.3, center='reltime') + scale_x_range() + theme_tree2() #用geom_range显示时间节点的95%置信区间，'reltime_0.95_CI'，让中心点位于评估的时间点'reltime'，scale_x_range()添加x轴尺度
```

##### 2.2.2.3.2. 系统发育网 —— geom_taxalink
系统发育网一般是在系统发育树绘制的基础上添加物种间的杂交和基因流关系。

```R
ggtree(tree) + geom_tiplab()
+ geom_taxalink('A', 'E') #在tipA和E之间添加关联线，默认是黑色实线。
+ geom_taxalink('F', 'K', color='red', linetype = 'dashed', arrow=grid::arrow(length=grid::unit(0.02, "npc"))) #在tipF和K之间添加关联线，红色虚线，并添加F到K的箭头，箭头大小为0.02npc。
```

##### 2.2.2.3.3. 多棵图共同展示
1. multiPhylo图
多棵树并列展示。

```R
trees <- lapply(c(10, 20, 40), rtree) #随机生成3棵树，节点分别为10，20，40
class(trees) <- "multiPhylo"
ggtree(trees) + facet_wrap(~.id, scale="free") + geom_tiplab() #分面展示3棵树

btrees = read.tree(system.file("extdata/RAxML", "RAxML_bootstrap.H3", package="ggtree")) #读取系统的100棵树
ggtree(btrees) + facet_wrap(~.id, ncol=10) #分面展示100棵树，每行10棵。
```

2. DensityTree图
多棵树重叠展示。

```R
trees <- read.tree(system.file("extdata/RAxML", "RAxML_bootstrap.H3", package="treeio")) #读取系统自带树文件RAxML_bootstrap.H3，包含100棵树。
ggdensitree(trees, alpha=.3, colour='steelblue') + geom_tiplab(size=3) + xlim(0, 45) #重叠多棵树。bootstrap值高的线条一致性，低的线条较乱。
```

#### 2.2.2.4. 一个生成树的例子

```R
library(treeio)
library(ggtree)

treetext<-"((((Mc:0.418672[&&NHX:I=+5728:D=-1170:G=Malvids],Eg:0.201388[&&NHX:I=+1420:D=-2519:G=Malvids])[&&NHX:C=1]:0.148853,((Pt:0.219587[&&NHX:I=+5355:D=-1003:G=Fabids],(Cs:0.209405[&&NHX:I=+1026:D=-3516:G=Malvids],(Gr:0.224188[&&NHX:I=+4875:D=-998:G=Malvids],At:0.492461[&&NHX:I=+1776:D=-2891:G=Malvids]):0.027774[&&NHX:C=2]):0.012564[&&NHX:C=3]):0.019343[&&NHX:C=4],(Pp:0.194515[&&NHX:I=+1159:D=-2949:G=Fabids],(Mt:0.354580[&&NHX:I=+3310:D=-1984:G=Fabids],Cl:0.338644[&&NHX:I=+794:D=-3735:G=Fabids]):0.022063[&&NHX:C=5]):0.019111[&&NHX:C=6]):0.011710[&&NHX:C=7]):0.029914[&&NHX:C=8],Vv:0.169100[&&NHX:I=+1522:D=-3045:G=Vitales]):0.241611[&&NHX:C=9],Mg:0.241611[&&NHX:I=+2278:D=-1909:G=Asterids])[&&NHX:C=10];"
tree <- read.nhx(textConnection(treetext))

p1 <- ggtree(tree,size=0.8) #画树，size指定线条粗细
+ geom_tiplab(color="black",size = 4.5,hjust = -0.1, align=TRUE, linesize=0.7) #添加tiplab-物种名
+ geom_nodepoint(color="#DCDCDC", size=6, alpha=0.9) #节点处添加圆点，指定颜色灰色和尺寸6
+ geom_nodelab(aes(label=C), size=3.5, hjust = 1) #在注释了标签C的节点处添加C的值，这里的标签C用于注释节点序号
+ geom_label2(aes(x=0.79,label=D),color="black",vjust=-0.2,fill="#7FFF00",alpha=0.6) # 在注释了D的位置添加D的值，这里的标签D用于注释收缩基因的数量，x的值用于指定添加位置。
+ geom_label2(aes(x=0.73,label=I),color="black",vjust=-0.2,fill="#FF7D40",alpha=0.6) # 在注释了I的位置添加I的值，这里的标签I用于注释扩张基因的数量，x的值用于指定添加位置。
+ xlim(NA,1.1) # 如果物种名或者其他标签超出显示范围，用xlim增大x轴显示。

p2<- ggtree::rotate(p1,13) #旋转节点13的上下分支clade
p2<- p1 %>% rotate(node=14) %>% rotate(node=16) # 旋转节点14和节点16

p2+geom_strip(10,10,label="Vitales",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5)+geom_strip(3,7,label="Fabids",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5)+geom_strip(1,4,label="Fabids",offset = 0.14, offset.text =0.01,color = "#D2691E",barsize = 1.5,fontsize = 5) # 给图加上分支clade的条带和文字注释
```

# 3. reference
1. [ggtree paper](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.12628)
2. https://yulab-smu.top/treedata-book/index.html
3. [facet_plot](https://guangchuangyu.github.io/2016/10/facet_plot-a-general-solution-to-associate-data-with-phylogenetic-tree/)
4. [不同树的绘制](https://www.codenong.com/js9fda2b82cedb/)