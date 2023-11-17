---
title: scripts
date: 2021-04-01 10:56:00
categories: 
- programming
- scripts
tags: 
- scripts
- biosoft
- batch_run.sh
- ROUSFinder2.0.py
- extract_intron_info.pl
- multi-transcriptome_assembly_with_ref.sh
- extract_single_copy_genes.sh
- concat_msa_with_differenct_species.sh

description: store some scripts using in biology and bioinformatics
---

<div align="middle"><iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=298 height=52 src="//music.163.com/outchain/player?type=2&id=476751845&auto=1&height=32"></iframe></div>

# 1. 我写的脚本
## 1.1. 多个RNA-seq数据，基于参考基因组的转录本组装和ORF预测
[multi-transcriptome_assembly_with_ref.sh](https://github.com/yanzhongsino/bioscripts/blob/main/my_scripts/multi-transcriptome_assembly_with_ref.sh)

## 1.2. 提取，合并，和比对单拷贝基因和蛋白
[extract_single_copy_genes.sh](https://github.com/yanzhongsino/bioscripts/blob/main/my_scripts/extract_single_copy_genes.sh)

## 1.3. 合并不同物种数量的多序列比对文件，保持序列的比对
[concat_msa_with_differenct_species.sh](https://github.com/yanzhongsino/bioscripts/blob/main/my_scripts/concat_msa_with_differenct_species.sh)

# 2. 我修改的脚本（其他人写的）
## 2.1. 从包含mRNA和CDS的gff注释文件中提取intron的位置和长度信息
[extract_intron_info.pl](https://github.com/yanzhongsino/bioscripts/blob/main/modifiedscripts/extract_intron_info.pl)

# 3. 保存的脚本
## 3.1. batch_run.sh —— 多线程并行运行批量化命令
[batch_run.sh](https://github.com/yanzhongsino/bioscripts/blob/main/saved_scripts/batch_run.sh)

```shell
#!/bin/bash
# batch_run.sh脚本用于多线程并行批量化循环命令，并控制并行数量始终为${thread}。
# usage: 修改thread=后的值为想要设定的并行线程数，把需要并行的命令替换23行的run something。运行脚本batch_run.sh，实现并行运行循环任务。
# ref：网上找的脚本，来源找不到了，侵删。

thread=16

start_time=`date +%s`              # 定义脚本运行的开始时间
echo ${start_time}
tmpFifo=/tmp/$$.fifo # 声明管道名称，$$表示脚本当前运行的进程PID
mkfifo ${tmpFifo} # 创建有名管道
exec 3<>${tmpFifo}                   #创建文件描述符，以可读（<）可写（>）的方式关联管道文件，这时候文件描述符3就有了有名管道文件的所有特性
rm -rf ${tmpFifo}                    #关联后的文件描述符拥有管道文件的所有特性,所以这时候管道文件可以删除，我们留下文件描述符来用就可以了
for ((i=1;i<=${thread};i++))
do
        echo "" >&3                   #&3代表引用文件描述符3，这条命令代表往管道里面放入了一个"令牌"
done

for sample in $(cat sample_list.txt) #并行运行的循环
do
        read -u3                           #代表从管道中读取一个令牌
        {
                echo ${sample}
                run something # 需要并行运行的真实命令
                echo 'success' ${sample}       
                echo "" >&3                   #代表我这一次命令执行到最后，把令牌放回管道
        } &
done

## ps:可以使用while read line; do echo $line; done < file.txt代替for i in $(cat file.txt);do echo $i; done循环。但是不能用cat file.txt | while read line; do echo $line; done代替。

wait
 
stop_time=`date +%s`  #定义脚本运行的结束时间
 
echo "TIME:`expr $stop_time - $start_time`"
exec 3<&-                       #关闭文件描述符的读
exec 3>&-                       #关闭文件描述符的写
```

## 3.2. infernal-tblout2gff.pl —— 基因组ncRNA的infernal注释结果tblout格式转换成gff格式
1. 背景
用Infernal注释基因组的ncRNA，得到的结果可以用脚本整理成gff3格式。

脚本来源：https://www.cnblogs.com/jessepeng/p/15392809.html

2. 脚本

```perl
#!/usr/bin/env perl
# infernal-tblout2gff.pl: convert cmsearch or cmscan tblout files to GFF format

# an important point of above ref:
# "Start is always less than or equal to end"
# EPN, Fri Jun  7 11:07:38 2019
# 
#
use strict;
use warnings;
use Getopt::Long;

my $in_tblout  = "";   # name of input tblout file

my $usage;
$usage  = "infernal-tblout2gff.pl\n\n";
$usage .= "Usage:\n\n";
$usage .= "infernal-tblout2gff.pl [OPTIONS] <cmsearch tblout file>\n\tOR\n";
$usage .= "infernal-tblout2gff.pl --cmscan [OPTIONS] <cmscan tblout file>\n\tOR\n";
$usage .= "infernal-tblout2gff.pl --cmscan --fmt2 [OPTIONS] <cmscan --fmt 2 tblout file>\n\n";
$usage .= "\tOPTIONS:\n";
$usage .= "\t\t-T <n>       : minimum bit score to include is <n>\n";
$usage .= "\t\t-E <x>       : maximum E-value to include is <x>\n";
$usage .= "\t\t--cmscan     : tblout file was created by cmscan\n";
$usage .= "\t\t--source <s> : specify 'source' field should be <s> (e.g. tRNAscan-SE)\n";
$usage .= "\t\t--fmt2       : tblout file was created with cmscan --fmt 2 option\n";
$usage .= "\t\t--all        : output all info in 'attributes' column   [default: E-value]\n";
$usage .= "\t\t--none       : output no info in 'attributes' column    [default: E-value]\n";
$usage .= "\t\t--desc       : output desc field in 'attributes' column [default: E-value]\n";
$usage .= "\t\t--version <s>: append \"-<s>\" to 'source' column\n";
$usage .= "\t\t--extra <s>  : append \"<s>;\" to 'attributes' column\n";
$usage .= "\t\t--hidedesc   : do not includ \"desc\:\" prior to desc value in 'attributes' column\n";

my $do_minscore = 0;       # set to '1' if -T used
my $do_maxevalue = 0;      # set to '1' if -E used
my $minscore   = undef;    # defined if if -T used
my $maxevalue  = undef;    # defined if -E used
my $do_cmscan  = 0;        # set to '1' if --cmscan used
my $do_fmt2    = 0;        # set to '1' if --fmt
my $do_all_attributes = 0; # set to '1' if --all
my $do_no_attributes  = 0; # set to '1' if --none
my $do_de_attributes  = 0; # set to '1' if --desc
my $version = undef;       # defined if --version used
my $extra = undef;         # defined if --extra used
my $do_hidedesc = 0;       # set to 1 if --hidedesc used
my $opt_source = undef;    # set to <s> if --source <s> used

&GetOptions( "T=s"       => \$minscore,
             "E=s"       => \$maxevalue,
             "cmscan"    => \$do_cmscan,
             "source=s"  => \$opt_source,
             "fmt2"      => \$do_fmt2,
             "all"       => \$do_all_attributes,
             "none"      => \$do_no_attributes,
             "desc"      => \$do_de_attributes,
             "version=s" => \$version, 
             "extra=s"   => \$extra,
             "hidedesc"  => \$do_hidedesc);

if(scalar(@ARGV) != 1) { die $usage; }
my ($tblout_file) = @ARGV;

if(defined $minscore)  { $do_minscore  = 1; }
if(defined $maxevalue) { $do_maxevalue = 1; }
if($do_minscore && $do_maxevalue) { 
  die "ERROR, -T and -E cannot be used in combination. Pick one.";
}
if(($do_all_attributes) && ($do_no_attributes)) { 
  die "ERROR, --all and --none cannot be used in combination. Pick one.";
}
if(($do_all_attributes) && ($do_de_attributes)) { 
  die "ERROR, --all and --desc cannot be used in combination. Pick one.";
}
if(($do_no_attributes) && ($do_de_attributes)) { 
  die "ERROR, --none and --desc cannot be used in combination. Pick one.";
}
if(($do_fmt2) && (! $do_cmscan)) { 
  die "ERROR, --fmt2 only makes sense in combination with --cmscan"; 
}
if((defined $opt_source) && (defined $version)) { 
  die "ERROR, --source and --version are incompatible";
}

if(! -e $tblout_file) { die "ERROR tblout file $tblout_file does not exist"; }
if(! -s $tblout_file) { die "ERROR tblout file $tblout_file is empty"; }

my $source = ($do_cmscan) ? "cmscan" : "cmsearch";
if(defined $version)    { $source .= "-" . $version; }
if(defined $opt_source) { $source = $opt_source; }

open(IN, $tblout_file) || die "ERROR unable to open $tblout_file for reading"; 
my $line;
my $i;
while($line = <IN>) { 
  if($line !~ m/^\#/) { 
    chomp $line;
    my @el_A = split(/\s+/, $line);
    my $nfields = scalar(@el_A);
    if((! $do_fmt2) && ($nfields < 18)) { 
      die "ERROR expected at least 18 space delimited fields in tblout line (fmt 1, default) but got $nfields on line:\n$line\n"; 
    }
    if(($do_fmt2) && ($nfields < 27)) { 
      die "ERROR expected at least 27 space delimited fields in tblout line (fmt 2, --fmt2) but got $nfields on line:\n$line\n"; 
    }
    # ref Infernal 1.1.2 user guide, pages 59-61
    my $idx     = undef;
    my $seqname = undef;
    my $seqaccn = undef;
    my $mdlname = undef;
    my $mdlaccn = undef;
    my $clan    = undef;
    my $mdl     = undef;
    my $mdlfrom = undef;
    my $mdlto   = undef;
    my $seqfrom = undef;
    my $seqto   = undef;
    my $strand  = undef;
    my $trunc   = undef;
    my $pass    = undef;
    my $gc      = undef;
    my $bias    = undef;
    my $score   = undef;
    my $evalue  = undef;
    my $inc     = undef;
    my $olp     = undef;
    my $anyidx  = undef;
    my $anyfrct1= undef;
    my $anyfrct2= undef;
    my $winidx  = undef;
    my $winfrct1= undef;
    my $winfrct2= undef;
    my $desc    = undef;

    if($do_fmt2) { # 27 fields
      $idx     = $el_A[0];
      $seqname = ($do_cmscan) ? $el_A[3] : $el_A[1];
      $seqaccn = ($do_cmscan) ? $el_A[4] : $el_A[2];
      $mdlname = ($do_cmscan) ? $el_A[1] : $el_A[3];
      $mdlaccn = ($do_cmscan) ? $el_A[2] : $el_A[4];
      $clan    = $el_A[5];
      $mdl     = $el_A[6];
      $mdlfrom = $el_A[7];
      $mdlto   = $el_A[8];
      $seqfrom = $el_A[9];
      $seqto   = $el_A[10];
      $strand  = $el_A[11];
      $trunc   = $el_A[12];
      $pass    = $el_A[13];
      $gc      = $el_A[14];
      $bias    = $el_A[15];
      $score   = $el_A[16];
      $evalue  = $el_A[17];
      $inc     = $el_A[18];
      $olp     = $el_A[19];
      $anyidx  = $el_A[20];
      $anyfrct1= $el_A[21];
      $anyfrct2= $el_A[22];
      $winidx  = $el_A[23];
      $winfrct1= $el_A[24];
      $winfrct2= $el_A[25];
      $desc    = $el_A[26]; 
      for($i = 27; $i < $nfields; $i++) { $desc .= "_" . $el_A[$i]; }
    }
    else { # fmt 1, default
      $seqname = ($do_cmscan) ? $el_A[2] : $el_A[0];
      $seqaccn = ($do_cmscan) ? $el_A[3] : $el_A[1];
      $mdlname = ($do_cmscan) ? $el_A[0] : $el_A[2];
      $mdlaccn = ($do_cmscan) ? $el_A[1] : $el_A[3];
      $mdl     = $el_A[4];
      $mdlfrom = $el_A[5];
      $mdlto   = $el_A[6];
      $seqfrom = $el_A[7];
      $seqto   = $el_A[8];
      $strand  = $el_A[9];
      $trunc   = $el_A[10];
      $pass    = $el_A[11];
      $gc      = $el_A[12];
      $bias    = $el_A[13];
      $score   = $el_A[14];
      $evalue  = $el_A[15];
      $inc     = $el_A[16];
      $desc    = $el_A[17]; 
      for($i = 18; $i < $nfields; $i++) { $desc .= "_" . $el_A[$i]; }
    }
    # one sanity check, strand should make sense
    if(($strand ne "+") && ($strand ne "-")) { 
      if(($do_fmt2) && (($seqfrom eq "+") || ($seqfrom eq "-"))) { 
        die "ERROR problem parsing, you specified --fmt2 but tblout file appears to have NOT been created with --fmt 2, retry without --fmt2\nproblematic line:\n$line\n"; 
      }
      if((! $do_fmt2) && (($pass eq "+") || ($pass eq "-"))) { 
        die "ERROR problem parsing, you did not specify --fmt2 but tblout file appears to have been created with --fmt 2, retry with --fmt2\nproblematic line:\n$line\n"; 
      }
      die "ERROR unable to parse, problematic line:\n$line\n";
    }
    if(($do_minscore) && ($score < $minscore)) { 
      ; # skip
    }
    elsif(($do_maxevalue) && ($evalue > $maxevalue)) { 
      ; # skip
    }
    else { 
      my $attributes = "evalue=" . $evalue; # default to just evalue
      if($do_all_attributes) { 
        if($do_fmt2) { 
          $attributes .= sprintf(";idx=$idx;seqaccn=$seqaccn;mdlaccn=$mdlaccn;clan=$clan;mdl=$mdl;mdlfrom=$mdlfrom;mdlto=$mdlto;trunc=$trunc;pass=$pass;gc=$gc;bias=$bias;inc=$inc;olp=$olp;anyidx=$anyidx;anyfrct1=$anyfrct1;anyfrct2=$anyfrct2;winidx=$winidx;winfrct1=$winfrct1;winfrct2=$winfrct2;%s$desc", ($do_hidedesc ? "" : "desc="));
        }
        else { 
          $attributes .= sprintf(";seqaccn=$seqaccn;mdlaccn=$mdlaccn;mdl=$mdl;mdlfrom=$mdlfrom;mdlto=$mdlto;trunc=$trunc;pass=$pass;gc=$gc;bias=$bias;inc=$inc;%s$desc", ($do_hidedesc ? "" : "desc="));
        }
      }
      elsif($do_no_attributes) { 
        $attributes = "-";
      }
      elsif($do_de_attributes) { 
        $attributes = $desc;
      }
      if(defined $extra) { 
        if($attributes eq "-")       { $attributes = ""; }
        elsif($attributes !~ m/\;$/) { $attributes .= ";"; }
        $attributes .= $extra . ";";
      }
      printf("%s\t%s\t%s\t%d\t%d\t%.1f\t%s\t%s\t%s\n", 
             $seqname,                             # token 1: 'sequence' (sequence name)
             $source,                              # token 2: 'source'
             $mdlname,                             # token 3: 'feature' (model name) you may want to change this to 'ncRNA'
             ($strand eq "+") ? $seqfrom : $seqto, # token 4: 'start' in coordinate space [1..seqlen], must be <= 'end'
             ($strand eq "+") ? $seqto : $seqfrom, # token 5: 'end' in coordinate space [1..seqlen], must be >= 'start'
             $score,                               # token 6: 'score' bit score
             $strand,                              # token 7: 'strand' ('+' or '-')
             ".",                                  # token 8: 'phase' irrelevant for noncoding RNAs
             $attributes);                         # token 9: attributes, currently only E-value, unless --all, --none or --desc
    }
  }
}
```

3. 运行
`perl infernal-tblout2gff.pl --cmscan --fmt2 genome.tblout >genome.infernal.ncRNA.gff3`

输入是Infernal的table格式输出文件，输出是gff3格式的ncRNA注释文件。


-------

- 欢迎关注微信公众号：**生信技工**
- 公众号主要分享生信分析、生信软件、基因组学、转录组学、植物进化、生物学概念等相关内容，包括生物信息学工具的基本原理、操作步骤和学习心得。

<img src="https://github.com/yanzhongsino/yanzhongsino.github.io/blob/hexo/source/wechat/Wechat_public_qrcode.jpg?raw=true" width=50% title="wechat_public_QRcode.png" align=center/>