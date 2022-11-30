---
title: transcript assembly
date: 2021-07-16
categories: 
- bio
- concept
tags:
- 

description: 
---

<div align="middle"><music URL></div>

# 组装转录组
## 组装
1. Trinity无参考组装

nohup Trinity --seqType fq --max_memory 100G --left/disk1/zhongyan/project/20220630_Melastoma.candidum/input/Medinilla_magnifica/WWQZ-read_1.fq --right /disk1/zhongyan/project/20220630_Melastoma.candidum/input/Medinilla_magnifica/WWQZ-read_2.fq --CPU 24 --output trinity > Medinilla_magnifica_trinity.log 2>&1 &

## 去冗余
1. cd-hit-est去冗余
nohup cd-hit-est -i ../trinity/Trinity.fasta -o ./trinity_cd-hit-est.fa -c 0.95 -d 0 -M 64000 -T 12 > cd-hit-est.log 2>&1 &

2. trinity自带脚提取最长转录本
Trinity-v2.11.0/util/misc/get_longest_isoform_seq_per_trinity_gene.pl Trinity.fasta > Trinity_unigene.fa

## 
3. transdecoder鉴定orf
TransDecoder.LongOrfs -t trinity_cd-hit-est.fa  -O ./
TransDecoder.Predict -t trinity_cd-hit-est.fa -O ./

# 计算Ks
## blastp自我比对
makeblastdb -in sample.transdecoder.pep -dbtype prot -out index/sample.pep
blastp -query sample.transdecoder.pep -db index/sample.pep -out sample.pep.blast -evalue 1e-5 -num_threads 12 -outfmt "7 std qlen slen"

## 计算Ks
https://github.com/EndymionCooper/KSPlotting/blob/master/kSPlotter.py

kSPlotter.py脚本的运行步骤是：
1. 使用mcl-blastline pipeline基于blastp输出的比对情况进行简单的聚类，构建基因家族。
2. 用MUSCLE对每个基因家族进行比对。
3. 用PAML包的codeml计算每个基因家族中的所有基因对的最大似然估计Ks值。
4. 对每个基因家族内的Ks值进行去冗余，保留基因复制事件的Ks值。
- 去冗余的方法有两种（M1和M2），可以用-R参数指定，具体细节在软件的github页面有解释，个人倾向于选择M2。

### 运行
`nohup python2 kSPlotter.py -b sample.pep.blast -aa sample.pep -nt sample.cds -o sample -R M2 > ksplotter.log 2>&1 &`

一个Trinity组装出来200Mb，CD-HIT去冗余留下155Mb的转录组，运行kSPlotter用时约32小时。

### 结果

1. sample_ALL_KS.txt：所有基因对的Ks

```
CL100001	TRINITY_DN10006_c0_g1_i4.p1	TRINITY_DN10006_c0_g1_i2.p1	0.2759
CL100002	TRINITY_DN10008_c0_g1_i6.p1	TRINITY_DN10008_c0_g1_i1.p1	0.0788
...
CL107822	TRINITY_DN35_c0_g2_i9.p3	TRINITY_DN35_c0_g2_i5.p3	0.0000
CL107822	TRINITY_DN35_c0_g2_i10.p3	TRINITY_DN35_c0_g2_i5.p3	0.0000
```

2. sample_KS_by_cluster.txt：去冗余后聚类的Ks

```
CL106371 ['0.0217', '0.6017']
CL107615 ['0.0000', '1.961']
CL106451 ['0.0990', '0.2423', '4.29865', '34.351']
```

3. sample_REDUCED_KS.txt：去冗余后的Ks，可用于画Ks分布图

```
0.0217
0.6017
0.0000
1.961
0.0990
```

4. samplelog.txt

# 正态分布拟合
## 背景
在Lynch和Conery在2000年发表在Science的论文中，他们证明了小规模基因复制的Ks分布是L型，而在L型分布背景上叠加的峰则是来自于演化历史中某个突然的大规模复制事件。
L型分布（呈指数分布, exponential distribution)的峰可能是近期的串联复制引起，随着时间推移基因丢失，形成一个向下的坡。正态分布(normal distribution)的峰则是由全基因组复制引起。

这就意味着我们可以根据ks频率分布图的正态分布峰来判断物种历史上发生过的全基因组复制事件，并通过ks值拟合峰值获得WGD事件发生的时间。

## 模型
用高斯混合模型对Ks进行正态分布拟合，排除假阳性峰。

R包mclust可以实现

```R
install.packages("mclust") # 安装
library(mclust) # 加载
data<-read.csv(sample_REDUCED_KS.txt,header=F)
data<-data[data$V1>0.05,,drop=F] #只保留>0.05的数据
data<-data[data$V1<5,,drop=F] #只保留<5的数据
mb=Mclust(data)
summary(mb,parameters=TRUE)


```


# 画Ks分布图，找峰值

```R
library(ggplot2)
library(ggpmisc)
data <- read.table("sample_REDUCED_KS.txt",header=F)
p <- ggplot(data, aes(V1)) + geom_density(size=1,color="black")+xlab("Synonymous substitution rate(Ks)")+ylab("Percent of Total Paralogs")+theme_classic()+scale_x_continuous(name="Ks", limits=c(0,2),breaks = seq(0,2,0.1))
ggsave(file="Ks.pdf",plot=p,width=10,height=5)
```

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>