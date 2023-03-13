---
title: 使用Rldeogram的ideogram函数画两物种的同线性图
date: 2022-08-03
categories: 
- bioinfo
- synteny
- plot
tags: 
- R
- R package
- Rldeogram
- ideogram
- synteny
- plot
description: 使用R包Rldeogram的ideogram函数画两物种的同线性图
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=20707476&auto=1&height=32"></iframe><music URL></div>

# 1. 使用Rldeogram的ideogram函数画两物种的共线性图
## 1.1. 输入文件
1. karyotype.txt
- Chr: 染色体号
- Start: 起始
- End: 终止
- fill: 染色体填充色
- species：物种名
- size: 物种名字体大小
- color: 物种名字体颜色

文件示例：
```
Chr Start      End   fill species size  color
1   1 23037639  FF8C00   Grape   12 252525
2   1 18779884  FF8C00   Grape   12 252525
3   1 17934068  FF8C00   Grape   12 252525
4   1 17349521  FF8C00   Grape   12 252525
1   1 22042719  4682B4   Populus   12 252525
2   1 19858802  4682B4   Populus   12 252525
3   1 19278319  4682B4   Populus   12 252525
```

2. synteny.txt
- Species_1：物种1染色体号
- Start_1，End_1：物种1染色体区域位置
- Species_2：物种2染色体号
- Start_2，End_2：物种2染色体区域位置

文件示例：
```
Species_1  Start_1    End_1 Species_2 Start_2   End_2   fill
1   12226377    12267836    1   5900307 5827251 cccccc
1  5635667 5667377 2 4459512  4393226 cccccc
1   7916366 7945659 3 8618518   8486865 cccccc
2   8214553 8242202 1 5964233  6027199 cccccc
3  2330522 2356593 1 6224069  6138821 cccccc
3  10861038    10886821    2  8099058 8011502 cccccc
4  9487312    9540261    3  7657579 7701112 cccccc
```

## 1.2. 运行

```R
install.packages('RIdeogram') #安装RIdeogram
library('RIdeogram') #载入RIdeogram
ka <- read.table("karyotype.txt",sep="\t",header = TRUE,stringsAsFactors = F) #读取karyotype.txt文件
sy <- read.table("synteny.txt",sep="\t",header = TRUE,stringsAsFactors = F) #读取synteny.txt文件
ideogram(karyotype = ka, synteny = sy) #使用ideogram函数，生成chromosome.svg文件用于绘图
convertSVG("chromosome.svg", device = "pdf",dpi=1600) #转化成chromosome.pdf文件，还可选择转化的格式：tiff，png，jpg，分辨率1600。
```

## 1.3. 结果
结果如下图：

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/ideogram.png?raw=true" title="Rldeogram绘制的同线性图" width="80%" />

**<p align="center">Figure 1. Rldeogram绘制的同线性图</p>**

# 2. references
1. https://www.jianshu.com/p/07ae1fe18071

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>