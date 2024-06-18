---
title: 转换GenBank文件为tbl格式，为提交注释做准备
date: 2022-06-16
categories: 
- bioinfo
- fileformat
tags: 
- GenBank
- tbl
- organellar genome
- genome annotation
- genome submit
- Sequin
- tbl2asn
- GB2sequin
- gbf2tbl.pl
- genbank_to_tbl.py
- Genbank2Sequin.py
description: 介绍GenBank格式文件转换成tbl格式的工具，包括可用于准备上传NCBI注释文件的在线工具GB2sequin，脚本gbf2tbl.pl，脚本genbank_to_tbl.py和脚本Genbank2Sequin.py。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=157092&auto=1&height=32"></iframe></div>

# 1. 简介
- GenBank格式常常出现在细胞器基因组的注释软件的结果中，但上传细胞器注释到公共数据库（eg NCBI）需要tbl格式文件。
- 这篇文章介绍了几种把GenBank格式文件转换成tbl格式的方法。包括在线工具GB2sequin（此工具还包含验证和运行tbl2asn功能，可用于准备上传NCBI的注释文件），以及两个python2写的转换脚本。

## 1.1. GenBank格式
GenBank是美国国立卫生研究院维护的基因序列数据库，数据库中用gb格式文件（后缀.gb，.gbk）记录基因注释信息。

GenBank文件格式标准参考：https://www.ncbi.nlm.nih.gov/Sitemap/samplerecord.html

## 1.2. tbl格式(for Sequin)
用于提交基因注释到GenBank的一种tab分隔的五栏的特征表（feature table）文件格式（后缀.tbl）。

tbl文件格式标准参考：https://www.ncbi.nlm.nih.gov/genbank/feature_table/

# 2. GB2sequin在线工具【推荐】
## 2.1. 简介
GB2sequin是用于转换GenBank格式文件为NCBI提交格式Sequin（包括tbl文件和sqn文件）的在线工具，包含验证GenBank注释的功能和运行tbl2asn的功能。

网址是：

https://chlorobox.mpimp-golm.mpg.de/GenBank2Sequin.html

## 2.2. 优势
1. 通过网页在线转换，方便快捷。
2. 自动对注释进行验证，并生成错误报告，可根据错误报告修正注释。
3. 支持在输入中加入作者信息的template等文件。
4. 自动运行tbl2asn，生成sqn文件，方便提交tbl文件到NCBI。
5. 自动生成标准的GenBank格式文件，去除冗余。

## 2.3. 使用步骤
1. 上传GenBank格式文件，要在文件末包含注释的序列，否则运行结果空白。
2. 【推荐】上传Author Submission Template。可以在https://submit.ncbi.nlm.nih.gov/genbank/template/submission/上填写信息后导出template.sbt文件用于上传。
3. 【可选】上传Source Modifier Table，Gene Product Specification Table。
4. 选择分子类别"Molecule Type"，一般默认genomic DNA即可。
5. 选择是否是完整(complete)，环状(circular)的基因。
6. 选择编码方式"Genetic Code"，植物样品选标准Standard。
7. 选择基因位置Location，包括叶绿体chloroplast，线粒体mitochondrion等。
8. 点击Start Conversion，等待几分钟即可得到结果，点击每项结果然后下载（大多是文本文件）。
9. 根据验证结果文件的错误ERROR和警告WARNING逐一修改原始GenBank文件。
10. 再次运行，重复以上操作，直至无错误和警告，即可使用sqn文件上传至NCBI。

验证结果的含义（tbl2asn运行的部分）可以参考之前写的**基因组提交的博客**：https://yanzhongsino.github.io/2022/03/22/omics_genome.submit/

## 2.4. 结果
结果文件包括：
1. FASTA：从GenBank提取的fasta文件。
2. Annotation Table：tbl文件。
3. tbl2asn log：tbl2asn运行的log文件。
4. Sequin：sqn文件，用于提交给NCBI。
5. GenBank：标准化的GenBank文件。
6. Validation：验证结果文件，包括细节。
7. Validation Summary：验证结果的概括。

# 3. genbank2tbl的脚本
- **只要GB2sequin可以用，就强烈推荐GB2sequin，毕竟自带验证功能和自动运行tbl2asn。**
- 如果GB2sequin不可用（比如要用的时候服务器宕机了之类的原因，我才不会告诉你我就是这么走运才探索了一下脚本），可用脚本进行GenBank到tbl的格式转换。
- 由于都是陈年老脚本，以下脚本都是用python2写的，并且调用了biopython（提前安装biopython）。

## 脚本gbf2tbl.pl【NCBI 官方出品】
1. 简介
- 脚本gbf2tbl.pl是NCBI官方提供的perl脚本。
- 我暂时还没用过，先记在这。

2. 下载
`wget ftp://ftp.ncbi.nlm.nih.gov//toolbox/ncbi_tools/converters/scripts/gbf2tbl.pl`

## 3.1. 脚本genbank_to_tbl.py【推荐】
### 3.1.1. 简介
genbank_to_tbl.py脚本用于转换genbank格式的gene，CDS，exon，intron，tRNA，rRNA，pseudo genes等注释为tbl格式。

### 3.1.2. 脚本
下载地址：https://gist.github.com/nickloman/2660685

