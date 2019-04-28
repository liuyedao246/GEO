# GEO数据库挖掘笔记

## 安装软件

### R语言环境变量的设置 环境变量函数为options（）

环境设置函数为options（），用options（）命令可以设置一些环境变量，使用help（options）可以查看详细的参数信息。

1. 数字位数的设置，options（digits = n），n一般默认情况下是7位，但实际上的范围是1~22，可以随意设置位数。

   ~~~R
   #这个命令，可以把R的整数表示能力设为10位。
   options（digits = 10）
   ~~~

   2. 扩展包的安装，使用下面的命令，可以联网安装扩展包。

      ~~~R
      options（CRAN = "http://cran.r-project.org"）
      install.packages("扩展包名")
      ~~~

   3. 利用R的options（）函数进行光标设置。

      ~~~R
      # 可以随意设置你的光标类型（prompt参数设置）
      options(prompt = "|")
      # 光标为“|”
      ~~~

   4. R里的options（）函数进行错误信息显示（忽略）设置

      ~~~R
      # 这个命令可以忽视任何警告
      options(warn = -1)
      # 这个命令，不放过任何警告
      options(warn = 1)
      ~~~

   5. options（）常用于设置R控制台、R语言计算相关的属性，常用属性名称几默认值如下：

      ~~~R
      add.smooth TRUE
      check.bounds FALSE
      continue "+ "
      digits 7
      echo TRUE
      encoding "native.enc"
      error NULL
      expressions 5000
      keep.source interactive()
      keep.source.pkgs FALSE
      max.print 99999
      OutDec "."
      prompt "> "
      scipen 0
      show.error.messages TRUE
      timeout 60
      verbose FALSE
      warn 0
      warning.length 1000
      width 80
      # 装载不同的扩展包还会增加一些新的属性信息
      ~~~

### bioconductor介绍

Bioconductor provides tools for the analysis and comprehension of  high-throughput genomic data. Bioconductor uses the R statistical  programming language, and is open source and open development. It  has two releases each year,and an active user community. 

