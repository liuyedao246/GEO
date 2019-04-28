# 用GEOquery从GEO数据库下载数据

#### _jmzeng_

#### _2016年3月12日_

> [Gene Expression Omnibus database (GEO)](http://www.ncbi.nlm.nih.gov/geo/)是由NCBI负责维护的一个数据库，设计初衷是为了收集整理各种表达芯片数据，但是后来也加入了甲基化芯片，甚至高通量测序数据！

[GEO数据库基础知识]("http://www2.warwick.ac.uk/fac/sci/moac/people/students/peter_cock/r/geo/")
--------------------------------------------------------------------------------------

*   GEO Platform (GPL) 芯片平台
    
*   GEO Sample (GSM) 样本ID号
    
*   GEO Series (GSE) study的ID号
    
*   GEO Dataset (GDS) 数据集的ID号 ## 用法
    

> 只需要记住三个函数，以及每个函数返回的对象该如何处理即可

getGEO/getGEOfile/getGEOSuppFiles

这三个函数根据上面的四种ID号下载数据时候，返回的对象还不一样！

### 首先是下载和加载包：

      source("http://www.bioconductor.org/biocLite.R")
      biocLite("GEOquery")
      library(GEOquery)

然后是使用它！
-------

> 首先，我们介绍getGEO函数

*   gds858 <- getGEO(‘GDS858’, destdir=“.”) ##根据GDS号来下载数据，下载soft文件
    
*   gpl96 <- getGEO(‘GPL96’, destdir=“.”) ##根据GPL号下载的是芯片设计的信息！
    
*   gse1009 <- getGEO(‘GSE1009’, destdir=“.”)##根据GSE号下载数据，下载\_series\_matrix.txt.gz
    

下载的文件都会保存在本地，destdir参数指定下载地址。

还有很多其它参数可以调整，学一个函数只需要看看它的帮助即可。

比较重要的三个参数是：GSEMatrix=TRUE,AnnotGPL=FALSE,getGPL=TRUE

返回的对象不一样！针对返回对象的方法也不一样！

### 下载GDS返回的对象

gds858返回的对象很复杂

用Table(gds858)可以得到表达矩阵！

用Meta(gds858)可以得到描述信息

    options(warn=-1)
    suppressMessages(library(GEOquery))
    gds858 <- getGEO('GDS858', destdir=".")
    names(Meta(gds858))
    Table(gds858)[1:5,1:5]

然后还可以用 **GDS2eSet**函数把它转变为expression set 对象

eset <- GDS2eSet(gds858, do.log2=TRUE)

### 下载GSE返回的对象

也就是直接根据GSE号返回的对象：gse1009

我们的处理函数有：**geneNames/sampleNames/pData/exprs**(这个是重点，对expression set 对象的操作函数)

### 下载GPL返回的对象

但是根据GPL号下载返回的对象跟GDS一样，也是用Table/Meta处理！

    options(warn=-1)
    suppressMessages(library(GEOquery))
    gpl96 <- getGEO('GPL96', destdir=".")
    names(Meta(gpl96))
    Table(gpl96)[1:10,1:4]
    ##下面这个就是芯片ID的基因注释信息
    Table(gpl96)[1:10,c("ID","GB_LIST","Gene.Title","Gene.Symbol","Entrez.Gene")]

getGEO除了可以下载数据，还可以打开本地数据！

gds858 <- getGEO(filename=‘GDS858.soft.gz’)

还可以下载所有的cel原始文件！
----------------

    tmp=getGEOSuppFiles(GSE1009)
    if (is.null(tmp)) {
      warning("Supplementary data files not provided!\nyou should check this GEO ID in NCBI\n")
    }
