---
title: 用Hifiasm组装基因组：（四）用Hifiasm软件组装基因组的操作
date: 2024-06-22
categories: 
- omics
- genome
- genome assembly
tags:
- genome assembly
- third generation sequencing
- TGS
- Hifiasm
- HiFi
- PacBio
- Hi-C

description: 基于三代HiFi reads用软件Hifiasm对非模式生物进行基因组的从头组装的详细讲解。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=2153862323&auto=1&height=32"></iframe></div>

# 1. Hifiasm基本用法
## 1.1. 安装Hifiasm
1. git安装

```shell
git clone https://github.com/chhylp123/hifiasm
cd hifisam && make
```

2. conda安装
- `conda install -c bioconda hifiasm`

## 1.2. Hifiasm的基础命令
1. 命令
- `nohup hifiasm -o sample_prefix -t 32 --h1 sample_HiC_1.fq.gz --h2 sample_HiC_2.fq.gz Hifi.fastq.gz 2>&1 > hifiasm.log &`
2. 参数
- -o指定输出文件前缀；-t指定线程；--h1和--h2指定Hi-C数据；后接HiFi数据。
- 用`2>&1 >hifiasm.log`保存日志和报错内容到hifiasm.log文件。这个log文件用于后续判断是否需要调整参数。
3. 运行时效
- 300Mb基因组，48线程，组装耗时3.5小时。

## 1.3. Hifiasm的输出文件
### 1.3.1. 输出文件
1. Hifiasm生成gfa格式的组装图（assembly graphs），常见输出文件：
- prefix.r_utg.gfa: haplotype-resolved raw unitig graph. 这个组装图保留了所有单倍型信息。
- prefix.p_utg.gfa: haplotype-resolved processed unitig graph without small bubbles. 小bubbles可能是体细胞突变或者数据污染造成，不是真实的单倍型信息。Hifiasm 会基于coverage自动清除那样的小bubbles。参数--hom-cov会影响这个结果。参数-p强制清除bubbles。
- prefix.p_ctg.gfa: assembly graph of primary contigs. This graph includes a complete assembly with long stretches of phased blocks. 主要contigs的组装图。
- prefix.a_ctg.gfa: assembly graph of alternate contigs. This graph consists of all contigs that are discarded in primary contig graph. 可选contigs的组装图。
- prefix.hap_n.p_ctg.gfa: phased contig graph. 分型contig图。
2. Hifiasm生成的其他格式文件
- prefix.ec.bin：二进制文件，保持了纠错reads。
- prefix.ovlp.source.bin 和 prefix.ovlp.reverse.bin：保存了重叠。
3. trio-binning模式还会输出以下文件
- prefix.dip.hap1.p_ctg.gfa：完全分型的父本/单倍型1的contig图，保留了分型的父本/单倍型1的组装。
- prefix.dip.hap2.p_ctg.gfa：完全分型的母本/单倍型2的contig图，保留了分型的母本/单倍型2的组装。
4. Hi-C partition模式还会输出以下文件
- prefix.hic.p_ctg.gfa：primary contigs的组装图。
- prefix.hic.hap1.p_ctg.gfa：完全分型的单倍型1的contig图，每个contig都被完全分型。
- prefix.hic.hap2.p_ctg.gfa：完全分型的单倍型2的contig图，每个contig都被完全分型。
- prefix.hic.a_ctg.gfa：可选（alternate）contigs的组装图。（--primary参数下生成）
5. HiFi only模式才会输出的文件
- prefix.bp.p_ctg.gfa：primary contigs的组装图。
- prefix.bp.hap1.p_ctg.gfa：单倍型1的部分分型的contig图。
- prefix.bp.hap2.p_ctg.gfa：单倍型2的部分分型的contig图。
6. --primary或者-l0参数下输出的文件
- prefix.p_ctg.gfa：primary contigs的组装图。
- prefix.a_ctg.gfa：可选（alternate）contigs的组装图。

对于输出的每个图（graph），Hifiasm也会输出一个简化版本（xx_nnoseq_xx.gfa），这个版本没有易于可视化的序列。低质量区域的坐标会以bed格式写入lowQ.bed文件中。