[Install](https://www.bioconductor.org/install/#install-R) the latest release of R, then get the latest version of  *Bioconductor* by starting R and entering the commands

```R
# 安装bioconductor包
if (!requireNamespace("BiocManager"))
    install.packages("BiocManager")
BiocManager::install()
```

## 从GEO数据库下载数据

### GEO数据库知识

> Gene Expression Omnibus database(GEO)是由NCBI负责维护的一个数据库，设计初衷是为了收集整理各种表达芯片，但是后来也加入了甲基化芯片，甚至高通量测序数据！

**GEO数据库包含下面四种信息**

- GEO Platform （GPL）芯片平台
- GEO Sample （GSM）样本ID号
- GEO Series （GSE） study的ID号
- GEO Dataset（GDS） 数据集的ID号

### 下载方法

使用“GEOquery”包中的3个函数getGEO/getGEOfile/getGEOSuppfiles，根据上面四种ID号下载数据，但是，返回的对象不一样。

1. 首先下载和加载包：

   ```R
BiocManager::install('GEOquery')
library(GEOquery)
   ```

2. 使用GEOquery包：

   ```R
   gds858 <- getGEO(‘GDS858’, destdir=“.”) ##根据GDS号来下载数据，下载soft文件
   gpl96 <- getGEO(‘GPL96’, destdir=“.”) ##根据GPL号下载的是芯片设计的信息！
   gse1009 <- getGEO(‘GSE1009’, destdir=“.”)##根据GSE号下载数据，下载_series_matrix.txt.gz
   ```

   下载的文件都会保存到本地，destdir参数指定下载地址；比较重要的3个参数：GSEMatrix = TRUE， AnnotGPL = FALSE， getGPL = TRUE。因为返回的对象不一样，针对不同的返回对象的处理方法也不一样。

   1. 下载GDS返回的对象

      gds858返回的对象很复杂，用Table（gds858）可以得到表达矩阵，用Meta（gds858）可以得到描述信息。

      ```R
      options(warn=-1)
      suppressMessages(library(GEOquery))
      gds858 <- getGEO('GDS858', destdir=".")
      names(Meta(gds858))
      Table(gds858)[1:5,1:5]
      ```

      然后用GDS2eSet函数把它转变为expression set对象

      ```R
      eset <- GDS2eSet(gds858, do.log2=TRUE)
      ```

      b. 下载GSE返回的对象

      处理GSE返回对象的函数有：geneNames/sampleNames/pData/exprs

      c. 下载GPL返回对象

      但是根据GPL号下载所返回的对象跟GDS一样，也可以用Table/Meta处理！ 

      ```R
      options(warn=-1)
      suppressMessages(library(GEOquery))
      gpl96 <- getGEO('GPL96', destdir=".")
      names(Meta(gpl96))
      Table(gpl96)[1:10,1:4]
      ##下面这个就是芯片ID的基因注释信息
      Table(gpl96)[1:10,c("ID","GB_LIST","Gene.Title","Gene.Symbol","Entrez.Gene")]
      ```

getGEO除了可以下载数据，还可以打开本地数据，

```R
gds858 <- getGEO(filename=‘GDS858.soft.gz’)
```

还可以下载所有的cel原始文件！

```R
tmp=getGEOSuppFiles(GSE1009)
if (is.null(tmp)) {
  warning("Supplementary data files not provided!\nyou should check this GEO ID in NCBI\n")
}
```

### 补充知识：ExpressionSet对象讲解

> 这个对象其实是对表达矩阵加上样本信息的一个封装，由biobase这个包引入。它是eSet这个对象的继承。这个对象其实很简单，就是表达矩阵加上样本分组信息的一个封装。

在Biobase基础包中，ExpressionSet是非常重要的类，因为Bioconductor设计之初是为了对基因芯片数据进行分析，而ExpressionSet正是Bioconductor为基因表达数据格式所定制的标准。它是所有涉及基因表达量相关数据在Bioconductor中进行操作的基础数据类型，比如affyPLM，affy，oligo，limma，arrayMagic等等。

ExpressionSet 的组成：

- assayData: 一个matrix类型或者environment类型数据。用于保存表达数据值。当它是一个matrix时，它的行表示不同的探针组（probe sets）（也是features，总之是一个无重复的索引值）的值，它的列表示不同的样品。如果有行号或者列号的话，那么行号必须与featureData及phenoData中的行号一致，列号就是样品名。当我们使用exprs()方法时，就是调取的这个assayData的matrix。当它是一个enviroment时，它必须有两个变量，一个就是与上一段描述一致的matrix，另一个就是epxrs，而这个exprs会响应exprs()方法，返回表达值。
- 头文件：用于描述实验平台相关的数据，其中包括phenoData, featureData，protocolData以及annotation等等。其中phenoData是一个存放样品信息的data.frame或者AnnotatedDataFrame类型的数据。如果有行号的话，其行号必须与assayData的列号一致（也就是样品名）。如果没有行号，则其行数必须与assayData的列数一致。featureData是一个存放features的data.frame或者AnnotatedDataFrame类型的数据。它的行数必须与assayData的行数一致。如果有行号的话，那么它的行号必须和assayData的行号一致。annotation是用于存放芯片类型的字符串，比如hgu95av2之类。protocolData用于存放设备相当的数据。它是AnnotatedDataFrame类型。它的维度必须与assayData的维度一致。
- experimentData: 
  一个MIAME类型的数据，它用于保存和实验设计相关的资料，比如实验室名，发表的文章，等等。那么什么是MIAME类呢？MIAME是Minimum 
  Information About a Microarray Experiment的首字母缩写，它包括以下一些属性（slots）：
  1. name：字符串，实验名称；
  2. lab：字符串，实验室名称；
  3. contact：字符串，联系方式；
  4. title：字符串，一句话描述实验的内容；
  5. abstract：字符串，实验摘要；
  6. url：字符串，实验相关的网址；
  7. samples：list类，样品的信息；
  8. hybridizations： list类，样品的信息；
  9. normControls：list类，对照信息，比如一些持家基因（house keeping genes）
  10. preprocessing：list类，原始数据预处理过程；
  11. pubMedlds：字符串，pubMed索引号；
  12. others：list类，其他相关信息。

ExpressionSet继承了eSet类，属性基本和eSet保持一致。

那么，对于一个ExpressionSet，哪些属性是必须的？哪些有可能缺失呢？很显然，assayData是必须的，其它的可能会缺失，但是不能都缺失，因为那样的话就无法完成数据分析的工作。

**重点**是“exprs”函数提取表达矩阵，“pData”函数看对象的样本分组信息。