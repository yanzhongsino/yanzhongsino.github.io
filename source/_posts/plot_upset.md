---
title: 集合可视化 —— upset图
date: 2021-09-29 15:56:36
categories:
- plot
- R
tags:
- plot
- R
- UpSetR
description: 记录了用R包UpSetR制作upset图展示多集合共享关系的方法。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

# 1. 多集合展示UpSet图
韦恩Venn图和UpSet图都可用于展示多集合直接的共享关系。当集合数量少于等于5个时，多用韦恩图，当集合数量大于5个时，多用upset图。

## 1.1. 应用场景
1. 利用orthofinder找到的不同物种的orthogroups的结果来绘制Venn Diagram韦恩图或者UpSet plot图，查看不同物种间共享的orthogroups的数量关系。

## 1.2. 数据准备
UpSetR的输入文件是表格形式，第一列是数据分类信息，后面列是每一个集合占一列（每个物种占一列），表格内容是1/0表示集合在每一个类别是否有数据。

### 1.2.1. UpSetR自带数据
```
movies <- read.csv(system.file("extdata", "movies.csv", package = "UpSetR"), header = T, sep=";") # 载入UpSetR包提供的电影数据用作示例。第一行是电影的类别名称，第一列是电影名，第二列是电影上映年度，后面列中的1/0是代表电影是否属于相应的类别。
mutations <- read.csv(system.file("extdata", "mutations.csv", package = "UpSetR"), header = T, sep = ",") # 载入UpsetR包提供的突变数据用作示例。第一行是不同的基因简称，第一列是基因特征ID，数据中的1/0是代表基因是否有对应的特征。

head(movies) # 大致浏览一下该数据集,数据集太长，就只看前几列
View(movies) # 弹出窗口，可查看数据。

head(mutations) # 大致浏览一下该数据集,数据集太长，就只看前几列
View(mutations) # 弹出窗口，可查看数据。
```

<img src="movies.png" width=50% height=50% title="movies" align=center/>
<br>
<img src="mutations.png" width=50% hseight=50% title="mutations" align=center/>

### 1.2.2. orthofinder数据
从orthofinder的结果文件Results_Aug14/Orthogroups/Orthogroups.GeneCount.tsv稍加处理就可以作为输入文件，展示不同物种的orthogroups集合的共享情况。

`sed -E "s/\t[1-9][0-9]*/\t1/g" Orthogroups.GeneCount.tsv >orthogroups.upset` # 把Orthogroups.GeneCount.tsv中的非零数字替换成1。

## 1.3. UpSetR包安装
```
install.packages("UpSetR"); # 安装
library(UpSetR); # 载入UpSetR
require(ggplot2); require(plyr); require(gridExtra); require(grid); # 载入包
```

## 1.4. UpSetR包使用
### 1.4.1. upset函数
1. `upset(mutations)`可以看到upset图的效果

2. 调整参数做指定数据显示

```
upset(mutations, 
sets = c("MUC16","EGFR","TP53","TTN"),# 查看特定的几个集合/几种电影类别
nset = 4, # 最多展示多少个集合数据
nintersects = 20, # 展示数量多的前多少个交集
mb.ratio = c(0.55, 0.45), # 控制上方条形图以及下方点图的比例
order.by = c("freq", "degree"), # 交集如何排序，这里先根据freq，然后根据degree
keep.order = TRUE, # keep.order按照sets参数的顺序排序
decreasing = c(TRUE,FALSE), # 变量如何排序；对应order.by参数。这里表示freq降序，degree升序。
number.angles = 30, # 调整柱形图上数字角度
point.size = 2, line.size = 1, # 点和线的大小
mainbar.y.label = "Genre Intersections", sets.x.label = "Movies PerGenre", # 坐标轴名称
text.scale = c(1.3, 1.3, 1, 1, 1.5, 1)) # 六个数字，分别控制c(intersectionsize title, intersection size tick labels, set size title, set size ticklabels, set names, numbers above bars)
```

<img src="upset_mutations.png" width=50% height=50% title="upset_mutations" align=center/>

### 1.4.2. queries参数
upset函数中可以添加queries参数，用于突出显示（上色）部分数据。
1. queries
queries是一个由多个query组成的list；每个query也是一个list，作为一次查询数据并突出显示的请求。

