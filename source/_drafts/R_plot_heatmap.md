---
title: 绘制热图
date: 2022-10-18
categories:
- R
- plot
- heatmap
tags:
- heatmap
- pheatmap

description: 
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2117115&auto=1&height=32"></iframe></div>


# 热图
热图是用不同颜色代表不同值的矩形图，还可以通过颜色进行聚类。

# 绘制热图

## pheatmap包

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

## ggplot2包

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

# references
1. https://zhuanlan.zhihu.com/p/464964887

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>