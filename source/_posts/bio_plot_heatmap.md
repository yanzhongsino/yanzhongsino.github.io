---
title: 热图的简介和绘制
date: 2022-11-06
categories:
- bio
- plot
tags:
- R package
- heatmap
- pheatmap
- ggplot2

description: 记录了热图基本概念，热图的分类，以及用R包pheatmap和ggplot2绘制网格热图的代码。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=105140&auto=1&height=32"></iframe></div>


# 1. 热图(heatmap)
热图(heatmap)是在两个维度用不同颜色展示一种现象的大小的一种数据可视化技术。有两种本质上不同的热图，网格热图(grid heatmap)和空间热图(spatial heatmap)。

1. 网格热图(grid heatmap)
网格热图用不同颜色代表不同数值的二维矩形图，同时两个维度还可以通过颜色进行聚类。

比如不同物种在不同基因上的表达水平。

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/48/Heatmap.png/1024px-Heatmap.png" width=90% title="heatmap集群热图" align=center/>

**<p align="center">Figure 1. heatmap集群热图</p>**

2. 空间热图(spatial heatmap)
空间热图将空间现象的大小通过颜色投射到地图上。

比如世界的温度分布图。

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e7/World_heat_map.png/1024px-World_heat_map.png" width=90% title="heatmap空间热图" align=center/>

**<p align="center">Figure 2. heatmap空间热图</p>**

3. 等值线图(choropleth map)
此外，还有与空间热图相似的，常用于地理上可视化的等值线图(choropleth map)。等值线图按照地理边界分组，如国家，州，省或者人为划分的植被区和气候区等，常常具有不规则边界。

比如美国各州人口密度图。

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/34/U.S._states_and_territories_by_population_density.svg/1024px-U.S._states_and_territories_by_population_density.svg.png" width=90% title="heatmap空间热图" align=center/>

**<p align="center">Figure 3. choropleth map等值线图</p>**

# 2. 绘制热图
这篇文章记录的是绘制网格热图的方法，包括常用的R包pheatmap和ggplot2。

## 2.1. pheatmap包
1. 输入数据(gene_expression.txt)的格式
- 第一行标题行
- 第一列是基因名称
- 第二列是数值（比如表达量）
- 可以是一组数据，则只有第二列；也可以是多组数据，依次是第二列及之后列。

```
tf ath_1 ath_2 egr_1 egr_2
AP2 1.0043 0 0.942273 0
ARF 1.20516 1.58619 1.13073 1.48129
```

2. 读取数据和聚类
```R
df<-read.table("gene_expression.txt",sep= " ", header = T,row.names = 1)
df_row <- hclust(dist(df)) #对行聚类
df <- df[df_row$order,] #按行聚类结果排序
df_column <- hclust(dist(t(df))) #对列聚类
df <- df[,df_column$order] #按列聚类结果排序
```

3. 绘制热图
```R
BiocManager::install("pheatmap")
library(pheatmap)
pheatmap(mat=df,color = colorRampPalette(c("lightgreen", "yellow","orange","red"))(20),legend_breaks = c(1:4), legend_labels = c("1.0","2.0","3.0","4.0"), border_color="white",treeheight_row = 50, treeheight_col = 8, display_numbers = TRUE, number_color = "black",main = "TF heatmap",cellwidth = 50, cellheight = 10)
# 其中color = colorRampPalette(c("lightgreen", "yellow","orange","red"))(20) #设置颜色渐变，值从低到高依次是浅绿色-黄色-橙色-红色，共20个颜色。
```

4. 更完整的pheatmap参数

```R
pheatmap(mat = mat, # 表达矩阵
               scale = "row", # 数据标准化方法（col/row/none）
               clustering_method = "complete", # 聚类方法（ward/ward.D/ward.D2/single/complete/average/mcquitty/median/centroid）
               clustering_distance_rows = "euclidean", # 行距离度量（correlation/euclidean）
               clustering_distance_cols = "euclidean", # 列距离度量（correlation/euclidean）
               cluster_cols = TRUE,cluster_rows = TRUE, # 行/列聚类（TRUE/FALSE）
               treeheight_col = 35,treeheight_row = 45, # 行/列聚类树高度
               annotation_col = coldata, # 分组矩阵（可不提供）
               cutree_cols = 2, # 列等分为2
               cutree_rows = 2, # 行等分为2
               show_colnames = TRUE,show_rownames = TRUE, # 是否显示行/列标签（TRUE/FALSE）
               fontsize_row = 7.5,fontsize_col = 7.5, # 横/纵轴标签大小
               color = colorRampPalette(colors = c("blue","white","red"))(100), # 颜色设置
               main = paste("Cluster Heatmap of",nrow(mat),"features",sep = " "), # 标题设置
               filename = "heatmap.pdf", # 输出文件设置
               width = 7,height = 7) # 图片长宽设置
```

## 2.2. ggplot2包
这个我还没试过，把https://zhuanlan.zhihu.com/p/464964887提到的代码摘抄在这。

```R
# Step1 根据实际情况确定是否对数据进行标准化
data <- scale(data,center = TRUE,scale = TRUE)

# Step2 数据重排及格式转换
row_clust <- hclust(dist(mat,method = "euclidean"),method = "complete") # 行(特征)聚类
rowInd <- row_clust$order # 行(特征)的顺序
col_clust <- hclust(dist(t(mat),method = "euclidean"),method = "complete") # 矩阵转置,列(样本)聚类
colInd <- col_clust$order # 列(样本)的顺序
mat <- mat[rowInd,colInd] # 将数据按照聚类结果重新排序
melt_mat <- melt(mat) # 融合数据,使之适用ggplot
colnames(melt_mat) <- c("Feature","Sample","Value")

# 聚类方法：ward.D/ward.D2/single/complete/average/mcquitty/median/centroid（后两种可能会出现边为负的情况）
# 距离度量：euclidean/maximum/manhattan/canberra/binary/minkowski

# Step3 绘制聚类树
h <- ggtree(row_clust,layout = "rectangular",branch.length = "none") # 行聚类树
v <- ggtree(col_clust)+layout_dendrogram() # 列聚类树

# Step4 绘制热图
p.ggplot <- ggplot(data = mat_new,aes(x = Sample,y = Feature,fill = Value))+
  geom_tile()+
  theme_minimal()+
  scale_fill_gradient2(low = "blue",high = "red",mid = "white",name = "Expression")+
  scale_y_discrete(position = "right")+
  labs(x = "",y = "")+
  theme(axis.title = element_text(size = 15),
        axis.text.y = element_text(size = 10),axis.text.x = element_text(angle = 90,size = 11),
        legend.title = element_text(size = 15),legend.text = element_text(size = 11))

# Step5 绘制分组信息条形图（可不提供）
p.group <- ggplot(data = coldata,aes(x = Sample,y = Condition,fill = Group))+
  geom_tile()+
  scale_y_discrete(position = "right")+
  theme_minimal()+
  theme(axis.text = element_blank(),axis.title = element_blank(),
        legend.title = element_text(size = 15),legend.text = element_text(size = 11))+
  labs(fill = "Group")

# Step6 拼接图
p.all <- p.ggplot %>% insert_left(h,width = 0.15) %>% insert_top(p.group,height = 0.05) %>% insert_top(v,height = 0.1)
```

# 3. references
1. https://en.wikipedia.org/wiki/Heat_map
2. https://zhuanlan.zhihu.com/p/464964887

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>