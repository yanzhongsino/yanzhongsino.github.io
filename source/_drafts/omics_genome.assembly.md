---
title: genome assembly
date: 2021-07-16 14:30:00
categories: 
- bio
	- concept
tags:

- crown groups
- stem groups
- fossil time

description: 记录了冠群和干群及在古生物学和进化生物学上应用的相关概念。
---

<div align="middle"><music URL></div>




# pacbio
三代下机数据是subreads.bam,subreads.bam.pbi,subreads.xml三个文件。

subreads.bam与reads比对到参考基因组上生成的bam文件格式一致，但内容有差异。

1. bam2fasta
- conda安装
```
conda install -c hcc smrtlink-tools #安装PacBio官方软件SMRTlink
bam2fasta subreads.bam -o sample #bam2fasta转换成sample.fasta
```

- 下载安装
https://www.pacb.com/support/software-downloads/ 下载最新版的SMRT LINK，目前是V11.0，压缩包有点大(1.8GB)。

```
wget https://downloads.pacbcloud.com/public/software/installers/smrtlink_11.0.0.146107.zip
unzip smrtlink_11.0.0.146107.zip #生成smrtlink_11.0.0.146107.run文件和它的md5文件
./smrtlink_11.0.0.146107.run --smrttools-only #安装smrttools，会在当前目录下生成smrtlink目录
```

`./smrtlink/smrtcmds/bin/`下会有很多命令，包括bam2fasta，samtools，minimap2，falconc等。
`./smrtlink/admin/bin/`下也有一些可用的命令。


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>