---
title: 用RSEM进行转录本定量分析
date: 2024-10-28
categories: 
- omics
- transcriptome
- express
tags:
- expression analysis
- RSEM
- RNA-Seq by Expectation-Maximization
- Simulation
- EBSeq
- Trinity
description: 本文记录软件RSEM（RNA-Seq by Expectation-Maximization）进行转录本定量分析。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1400229069&auto=1&height=32"></iframe></div>

**写在前面**：如果是Trinity组装的转录组，可以直接在Trinity自带脚本`align_and_estimate_abundance.pl`中实现转录本的定量分析，调用RSEM，更方便快捷，参考博客[用Trinity的脚本进行转录本定量分析](https://yanzhongsino.github.io/2024/10/28/omics_transcriptome_expression_Trinity.estimat.abundance/)。

# 1. 常用的转录本定量分析软件
常用的转录本定量分析软件包括RSEM, Salmon, Kallisto, FeatureCounts等。
1. RSEM是基于比对进行的转录本表达定量分析
- 优点是**高精度定量**和**详细的输出指标**。RSEM支持有参和无参，基因和转录本，多种比对软件（Bowtie, Bowtie2, STAR），输出更详细的定量信息（包括reads count, TPM, FPKM等定量指标）。
- 缺点是基于比对，所以运算速度较慢，需要更多计算资源，更适应小规模数据集。
2. Salmon和Kallisto两者都是采用伪比对技术，不需要比对，速度更快。
- 优点是速度快，适合大规模数据集。
- 缺点是对序列的准确性要求更高，输出文件的信息相对RSEM较少，只包含reads count，TPM。
3. FeatureCounts直接从已比对的BAM文件中进行read计数，只适用于有参考基因组的情况，需要自行使用hisat2或bowtie2比对。

# 2. RSEM简介
RSEM（RNA-Seq by Expectation-Maximization）是从RNA-Seq数据定量估计基因和转录本表达水平的软件包。
1. RSEM的特点
- RSEM包提供了一个用户友好的界面，支持并行计算EM算法的线程，单端和双端reads，质量评分，长reads读取和RSPD估计。
- 此外，它还提供了表达水平的后验均值和95%可信区间估计。
- RSEM也有自己的脚本来生成pdf格式的文本阅读深度图。RSEM的独特之处在于，读取深度图可以堆叠，黑色表示对唯一读取的贡献，红色表示对多读取的贡献。
- 此外，从数据中学习到的模型也可以可视化。
2. RSEM包含的功能和模块
- 计算定量表达结果，包括reads count, TPM, FPKM。
- 可视化表达结果，包括bam2gbam，可以生成转录坐标和基因组坐标的BAM和Wiggle文件，Wiggle图。基因组坐标文件可以通过UCSC基因组浏览器和Broad研究所的IGV可视化。转录坐标文件可以通过IGV可视化。
- 建模 (Simulation)：还可以用`rsem-simulate-reads`模块基于参数建模，生成RNA-seq数据。
- 差异表达分析：RSEM推荐用软件EBSeq来进行差异表达分析。常用的差异表达分析工具，如 edgeR 和 DESeq，并没有考虑到reads映射不确定性造成的变异。 EBSeq 是UW-Madison开发的一种经验贝叶斯差异表达分析工具，它可以根据父基因的同工型数量对同工型进行分组，从而将读图映射不确定性造成的变异考虑在内。 此外，它对异常值也有更强的鲁棒性。 有关 EBSeq 的更多信息，请访问 EBSeq 网站：https://www.biostat.wisc.edu/~kendzior/EBSEQ/。RSEM带有EBSeq包，只需安装`make ebseq`即可使用EBSeq。
- 先验增强的RSEM(Prior-Enhanced RSEM, pRSEM)：pRSEM使用更多的数据（如 ChIP-seq 数据，scRNA-seq数据）来分配 RNA-seq 多映射片段。 在 pRSEM 子文件夹以及 RSEM 的 rsem-prepare-reference 和 rsem-calculate-expression 脚本中包含了 pRSEM 代码。

# 3. RSEM安装
1. 依赖
- C++编译器：如GCC。
- Perl：用于运行一些脚本。
- R：用于数据分析和可视化。
- Python（可选）：如果需要使用--gff3选项。
- Bowtie/Bowtie2/STAR/HISAT2（可选）：如果需要使用这些比对工具。
2. RSEM安装
- conda安装：`conda install bioconda::rsem`
- github下载和安装

```shell
git clone https://github.com/deweylab/RSEM.git
cd RSEM
make
make install
```

3. 添加到环境变量
- 添加之后，如果可以使用`rsem-calculate-expression`命令则表示安装成果。

# 4. RSEM使用
## 4.1. 建立转录组的索引文件
1. 有参考基因组
- 提供参考基因组genome.fa序列和GTF注释文件genome.annotation.gtf（或者用`-gff3 genome.annotation.gff3`指定gff3格式的注释文件），RSEM则会通过GTF注释文件从参考基因组序列中提取出各个转录本的序列，然后利用Bowtie2 or STAR等软件来建索引，索引文件前缀ref。
- `rsem-prepare-reference -gtf genome.annotation.gtf --bowtie2 genome.fa ref`
- 生成提取的转录本序列文件，和bowtie2生成的索引文件ref.bt2
2. 无参考基因组 
- 没有参考基因组的情况，可以用Trinity无参组装的转录本文件Trinity.fasta，生成转录组的索引，索引文件前缀Trinity_ref。
- `rsem-prepare-reference --bowtie2 Trinity.fasta Trinity_ref`
- - 用bowtie2生成索引文件Trinity_ref.bt2
3. 其他参数
- `--bowtie2`：指定建立索引文件的软件，可选`--bowtie`,`--bowtie2`,`--star`。
- `--transcript-to-gene-map map.txt`：如果想生成基于基因水平的定量结果，则需要加上这个参数指定包含转录本和基因的对应关系的map.txt文件，文件包含两列gene_id transcript_id。指定这个参数后，除了生成transcript的定量结果，还会生成gene（一个基因有多个转录本）的定量结果。推荐使用。RSEM还提供了脚本从Trinity的结果生成map.txt文件，`extract-transcript-to-gene-map-from-trinity Trinity.fasta map.txt`。

## 4.2. 计算表达量
1. 命令
- `rsem-calculate-expression --bowtie2 --paired-end -p 12 read_1.fastq read_2.fastq Trinity_ref sample_output`
2. 参数
- --bowtie2: 指定使用 Bowtie2 进行比对。
- --paired-end: 数据是PE测序的。
- -p 12: 指定使用 12 个线程进行计算。
- read_1.fastq read_2.fastq: 输入的配对 RNA-Seq 读段文件。
- Trinity_ref: RSEM 参考数据索引文件的前缀，上一步的ref或Trinity_ref。
- sample_output: 指定输出文件的前缀。
3. 常用参数
- --estimate-rspd: 估算每个转录本的reads位置分布，默认启用。estimate the read start position distribution，看是否有positional biases。
- --calc-ci: 计算转录本丰度的置信区间。
- --append-names：用于在结果中附加上gene name和transcript name。
- --seed-length: 设置用于比对的种子长度，默认是 25。
- --output-genome-bam: 输出基于基因的 bam 文件（默认基于转录本的bam文件）。
- --no-bam-output: 不输出 BAM 文件。
- --time: 显示命令执行时间。
4. bowtie2的参数
- RSEM调用bowtie2的参数可以通过--bowtie2-mismatch-rate，--bowtie2-k以及--bowtie2-sensitivity-level来设定。
- 默认参数是：`bowtie2 -q --phred33 --sensitive --dpad 0 --gbar 99999999 --mp 1,1 --np 1 --score-min L,0,-0.1 -I 1 -X 1000 --no-mixed --no-discordant -p 6 -k 200 -x /path/to/index/ -1 RNA-seq.clean_1.fastq -2 RNA-seq.clean_2.fastq | samtools view -S -b -o sample.bam`
- 默认参数中：--gbar表示，RSEM不支持有gap的比对；--no-mixed则表示RSEM不支持单个reads的比对；--no-discordant则告诉我们RSEM也不支持paired reads discordant的比对; -k 200表明RSEM希望bowtie2输出最佳的200个比对结果。
5. 输出文件
- sample_output.genes.results: 包含基因水平的表达定量结果。
- sample_output.isoforms.results: 包含转录本水平的表达定量结果。
- sample_output.transcript.bam: 如果没有使用 --no-bam-output，会生成基于转录本比对的 BAM 文件。
- sample_output.genome.bam: 如果用 --output-genome-bam，会生成基于基因的比对的 BAM 文件。
- sample_output.stat：文件夹，包含一些统计结果。

## 4.3. 一些解释
1. 结果文件sample_output.isoforms.results的每一列
- 第1，2列分别是transcript id和gene id，如果像我一样设置了--append-names参数，那么在ID后面还会附上transcript name和gene name（以下划线_分隔）
- 第3，4列分别是transcript length和effective_length，这里是length是没有包括转录本的poly(A) tail的，而effective_length则是指可以有fragment覆盖的区域，计算公式：transcript length - mean fragment length + 1；如果是.genes.results中的length，则是所有transcript length的加权平均数
- 第4，5，6列则分别是expected_count，TPM，FPKM；expected_count比较复杂，跟一般的count数定义有点不一样，是指the sum of the posterior probability of each read comes from this transcript over all reads，RSEM给出的解释是（复制黏贴了）：Because (1) each read aligning to this transcript has a probability of being generated from background noise; (2) RSEM may filter some alignable low quality reads, the sum of expected counts for all transcript are generally less than the total number of reads aligned；TPM和FPKM则比较常见了
- 第7列则是IsoPct，其含义是转录本上的丰度占其基因上的丰度的百分比。
2. multimapping reads（多重比对reads）的处理
- RSEM官方教程中提到，以Ccl6基因的3个转录本的比对结果为例，其第2个转录本全长与第1个转录本重叠，所以在这重叠区域会产生大量的multimapping reads。RSEM的处理方法是这样的：RSEM先尽量让第1个转录本在其重叠区域的reads（包括unique reads和multimapping reads）分布趋于平滑？因此再将在区域的剩下multimapping reads算在第2个转录本上；至于第3个转录本，其没有reads比对上，所以就空的。

<img src="https://github.com/bli25/RSEM_tutorial/raw/master/images/Ccl6_transcript_wiggle.png" width=80% title="Cc16基因的例子" align=center/>

**<p align="center">Ccl6_transcript_wiggle.pdf</p>**


# 5. references
1. 官方教程：https://deweylab.github.io/RSEM/
2. github：https://github.com/deweylab/RSEM
3. https://www.plob.org/article/22918.html

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" 