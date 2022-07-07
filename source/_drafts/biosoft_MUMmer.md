---
title: 共线性分析软件——MUMmer
date: 2022-07-08
categories: 
tags: 
description: 
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=20707476&auto=1&height=32"></iframe></div>




# install

安装
https://mummer4.github.io/install/install.html


```
wget https://github.com/mummer4/mummer/releases/download/v4.0.0rc1/mummer-4.0.0rc1.tar.gz
tar -xzf mummer-4.0.0rc1.tar.gz
cd mummer-4.0.0rc1
./configure --prefix=/path/to/installation
make
make install
nucmer -h
```


# 使用
## nucmer
`nucmer ref.fa query.fa -p out`

生成
- out.delta

## dnadiff
`dnadiff ref.fa query.fa`

生成
- out.delta
- out.1coords
- out.1delta
- out.mcoords
- out.mdelta
- out.qdiff
- out.rdiff
- out.report
- out.snps

已有delta文件，则可运行

`dnadiff -d out.delta`

# references
1. https://www.jianshu.com/p/c12f2a117892

