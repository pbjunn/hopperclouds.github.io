<h2>使用R并行方式对数值型数据离散化</h2>

数据的特征按照其取值可以分为连续型和离散型。离散数值属性在数据挖掘的过程中具有重要的作用。比如在信用卡评分模型中，当自变量很多时，并非所有字段对于目标字段来说都是有效的，因此通常的做法是通过计算woe值和iv值(类似于信息增益)来初步挑选通过对目标变量重要的字段，然后建模逻辑回归模型。而这当中就需要对数值型数据离散化。
数值型数据离散化通常分为有监督离散化和无监督离散化。考虑到数据建模通常是建立目标字段和其影响因素之间的关系的量化，因此会选择有监督离散化。
R语言中用于数值型数据离散化的包discretization。安装和加载如下：
```r
> install.packages("discretization")
> library(discretization)
```
<!--more-->
以R自带数据集iris为例，以"Species" 为目标字段，对"Sepal.Length"、"Sepal.Width"、"Petal.Length"、"Petal.Width"四个数值型属性离散化。

```r
> lisan_result <- mdlp(iris)
> class(lisan_result)
[1] "list"
> names(lisan_result)
[1] "cutp"      "Disc.data"
```

使用mdlp()方法对iris离散化，该方法默认数据框最后一列最后为目标字段。返回结果为列表。"cutp"为各列的分割点向量。"Disc.data"为离散化后的数据框。
该方法对于较小的样本量和维度时，程序运行时间还可以接受。但随着数据量的增大，数据维度的增加，程序运行时间会越来越长。因此考虑采用并行的方式对数据进行离散化。介绍R用于离散化的包parallel。
```r
> install.packages("parallel")
> library(parallel)
> cores <- detectCores()  ##查看本机虚拟核心数
cores
[1] 4
```
现在考虑以并行的方式实现离散化方法。考虑设计思路如下：
1.将字段10个为一组分别与目标字段组合成数据框，(不足10个时以实际数量字段与目标字段组合)存放在一个列表中。列表的元素即离散字段与目标字段构成的数据框。
2.启动M个附属进程，并初始化。M<=本机虚拟核心数。使用parLapply()作用于步骤1中建立的列表数据。此时既有M个附属进程对数据进行离散化。
3.将步骤2中的离散化结果合并。
4.将上述步骤封装成函数。整理后使得返回结果与mdlp()函数一致。这样方便调用。
将上述设计思路写成R代码，如下：输入离散数据、使用核心数，返回结果与使用mdlp()函数相同
```r
parallel_lisan <- function(lisan_data,cores_num){

   library(parallel)
   library(discretization)

    res <- list()
    lisan_data_v <- list()
    cut_point <- list()
    Disc.data <- data.frame(c(rep(NA,nrow(lisan_data))))

    name_num = ncol(lisan_data)-1      ##将原始数据分割成多列，先考虑每组10列。不足的单独分为一组。
    group_num = floor(name_num/10)
    last_group_num = name_num%%10

    if( name_num > 10){                ##当原始数据列数多余10列
        for(i in 1:group_num){
            lw_flag <- lisan_data[,ncol(lisan_data)]
            lisan_data_v[[i]] <- cbind(lisan_data[,(10*i-9):(10*i)],lw_flag)
          }
        lisan_data_v[[group_num+1]] <- lisan_data[,(10*group_num+1):ncol(lisan_data)]

    }else{
        lisan_data_v[[group_num+1]] <- lisan_data[,(10*group_num+1):ncol(lisan_data)]
    }

    cl <- makeCluster(cores_num)                   ##初始化核心
    results <- parLapply(cl,lisan_data_v,mdlp)     ##对列表数据使用mdlp函数并行离散化

    for(i in 1:length(results)){
         for(j in 1:length(results[[i]][[1]])){
              cut_point[[(i-1)*10+j]] <- results[[i]][[1]][[j]]
          }
        temp <- as.data.frame(results[[i]][[2]])
        Disc.data <- cbind(Disc.data,temp[,1:(ncol(temp)-1)])      ##合并离散数据结果
    }

    Disc.data <- Disc.data[,2:ncol(Disc.data)]
    Disc.data$lw_flag <- lisan_data[,ncol(lisan_data)]
    names(Disc.data) <- names(lisan_data)
    stopCluster(cl)

    res[["cutp"]] <- cut_point
    res[["Disc.data"]] <- Disc.data
   
    return(res)
  }
```
