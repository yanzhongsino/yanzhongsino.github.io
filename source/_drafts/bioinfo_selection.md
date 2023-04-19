---
title: 自然选择的分析
date: 2023-04-12
categories: 
- bioinfo
- R
- plot
tags:
- 
description: 用plink做GWAS分析
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1697043&auto=1&height=32"></iframe></div>

判断一棵树上的哪些支受到了自然选择


# 从SNP生成一致性序列

for i in $(cat sample.list);do bcftools consensus -f ref.fa -s $i samples.vcf.gz -o $i.fa; done # 用变异位点替换参考基因组，生成所有样本的一致性基因组序列
for i in $(cat sample.list);do gffread -x $i.cds.fa -g $i.fa ref.gff; done # 生成CDS序列
for i in $(cat sample.list);do seqkit seq -w 0 -u $i.cds.fa |sed "s/>/>$i/g" > $i.cds; done # 把CDS序列转换成单行格式
for i in $(cat sample.list);do rm $i.cds.fa ; done # 删除多行格式的CDS序列
for i in $(cat species_sample.txt);do grep -A 1 mc02226 consensus/$i.cds|sed "s/>.*/>$i/g" >>mc02226.cds; done # 提取基因mc02226的CDS序列

下面方法批量处理基因
for j in $(cat genes.list);do for i in $(cat sample.txt);do grep -A 1 $j $i.cds|sed "s/>.*/>$j_$i/g" >>$j.cds; done;done

for j in $(cat genes.list);
    do
        for i in $(cat sample.list);
            do grep -A 1 $j $i.cds|sed "s/>.*/>$j_$i/g" >>$j.cds;
            done;
    done


sed -i 's/TGA$//' mc02226.cds # 删除终止密码子



# references
1. https://yuzhenpeng.github.io/2019/12/24/paml/#%E4%BA%8C-%E5%A6%82%E4%BD%95%E8%BF%90%E8%A1%8CPAML
2. https://www.jianshu.com/p/372cf1f02d80

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>
