---
title: 用k-mer分析进行基因组调查：（五）用GenomeScope评估基因组特征
date: 2022-06-05
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

description: 介绍GenomeScope，用GenomeScope做基因组调查(genome survey)的基因组特征评估。用GenomeScope1.0评估二倍体物种的基因组大小、杂合度、重复率等基因组特征，用GenomeScope2.0评估多倍体物种的基因组大小、杂合度、重复率、基因型比例和基因组结构(同源/异源多倍体)等基因组特征，用Smudgeplot估计物种的倍性。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=283098&auto=1&height=32"></iframe> </div>

**【推荐】用Smudgeplot评估物种倍性后，用组合jellyfish+GenomeScope1.0做二倍体物种的基因组调查，用组合KMC+GenomeScope2.0做多倍体物种的基因组调查。**

# 1. k-mer进行基因组调查的软件概况
k-mer进行基因组调查分为**k-mer频数统计**和**基因组特征评估**两步。
- GenomeScope可以实现第二步基因组特征评估。
- 需要在jellyfish/KMC等软件的第一步结果k-mer频数分布表的基础上，GenomeScope才可实现。

推荐第一步获取k-mer频数分布表的命令：
1. jellyfish
```
jellyfish count -C -m 21 -s 1000000000 -t 10 *.fastq -o sample.jf #计算k-mer频率，生成sample.jf
jellyfish histo -t 10 sample.jf > sample.histo #生成k-mer频数直方表sample.histo和k-mer直方图
```

2. KMC
```
mkdir tmp
ls *.fastq.gz > FILES
kmc -k21 -t16 -m64 -ci1 -cs10000 @FILES kmcdb tmp #计算k-mer频率
kmc_tools transform kmcdb histogram sample.histo -cx10000 #生成k-mer直方图
```

# 2. GenomeScope概况
- GenomeScope可以利用第一步jellyfish或KMC等其他软件分析得到k-mer频数分布表(sample.histo文件)实现第二步**基因组特征评估**。
- GenomeScope1.0在2017年发表，用于二倍体物种的基因组调查；2020年又发表了GenomeScope 2.0版本，用于多倍体物种的基因组调查，并发布了用于判断物种倍性的Smudgeplot。
- **GenomeScope1.0**的基因组特征结果包括基因组大小(genome size)，杂合度(heterozygosity)，重复序列比例，GC含量等。
- **GenomeScope2.0**的基因组特征结果包括基因组大小(genome size)，杂合度(heterozygosity)，重复序列比例，GC含量，基因型比例，和基因组结构(同源/异源多倍体)等。
- GenomeScope有网页版和Linux本地版，功能一样；推荐网页版，免去安装的麻烦。

