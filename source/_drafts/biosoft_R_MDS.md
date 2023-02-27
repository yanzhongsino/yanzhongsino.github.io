---
title: MDS
date: 2023-02-24
categories:
- bio
- bioinfo
tags:
- R packages
- MSCquartets

description: 
---

<div align="middle"><music URL></div>

### 核质冲突
评估基因树与物种树的不一致程度
1. R包ape计算基因树成对间的Robinson-Foulds（RF）距离
- 载入包`library(ape)`
- 读取树`trees<-read.tree("species_trees.tre")`
- 计算RF`trees_dist<-dist.topo(trees, method = "PH85")`trees_dist是dist类别的数据
- 转化dist到矩阵matrix`trees_matric<-as.matrix(trees_dist)`
- 保存矩阵数据`write.table (trees_matric, file ="RF_pairs.tsv", sep ="\t")`

2. MDS绘制不同维度的RF距离
- 经典MDS法
```R
mds <- cmdscale(trees_dist, eig=TRUE, k=2) #k代表维度
x <- mds$points[,1]
y <- mds$points[,2]
plot(x, y, xlab="Coordinate 1", ylab="Coordinate 2", main="Metric MDS", type="n")
text(mds$points, labels = rownames(trees_matric), cex = 0.6)
```
- nonmetric MDS法
```R
library(MASS)
mds_mass <- isoMDS(trees_dist, k=2) #k代表维度
x1 <- mds_mass$points[,1]
y1 <- mds_mass$points[,2]
plot(x1, y1, xlab="Coordinate 1", ylab="Coordinate 2", main="Nonmetric MDS", type="n")
text(x, y, labels = row.names(trees_matric), cex=.7)
```

- 绘图
```R
mds.var.per <- round(mds$eig/sum(mds$eig)*100,1)
mds.values <- mds$points
mds.data <- data.frame(Sample=rownames(mds.values),
                       X=mds.values[,1],
                       Y=mds.values[,2])
write.table(mds.data,file="mds.tsv",sep="\t") #把MDS的二维值保存到mds.tsv，里面用tree1,tree2,...,treen等依次命名，与species_trees.tre树文件中树的顺序一致。如果在合并了基因树和物种树，可以依据顺序找到物种树。
ggplot(data=mds.data, aes(x=X, y=Y, label=Sample))+
  geom_text() +
  xlab(paste("MDS1 - ", mds.var.per[1], "%", sep=""))+
  ylab(paste("MDS2 - ", mds.var.per[2], "%", sep=""))+
  theme_bw()+
  ggtitle("MDS plot using Euclidean distance")
```

3. 建议
还可以把每个基因树和物种树的支持率的平均值计算出来之后，用支持率给每棵树上色，从而查看是否是离物种树越近的基因树的平均支持率越高。
也可以用基因树的平均支持率和基因树离物种树的RF距离画散点图，并做线性回归分析判断相关性。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>