### 1.3.2. 输出文件格式
Hifiasm输出的gfa格式的组装图文件，大致遵循[GFA 1.0格式规范](https://github.com/GFA-spec/GFA-spec/blob/master/GFA1.md)，但有几个Hifiasm特有的内容。

gfa文件第一列表示序列图的类型（通常有S,L,A几种类型）。
S表示Segment，L表示Link。Hifiasm输出的gfa格式还有一种特有的A类型的行，用来保存用于构建contig/unitig的reads的信息。

1. S (Segment) 行：
- S 行示例：`S ptg000001l   ATTTGG...  LN:i:17404864   rd:i:41`
- 其中第二列，`ptg000001l`是contig/unitig的ID。
- 第三列是序列的碱基，通常比较长。
- 第四列是序列长度。`LN:i:17404864`中LN表示序列长度（segment length）标签，i表示数据类型是整数(int)，17404864表示序列长度。
- 第五列是read覆盖度。`rd:i::41`中rd是read coverage标签，i表示数据类型是整数(int)，41表示read覆盖度值，这个值使用相同的contig/unitig的reads计算而来。
2. A行保存了用于构建contig/unitig的reads的信息。
- A行示例： `A	ptg000001l	0	-	m84075_240209_054407_s3/1115221/ccs	0	13485	id:i:711340	HG:A:a`
- A行的每一列的含义如下：

<table class="docutils align-default">
    <thead>
        <tr class="row-odd">
            <th class="head"><p>Col</p></th>
            <th class="head"><p>Type</p></th>
            <th class="head"><p>Description</p></th>
        </tr>
    </thead>
    <tbody>
        <tr class="row-even">
            <td><p>1</p></td>
            <td><p>string</p></td>
            <td><p>Should be always <code class="docutils literal notranslate"><span class="pre">A</span></code></p></td>
        </tr>
        <tr class="row-odd">
            <td><p>2</p></td>
            <td><p>string</p></td>
            <td><p>Contig/unitig name</p></td>
        </tr>
        <tr class="row-even">
            <td><p>3</p></td>
            <td><p>int</p></td>
            <td><p>Contig/unitig start coordinate of subregion constructed by read</p></td>
        </tr>
        <tr class="row-odd">
            <td><p>4</p></td>
            <td><p>char</p></td>
            <td><p>Read strand: “+” or “-”</p></td>
        </tr>
        <tr class="row-even">
            <td><p>5</p></td>
            <td><p>string</p></td>
            <td><p>Read name</p></td>
        </tr>
        <tr class="row-odd">
            <td><p>6</p></td>
            <td><p>int</p></td>
            <td><p>Read start coordinate of subregion which is used to construct contig/unitig</p></td>
        </tr>
        <tr class="row-even">
            <td><p>7</p></td>
            <td><p>int</p></td>
            <td><p>Read end coordinate of subregion which is used to construct contig/unitig</p></td>
        </tr>
        <tr class="row-odd">
            <td><p>8</p></td>
            <td><p>id:i:int</p></td>
            <td><p>Read ID</p></td>
        </tr>
        <tr class="row-even">
            <td><p>9</p></td>
            <td><p>HG:A:char</p></td>
            <td><p>Haplotype status of read. <code class="docutils literal notranslate"><span class="pre">HG:A:a</span></code>, <code class="docutils literal notranslate"><span class="pre">HG:A:p</span></code>, <code class="docutils literal notranslate"><span class="pre">HG:A:m</span></code> indicate read is non-binnable, father/hap1-specific and mother/hap2-specific, respectively.</p></td>
        </tr>
    </tbody>
</table>

## 1.4. 组装后格式转换
- `awk '/^S/{print ">"$2;print $3}' test.bp.p_ctg.gfa > test.p_ctg.fa`
- 用awk从gfa文件提取主要的contigs（第一列为S的contigs），生成fasta格式的基因组文件。

# 2. 建议这样跑Hifiasm组装二倍体物种
一般来说，使用Hifiasm的默认参数进行组装就可以达到大部分情况的要求，但根据不同样本的情况，可以调整参数。

熟读官方manual后，总结了下面一套跑法，来确定参数和检验组装结果。

1. 先用默认参数跑一遍

`nohup hifiasm -o sample_prefix -t 48 --h1 sample_HiC_1.fq.gz --h2 sample_HiC_2.fq.gz Hifi.fastq.gz 2>&1 > hifiasm.log &`

2. 查看hifiasm.log文件，如果k-mer plot只有一个峰代表是纯合子样本，则加-l0参数关闭purge duplication步骤，再跑一遍。

`nohup hifiasm -o sample_prefix -t 48 -l0 --h1 sample_HiC_1.fq.gz --h2 sample_HiC_2.fq.gz Hifi.fastq.gz 2>&1 > hifiasm.log &`

3. 查看hifiasm.log文件，如果k-mer plot有两个峰代表是杂合子样本。
- 首先，确定纯合子覆盖值（Ho_coverage值，即homozygous read coverage threshold）是否判断正确，接近k-mer图中的较大的峰值Ho_peak值即为判断正确。
- 然后，检查碱基数量是否异常。Hi-C模式下，纯合碱基比杂合碱基要多即为异常，则会错误判断纯合子覆盖度值。
- 如果有问题，需要使用参数--hom-cov指定纯合子覆盖度为纯合峰值（Ho_peak值，比如90）重跑一遍。

`nohup hifiasm -o sample_prefix -t 48 --hom-cov 90 --h1 sample_HiC_1.fq.gz --h2 sample_HiC_2.fq.gz Hifi.fastq.gz 2>&1 > hifiasm.log &`

4. 检查组装结果是否异常，调整参数
- 组装的基因组大小是否符合预期（genome survey估计的或者流式细胞仪测定的基因组大小）：组装的基因组过大或过小通常是由于hifiasm判断错了纯合子覆盖值。检查日志文件，如果判断错了，用参数--hom-cov指定，重跑一遍。
- 分型的两套基因组大小是否相差不大：如果两套基因组差别过大，可能是做purge的程度不够，可以调低-s值（默认0.55）来改善。
- 组装的基因组序列是否足够长：如果序列不够长，片段化明显，则可以尝试增加 -D 和 -N, 会提高重复区域的分辨率，同时也会增加运行时间。
- 是否存在错误组装：如果后续的Hi-C，或者BioNano发现hifiasm组装结果有比较多错误组装，则可以适当降低 --purge-max, -s和 -O。或者设置 -u 关闭post-join 步骤，hifiasm通过该步骤提高组装的连续性。

5. 还有几个Hi-C辅助组装（Hi-C integrated assembly）的参数。调高以下参数可能改善结果，但增加耗时。
- --n-weight：rounds of reweighting Hi-C links。默认是3轮。
- --n-perturb：rounds of perturbation。默认是10000。
- --f-perturb：fraction to flip for perturbation。默认是0.1。
- --l-msjoin：detect misjoined unitigs of >=INT in size。默认是500000，这个参数非常棘手，尽量不调整。

# 3. Hifiasm作者建议
1. Hifiasm默认运行清除单倍型重复（purges haplotig duplications）的步骤（Trio-binning模式默认不运行）。对于纯合基因组（inbred or homozygous genomes），需要用-l0参数禁止运行清除步骤。
2. 旧的HiFi reads可能在两端包含短的接头（adapter）序列。可以用-z20参数修剪掉reads两端的20bp序列。
3. 对于小的基因组，可以使用-f0参数禁止初始过滤步骤（initial bloom filter）来节约内存，这个步骤在程序早期消耗16GB内存。
4. 对于比人基因组（3Gb）大得多的基因组，使用-f38或-f39参数来节约k-mer计算的内存。

# 4. log文件解释（debug）
Hifiasm在log文件（前面生成的hifiasm.log）中打印的几个信息可用于debug，主要是判断Hifiasm是否根据k-mer频数分布图正确判断了纯合子覆盖度值。

1. k-mer频数分布图（k-mer plot）（这与做genome survey时原理一致）
- log文件会先打印一个k-mer plot，如果指定了Hi-C数据，还会再接着打印几轮校正（round 1，2，3，finally）的k-mer plot。我们做debug关注第一个总的k-mer plot即可。
- 对于纯合子样本（homozygous samples），k-mers 频数分布图应该有一个峰（代表纯合reads的频数分布中心），峰值位置k-mer频数用Ho_peak来代表。
- 对于杂合子样本（heterozygous samples），k-mers 频数分布图应该有两个峰。 频数大一点的峰代表纯合reads覆盖和分布中心（峰值位置k-mer频数用Ho_peak代表）；频数小一点的峰代表杂合reads覆盖和分布中心（峰值位置k-mer频数用He_peak代表）。
- 如果出现非典型的单峰或双峰的k-mer频数分布图，那么可能是样本污染导致。
2. 纯合子覆盖度（homozygous coverage）
- log文件会在打印完k-mer plot后，打印一行：`[M::purge_dups] homozygous read coverage threshold: 90.` 
- 这个90是hifiasm确定的纯合子覆盖度值（用Ho_coverage代表）。要检查Ho_coverage值是否接近k-mer plot中确定的纯和峰值Ho_peak值，如果不接近（比如更接近He_peak值）那表明hifiasm错误地确定了纯合子覆盖度值，此时组装的基因组要么太大，要么太小。要用--homo-cov参数设置纯合子覆盖度值为Ho_peak值。
3. 纯合/杂合碱基数量（number of het/hom bases）
- log文件会在打印一行：`[M::stat] # heterozygous bases: 437440353; # homozygous bases: 93698357`
- 分别代表在Hi-C定向组装（Hi-C phased assembly）时，unitig graph中多少碱基是纯合的（homozygous bases），多少碱基是杂合的（heterozygous bases）。
- 对于杂合子样本，通常杂合碱基比纯合碱基数量多。如果log文件中显示纯合碱基数量比杂合碱基数量还多，代表hifiasm错误地确定了纯合子覆盖度值（Ho_coverage），需要用--homo-cov参数设置纯合子覆盖度值为Ho_peak值。



# 5. references
1. hifiasm manual：https://hifiasm.readthedocs.io/_/downloads/en/latest/pdf/
2. http://xuzhougeng.com/archives/assemble-hifi-pacbio-with-hifiasm

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>