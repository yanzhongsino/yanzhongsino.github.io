---
title: 利用Hi-C数据辅助组装基因组至染色体水平
date: 2022-04-15
categories: 
- omics
- genome
tags:
- Hi-C
- 3D-DNA

description: 记录了用Hi-C数据辅助组装基因组至染色体水平的步骤。
---

<div align="middle"><music URL></div>

利用Hi-C数据辅助组装基因组至染色体水平
pipeline：juicer，3D-DNA，JuiceBox

Juicer,JuiceBox,3D-DNA pipeline

酶切：dpn II

六碱基识别序列限制性内切酶（通常为HindIII）
四碱基识别序列限制性内切酶（通常为MboI／DpnII）

## 准备输入文件
在运行目录下新建两个文件，一个references存放基因组，一个fastq存放Hi-C的测序数据为`*_R1.fastq`和`*_R2.fastq`文件名，如果文件太大，`ln -s`生成文件的软链接也可以。

1. 建立基因组索引
`cd references`-`bwa index genome.draft.fa`-`cd ..`
2. 生成限制位点文件
`generate_site_positions.py HindIII genome ./references/genome.draft.fa`
3. 生成contigs长度文件
`awk 'BEGIN{OFS="\t"}{print $1, $NF}' genome_HindIII.txt > genome.chrom.sizes`

## 运行juicer
1. 运行
`nohup juicer.sh -s HindIII -z ./references/genome.draft.fa -y genome_HindIII.txt -p genome.chrom.sizes -D /path/to/juicer/ -d ./ -t 12 -g genome.draft >juicer.log 2>&1 &`

其中，-z,-y,-p是必须指定的，-z,-y,-p指定输入文件;-s指定酶切位点使用的酶的类型;-t指定线程;-g指定基因组ID;-d指定运行目录，代表储存了Hi-C数据的fastq目录在运行目录下，且结果生成在运行目录下;-D指定juicer的安装目录。

2. 结果文件
最终会生成aligned目录

结果保存在aligned目录下，其中`merged_nodups.txt`即为结果文件，给3D-DNA作为输入文件。

3. 报错处理
如果报错
```
Using genome_HindIII.txt as site file
(-: Looking for fastq files...fastq files exist
***! Move or remove directory ".//aligned" before proceeding.
***! Type "juicer.sh -h " for help
```

就先运行
```
mkdir splits
cd splits
split -a 3 -l 90000000 -d --additional-suffix=_R1.fastq ../fastq/hic_R2.fastq
split -a 3 -l 90000000 -d --additional-suffix=_R2.fastq ../fastq/hic_R1.fastq
cd ..
```


# references
1. juicer github：https://github.com/aidenlab/juicer
2. 3D-DNA github：https://github.com/aidenlab/3d-dna

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>