# 3. GenomeScope1.0 网页版【推荐】—— 适用于二倍体物种
[GenomeScope1.0 网页版](http://qb.cshl.edu/genomescope)：http://qb.cshl.edu/genomescope

1. 上传第一步获得的k-mer频数分布表sample.histo文件；
2. 设置参数Kmer length为第一步选择的k-mer长度值，这里是17；
3. 参数Read length为序列读长，一般为150；
4. 参数Max kmer coverage默认是1000。
    
    建议按照物种情况修改，比如10000，以统计更准确。
    
    这个参数太小，可能造成过滤过多的Kmer，导致估计的基因组大小偏小的情况。
    
    这个参数太大则可能把高拷贝数量的DNA，比如叶绿体DNA，包括进Kmer的统计，造成GenomeScope算法的误差，所以还是不推荐使用-1或太大的值。

5. 提交后几分钟就可以得到结果，保存结果图片可用于发表。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/omics_genome.survey_GenomeScope1.0.png?raw=true" width=70% title="GenomeScope1.0结果示例" align=center/>

**<p align="center">Figure 1. GenomeScope1.0结果示例</p>**

# 4. GenomeScope2.0 网页版 【推荐】 —— 适用于多倍体物种
[GenomeScope2.0 网页版](http://qb.cshl.edu/genomescope/genomescope2.0)：http://qb.cshl.edu/genomescope/genomescope2.0

GenomeScope2.0版本相较于1.0，进行了许多改进，主要是增加了多倍体物种的基因组调查，并提出Smudgeplot方法来估计基因组的倍性和基因组结构。

## 4.1. GenomeScope 2.0 使用步骤
1. 上传第一步获得的k-mer频数分布表histo文件；
2. 设置参数Kmer length为第一步选择的k-mer长度值，这里是17；
3. 参数倍性Ploidy根据物种的倍性设定，默认是二倍体，设置成2；
4. 参数Max k-mer coverage默认是-1，即不限制最大k-mer深度。建议按照物种情况修改，比如10000，以统计更准确。这个参数太小，可能造成过滤过多的Kmer，导致估计的基因组大小偏小的情况。这个参数太大则可能把高拷贝数量的DNA，比如叶绿体DNA，包括进Kmer的统计，造成GenomeScope算法的误差，所以还是不推荐使用-1或太大的值。
5. 参数Average k-mer coverage for polyploid genome默认是-1，即不进行筛选，可以根据情况调整。
6. 提交后几分钟就可以得到结果，保存结果图片可用于发表。

## 4.2. GenomeScope 2.0 结果
### 4.2.1. **二倍体**结果
二倍体的GenomeScope 2.0 结果与GenomeScope 1.0 结果的主要不同之处在于杂合度结果(het)变成了2.0版本的代表基因型的aa和ab的比例，其中杂合基因型ab的比例即为杂合度。2.0结果中的p值代表设置的物种倍性。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/omics_genome.survey_GenomeScope2.0.png?raw=true" width=70% title="GenomeScope2.0二倍体结果示例" align=center/>

**<p align="center">Figure 2. GenomeScope2.0 二倍体结果示例</p>**

### 4.2.2. 区分**异源四倍体**和**同源四倍体**
GenomeScope2.0添加了参数倍性**Ploidy**，可以评估多倍体的基因组特征。

1. 四倍体共有两种可能的拓扑结构，代表着同源四倍体和异源四倍体，每种拓扑包含三种杂合基因型和一种纯合基因型，共有五种基因型。(五倍体有五种可能的拓扑，六倍体有十六种)
2. 根据结果中杂合基因型的分布模式可以区分异源四倍体和同源四倍体。

<img src="https://media.springernature.com/full/springer-static/image/art%3A10.1038%2Fs41467-020-14998-3/MediaObjects/41467_2020_14998_Fig2_HTML.png?as=webp" width=80% title="四倍体的两种拓扑结构可能性" align=center/>

**<p align="center">Figure 3. 四倍体的两种拓扑结构可能性 a异源四倍体，b同源四倍体。 图片来源：[GenomeScope 2.0 paper](https://www.nature.com/articles/s41467-020-14998-3)</p>**

3. GenomeScope2.0的**四倍体**结果

在GenomeScope 2.0 的结果中，如果杂合基因型aaab的比例大于aabb，则认为该物种是异源四倍体；如果杂合基因型aaab的比例小于aabb，则认为该物种是同源四倍体。

<img src="https://media.springernature.com/full/springer-static/image/art%3A10.1038%2Fs41467-020-14998-3/MediaObjects/41467_2020_14998_Fig6_HTML.png?as=webp" width=100% title="GenomeScope2.0多倍体结果示例" align=center/>

**<p align="center">Figure 4. GenomeScope2.0 多倍体结果示例 a异源四倍体，b同源四倍体。 图片来源：[GenomeScope 2.0 paper](https://www.nature.com/articles/s41467-020-14998-3)</p>**

# 5. GenomeScope1.0本地版 —— 适用于二倍体物种
GenomeScope1.0的本地版是用一个R脚本实现的，在[GenomeScope github](https://github.com/schatzlab/genomescope)可以下载genomescope.R脚本，下载后把genomescope.R文件加入环境变量即可使用。。

`Rscript genomescope.R sample.histo k-mer_length read_length output_dir [kmer_max] [verbose]`

必需参数：
- sample.histo：频数分布直方表，jellyfish的结果。
- k-mer_length：k-mer长度，通常是17，21，与jellyfish一致。
- read_length：reads长度，这里是150bp的PE reads，所以是150。
- output_dir：输出目录，结果图和文本都输出到这个目录。

# 6. GenomeScope2.0本地版 —— 适用于多倍体物种
## 6.1. 下载和安装

```shell
git clone https://github.com/tbenavi1/genomescope2.0.git #下载
cd genomescope2.0/
mkdir ~/R_libs #创建主目录下的R_libs文件夹用于安装本地R库
echo "R_LIBS=~/R_libs/" >> ~/.Renviron #创建/编辑.Renviron文件，使得R在创建的R_libs文件夹加载库
Rscript install.R #安装
```

安装后把目录下的genomescope.R文件加入环境变量即可使用。

## 6.2. 使用

`genomescope.R -i histogram_file -o output_dir -k k-mer_length`

参数：
- -i histogram_file：频数分布直方表，jellyfish或KMC的结果。
- -k k-mer_length：k-mer长度，通常是17，21，与jellyfish/KMC的设置一致。
- -o output_dir：输出目录，结果图和文本都输出到这个目录。
- -p ploidy：设置倍性。
- -l lambda：设置测序的平均k-mer覆盖率的初始猜测。
- -n name_prefix：设置输出文件的前缀。
- -m max_kmercov：设置从分析中排除高频k-mers的截止值，根据物种情况确定，推荐1000或10000。

# 7. GenomeScope实践经验
1. 实际使用中发现，GenomeScope1.0和2.0常常估算差异较大。建议二倍体还是使用GenomeScope1.0。
- 在估算一个约300Mb的二倍体基因组时，GenomeScope1.0估算出来267Mb，GenomeScope2.0估算出来149Mb。
- 在估算一个约6Gb的四倍体基因组时发现，GenomeScope1.0估算出来5.5Gb，GenomeScope2.0估算出来2.7Gb。

# 8. references
1. GenomeScope 1.0 github：https://github.com/schatzlab/genomescope
2. GenomeScope 2.0 github：https://github.com/tbenavi1/genomescope2.0
3. GenomeScope 1.0 paper：https://academic.oup.com/bioinformatics/article/33/14/2202/3089939
4. GenomeScope 2.0 + Smudgeplot paper：https://www.nature.com/articles/s41467-020-14998-3
5. Smudgeplot github：https://github.com/KamilSJaron/smudgeplot

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>