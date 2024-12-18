---
title: 使用BankIt提交细胞器基因组组装和注释到NCBI
date: 2022-06-30
categories: 
- NCBI
- GenBank
- submit
- organelle
tags:
- mitogenome
- plastome
- submit
- tbl
- feature table

description: 记录了上传细胞器基因组的组装和注释数据到NCBI的步骤。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=509092165&auto=1&height=32"></iframe></div>

# 1. 准备上传的文件
1. 基因组fasta文件
2. 注释tbl(feature table file)格式文件
- tbl格式文件是用tab作为列分隔符的五列的文本格式文件。
- 推荐用GB2sequin来准备。
- 参考博客-转换GenBank文件为tbl格式：https://yanzhongsino.github.io/2022/06/16/bioinfo_fileformat_gb2tbl/。
- tbl文件格式：https://www.ncbi.nlm.nih.gov/WebSub/html/help/feature-table.html
- tbl文件的官方定义：https://www.insdc.org/submitting-standards/feature-table/

# 2. 上传步骤
## 2.1. 上传工具
BankIt提交入口：https://www.ncbi.nlm.nih.gov/WebSub/index.cgi

目前好像只支持用BankIt上传细胞器基因组，允许一次提交多个细胞器基因组序列。

提交之后可以在这里查询提交记录：
- BankIt历史提交记录：https://www.ncbi.nlm.nih.gov/WebSub/?tool=genbank&form=history

## 2.2. 上传步骤
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
7. Submission Category：细胞器基因组是上传者首次上传的还是其他已上传数据。如果是首次上传就选Original Submission; 如果是使用其他已上传数据，只是提交新的注释信息，则选择Third Party Annotation (TPA)，并且需要提供已上传数据的accession number。
8. Source Modifiers：资源信息。
- Organelle/Location: 叶绿体/线粒体/其他器官。
- Source Modifier可以填写Country；对应的value填写China。
9.  Features
- 提供基因注释信息。
- 可以选择tbl文件或者手动填写注释表格，tbl文件的ID和提交的序列ID需要一致。
- 最好包含CDS注释信息，否则可能被检查，CDS的注释是包含一个基因的所有exons的位置的，用这样的方式连接所有位置区间：`join(100..200,complement(480..360),520..600)`。
- BankIt会自动识别上传的tbl注释信息是否有错误，比如注释的区间超过了染色体长度，比如中间终止密码子，并给出Error和Warning提示信息，可以根据提示信息检查注释的问题，修正后再次上传。
- 比如错误提示：Error: Intervals specified for this feature extend beyond the sequence length. Please correct your intervals so it is within the sequence length.
10. Review and Correct：回顾和确认填写的信息，即可完成提交。

没什么问题的话，两个工作日内会发邮件告知GenBank accession numbers，可用于文章引用。

如果序列或者注释有问题，会收到通知邮件，简单问题可以回复邮件请GenBank工作人员帮忙修改，复杂问题通常会要求修改后重新提交（resubmit），重新提交的时候可以在最后Review时勾选resubmit的选项，并附上上一次提交的BankIt ID。

# 注释的tbl文件的一些规范
1. tbl文件
- tbl文件格式：https://www.ncbi.nlm.nih.gov/WebSub/html/help/feature-table.html
- tbl文件的官方定义：https://www.insdc.org/submitting-standards/feature-table/
2. 格式转换的脚本
- 参考博客-转换GenBank文件为tbl格式：https://yanzhongsino.github.io/2022/06/16/bioinfo_fileformat_gb2tbl/。
- 参考脚本-转换Excel文件为tbl格式：https://github.com/yanzhongsino/bioscripts/blob/main/my_scripts/convert_excel_to_tbl.py
3. 常见注释方式
- 多个exons的CDS的注释

```
1080    1315    gene
                gene    atp1
1080    1210    CDS
1276    1315
                product atp1
                note    atp1
```

- 基因片段的注释

```
3453    3215    gene fragment
                    gene fragment   atp6*
                    note            atp6*
```

4. 常见报错和修改方法
- `Error: Intervals specified for this feature extend beyond the sequence length. Please correct your intervals so it is within the sequence length.`：注释的区间超过了fasta文件的序列长度，修改错误注释。
- 如果有特殊情况，例如RNA editing或者序列异质性，导致CDS区域不是3的倍数（indels的异质性），可以在note行写上特殊情况说明。参考MZ269392-MZ269412的注释。


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>