之前得推文中介绍了grep和awk这两个Linux常用得模式匹配、字符搜索命令，在这篇推文中，将介绍另外一个常用得命令——sed。

Linux中的grep、awk以及sed被称为文本处理三剑客，三者的功能虽都是处理文本，但侧重点各不相同，grep更适合单纯的查找或匹配文本，awk更适合格式化文本，对文本进行较复杂格式处理，sed则更适合编辑匹配到的文本。掌握好这三个命令，能极大提高我们处理生物数据的效率。

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/K5uWlWfKFaUZbU2tiaI4BVfBeWIaeYO3Se3PMLny5VBbfHx3FYmdynQ87CymE29K2MkcSvRfSeibvcype5BENCtw/640?wx_fmt=png&wxfrom=13)

sed可以看作是是一个流编辑器，用于对输入流（文件或来自管道的输入）执行基本的文本转换。其基本的使用格式为：

```
sed 'OPERATION/REGEXP/REPLACEMENT/FLAGS' FILENAME
```

OPERATION是指定要执行的操作，最常见和广泛使用的操作是s，它执行替换操作。其他常用的操作符包括y用于转换，i用于插入，d用于删除等。

REGEXP和REPLACEMENT分别指定正在执行的操作的搜索项和替代项。

FLAGS是控制操作的附加参数。一些常见的FLAGS包括：

g：用REPLACEMENT替换所有REGEXP（全局替换） 

N：N是任何数字，用于用REPLACEMENT替换每一行中第N个REGEXP

i：忽略匹配REGEXP时的大小写

以下介绍一些sed在处理生物数据文件中常见的用法：

- 假设我们有一个FASTA文件（文件名称为test.fasta），其中有5条序列：
```
>Sequence1
ATCGATCGATCGATCG
>Sequence2
GATCGATCGATCGATC
>Sequence3
TGCATGCATGCATGCA
>Sequence4
CGATCGATCGATCGAT
>Sequence5
AGCTAGCTAGCTAGCT
```

- 将文件中的Sequence替换成Seq
    

```
sed 's/Sequence/Seq/g' test.fasta > NEWFILE
```

*由于sed不会直接在原文件上进行替换，所以需要写入一个新的文件。如果想直接在原文件上替换，需要加上参数-i

```
sed -i 's/Sequence/Seq/g' test.fasta
```

- 如果还想保留一个备份文件，则需要在-i后面加上一个后缀，例如.old
    

```
sed -i.old 's/Sequence/Seq/g' test.fasta
```

- 将每条序列中出现的第一个A替换成T
    

```
sed 's/A/T/1' test.fasta > NEWFILE
```

- sed 默认情况下是将整个文件作为输入流，如果我们只对某一行感兴趣，则需要加上参数-n，例如输出文件中的第二行
    

```
sed -n '2p' test.fasta
```

- 使用逗号分隔可以表示范围，例如第2行到第10行
    

```
sed -n '2,10p' test.fasta
```
- 删除特定行
    

```
# 删除第一行
sed '1d' test.fasta
# 删除1-3行
sed '1,3d' test.fasta
```

- 删除空白行
    

```
# 删除空白行
sed 's/^$//g' test.fasta
```

- 在特定行插入内容
    

```
# 在第一行插入
sed '1 i # Mysequence' test.fasta
```

![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/K5uWlWfKFaUZbU2tiaI4BVfBeWIaeYO3Sl5dbjAybSUiaA38Hxz5wibHKhic5icmTNLzsbeqgibWEsNocCoLSVicXbWfg/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)