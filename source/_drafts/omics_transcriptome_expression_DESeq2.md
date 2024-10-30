---
title: 用DESeq2做差异表达基因的分析
date: 2024-10-25
categories: 
- omics
- transcriptome
- expression
- Differential expression analysis
tags:
- map
- Differential expression analysis
- DEA
- hisat2
- DESeq2
description: 记录用DESeq2对RNA-Illumina-seq数据做差异表达分析的流程。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=5255807&auto=1&height=32"></iframe></div>

本文记录从原始下机数据，到差异表达分析的全流程。
1. 原始数据前处理
- 原始RNA-seq数据的质控【fastp/FastQC/Trimmomatic】
- clean数据比对【HISAT2/STAR】
- 生成计数矩阵【featureCounts/HTSeq-count】
2. 差异表达分析【DESeq2】
- 安装和加载 DESeq2
- 构建 DESeqDataSet：生成的基因计数矩阵（行：基因，列：样本）和样本信息（如样本条件）数据框准备好。
- 数据预处理： 过滤掉低表达基因，通常只保留在至少某些样本中有可检测信号的基因。
- 差异表达分析： 调用主函数 DESeq() 进行归一化和差异分析。
- 提取结果： 使用 results() 提取差异表达结果，并应用适当的 p 值校正（如 Benjamini-Hochberg 法）控制假阳性率。
3. 结果可视化
- 可视化：使用 MA 图、火山图等可视化数据。
- 查看显著差异基因： 对结果进行筛选，通常依据调整后的 p 值 (padj) < 0.05 并且 |log2FoldChange| > 1。
4. 后续分析
- 对显著差异表达基因可进行功能注释、通路分析等，了解其生物学意义。

# 原始数据的前处理
## RNA-seq的QC和map
首先对原始测序的RNA-Illumina-seq数据进行质控和比对，生成BAM/SAM文件用于计数。
1. QC
- `fastp -i rna1_raw_R1.fq -I rna1_raw_R2.fq -o rna1_clean_R1.fq rna1_clean_R2.fq -q 20 -l 50 -w 12`
- `fastp -i rna2_raw_R1.fq -I rna2_raw_R2.fq -o rna2_clean_R1.fq rna2_clean_R2.fq -q 20 -l 50 -w 12`
2. mapping
- `hisat2-build ref.fa ref.hisat` # 建索引
- `hisat2 --dta -p 8 -x ref.hisat -1 rna1_clean_R1.fa -2 rna1_clean_R2.fa 2>rna1_hisat.log |samtools sort -O BAM -@ 12 > rna1_hisat.bam &` #样品1，保存日志到rna1_hisat.log文件。
- `hisat2 --dta -p 8 -x ref.hisat -1 rna2_clean_R1.fa -2 rna2_clean_R2.fa 2>rna2_hisat.log |samtools sort -O BAM -@ 12 > rna2_hisat.bam &` #样品2，保存日志到rna2_hisat.log文件。

## 生成计数矩阵
1. featureCounts 是 Subread 软件包的一部分，用于从比对好的 BAM 文件中提取计数。它支持多线程，可以处理大数据集。

```shell
conda install bioconda::subread
featureCounts -T 4 -a genes.gtf -o counts.txt sample1.bam sample2.bam sample3.bam
```

- -T 指定使用的线程数。
- -a 指定 GTF 或 GFF 格式的注释文件。
- -o 指定输出文件名。
- 后面依次列出需要处理的 BAM 文件。

2. HTSeq-count 是 HTSeq 软件包的一部分，可以用来从 BAM 文件中生成计数。

```shell
conda install bioconda::htseq # 安装HTSeq
htseq-count -f bam -r pos -s no -i gene_id sample1.bam genes.gtf > counts.txt
```

- -f 指定输入文件格式，通常为 bam。
- -r 指定输入文件中排序的方式，常用 pos（位置排序）或 name（名称排序）。
- -s 指定是否有链特异性（strand-specific），可以是 yes、no 或 reverse。
- -i 指定在注释文件 GTF 中用作计数的特征属性（例如 gene_id）。
- <input_file.bam> 是 BAM 格式的输入文件。
- <annotation_file.gtf> 是 GTF 格式的注释文件。

# 差异表达分析
1. 在R中安装DESeq2

```shell
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install("DESeq2")
```

2. 加载DESeq2并导入数据
```shell
library(DESeq2)

# 导入计数矩阵和元数据
countData <- read.table("counts.txt", header = TRUE, row.names = 1)
colData <- read.csv("colData.csv", header = TRUE, row.names = 1)

# 确保样本顺序匹配
all(rownames(colData) == colnames(countData)) # 应该返回 TRUE
```

3. 构建DESeqDataSet对象

```shell
dds <- DESeqDataSetFromMatrix(countData = countData,
                              colData = colData,
                              design = ~ condition)
```

4. 数据预处理
- 过滤低计数的基因：
```shell
dds <- dds[rowSums(counts(dds)) > 1, ]
```

5. 运行差异表达分析
- `dds <- DESeq(dds)`
6. 提取结果
- 提取对比分析结果（例如，条件"B" 对比 "A"）：`res <- results(dds, contrast = c("condition", "B", "A"))`

7. 检查和总结结果
- 可视化 p 值和对数折叠变化：
+
```shell
# 评估调整后的 p 值小于 0.05 的显著基因
summary(res)
# 绘制火山图（Volcano Plot）
plot(log2FoldChange ~ -log10(padj), data = as.data.frame(res), pch = 20, main = "Volcano Plot")
```

8. 结果导出
- 将结果导出到 CSV 文件中：`write.csv(as.data.frame(res), file = "DESeq2_results.csv")`


- 在数据导入和处理阶段需要确保数据的正确性，特别是样本名称和条件应该一致。
- 根据具体实验设计，可能需要修改 design 参数以反映更复杂的实验结构。
- 在解释结果时，要考虑生物学背景以及统计结果。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" 