```python
# requires biopython
# run like:
#   genbank_to_tbl.py "my organism name" "my strain ID" "ncbi project id" < my_sequence.gbk
#   writes seq.fsa, seq.tbl as output

import sys
from copy import copy
from Bio import SeqIO

def find_gene_entry(features, locus_tag):
    for f in features:
        if f.type == 'gene':
            if f.qualifiers['locus_tag'][0] == locus_tag:
                return f
    print locus_tag
    raise ValueError

coding = ['CDS', 'tRNA', 'rRNA']

def go():
    seqid = 0
    fasta_fh = open("seq.fsa", "w")
    feature_fh = open("seq.tbl", "w")
    allowed_tags = ['locus_tag', 'gene', 'product', 'pseudo', 'protein_id', 'gene_desc', 'old_locus_tag']
    records = list(SeqIO.parse(sys.stdin, "genbank"))

    for rec in records:
        for f in rec.features:
            if f.type in coding and 'gene' in f.qualifiers:
                print f.qualifiers['locus_tag'][0]

                f2 = find_gene_entry(rec.features, f.qualifiers['locus_tag'][0])
                f2.qualifiers['gene'] = f.qualifiers['gene']

                del f.qualifiers['gene']

    for rec in records:
        seqid += 1

        if len(rec) <= 200:
            print >>sys.stderr, "skipping small contig %s" % (rec.id,)
            continue

#        rec.id = rec.name = "%s%08d" % (sys.argv[4], seqid,)

        circular = rec.annotations.get('molecule', 'linear')
        rec.description = "[organism=%s] [strain=%s] [topology=%s] [molecule=DNA] [tech=wgs] [gcode=11]" % (sys.argv[1], sys.argv[2], circular)
        SeqIO.write([rec], fasta_fh, "fasta")

        print >>feature_fh, ">Feature %s" % (rec.name,)
        for f in rec.features:
            if f.strand == 1:
                print >>feature_fh, "%d\t%d\t%s" % (f.location.nofuzzy_start + 1, f.location.nofuzzy_end, f.type)
            else:
                print >>feature_fh, "%d\t%d\t%s" % (f.location.nofuzzy_end, f.location.nofuzzy_start + 1, f.type)

            if f.type == 'CDS' and 'product' not in f.qualifiers:
                f.qualifiers['product'] = ['hypothetical protein']

            if f.type == 'CDS':
                f.qualifiers['protein_id'] = ["gnl|ProjectID_%s|%s" % (sys.argv[3], f.qualifiers['locus_tag'][0])]

            if f.type in coding:
                del f.qualifiers['locus_tag']

            for key, vals in f.qualifiers.iteritems():
                my_allowed_tags = copy(allowed_tags)
                if 'pseudo' in f.qualifiers:
                    my_allowed_tags.append('note')

                if key not in my_allowed_tags:
                    continue

                for v in vals:
                    if len(v) or key == 'pseudo':
                        print >>feature_fh, "\t\t\t%s\t%s" % (key, v)


go()
```

### 3.1.3. 使用
1. 命令
`python2 genbank_to_tbl.py sample mc_01 ncbi_id < sample.gb`

2. 参数
- sample：样品名称，organism name
- mc_01：品种编号，strain ID
- ncbi_id：NCBI项目ID
- < sample.gb：输入genbank格式文件

四个参数都是必需参数。

3. 输出
- seq.fsa：基因组序列
- seq.tbl：tbl格式文件

4. notes
- seq.tbl结果中包含genbank注释的gene，CDS，exon，intron，tRNA，rRNA，pseudo genes等结果。
- 需要在输入的genbank文件中包含locus_tag的值（可以用基因名称），否则报错。

## 3.2. Genbank2Sequin.py
### 3.2.1. 简介
Genbank2Sequin.py脚本用于批量转换genbank格式的gene和CDS注释为tbl格式。

### 3.2.2. 脚本
下载地址：https://github.com/Van-Doorslaer/Genbank2sequin/blob/master/Genbank2Sequin.py

```python
from Bio import SeqIO
import re, glob, os

for f in glob.glob("*.gb"):
    ID = f.split(".")[0]
    for r in SeqIO.parse(f,"genbank"):
        #print r.features
        with open(ID+'.fsa', 'w') as fasta, open(ID+".tbl", 'w') as table:
            print >> fasta, ">%s [organism=Human papillomavirus %s]\n%s" %(ID, ID, r.seq)
            print >>table, ">Feature "+ID 
            for (index, feature) in enumerate(r.features):
                if feature.type ==  'CDS':
                    m=re.search('\[(\d*)\:(\d*)\]',str(feature.location))
                    if m:
                        #print feature.qualifiers
                        print >>table, "%s\t%s\tgene\n\t\t\tgene\t%s" %(int(m.groups()[0])+1, m.groups()[1],feature.qualifiers['note'][0])
                        print >>table, "%s\t%s\tCDS\n\t\t\tproduct\t%s\n\t\t\tgene\t%s\n\t\t\tcodon_start\t1" %(int(m.groups()[0])+1, m.groups()[1], feature.qualifiers['note'][0], feature.qualifiers['note'][0])
```

### 3.2.3. 使用
1. 命令
`python2 Genbank2Sequin.py`

- 运行后会自动搜索当前文件夹下的所有.gb后缀文件，然后把gene和CDS的注释转换为.tbl文件。

2. 输出
- *.tbl：tbl格式文件。

3. notes
- *.tbl结果中只包含genbank注释的gene，CDS结果。
- 需要在输入的genbank文件中包含note的值（可以用基因名称），否则报错。
- 可批量操作多个gb文件。

# 4. reference
1. GB2sequin paper：https://www.sciencedirect.com/science/article/pii/S0888754318301897?via%3Dihub
2. GB2sequin：https://chlorobox.mpimp-golm.mpg.de/GenBank2Sequin.html
3. genbank_to_tbl.py：https://gist.github.com/nickloman/2660685
4. Genbank2Sequin.py：https://github.com/Van-Doorslaer/Genbank2sequin/blob/master/Genbank2Sequin.py
5. genbank格式：https://www.ncbi.nlm.nih.gov/Sitemap/samplerecord.html
6. tbl格式：https://www.ncbi.nlm.nih.gov/genbank/feature_table/

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>