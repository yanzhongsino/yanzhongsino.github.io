---
title: title
date: 2021-03-27 16:50:00
categories: 
tags: 
description: 
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=20707476&auto=1&height=32"></iframe><music URL></div>




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
nucmer ref.fa query.fa -p out

生成
- out.delta

## dnadiff
dnadiff -d out.delta

生成
- out.1coords
- out.1delta
- out.mcoords
- out.mdelta
- out.qdiff
- out.rdiff
- out.report
- out.snps


dnadiff ref.fa query.fa

# references
1. https://www.jianshu.com/p/c12f2a117892

