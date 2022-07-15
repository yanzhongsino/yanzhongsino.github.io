---
title: 使用BankIt提交细胞器基因组组装和注释到NCBI
date: 2022-06-30
categories: 
- omics
- organelle
tags:
- mitogenome
- plastome
- submit

description: 记录了上传细胞器基因组的组装和注释数据到NCBI的步骤。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=509092165&auto=1&height=32"></iframe></div>

# 准备上传的文件
1. 基因组fasta文件
2. 注释tbl格式文件

tbl格式文件推荐用GB2sequin来准备。

参考博客-转换GenBank文件为tbl格式：https://yanzhongsino.github.io/2022/06/16/biosoft_fileformat_gb2tbl/。


# 上传步骤
## 上传工具
BankIt：https://www.ncbi.nlm.nih.gov/WebSub/index.cgi

使用BankIt在线上传，允许一次提交多个细胞器基因组序列。

## 上传步骤
1. 选择上传的数据类别。细胞器基因组选择Sequence data not listed above：organelle。下面列几个常用的：
- Eukaryotic and Prokaryotic Genomes(WGS or Complete): 组装好的真核和原核物种的基因组
- Transcriptome Shotgun Assembly (TSA)：组装好的转录组
- Unassembled sequence reads (SRA)：未组装的测序reads
- Sequence data not listed above：mRNA, genomic DNA, organelle, ncRNA, plasmids...：其他测序数据，细胞器基因组选这个。
2. Contact：填写上传人的信息，包括姓名，学院，学校，地址，城市，地区/省份，邮编，国家，和接收上传信息的邮箱【重要】。
3. Reference：填写提供序列的作者和出版信息。
4. Sequencing Technology：填写测序方法信息，包括测序平台，是否是组装的数据，组装软件和版本，组装样品名称，覆盖度。
5. Nucleotide：填写序列的信息。
- 序列发布的时间，可以指定日期，也可以一通过上传审核就发布。
- 分子类型（Molecule Type）：细胞器基因组选的genomic DNA
- 拓扑结构（Topology）：线型分子（Linear）还是环形分子（Circular）。
- 是否是完整的细胞器基因组：yes/no。
- 核苷酸序列格式：fasta或者alignment，选的fasta
- 上传细胞器基因组的fasta文件
6. Organism：填写Organism name信息，可填物种的学名。
7. Submission Category：测序reads是上传者测序的还是使用的其他已上传序列数据。如果是使用其他已上传reads进行的组装则需要提供已上传reads的accession number。
8. Source Modifiers：资源信息。
- Organelle/Location: 叶绿体/线粒体/其他器官。
- Source Modifier可以填写Country；对应的value填写China。
9.  Features：提供注释信息，可以选择tbl文件或者手动填写注释表格，tbl文件的ID和提交的序列ID需要一致。
10. Review and Correct：回顾和确认填写的信息，即可完成提交。

没什么问题的话，两个工作日内会发邮件告知GenBank accession numbers，可用于文章引用。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>