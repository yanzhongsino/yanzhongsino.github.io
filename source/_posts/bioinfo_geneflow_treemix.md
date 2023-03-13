---
title: 系统发育网络推断 —— TreeMix
date: 2022-03-20
categories: 
- bioinfo
- gene flow
tags: 
- TreeMix
- gene flow
- hybridization
- introgressive
- population networks
description: 这篇博客记录的是用TreeMix软件推测居群间系统网络，确定最佳基因流数量的具体方法。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1296539261&auto=1&height=32"></iframe></div>

这篇博客记录的分析步骤是：
1. 使用基因组尺度的snp数据计算等位基因频率
2. 转换成TreeMix的输入文件格式
3. 用TreeMix推断基因流
4. 确定最佳基因流数量
5. 绘制系统发育网络图

# 1. TreeMix
TreeMix是一个利用全基因组尺度的等位基因频率数据推断居群的分化和和杂合（基因流动或基因渗入）的一个软件，适用于推断一个物种多个居群的系统发育网。TreeMix用多居群的等位基因频率来推测最大似然树(ML树)，并可推断指定数量基因流事件。

1. TreeMix简介
   - TreeMix利用等位基因频率来推断群体间分化和杂合（基因流动或基因渗入）
2. TreeMix输入输出
   - 输入：基因组snp的vcf文件，和居群系统树(可选optional)
   - 输出：最佳杂交次数和系统发育网络(包含杂交方向和强度)
3. TreeMix优势和不足
   - TreeMix和PhyloNetworks一样，也是推断系统发育网络。
   - 我自己用时，有些PhyloNetworks报错无法定根和边缘错误的情况TreeMix可以找到最佳杂交次数。
   - 不足是比PhyloNetworks更耗时，超级耗时。

## 1.1. TreeMix install

1. `conda install -c bioconda treemix` #现在conda可以下到最新版本treemix 1.13

