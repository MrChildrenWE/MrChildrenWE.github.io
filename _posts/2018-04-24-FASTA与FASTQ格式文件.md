---
layout:     post   				          # 使用的布局（不需要改）
title:      FASTA与FASTQ格式文件			# 标题 
subtitle:   fasta 与 fastq 格式的详细内容              #副标题
date:       2018-04-24		      # 时间
author:     Mr.Children						# 作者
header-img: img/1.jpg 	#这篇文章标题背景图片
catalog: true 						# 是否归档
tags:								#标签
    - NGS
---
#### 1. FASTA 文件
> FASTA文件主要由两个部分组成:序列头信息(有时包括一些描述信息)和具体的序列数据.头信息独占一行,以大于号(>) 开头作为标记

```
>ENSMUSG00000020122|ENSMUST00000138518
CCCTCCTATCATGCTGTCAGTGTATCTCTAAATAGCACTCTCAACCCCCGTGAACTTGGT
TATTAAAAACATGCCCAAAGTCTGGGAGCCAGGGCTGCAGGGAAATACCACAGCCTCAGT
TCATCAAAACAGTTCATTGCCCAAAATGTTCTCAGCTGCAGCTTTCATGAGGTAACTCCA
GGGCCCACCTGTTCTCTGGT
```
>头部信息没有严格的限制.但一般用空格把头信息分为两个部分:第一部分序列名字紧贴> [注释信息]

```
>gene_00284728 length=231;type=dna
GAGAACTGATTCTGTTACCGCAGGGCATTCGGATGTGCTAAGGTAGTAATCCATTATAAGTAACATG
CGCGGAATATCCGGGAGGTCATAGTCGTAATGCATAATTATTCCCTCCCTCAGAAGGACTCCCTTGC
```

#### 2.FASTQ 文件
> 这是目前存储测序数据最普遍、最公认的一个数据格式,另一个是uBam格式,上面所讲的FASTA文件,它所存的都是已经排列好的序列（如参考序列）,FASTQ存的则是产生自测序仪的原始测序数据,它由测序的图像数据转换过来,也是文本文件,文件大小依照不同的测序量（或测序深度）而有很大差异,小的可能只有几M,大的则常常有几十G上百G,文件后缀通常都是.fastq,.fq或者.fq.gz(gz压缩)

```
@DJB775P1:248:D0MDGACXX:7:1202:12362:49613
TGCTTACTCTGCGTTGATACCACTGCTTAGATCGGAAGAGCACACGTCTGAA
+
JJJJJIIJJJJJJHIHHHGHFFFFFFCEEEEEDBD?DDDDDDBDDDABDDCA
@DJB775P1:248:D0MDGACXX:7:1202:12782:49716
CTCTGCGTTGATACCACTGCTTACTCTGCGTTGATACCACTGCTTAGATCGG
+
IIIIIIIIIIIIIIIHHHHHHFFFFFFEECCCCBCECCCCCCCCCCCCCCCC
```
它有自己的格式,每四行成为一个独立的单元,我们称之为read.具体的格式描述如下：
###### 第一行：以‘@’开头,是这一条read的名字,这个字符串是根据测序时的状态信息转换过来的,中间不会有空格,它是每一条read的唯一标识符,同一份FASTQ文件中不会重复出现,甚至不同的FASTQ文件里也不会有重复； 
###### 第二行：测序read的序列,由A,C,G,T和N这五种字母构成,这也是我们真正关心的DNA序列,N代表的是测序时那些无法被识别出来的碱基；
###### 第三行：以‘+’开头,在旧版的FASTQ文件中会直接重复第一行的信息,但现在一般什么也不加（节省存储空间）；
###### 第四行：测序read的质量值,这个和第二行的碱基信息一样重要,它描述的是每个测序碱基的可靠程度,用ASCII码表示.


#### 3.质量值
&emsp;&emsp;碱基质量值就是能够用来定量描述碱基好坏程度的一个数值.它该如何才能恰当地描述这个结果呢？我们试想一下,如果测序测得越准确,这个碱基的质量就应该越高；反之,测得越不准确,质量值就应该越低.也就是说可以利用碱基被测错的概率来描述它的质量值,错误率越低,质量值就越高！  
如下图,红线代表错误率,蓝线代表准确率（反映质量值）,这便是我们希望达到的效果：

