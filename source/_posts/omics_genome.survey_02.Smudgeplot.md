---
title: 用k-mer分析进行基因组调查：（二）用Smudgeplot估计倍性
date: 2022-12-31
categories:
- omics
- genome
- genome survey
tags:
- genome
- genome survey
- k-mer
- GenomeScope
- Smudgeplot
- KMC

description: 介绍Smudgeplot，用Smudgeplot估计物种的倍性。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=276895&auto=1&height=32"></iframe> </div>

**用k-mer分析进行基因组调查系列：**
- （一）基本原理：https://yanzhongsino.github.io/2022/05/25/omics_genome.survey_01.intro/
- （二）用Smudgeplot估计倍性：https://yanzhongsino.github.io/2022/12/31/omics_genome.survey_02.Smudgeplot/
- （三）用jellyfish进行k-mer频数统计：https://yanzhongsino.github.io/2022/05/27/omics_genome.survey_03.jellyfish/
- （四）用KMC进行k-mer频数统计：https://yanzhongsino.github.io/2022/06/05/omics_genome.survey_04.KMC/
- （五）用GenomeScope评估基因组特征：https://yanzhongsino.github.io/2022/06/05/omics_genome.survey_05.GenomeScope/
- （六）用GCE分步实现：https://yanzhongsino.github.io/2022/06/07/omics_genome.survey_06.GCE/
- （七）用KmerGenie一步实现：https://yanzhongsino.github.io/2022/06/19/omics_genome.survey_07.KmerGenie/

**【推荐】用Smudgeplot评估物种倍性后，用组合jellyfish+GenomeScope1.0做二倍体物种的基因组调查，用组合KMC+GenomeScope2.0做多倍体物种的基因组调查。**

# 1. Smudgeplot
Smudgeplot是2020年与GenomeScope2.0一起发表的用于估计物种的倍性的软件。开发者计划接下来把Smudgeplot整合进GenomeScope。

# 2. Smudgeplot原理
Smudgeplot从k-mer数据库中提取杂合k-mer对，然后训练杂合k-mer对。

通过比较k-mer对覆盖度的总数(CovA + CovB)和相对覆盖度(CovB / (CovA + CovB))，统计杂合k-mers对的数量，Smudgeplot可以解析基因组结构。