2. 在[treemix software](https://bitbucket.org/nygcresearch/treemix/downloads/)下载压缩包再安装

```shell
tar -xvf treemix-1.0.tar.gz
cd treemix-1.0
./configure
make
make install
```

## 1.2. 准备文件
1. sample.snp.vcf.gz：基因组尺度的snp文件，vcf格式
2. sample.tre【optional】：居群的系统发育树文件，Newick格式，把支持率等信息删除；也可以不提供，用TreeMix推断ML树。**这个树文件是居群树文件，不是个体树文件**。

## 1.3. 分析步骤
### 1.3.1. 计算等位基因频率

1. `vcftools --gzvcf sample.snp.vcf.gz --plink-tped --out sample` # 转换为tped格式，生成sample.tped和sample.tfam文件。
如果报错**Unrecognized values used for CHROM: scaffold_1 - Replacing with 0**，可以用`bcftools view -H sample.snp.vcf.gz | cut -f 1 | uniq | awk '{print $0"\t"$0}' > chrom-map.txt`生成染色体ID的文件，然后再用--chrom-map指定文件来运行`vcftools --gzvcf sample.snp.vcf.gz --plink-tped --chrom-map chrom-map.txt --out sample`。

2. sample.tfam文件共6列，前两列都为样本ID，修改第一列数据为群体ID。（可以是同一物种不同种群作为单独群体，也可以是一个物种作为一个群体，这里sample的一个物种作为一个群体，群体ID为物种名）。

3. `cat sample.tfam |awk '{print $1"\t"$2"\t"$1}' >sample.pop.cov`编辑文件sample.pop.cov，格式为：共三列，前两列与修改后的sample.tfam前两列一样，为群体ID和样本ID，第三列和第一列一致，tab分隔。

4. 计算等位基因组的频率
- `plink --threads 12 --tfile sample --freq --allow-extra-chr --within sample.pop.cov` # 计算等位基因组的频率，生成plink.frq.strat和plink.nosex文件，加上--allow-extra-chr是防止识别不了染色体名称。
- `pigz plink.frq.strat` #压缩等位基因频率文件

5. 转换格式【耗时小时计】
- `python2 /path/to/plink2treemix.py plink.frq.strat.gz sample.treemix.in.gz` #用treemix自带脚本进行格式转换，notes：输入输出都为压缩文件，plink2treemix.py使用python2并需要绝对路径（否则报错）。

### 1.3.2. treemix推断基因流
1. treemix默认功能
`treemix -i sample.treemix.in.gz -o sample` # treemix的默认功能是在所有位点都独立的假设下建一棵种群ML树

2. TreeMix参数解释
- -i 指定基因频率输入文件
- -o 指定输出文件前缀
- -tf 【可选】指定树文件，指定后就使用指定树的拓扑结构(最好把支持率和其他无关信息都删除，只留最简单的Newick格式)，否则treemix会推断拓扑
- -root 【可选】指定外类群(指定的是居群名称)，多个用逗号分隔；最好指定，否则后面plot_tree画树没找到更换外类群的参数会很麻烦
- -m 为the number of migration edges即基因渗入的次数
- -k 1000 因为SNP之间不是独立位点，为了避免连锁不平衡，用k参数指定SNP数量有连锁，比如这里指定用1000个SNP组成的blocks评估协方差矩阵
- -se 计算迁移权重的标准误差(计算成本高)，如果想省时间可以不用这个参数
- -bootstrap 为了判断给定树拓扑的可信度，在blocks运行bootstrap重复，没太理解
- -global 在增加所有种群后做一轮全局重组。
- noss 关闭样本量校正。TreeMix计算协方差会考虑每个种群的样本量，有些情况(如果有种群的样本只有1个)会过度校正，可以关闭。
- -g old.vertices.gz old.edges.gz #使用之前生成的树和图结果，用-g指定之前的两个结果文件
- -cor_mig known_events and -climb #合并已知的迁移事件

3. 推荐参数
检测物种间基因流，每个物种一个个体或多个个体的数据，用的参数：
`treemix -i sample.treemix.in.gz -o sample -tf sample.tre -root outgroup1,outgroup2 -m 3 -k 1000 -se -bootstrap -global -noss`

4. 多次分析以评估最佳m值
比如m取1-10(常用1-5,1-10)，每个m值重复5次(至少两次)。
```shell
for m in {1..10}
do
	{
	for i in {1..5}
	do
		{
			nohup time treemix -i sample.treemix.in.gz -o sample.${m}.${i} -tf sample.tre -root outgroup1,outgroup2 -m ${m} -k 1000 -se -bootstrap -global -noss>${m}.${i}.out 2>&1 &
		}
	done
	}
done
```

5. 运行结果
每一次运行都将生成一套后缀为llik,modelcov,treeout,vertices,cov,covse,edges的文件，这些文件都不大(KB级别)，但还会生成一个300+MB的out文件(out文件没报错就可以删除了)。

- .llik：保存了从0到指定基因渗入次数m的始末似然值
- .treeout.gz: 拟合树模型(fitted tree model)和迁移事件(migration events)。第一行是Newick格式的ML树，剩下行包含迁移边界(migration edges)
- .cov.gz: 种群间的协方差矩阵(covariance matrix)
- .covse.gz: 协方差矩阵的标准误差(standard errors)
- .modelcov.gz: 模型的拟合协方差(fitted covariance)
- .vertices.gz和.edges.gz: 包含推断图的内部结构

### 1.3.3. 确认最佳m值
R包OptM分析treemix的结果，确认最佳m值(Deltam值最小情况下的m值)。

在treemix运行的结果目录下运行R，需要使用每一次运行的.llik,.cov.gz,.modelcov.gz三个配套结果文件，每个m值至少重复/迭代两次，否则报错。

```R
>library(OptM) #载入OptM包
>evanno=optM("./",method = "Evanno"，tsv="evanno.txt") #用Evanno方法(默认)分析treemix结果文件，保存结果到evanno.txt文件，里面就有每个m对应的Deltam值(第15列)，Deltam值最小时对应的m即时最佳m值，这里的例子是m=3时最低。
  m runs   mean(Lm)   sd(Lm)   min(Lm)   max(Lm)
1 0   32 -329764.72 4802.416 -336972.0 -320838.0
2 1    5 -239329.60 3018.974 -243918.0 -236221.0
3 2    5 -139018.20 5332.074 -147433.0 -134177.0
4 3    5 -116765.80 1533.134 -119301.0 -115329.0
5 4    5  -94811.20 2239.830  -97001.2  -91664.1
6 5    2  -86727.45 1860.469  -88043.0  -85411.9
7 6    5  -79769.74 1545.658  -82041.1  -77782.6
8 7    5  -68844.08 2924.953  -73560.4  -66420.4
      L'(m)   sdL'(m)  minL'(m)   maxL'(m)    L''(m)
1        NA        NA        NA         NA        NA
2  90435.12 1783.4423 88651.676  92218.561  9876.281
3 100311.40 2313.0997 97998.300 102624.500 78059.000
4  22252.40 3798.9398 18453.460  26051.340   297.800
5  21954.60  706.6962 21247.904  22661.296 13870.850
6   8083.75  379.3613  7704.389   8463.111  1126.040
7   6957.71  314.8111  6642.899   7272.521  3967.950
8  10925.66 1379.2958  9546.364  12304.956        NA
    sdL''(m) minL''(m) maxL''(m)     Deltam
1         NA        NA        NA         NA
2  529.65744  9346.624 10405.939  3.2714033
3 1485.84010 76573.160 79544.840 14.6395203
4 3092.24366 -2794.444  3390.044  0.1942427
5  327.33487 13543.515 14198.185  6.1928138
6   64.55023  1061.490  1190.590  0.6052453
7 1064.48476  2903.465  5032.435  2.5671598
8         NA        NA        NA         NA
    mean(f)        sd(f)
1        NA           NA
2 0.9776741 1.214193e-04
3 0.9837528 7.790282e-04
4 0.9867460 2.483868e-04
5 0.9897553 8.854187e-05
6 0.9909923 5.825696e-05
7 0.9915973 8.194263e-05
8 0.9923055 2.114830e-04

>plot_optM(evanno,method = "Evanno",pdf="evanno.pdf") #把结果作图，更直观判断最佳m值，保存成evanno.pdf

>linear=optM("./",method = "linear",tsv="linear.txt") #还可以换linear线性回归方法分析一下treemix结果，m作为横坐标，Log Likelihood做纵坐标，保存结果到linear.txt文件，但不知道为啥没报错但没有结果文件输出
>plot_optM(linear,method = "linear",pdf="linear.pdf") #结果作图，这个结果我没找到怎么看最佳m值，猜测是在图中斜率变化最大的位置对应的m值，标记了黑色的change points处，这里是m在2-3的位置，接近2。
```

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/bioinfo_geneflow.treemix_evanno.png?raw=true" width=80% title="linear" align=center/>

**<p align="center">Figure 1. evanno 示例</p>**

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/images/bioinfo_geneflow.treemix_linear.png?raw=true" width=80% title="linear" align=center/>

**<p align="center">Figure 2. linear 示例</p>**

### 1.3.4. 最佳m值作图
用最佳m值对应的结果文件作图
```R
>source("path/to/plotting_funcs.R") #载入treemix/src文件夹中的R脚本
>plot_tree("sample.treemix.3.1") #用sample.treemix.3.1结果作图
```

**notes**：
- 虽然TreeMix的输入是个体的snp.vcf计算的基因型频率数据，但推断的系统发育网是居群的(居群名称组成的系统结构)，不是个体的系统发育。
- sample.tre是居群树(据群名称组成的树)，不是个体树
- treemix的参数-root指定的也是居群名称

# 2. reference
1. TreeMix paper：https://www.nature.com/articles/npre.2012.6956.1
2. TreeMix software：https://bitbucket.org/nygcresearch/treemix/downloads/
3. TreeMix manual：https://bitbucket.org/nygcresearch/treemix/downloads/treemix_manual_10_1_2012.pdf

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>