2. query
query也是list格式，由查询函数query和其他参数（param,color,active,query.name）组成，其中query,param是必须设置的参数。
    - query: 指定查询函数，UpSetR有内置(比如intersects)，也可以自定义函数后调用。
    - param: list格式, 指定query作用的数据。
    - color：设置颜色，可选设置。
    - active：显示类型，TRUE/T表示用颜色覆盖条形图，FALSE/F表示在条形图顶端显示三角形。
    - query.name：添加query图例名称。

3. queries参数示例

- 把"EGFR"和"TP53"两个基因共同拥有的突变标上蓝色；把"TTN"基因特有的突变在直方图上标为红色。

```
upset(mutations, sets=c("MUC16","EGFR","TP53","TTN"), 
queries = list(list(query = intersects,  
params = list("EGFR", "TP53"), # 指定作用的数据
color = "blue", # 设置颜色，未设置会调用默认调色板
active = T,   # 条形图被颜色覆盖
query.name = "share EGFR and TP53"), # 添加query图例
list(query = intersects, params=list("TTN"), color="red", active=T)))
```

<img src="upset_movies.png" width=50% height=50% title="upset_movies" align=center/>

- 把同属Drama和Thriller的电影突出显示，把1970-1980的电影标红。
```
between <- function(row, min, max){
  newData <- (row["ReleaseDate"] < max) & (row["ReleaseDate"] > min)
} # 自定义between函数

upset(movies, sets=c("Drama","Comedy","Action","Thriller","Western","Documentary"),
      queries = list(list(query = intersects, params = list("Drama", "Thriller")),
                     list(query = between, params=list(1970,1980), color="red", active=TRUE)))
```

<img src="upset_movies2.png" width=50% height=50% title="upset_movies2" align=center/>

### 1.4.3. 添加属性图
1. 添加箱线图
每次最多添加两个箱线图
`upset(movies, boxplot.summary = c("AvgRating", "ReleaseDate")) `

<img src="upset_boxplot.png" width=50% height=50% title="upset_boxplot" align=center/>

#### 1.4.3.1. attribute.plots参数
attribute.plots参数用于添加属性图，内置有柱形图，散点图，热图等。
如果想添加密度曲线图，可以自定义plot函数后添加。

1. 添加柱形图和散点图
```
upset(movies, sets=c("Drama","Comedy","Action","Thriller","Western","Documentary"),
      queries = list(list(query = intersects, params = list("Drama", "Thriller")),
                     list(query = between, params=list(1970,1980), color="red", active=TRUE)),
      attribute.plots=list(gridrows=60, # 添加属性图
      plots=list(
        list(plot=scatter_plot, # 散点图
        x="ReleaseDate", y="AvgRating", # 指定横纵坐标
        queries = T), # T表示显示queries定义的颜色
        list(plot= histogram, x="ReleaseDate", queries = F)), # 直方图
        ncols = 2), # 添加的图分两列
      query.legend = "top") # query图例放在上方
```

<img src="upset_scatter_histogram.png" width=50% height=50% title="upset_scatter_histograms" align=center/>

2. 添加密度曲线图

```
#自定义密度曲线
another.plot <- function(data, x, y) {
    data$decades <- round_any(as.integer(unlist(data[y])), 10, ceiling)
    data <- data[which(data$decades >= 1970), ]
    myplot <- (ggplot(data, aes_string(x = x)) + geom_density(aes(fill = factor(decades)), 
        alpha = 0.4) + theme(plot.margin = unit(c(0, 0, 0, 0), "cm"), legend.key.size = unit(0.4, "cm")))
}
```

```
library(plyr)
upset(movies, main.bar.color = "black", mb.ratio = c(0.5, 0.5), queries = list(list(query = intersects, 
    params = list("Drama"), color = "red", active = F), list(query = intersects, 
    params = list("Action", "Drama"), active = T), list(query = intersects, 
    params = list("Drama", "Comedy", "Action"), color = "orange", active = T)), 
    attribute.plots = list(gridrows = 50, plots = list(list(plot = histogram, 
        x = "ReleaseDate", queries = F), list(plot = scatter_plot, x = "ReleaseDate", 
        y = "AvgRating", queries = T), list(plot = another.plot, x = "AvgRating", 
        y = "ReleaseDate", queries = F)), ncols = 3))
```

<img src="upset_density.png" width=50% height=50% title="upset_density" align=center/>

# 2. references
https://github.com/hms-dbmi/UpSetR
https://www.jianshu.com/p/324aae3d5ea4
https://zhuanlan.zhihu.com/p/35303590