# 3. Smudgeplot安装
1. 依赖
依赖是[tbenavi1/KMC](https://github.com/tbenavi1/KMC)和[GenomeScope2.0](https://github.com/tbenavi1/genomescope2.0)。
- 用于统计k-mers频数的软件。建议[tbenavi1/KMC](https://github.com/tbenavi1/KMC)，里面包括一个smudge_pairs程序，用来找杂合k-mer对。也可以用jellyfish代替KMC，参考[manual of smudgeplot with jellyfish](https://github.com/KamilSJaron/smudgeplot/wiki/manual-of-smudgeplot-with-jellyfish)。
- [GenomeScope2.0](https://github.com/tbenavi1/genomescope2.0)

2. 安装
`conda install -c bioconda smudgeplot` #conda安装

# 4. Smudgeplot使用
## 4.1. KMC+Smudgeplot
### 4.1.1. 用KMC计算k-mer频率，生成k-mer频率表
1. 运行
```
mkdir tmp
ls *.fastq.gz > FILES
kmc -k21 -t8 -m64 -ci1 -cs10000 @FILES kmcdb tmp #计算k-mer频率，生成二进制文件kmcdb.kmc_pre和kmcdb.kmc_suf
kmc_tools transform kmcdb histogram kmcdb_k21.hist -cx10000 #生成k-mer频数直方表kmcdb_k21.hist
rm -rf tmp
```

2. kmc命令参数：
- -k21：k-mer长度设置为21
- -t8：线程8
- -m64：内存64G，设置使用RAM的大致数量，范围1-1024。
- -ci1 -cs10000：统计k-mer coverages覆盖度范围在[1-10000]的。
- @FILES：保存了输入文件列表的文件名为FILES
- kmcdb：KMC数据库的输出文件名前缀
- tmp：临时目录

3. kmc_tools命令参数：
- -cx10000：储存在直方图文件中counter的最大值。

### 4.1.2. 选择覆盖阈值
- 可以目视检查k-mer直方图，选择覆盖阈值上(U)下(L)限。
- 也可以用命令估计覆盖阈值上(U)下(L)限。L的取值范围是[20-200]，U的取值范围是[500-3000]。

```
L=$(smudgeplot.py cutoff kmcdb_k21.hist L)
U=$(smudgeplot.py cutoff kmcdb_k21.hist U)
echo $L $U # these need to be sane values
```

### 4.1.3. 提取阈值范围的k-mers，计算k-mer pairs
1. smudge_pairs
- 用`kmc_tools`提取k-mers，然后用KMC的`smudge_pairs`计算k-mer pairs。
- `smudge_pairs`比`smudgeplot.py hetkmers`使用更少内存，速度更快地寻找杂合k-mer pairs。
  - `kmc_tools transform kmcdb -ci"$L" -cx"$U" reduce kmcdb_L"$L"_U"$U"` 根据L和U过滤，生成二进制文件kmcdb_L10_U680.kmc_pre和kmcdb_L10_U680.kmc_suf
  - `smudge_pairs kmcdb_L"$L"_U"$U" kmcdb_L"$L"_U"$U"_coverages.tsv kmcdb_L"$L"_U"$U"_pairs.tsv > kmcdb_L"$L"_U"$U"_familysizes.tsv`

2. smudgeplot.py
- 如果没有安装KMC，可以用`kmc_dump`提取k-mers。
- 然后用`smudgeplot.py hetkmers`计算k-mer pairs。
- `kmc_tools transform kmcdb -ci"$L" -cx"$U" dump -s kmcdb_L"$L"_U"$U".dump #生成kmcdb_L10_U680.dump`
- `smudgeplot.py hetkmers -o kmcdb_L"$L"_U"$U" < kmcdb_L"$L"_U"$U".dump # 生成kmcdb_L10_U680_coverages.tsv和kmcdb_L10_U680_sequences.tsv，耗时约1h`

### 4.1.4. 生成热度图(smudgeplot)
1. 命令
`smudgeplot.py plot kmcdb_L"$L"_U"$U"_coverages.tsv -o smudgeplot`

- -o指定输出文件前缀，默认smudgeplot
- -t指定图中的title

2. 结果
- 生成两个基础的热度图。一个log尺度smudgeplot_smudgeplot_log10.png；一个线性尺度smudgeplot_smudgeplot.png
- smudgeplot_summary_table.tsv，下面是文件示例：

```
peak	kmers [#]	kmers [proportion]	summit B / (A + B)	summit A + B
AB	10703792	1	0.49	44.2
```

- smudgeplot_verbose_summary.txt，下面是文件示例：

```shell
1n coverage estimates (Coverage of every haplotype; Don't confuse with genome coverage whichis (ploidy * 1n coverage).)
* User defined 1n coverage:
* Subset 1n coverage estimate:	16.4
* Highest peak 1n coverage estimate:	23.1
1n coverage used in smudgeplot (one of the three above):	23.1
* Proposed ploidy:	2
* Minimal number of heterozygous loci:	509705
Note: This number is NOT an estimate of the total number heterozygous loci, it's merly setting the lower boundary if the inference of heterozygosity peaks is correct.
* Proportion of heterozygosity carried by pairs in different genome copies (table)
  genome_copies propotion_of_heterozygosity
1             2                           1
* Proportion of heterozygosity carried by paralogs:	0
* Summary of all detected peaks (table)
  peak kmers [#] kmers [proportion] summit B / (A + B) summit A + B
2   AB  10703792                  1               0.49         44.2
```

- smudgeplot_warnings.txt

## 4.2. jellyfish+Smudgeplot
用jellyfish代替KMC

1. 用jellyfish计算k-mer频率，生成k-mer频率表
```
jellyfish count -C -m 21 -s 1000000000 -t 8 *.fastq -o kmer_counts.jf
jellyfish histo kmer_counts.jf > kmer_k21.hist
```

2. 提取阈值范围的k-mers，计算k-mer pairs
```
L=$(smudgeplot.py cutoff kmer_k21.hist L)
U=$(smudgeplot.py cutoff kmer_k21.hist U)
echo $L $U
jellyfish dump -c -L $L -U $U kmer_counts.jf | smudgeplot.py hetkmers -o kmer_pairs
# note that if you would like use --middle flag, you would have to sort the jellyfish dump first
```

3. 生成smudgeplot污热度图
```
smudgeplot.py plot kmer_pairs_coverages_2.tsv -o my_genome
```

## 4.3. Smudgeplot结果
1. 热度图
- 热度图，横坐标是相对覆盖度 (CovB / (CovA + CovB)) ，纵坐标是总覆盖度 (CovA + CovB) ，颜色是k-mer对的频率。
- 每个单倍型结构都在图上呈现一个"污点(smudge)"，污点的热度表示单倍型结构在基因组中出现的频率，频率最高的单倍型结构即为预测的物种倍性结果。(比如这个图提供了三倍体的证据，AAB的频率最高)

<img src="https://user-images.githubusercontent.com/8181573/45959760-f1032d00-c01a-11e8-8576-ff0512c33da9.png" width=80% title="Smudgeplot热度图" align=center/>

**<p align="center">Figure 5. Smudgeplot热度图。
图片来源： [Smudgeplot github](https://github.com/KamilSJaron/smudgeplot)</p>**

2. smudgeplot_verbose_summary.txt
- 文件中可以提取预测的最终倍性结果：“* Proposed ploidy:	2”

# 5. KMC/jellyfish结果用于GenomeScope进行基因组调查
- 通过KMC/jellyfish获得的频数分布表结果kmcdb_k21.hist可用于GenomeScope进行基因组调查

`Rscript genomescope.R kmcdb_k21.hist <k-mer_length> <read_length> <output_dir> [kmer_max] [verbose]`

# 6. 批量运行脚本
如果样品数量非常多，可以用脚本自动遍历每个样品来运行。

把样品名称整理成样品列表（sample.list），修改下面脚本中的/path/to/data/为数据所在路径，即可运行。

```shell
for i in sample.list
do
	mkdir $i
	cd $i
    mkdir tmp

    # 运行KMC，获得频数分布表sample_k21.hist
	ls /path/to/data/$i* > FILES
	kmc -k21 -t8 -m64 -ci1 -cs10000 @FILES $i tmp
	kmc_tools transform $i histogram "$i"_k21.hist -cx10000
    rm -rf tmp

    # 运行genomescope2.0，估计基因组特征（默认是二倍体-p 2）
	genomescope.R -i "$i"_k21.hist -o ./ -k 21 -n "$i"_genomescope -p 2 >"$i"_genomescope.log 2>&1 &

    # 运行smudgeplot，估计
	L=$(smudgeplot.py cutoff "${i}"_k21.hist L)
	U=$(smudgeplot.py cutoff "${i}"_k21.hist U)
	echo $L $U
	kmc_tools transform $i -ci"$L" -cx"$U" dump -s "$i"_L"$L"_U"$U".dump
	smudgeplot.py hetkmers -o "$i"_L"$L"_U"$U" < "$i"_L"$L"_U"$U".dump # 耗时步骤，约1h
	smudgeplot.py plot "$i"_L"$L"_U"$U"_coverages.tsv -o $i -t $i
	cd ..
done
```

# 7. references
1. GenomeScope 2.0 + Smudgeplot paper：https://www.nature.com/articles/s41467-020-14998-3
2. Smudgeplot github：https://github.com/KamilSJaron/smudgeplot
3. genomescope2.0 github：https://github.com/tbenavi1/genomescope2.0

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>