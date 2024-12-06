---
title: 使用BankIt提交原始测序数据到NCBI的GenBank
date: 2024-12-06
categories: 
- NCBI
- GenBank
- submit
- SRA
tags:
- SRA
- Sequence Read Archive
- Unassembled sequence reads
- submit
- GenBank
- NGS
- TGS
- Illumina
- PacBio
description: 记录上传原始测序reads数据（二代数据和三代数据）到GenBank的步骤。
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=1392871582&auto=1&height=32"></iframe></div>

# 1. 上传的原始测序数据
二代或三代测序的原始序列读数应提交至Sequence Read Archive(SRA)：https://www.ncbi.nlm.nih.gov/sra。

# 2. 上传前准备
建议先注册和登录NCBI账号再进行上传，之后的所有提交记录都会保存在你的账号下，在这里可以查看提交记录：https://submit.ncbi.nlm.nih.gov/subs/。

# 3. 上传步骤
## 3.1. 选择数据类型
1. 打开BankIt网址：https://www.ncbi.nlm.nih.gov/WebSub。登录NCBI账号。
2. 选择上传的数据类别：
- Unassembled sequence reads (SRA)：未组装的测序reads
## 3.2. 进入Submission Portal门户网站
选完数据类型，会自动进入Submission Portal门户网站
1. 提交者信息Submitter
- 包括姓名First and Last name，邮箱Email，单位Submitting organization(学校)，部门Department(学院)，街道Street，城市City，省/州State/Province，邮编Postal code，国家Country。
- 在最后可以选择**Update my contact information in profile**会更新填写的信息到账号，下一次新填写就默认填入提交者信息了。
2. 一般性的信息General info
- BioProject和BioSample是所有提交到GenBank都需要的，属于哪个项目，用了哪个样本。
- 可以先注册这两项再在这里提供accession number；也可以选择还未注册提交之后会增加填写BioProject和BioSample信息的步骤然后自动注册新的；
- 同一个项目同一个样品多个测序文件，比如基因组测序最常见的方案（Illumina PE的 DNA-Seq和RNA-Seq，以及PacBio SMRT），可以在一次提交中一同提交。
- 释放日期Release date可以指定日期，也可以提交通过后立即释放；释放日期可以提交后发邮件更改的；
3. Project info【如果2填了没有已有的BioProject就会出现这一项】
- 提供创建新的BioProject的信息
- 项目题目Project title，项目描述Project description，和领域Relevance 
- 可选项：相关的外部链接External links，基金grants
4. BioSample Type【如果2填了没有已有的BioSample就会出现这一项】
- NCBI packages：样品类型，选Plant。可以填学名来筛选。
5. BioSample Attributes【如果2填了没有已有的BioSample就会出现这一项】
- 如果是批量提交，可以用excel或csv表格提交多个BioSamples信息。
- 样品名称Sample Name：区别于其他样品的唯一名称。可以用lab惯用的样品命名，比如YZ_Mc_01。
- 生物名称Organism，填物种学名；
- 个体描述isolate/栽培名称cultivar/生态型ecotype三选一填，填样本个体的唯一标识符ID。
- 年龄age/发育阶段development stage二选一填。
- 地理位置geographic location：可以填国家China
- 组织tissue：填写取样部位，eg. leaf，flower，root，stem.
6. SRA Metadata
- 如果是批量提交，可以用excel或csv表格提交多条信息。
- 样品名称Sample Name：选择BioSample Attributes中填的。
- Library ID：测序前建库的ID，是独一无二的。
- Title：通常用这种形式：{methodology} of {organism}:{sample info}，如RNA-Seq of Bauhinia variegata: mature flower。
- Library strategy：建库策略，二代和三代测序选全基因组测序**WGS**。
- Library source：DNA-Seq和RNA-Seq通常Genomic或Transcriptomic。
- Library selection：二代和三代测序选RANDOM
- Library layout：单端 single或双端 paired
- Platform：测序平台，Illumina或Pacbio_SMRT等。
- Instrument model：具体的测序仪器型号。
- Design description：建库方法的简短描述。如TruSeq library preparation from genomic DNA。
- Filetype：fastq或bam等。
- Filename：文件名。如果双端测序就两个文件名。
7. Files
- 提交SRA文件，fastq格式
- FTP或Aspera命令行上传文件夹(>10GB)，或者网页提交(<2GB)或Aspera Connect plugin(2-10GB)上传文件
- 提交之后会检查信息是否错误，按提示修改。
8. Review&Submit：最后检查一遍信息没错误就确认提交。


没什么问题的话，两个工作日内会发邮件告知GenBank accession numbers，可用于文章引用。

-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=20% title="wechat_public_QRcode.png" align=center/>