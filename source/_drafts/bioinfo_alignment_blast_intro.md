---
title: blast
date: 2022-11-12
categories: 
- bioinfo
- blast
tags: 
- 
description: 介绍blast
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=399341112&auto=1&height=32"></iframe></div>


blastn：将核苷酸序列比对至核苷酸数据库。
blastp：将氨基酸序列比对至氨基酸数据库。
blastx：将核苷酸序列比对至氨基酸数据库。
tblastn：将氨基酸序列比对至核苷酸数据库。比对时，将输入的氨基酸序列与数据库中核苷酸序列翻译后的氨基酸序列逐一比对。
tblastx：将核苷酸序列比对至核苷酸数据库。与blastn的区别是比对时，输入的核苷酸序列与数据库中的核苷酸序列都先翻译为氨基酸序列，而后再进行逐一比对。

# blast
参数-max_hsps 1：代表结果中subject ID和query ID完全相同的行中只保留evalue最小的一行

-max_target_seqs和-num_alginments都是设置每个query保留多少条匹配结果，目前来看二者的结果是一样的。
-max_target_seqs设置为1，保留下来的是最佳匹配


makeblastdb -in ref.fa -dbtype prot -out index/ref.pep
blastp -query sample.fa -db index/ref.pep -out out.blast -evalue 1e-5 -num_threads 12 -outfmt 6 -num_alignments 5 &

- sample.fa不能是压缩文件

blast的结果是以HSP（高度相似性片段）作为一行进行输出的。



















# references
1. blast manual：https://www.ncbi.nlm.nih.gov/books/NBK279690/
2. blast参数-max_hsps：https://segmentfault.com/a/1190000015827656
3. blast参数-max_target_seqs和-num_alignments：https://segmentfault.com/a/1190000015831475
4. blast自定义输出项：https://blog.ligene.cn/2017/07/05/BLAST-Tutorial/


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=30% title="wechat_public_QRcode.png" align=center/>