![](http://wx4.sinaimg.cn/mw690/8aa6de51ly1fr1p783tz8j20c509vdgu.jpg)

这里我们假定碱基的测序错误率为p_error,质量值为Q,它们之间的关系如下:  
`Q = -10log(p_error)`  
即,质量值是测序错误率的对数（10为底数）乘以-10（并取整）.这个公式也是目前测序质量值的计算公式,它非常简单,p_error的值和测序时的多个因素有关,体现为测序图像数据点的清晰程度,并由测序过程中的base calling 算法计算出来；  
&emsp;&emsp;公式左边的Q我们称之为Phred quality score,就是用它来描述测序碱基的靠谱程度.比如,如果该碱基的测序错误率是0.01,那么质量值就是20（俗称Q20）,如果是0.001,那么质量值就是30（俗称Q30）.Q20和Q30的比例常常被我们用来评价某次测序结果的好坏,比例越高就越好.  
下面一个表,更进一步地解释质量值高低的含义：
![](http://wx3.sinaimg.cnmw690/8aa6de51ly1fr1p7d2xa8j20hs06oq58.jpg)  
&emsp;&emsp;为了格式存储以及处理时的方便,这个数字被直接转换成了ASCII码,并与第二行的read序列构成一一对应的关系——每一个ASCII码都和它正上面的碱基对应,这就很完美.  
&emsp;&emsp;不过,值得一提的是,ASCII码虽然能够从小到大表示0-127的整数,但是并非所有的ASCII码都是可见的字符,比如所有小于33的ASCII码值所表示的都是不可见字符,比如空格,换行符等,因此为了能够让碱基的质量值表达出来,必须避开所有这些不可见字符.最简单的做法就是加上一个固定的整数.
&emsp;&emsp;但一开始对于要加哪一个整数,并没有什么指导标准,这就导致了在刚开始的时候,不同的测序平台加的整数也不同,总的来说有以下3种质量体系,演变到现在也基本只剩下第一种了,
![](http://wx4.sinaimg.cn/mw690/8aa6de51ly1fr1p7grcjtj20hs04sacj.jpg)  
&emsp;&emsp;从表中可以看到下限有33和64两个值,我们把加33的的质量值体系称之为Phred33,加64的称之为Phred64（Solexa的除外,它叫Solexa64).不过,**现在一般都是使用Phred33这个体系,而且33也恰好是ASCII的第一个可见字符.**
&emsp;&emsp;如果你在实际做项目的过程不知道所用的质量体系（经验丰富者是可以直接看出来的）,那么可以用我下面这一段代码,简单地做个检查:
```
less $1 | head -n 1000 | awk '{if(NR%4==0) printf("%s",$0);}' \
| od -A n -t u1 -v \
| awk 'BEGIN{min=100;max=0;} \
  {for(i=1;i<=NF;i++) {if($i>max) max=$i; if($i<min) min=$i;}}END \
  {if(max<=126 && min<59) print "Phred33"; \
  else if(max>73 && min>=64) print "Phred64"; \
  else if(min>=59 && min<64 && max>73) print "Solexa64"; \
  else print "Unknown score encoding"; \
  print "( " min ", " max, ")";}'
```
&emsp;&emsp;将上面这段代码复制到任意一份shell文件中（比如：fq_qual_type.sh）,就可以用它来进行质量值类型的检查了.代码的思路其实比较简单,就是截取FASTQ文件的前1000行数据,并抽取出质量值所在的行,分别计算出其中最小和最大的ASCII值,再比较一下就判断出来了.下面给出一个例子,这是我们在本文中用到的FASTQ文件,它是Phred33的：
```
$ sh fq_qual_type.sh untreated.fq
Phred33
( 34, 67 )
```
&emsp;&emsp;另外,在查看碱基质量值的过程中,如果你心中存有ASCII码表当然可以直接“看”出各个碱基的质量值,但在实际的场景中都是通过程序直接进行转换处理.下面我就用Python的ord()函数举个转换的例子：
```
qual='JJJJJIIJJJJJJHIHHHGHFFFFFFCEEEEEDBD'

[ord(q)-33 for q in qual]

[41, 41, 41, 41, 41, 40, 40, 41, 41, 41, 41, 41, 41, 39, 40, 39, 39, 39, 38,
 39, 37, 37, 37, 37, 37, 37, 34, 36, 36, 36, 36, 36, 35, 33, 35]
 ```
&emsp;&emsp;这里的ord()函数会将字符转换为ASCII对应的数字,减掉33后就得到了该碱基最后的质量值（即,Phred quality score）.  
&emsp;&emsp;另外,根据上面phred quality score的计算公式,我们可以很方便地获得每个测序碱基的错误率,这个错误率在我们的比对和变异检测中都十分重要,还是以上述qual为例子：
```
qual='JJJJJIIJJJJJJHIHHHGHFFFFFFCEEEEEDBD'
phred_score = [ord(q)-33 for q in qual]
[10**(-q/10.0) for q in phred_score]
[7.9e-05, 7.9e-05, 7.9e-05, 7.9e-05, 7.9e-05, 0.0001, 0.0001, 7.9e-05,
 7.9e-05, 7.9e-05, 7.9e-05, 7.9e-05, 7.9e-05, 0.000126, 0.0001, 0.000126,
 0.000126, 0.000126, 0.000158, 0.000126, 0.0002, 0.0002, 0.0002, 0.0002,
 0.0002, 0.0002, 0.000398, 0.000251, 0.000251, 0.000251, 0.000251, 0.000251,
 0.000316, 0.000501, 0.000316]
```
>这其实就是根据phred quality score的定义进行简单的对数运算.




