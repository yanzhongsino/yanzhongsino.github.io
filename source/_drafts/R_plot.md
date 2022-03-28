---
title: 通过R包实现功能
date: 2021-07-28 14:30:00
categories: 

tags:


description: 通过R包实现功能
---

<div align="middle"><music URL></div>

1. R包reshape可以把数据从宽格式(wide format)转成长格式(long format)
```R
library(reshape)
head(wide)

    var1   var2 var3
1   2.46    -0.78   2.31
2   3.15    -0.98   1.02
3   5.26    0.68    0.28
4   2.31    1.34    0.15
5   0.25    2.34    0.29
6   1.36    3.77    2.08

long<- melt(wide)
head(long)

    var1
1   2.46
2   3.15
3   5.26
4   2.31
5   0.25
6   1.36
```