​                

 

# 使用dplyr进行数据操作

来源：http://bioconnector.org/workshops/r-dplyr-yeast.html

 

数据分析涉及大量的清理工作——整理和清理数据以利于进行下游分析。**该课程主要讲述高级数据操作技术和利用“拆分-应用-组合”策略进行分析**。

我们将使用R中的`dplyr`包进行有效地操作和计算包含许多观测值的大数据子集的统计特征。

 

> Recommended reading: Review the [Introduction (10.1)](http://r4ds.had.co.nz/tibbles.html#introduction-4) and [Tibbles vs. data.frame (10.3)](http://r4ds.had.co.nz/tibbles.html#tibbles-vs.data.frame) sections of the R for Data Science book. We will initially be using the read_* functions from the [readr package](http://readr.tidyverse.org/).  These functions load data into a tibble instead of R’s traditional  data.frame. Tibbles are data frames, but they tweak some older behaviors  to make life a little easier. These sections explain the few key small  differences between traditional data.frames and tibbles.

 

**好，那我们先看看这两部分。**我捡要点讲。

 

## Introduction (10.1)

 

一个典型的数据科学项目中需要的工具模型应该是这样的：

![img](http://r4ds.had.co.nz/diagrams/data-science.png)

 

1. 首先你要**导入数据**（import）。这意思是把你存储在文件、数据库中或者来自网络API中的数据存储为R中的数据框。
2. 导入之后是**整理数据**（tidy）。简要来说，当你的数据是干净的时候，每一列应该是一个变量，每一行是一个观察值。
3. 整理之后是**转换数据**（transform）。转换包括缩小感兴趣的观察值，根据已知变量创造新的变量，计算一系列的统计特征。将整理和转换放在一起称为**争吵**（wrangling），因为把你的数据变成可以容易处理的形式经常就像一场争辩。
4. 一旦你把数据整理好了，有两种主要产生知识的“引擎”：**可视化**和**建模**。它们有互补的长处和短处，所以真正的分析经常会将两个过程重叠多次。
5. 数据科学的最后一步是**交流**，这绝对是数据分析项目最重要的一部分。除非你能和别人交流你的结果，不然你前面做的太多也是白扯。

伴随着整个过程的工具就是**编程**。

 

### 安装tidyverse包

 

`tidyverse`包含有几个包，它们享有共同的数据与R编程理念，它们被设计到一起进行流畅的工作。

 

```
# 安装
install.packages("tidyverse")
```

 

```R
# 载入
> library(tidyverse)
-- Attaching packages --------------------------------------- tidyverse 1.2.1 --
√ ggplot2 2.2.1     √ readr   1.1.1
√ tibble  1.3.4     √ purrr   0.2.4
√ tidyr   0.7.2     √ stringr 1.2.0
√ ggplot2 2.2.1     √ forcats 0.2.0
-- Conflicts ------------------------------------------ tidyverse_conflicts() --
x dplyr::filter() masks stats::filter()
x dplyr::lag()    masks stats::lag()
```

 

`tidyverse`里面的一些包可能会经常变动，我们可以运行`tidyverse_update()`进行更新。

 

## Tibbles vs. data.frame

tibble与data.frame主要有两点不同：**输出**和**取子集**。

 

### 打印输出

 

```R
> require(tidyverse)
> tibble(
+     a = lubridate::now() + runif(1e3) * 86400,
+     b = lubridate::today() + runif(1e3) * 30,
+     c = 1:1e3,
+     d = runif(1e3),
+     e = sample(letters, 1e3, replace = TRUE)
+ )
# A tibble: 1,000 x 5
                     a          b     c          d     e
                <dttm>     <date> <int>      <dbl> <chr>
 1 2017-11-18 15:34:33 2017-11-29     1 0.67316945     j
 2 2017-11-18 08:14:13 2017-11-20     2 0.37123387     q
 3 2017-11-17 21:30:13 2017-11-27     3 0.62118747     v
 4 2017-11-18 14:25:56 2017-12-16     4 0.05463553     d
 5 2017-11-17 22:47:24 2017-12-12     5 0.51881040     h
 6 2017-11-17 16:19:12 2017-12-09     6 0.03022423     d
 7 2017-11-18 00:41:48 2017-12-13     7 0.17127418     v
 8 2017-11-17 16:41:40 2017-12-01     8 0.42216380     l
 9 2017-11-18 15:44:46 2017-12-12     9 0.34529504     e
10 2017-11-17 23:38:13 2017-12-16    10 0.58544749     p
# ... with 990 more rows
```

 

**Tibbles有一个非常精确的方法，它只展示前10行数据，并且所有的列都会适配屏幕。**这样在处理大型数据时更加方便，而且，每一列都会报告它的类型。

 

如果不想要默认的这种输出方式，我们可以通过`print()`函数的`n`与`width`选项控制显示的行和宽度。`width = Inf`会显示所有列：

```R
> tibble(
+     a = lubridate::now() + runif(1e3) * 86400,
+     b = lubridate::today() + runif(1e3) * 30,
+     c = 1:1e3,
+     d = runif(1e3),
+     e = sample(letters, 1e3, replace = TRUE)
+ ) %>% print(n=20, width=Inf)
# A tibble: 1,000 x 5
                     a          b     c         d     e
                <dttm>     <date> <int>     <dbl> <chr>
 1 2017-11-18 00:34:08 2017-11-29     1 0.1639318     i
 2 2017-11-17 19:42:21 2017-11-30     2 0.2544355     b
 3 2017-11-18 16:09:00 2017-12-11     3 0.7345556     u
 4 2017-11-18 04:30:14 2017-12-07     4 0.2494946     n
 5 2017-11-18 05:54:08 2017-11-23     5 0.1666868     m
 6 2017-11-17 22:09:00 2017-12-15     6 0.4188294     g
 7 2017-11-18 09:01:39 2017-12-01     7 0.2267510     l
 8 2017-11-18 07:10:33 2017-12-11     8 0.7943061     l
 9 2017-11-18 13:46:35 2017-12-14     9 0.9832414     h
10 2017-11-17 18:36:46 2017-12-10    10 0.8545212     m
11 2017-11-18 10:49:32 2017-11-17    11 0.9460041     c
12 2017-11-18 02:57:14 2017-12-01    12 0.9911525     q
13 2017-11-18 16:24:18 2017-12-16    13 0.2692637     g
14 2017-11-17 18:57:07 2017-12-01    14 0.3437629     l
15 2017-11-18 08:50:36 2017-12-04    15 0.1135036     q
16 2017-11-18 09:20:08 2017-12-16    16 0.8855373     v
17 2017-11-18 02:50:50 2017-12-05    17 0.4273206     j
18 2017-11-18 11:19:09 2017-11-26    18 0.7004148     x
19 2017-11-18 14:47:48 2017-11-20    19 0.5334430     v
20 2017-11-18 12:45:52 2017-12-07    20 0.7103081     v
# ... with 980 more rows
```

 

使用`package?tibble`查看所有的选项。

 

当数据太多时我们也可以使用Rstudio内置的查看器来可视化完整的数据集。

```R
> tibble(
+     a = lubridate::now() + runif(1e3) * 86400,
+     b = lubridate::today() + runif(1e3) * 30,
+     c = 1:1e3,
+     d = runif(1e3),
+     e = sample(letters, 1e3, replace = TRUE)
+ ) %>% View()
```

 

### 取子集

如果我们想要取出单个变量，我们需要一些新的工具，`$`和`[[`。`[[`可以根据名称或是位置提取变量，`$`只能根据名称，不过可以少打点字。

 

```R
df <- tibble(
    x = runif(5),
    y = rnorm(5))
```

 

```R
# 根据名字
df$x
df[["x"]]
```

​      

1. 0.995860082330182
2. 0.997774066170678
3. 0.788461328251287
4. 0.698660581139848
5. 0.363849424524233

​      

1. 0.995860082330182
2. 0.997774066170678
3. 0.788461328251287
4. 0.698660581139848
5. 0.363849424524233

 

```R
# 根据位置
df[[1]]
```

​      

1. 0.995860082330182
2. 0.997774066170678
3. 0.788461328251287
4. 0.698660581139848
5. 0.363849424524233

 

如果想要在管道中使用，需要特殊的**占位符**`.`：

 

```R
df %>% .$x
```

​      

1. 0.995860082330182
2. 0.997774066170678
3. 0.788461328251287
4. 0.698660581139848
5. 0.363849424524233

 

```R
df %>% .[["x"]]
```

​      

1. 0.995860082330182
2. 0.997774066170678
3. 0.788461328251287
4. 0.698660581139848
5. 0.363849424524233

 

**注意**： **相比于数据框，tibbles显得更加严格，它不支持局部匹配。**

 

### 和旧代码交互

一些比较旧的函数不支持tibbles，如果你碰到了的话可以使用`as.data.frame()`函数将`tibble`转换为`data.frame`：

 

```R
class(as.data.frame(df))
```

​      

 'data.frame' 

 

### 练习

 

1. 你怎么知道一个变量是tibble？尝试使用`mtcars`

 

```R
class(mtcars)
```

​      

 'data.frame' 

 

```R
class(as.tibble(mtcars))
```

​      

1. 'tbl_df'
2. 'tbl'
3. 'data.frame'

 

```R
typeof(as.tibble(mtcars))
```

​      

 'list' 

 

```R
str(as.tibble(mtcars))
```

​      

```R
Classes 'tbl_df', 'tbl' and 'data.frame':	32 obs. of  11 variables:
 $ mpg : num  21 21 22.8 21.4 18.7 18.1 14.3 24.4 22.8 19.2 ...
 $ cyl : num  6 6 4 6 8 6 8 4 4 6 ...
 $ disp: num  160 160 108 258 360 ...
 $ hp  : num  110 110 93 110 175 105 245 62 95 123 ...
 $ drat: num  3.9 3.9 3.85 3.08 3.15 2.76 3.21 3.69 3.92 3.92 ...
 $ wt  : num  2.62 2.88 2.32 3.21 3.44 ...
 $ qsec: num  16.5 17 18.6 19.4 17 ...
 $ vs  : num  0 0 1 1 0 1 0 1 1 1 ...
 $ am  : num  1 1 1 0 0 0 0 0 0 0 ...
 $ gear: num  4 4 4 3 3 3 3 4 4 4 ...
 $ carb: num  4 4 1 1 2 1 4 2 2 4 ...
```

 

看来需要通过class判断

 

1. 比对用`data.frame`和`tibbles`进行下面的操作，有什么不同？为什么默认的数据框会令人沮丧？

```R
df <- data.frame(abc = 1, xyz = "a")
df$x
df[, "xyz"]
df[, c("abc", "xyz")]
```

 

```R
df <- data.frame(abc = 1, xyz = "a")
df$x
df[, "xyz"]
df[, c("abc", "xyz")]
```

​      

 a 

​      

 a 

​      

| abc  | xyz  |
| ---- | ---- |
| 1    | a    |

 

我们再使用tibble看看

 

```R
df = as.tibble(df)
df$x
df[, "xyz"]
df[, c("abc", "xyz")]
```

​      

```R
Warning message:
"Unknown or uninitialised column: 'x'."
```

​      

```R
NULL
```

​      

| xyz  |
| ---- |
| a    |

​      

| abc  | xyz  |
| ---- | ---- |
| 1    | a    |

 

很显然，tibble只支持完全匹配，这样可以防止一些误操作，或者匹配到多个结果。

 

Practice referring to non-syntactic names in the following data frame by:

```R
Extracting the variable called 1.

Plotting a scatterplot of 1 vs 2.

Creating a new column called 3 which is 2 divided by 1.

Renaming the columns to one, two and three.
annoying <- tibble(
  `1` = 1:10,
  `2` = `1` * 2 + rnorm(length(`1`))
)
```

 

```r
annoying <- tibble(
  `1` = 1:10,
  `2` = `1` * 2 + rnorm(length(`1`))
)
```

 

```R
annoying
```

​      

| 1    | 2         |
| ---- | --------- |
| 1    | 2.009973  |
| 2    | 4.369652  |
| 3    | 5.029929  |
| 4    | 7.068456  |
| 5    | 9.151830  |
| 6    | 12.523541 |
| 7    | 15.090779 |
| 8    | 15.550297 |
| 9    | 18.503531 |
| 10   | 19.554473 |

 

```R
annoying$`1`
```

​      

1. 1
2. 2
3. 3
4. 4
5. 5
6. 6
7. 7
8. 8
9. 9
10. 10

 

```R
plot(annoying$`1`, annoying$`2`)
```

​      

 ![img](data:image/png;base64,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) 

 

```R
annoying$`3` = annoying$`2` / annoying$`1`
```

 

```R
annoying$`3`
```

​      

1. 2.0099734854912
2. 2.18482601344398
3. 1.676642921539
4. 1.7671140762693
5. 1.83036597961312
6. 2.08725689375246
7. 2.15582555773269
8. 1.94378716935277
9. 2.05594783761419
10. 1.95544731728379

 

```R
names(annoying) = c("one", "two", "three")
```

 

```R
annoying
```

​      

| one  | two       | three    |
| ---- | --------- | -------- |
| 1    | 2.009973  | 2.009973 |
| 2    | 4.369652  | 2.184826 |
| 3    | 5.029929  | 1.676643 |
| 4    | 7.068456  | 1.767114 |
| 5    | 9.151830  | 1.830366 |
| 6    | 12.523541 | 2.087257 |
| 7    | 15.090779 | 2.155826 |
| 8    | 15.550297 | 1.943787 |
| 9    | 18.503531 | 2.055948 |
| 10   | 19.554473 | 1.955447 |

 

What does tibble::enframe() do? When might you use it?

 

```R
?tibble::enframe()
```

 

根据运行结果表明`enframe()`函数是将原子向量转为数据框。 （Converting atomic vectors to data frames, and vice versa）

 

**接下来让我们回到原来的话题，dplyr**

 

### 数据

**数据介绍**

This is a cleaned up version of a gene expression dataset from [Brauer  et al. Coordination of Growth Rate, Cell Cycle, Stress Response, and  Metabolic Activity in Yeast (2008) Mol Biol Cell 19:352-367](https://www.ncbi.nlm.nih.gov/pubmed/17959824). This  data is from a gene expression microarray, and in this paper the authors  are examining the relationship between growth rate and gene expression  in yeast cultures limited by one of six different nutrients (glucose,  leucine, ammonium, sulfate, phosphate, uracil). If you give yeast a rich  media loaded with nutrients except restrict the supply of a single  nutrient, you can control the growth rate to any rate you choose. By  starving yeast of specific nutrients you can find genes that:

- **Raise or lower their expression in response to growth rate.**  Growth-rate dependent expression patterns can tell us a lot about cell  cycle control, and how the cell responds to stress. The authors found  that expression of >25% of all yeast genes is linearly correlated  with growth rate, independent of the limiting nutrient. They also found  that the subset of negatively growth-correlated genes is enriched for  peroxisomal functions, and positively correlated genes mainly encode  ribosomal functions.
- **Respond differently when different nutrients are being limited.**  If you see particular genes that respond very differently when a  nutrient is sharply restricted, these genes might be involved in the  transport or metabolism of that specific nutrient.

 

下载清洗后的数据[brauer2007_tidy.csv](http://bioconnector.org/workshops/data/brauer2007_tidy.csv)，你也可以在我当前文件目录下找到。

 

### 读入数据

需要导入`dplyr`与`readr`两个包，确保在继续下面学习之前安装好它们。

 

```R
# 导入包
library(readr)
library(dplyr)
```

 

```R
# 读入数据
ydat = read_csv(file="./brauer2007_tidy.csv")
```

​      

```R
Parsed with column specification:
cols(
  symbol = col_character(),
  systematic_name = col_character(),
  nutrient = col_character(),
  rate = col_double(),
  expression = col_double(),
  bp = col_character(),
  mf = col_character()
)
```

 

```R
# 显示数据
ydat
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                                                           | mf                                                           |
| ------ | --------------- | -------- | ---- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| SFB2   | YNL049C         | Glucose  | 0.05 | -0.24      | ER to Golgi transport                                        | molecular function unknown                                   |
| NA     | YNL095C         | Glucose  | 0.05 | 0.28       | biological process unknown                                   | molecular function unknown                                   |
| QRI7   | YDL104C         | Glucose  | 0.05 | -0.02      | proteolysis and peptidolysis                                 | metalloendopeptidase activity                                |
| CFT2   | YLR115W         | Glucose  | 0.05 | -0.33      | mRNA polyadenylylation*                                      | RNA binding                                                  |
| SSO2   | YMR183C         | Glucose  | 0.05 | 0.05       | vesicle fusion*                                              | t-SNARE activity                                             |
| PSP2   | YML017W         | Glucose  | 0.05 | -0.69      | biological process unknown                                   | molecular function unknown                                   |
| RIB2   | YOL066C         | Glucose  | 0.05 | -0.55      | riboflavin biosynthesis                                      | pseudouridylate synthase activity*                           |
| VMA13  | YPR036W         | Glucose  | 0.05 | -0.75      | vacuolar acidification                                       | hydrogen-transporting ATPase activity, rotational mechanism  |
| EDC3   | YEL015W         | Glucose  | 0.05 | -0.24      | deadenylylation-independent decapping                        | molecular function unknown                                   |
| VPS5   | YOR069W         | Glucose  | 0.05 | -0.16      | protein retention in Golgi*                                  | protein transporter activity                                 |
| NA     | YOL029C         | Glucose  | 0.05 | -0.22      | biological process unknown                                   | molecular function unknown                                   |
| AMN1   | YBR158W         | Glucose  | 0.05 | 0.18       | negative regulation of exit from mitosis*                    | protein binding                                              |
| SCW11  | YGL028C         | Glucose  | 0.05 | -0.67      | cytokinesis, completion of separation                        | glucan 1,3-beta-glucosidase activity                         |
| DSE2   | YHR143W         | Glucose  | 0.05 | -0.59      | cell wall organization and biogenesis*                       | glucan 1,3-beta-glucosidase activity                         |
| COX15  | YER141W         | Glucose  | 0.05 | -0.28      | cytochrome c oxidase complex assembly*                       | oxidoreductase activity, acting on NADH or NADPH, heme protein as acceptor |
| SPE1   | YKL184W         | Glucose  | 0.05 | -0.19      | pantothenate biosynthesis*                                   | ornithine decarboxylase activity                             |
| MTF1   | YMR228W         | Glucose  | 0.05 | -0.42      | transcription from mitochondrial promoter                    | S-adenosylmethionine-dependent methyltransferase activity*   |
| KSS1   | YGR040W         | Glucose  | 0.05 | -0.76      | invasive growth (sensu Saccharomyces)*                       | MAP kinase activity                                          |
| NA     | YHR036W         | Glucose  | 0.05 | -0.91      | biological process unknown                                   | molecular function unknown                                   |
| NA     | YNL158W         | Glucose  | 0.05 | -0.47      | biological process unknown                                   | molecular function unknown                                   |
| YAP7   | YOL028C         | Glucose  | 0.05 | -0.51      | positive regulation of transcription from RNA polymerase II promoter | RNA polymerase II transcription factor activity              |
| NA     | YBR074W         | Glucose  | 0.05 | -1.01      | proteolysis and peptidolysis                                 | metalloendopeptidase activity                                |
| YVC1   | YOR087W         | Glucose  | 0.05 | -0.40      | cation homeostasis                                           | calcium channel activity*                                    |
| CDC40  | YDR364C         | Glucose  | 0.05 | -0.19      | nuclear mRNA splicing, via spliceosome*                      | RNA splicing factor activity, transesterification mechanism* |
| NA     | YPL162C         | Glucose  | 0.05 | -0.10      | biological process unknown                                   | molecular function unknown                                   |
| RMD1   | YDL001W         | Glucose  | 0.05 | -0.22      | biological process unknown                                   | molecular function unknown                                   |
| PCL6   | YER059W         | Glucose  | 0.05 | -0.25      | regulation of glycogen biosynthesis*                         | cyclin-dependent protein kinase regulator activity           |
| AI4    | Q0065           | Glucose  | 0.05 | -0.36      | RNA splicing*                                                | endonuclease activity                                        |
| GGC1   | YDL198C         | Glucose  | 0.05 | 0.76       | mitochondrial genome maintenance*                            | guanine nucleotide transporter activity                      |
| SUL1   | YBR294W         | Glucose  | 0.05 | -0.32      | sulfate transport                                            | sulfate transporter activity                                 |
| ...    | ...             | ...      | ...  | ...        | ...                                                          | ...                                                          |
| NA     | YNL046W         | Uracil   | 0.3  | 0.36       | biological process unknown                                   | molecular function unknown                                   |
| RPS0B  | YLR048W         | Uracil   | 0.3  | 0.30       | protein biosynthesis*                                        | structural constituent of ribosome                           |
| COS12  | YGL263W         | Uracil   | 0.3  | 1.33       | biological process unknown                                   | molecular function unknown                                   |
| NA     | YNR065C         | Uracil   | 0.3  | 0.34       | biological process unknown                                   | molecular function unknown                                   |
| IZH1   | YDR492W         | Uracil   | 0.3  | 0.44       | lipid metabolism*                                            | metal ion binding                                            |
| NA     | YPR064W         | Uracil   | 0.3  | 0.27       | NA                                                           | NA                                                           |
| IZH4   | YOL101C         | Uracil   | 0.3  | 1.15       | lipid metabolism*                                            | metal ion binding                                            |
| PST1   | YDR055W         | Uracil   | 0.3  | 0.64       | cell wall organization and biogenesis                        | molecular function unknown                                   |
| PRM10  | YJL108C         | Uracil   | 0.3  | 0.54       | conjugation with cellular fusion                             | molecular function unknown                                   |
| NA     | YJL107C         | Uracil   | 0.3  | 0.48       | biological process unknown                                   | molecular function unknown                                   |
| SFA1   | YDL168W         | Uracil   | 0.3  | 0.32       | formaldehyde catabolism                                      | alcohol dehydrogenase activity*                              |
| CAP2   | YIL034C         | Uracil   | 0.3  | 0.14       | filamentous growth*                                          | actin filament binding                                       |
| NA     | YMR122W-A       | Uracil   | 0.3  | 0.49       | biological process unknown                                   | molecular function unknown                                   |
| CIS3   | YJL158C         | Uracil   | 0.3  | 0.32       | cell wall organization and biogenesis                        | structural constituent of cell wall                          |
| NA     | YPR012W         | Uracil   | 0.3  | 0.27       | NA                                                           | NA                                                           |
| RGS2   | YOR107W         | Uracil   | 0.3  | -0.08      | G-protein signaling, coupled to cAMP nucleotide second messenger | GTPase activator activity                                    |
| NA     | YPR117W         | Uracil   | 0.3  | -0.16      | biological process unknown                                   | molecular function unknown                                   |
| NA     | YPR150W         | Uracil   | 0.3  | 0.09       | NA                                                           | NA                                                           |
| CSG2   | YBR036C         | Uracil   | 0.3  | -0.01      | calcium ion homeostasis*                                     | enzyme regulator activity                                    |
| SPO11  | YHL022C         | Uracil   | 0.3  | -0.34      | meiotic DNA double-strand break formation                    | endodeoxyribonuclease activity, producing 3'-phosphomonoesters |
| CHO1   | YER026C         | Uracil   | 0.3  | -0.20      | phosphatidylserine biosynthesis                              | CDP-diacylglycerol-serine O-phosphatidyltransferase activity |
| WSC2   | YNL283C         | Uracil   | 0.3  | -0.21      | cell wall organization and biogenesis*                       | transmembrane receptor activity                              |
| MYO2   | YOR326W         | Uracil   | 0.3  | -0.04      | endocytosis*                                                 | microfilament motor activity                                 |
| NA     | YPL066W         | Uracil   | 0.3  | 0.15       | biological process unknown                                   | molecular function unknown                                   |
| DOA1   | YKL213C         | Uracil   | 0.3  | 0.14       | ubiquitin-dependent protein catabolism*                      | molecular function unknown                                   |
| KRE1   | YNL322C         | Uracil   | 0.3  | 0.28       | cell wall organization and biogenesis                        | structural constituent of cell wall                          |
| MTL1   | YGR023W         | Uracil   | 0.3  | 0.27       | cell wall organization and biogenesis                        | molecular function unknown                                   |
| KRE9   | YJL174W         | Uracil   | 0.3  | 0.43       | cell wall organization and biogenesis*                       | molecular function unknown                                   |
| UTH1   | YKR042W         | Uracil   | 0.3  | 0.19       | mitochondrion organization and biogenesis*                   | molecular function unknown                                   |
| NA     | YOL111C         | Uracil   | 0.3  | 0.04       | biological process unknown                                   | molecular function unknown                                   |

 

```R
# 如果你觉得看得不爽，可以直接在Rstudio的交互式窗口看
# View(ydat)
```

 

## dplyr包

[`dplyr`包](https://github.com/hadley/dplyr)是一个相对比较新的R包，它使得数据操作变得更快更简单。它引入了所谓的`magrittr`包的功能特性，使得我们可以将所有的命令串成一个管道，这将改变我们对思考问题进行编写程序的方式。

 

当我们使用`read_csv()`函数读入数据的时候会自动加载`dplyr`包，读入的数据将被存为一个特殊的数据框，称为`tbl`（发音为"tibble"），这个可以使用`class(ydat)`进行查看。如果你有其他的标准表格数据，可以使用`as_tibble()`函数将它转换为`tbl`，它的数据显示效果更好。

除了上面我介绍的tibble，还可以查看[`tibbles vignette`](https://cran.r-project.org/web/packages/tibble/vignettes/tibble.html)。

 

## dplyr动词

dplyr包给出了一些有用的**动词**用于管理数据。下面是一些我们这篇文章内容中会使用的*单表格*动词。（**单表格**动词意思是只处理单个表格的动词——相比于**双表格**动词用于连接数据）

1. `filter()`
2. `select()`
3. `mutate()`
4. `arrange()`
5. `summarize()`
6. `group_by()`

这些函数的第一个输入参数都是数据框或者tibble，它们都会返回一个数据框或者tibble作为结果输出。

 

### filter()

如果你想要过滤数据中一些条件是真的**行**，使用`filter()`函数。

第一个参数是输入数据集，第二个是条件。

 

下面让我们实际操作一下，看一下[LEU1](http://www.yeastgenome.org/locus/Leu1/overview)，一个涉及白氨酸合成的基因。

 

```R
# 确保导入了dplyr包
library(dplyr)

# 查看涉及白氨酸合成通路的一个基因
filter(ydat, symbol == "LEU1")
```

​      

| symbol | systematic_name | nutrient  | rate | expression | bp                   | mf                                     |
| ------ | --------------- | --------- | ---- | ---------- | -------------------- | -------------------------------------- |
| LEU1   | YGL009C         | Glucose   | 0.05 | -1.12      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Glucose   | 0.10 | -0.77      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Glucose   | 0.15 | -0.67      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Glucose   | 0.20 | -0.59      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Glucose   | 0.25 | -0.20      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Glucose   | 0.30 | 0.03       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Ammonia   | 0.05 | -0.76      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Ammonia   | 0.10 | -1.17      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Ammonia   | 0.15 | -1.20      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Ammonia   | 0.20 | -1.02      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Ammonia   | 0.25 | -0.54      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Ammonia   | 0.30 | -0.22      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Phosphate | 0.05 | -0.81      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Phosphate | 0.10 | -0.51      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Phosphate | 0.15 | -0.61      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Phosphate | 0.20 | -0.42      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Phosphate | 0.25 | -0.18      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Phosphate | 0.30 | -0.07      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Sulfate   | 0.05 | -1.57      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Sulfate   | 0.10 | -1.55      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Sulfate   | 0.15 | -1.16      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Sulfate   | 0.20 | -1.10      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Sulfate   | 0.25 | -0.86      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Sulfate   | 0.30 | -0.76      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine   | 0.05 | 3.84       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine   | 0.10 | 3.36       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine   | 0.15 | 3.24       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine   | 0.20 | 2.84       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine   | 0.25 | 2.04       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine   | 0.30 | 0.87       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Uracil    | 0.05 | -2.07      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Uracil    | 0.10 | -1.09      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Uracil    | 0.15 | -0.52      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Uracil    | 0.20 | -0.29      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Uracil    | 0.25 | -0.34      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Uracil    | 0.30 | -0.16      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |

 

```R
# 查看多个基因
filter(ydat, symbol == "LEU1" | symbol == "ADH2")
```

​      

| symbol | systematic_name | nutrient  | rate | expression | bp                   | mf                                     |
| ------ | --------------- | --------- | ---- | ---------- | -------------------- | -------------------------------------- |
| LEU1   | YGL009C         | Glucose   | 0.05 | -1.12      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Glucose   | 0.05 | 6.28       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Glucose   | 0.10 | -0.77      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Glucose   | 0.10 | 5.81       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Glucose   | 0.15 | -0.67      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Glucose   | 0.15 | 5.64       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Glucose   | 0.20 | -0.59      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Glucose   | 0.20 | 5.10       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Glucose   | 0.25 | -0.20      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Glucose   | 0.25 | 1.89       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Glucose   | 0.30 | 0.03       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Glucose   | 0.30 | 0.83       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Ammonia   | 0.05 | -0.76      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Ammonia   | 0.05 | 0.55       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Ammonia   | 0.10 | -1.17      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Ammonia   | 0.10 | -1.55      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Ammonia   | 0.15 | -1.20      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Ammonia   | 0.15 | -2.52      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Ammonia   | 0.20 | -1.02      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Ammonia   | 0.20 | -2.46      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Ammonia   | 0.25 | -0.54      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Ammonia   | 0.25 | -2.40      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Ammonia   | 0.30 | -0.22      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Ammonia   | 0.30 | -2.15      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Phosphate | 0.05 | -0.81      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Phosphate | 0.05 | -4.60      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Phosphate | 0.10 | -0.51      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Phosphate | 0.10 | -6.15      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Phosphate | 0.15 | -0.61      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Phosphate | 0.15 | -4.36      | fermentation*        | alcohol dehydrogenase activity         |
| ...    | ...             | ...       | ...  | ...        | ...                  | ...                                    |
| LEU1   | YGL009C         | Sulfate   | 0.20 | -1.10      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Sulfate   | 0.20 | -2.05      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Sulfate   | 0.25 | -0.86      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Sulfate   | 0.25 | -2.48      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Sulfate   | 0.30 | -0.76      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Sulfate   | 0.30 | -1.59      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Leucine   | 0.05 | 3.84       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Leucine   | 0.05 | 4.15       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Leucine   | 0.10 | 3.36       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Leucine   | 0.10 | 2.61       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Leucine   | 0.15 | 3.24       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Leucine   | 0.15 | 1.89       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Leucine   | 0.20 | 2.84       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Leucine   | 0.20 | 1.90       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Leucine   | 0.25 | 2.04       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Leucine   | 0.25 | 1.60       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Leucine   | 0.30 | 0.87       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Leucine   | 0.30 | 1.56       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Uracil    | 0.05 | -2.07      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Uracil    | 0.05 | 0.63       | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Uracil    | 0.10 | -1.09      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Uracil    | 0.10 | -0.87      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Uracil    | 0.15 | -0.52      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Uracil    | 0.15 | -1.32      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Uracil    | 0.20 | -0.29      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Uracil    | 0.20 | -1.17      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Uracil    | 0.25 | -0.34      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Uracil    | 0.25 | -1.20      | fermentation*        | alcohol dehydrogenase activity         |
| LEU1   | YGL009C         | Uracil    | 0.30 | -0.16      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| ADH2   | YMR303C         | Uracil    | 0.30 | -1.19      | fermentation*        | alcohol dehydrogenase activity         |

 

**注意**，因为我是用jupyter notebook写的，有些输入的注释信息没有输出来，最好自己在rstudio试试。

 

```R
# 查看由于营养缺失导致低生长率的LEU1表达水平
# 注意白氨酸缺失时LEU1会高度上调
filter(ydat, symbol == "LEU1" & rate == .05)
```

​      

| symbol | systematic_name | nutrient  | rate | expression | bp                   | mf                                     |
| ------ | --------------- | --------- | ---- | ---------- | -------------------- | -------------------------------------- |
| LEU1   | YGL009C         | Glucose   | 0.05 | -1.12      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Ammonia   | 0.05 | -0.76      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Phosphate | 0.05 | -0.81      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Sulfate   | 0.05 | -1.57      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine   | 0.05 | 3.84       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Uracil    | 0.05 | -2.07      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |

 

```R
# 但是生长/营养限制解除后表达会下降
filter(ydat, symbol == "LEU1" & rate == .3)
```

​      

| symbol | systematic_name | nutrient  | rate | expression | bp                   | mf                                     |
| ------ | --------------- | --------- | ---- | ---------- | -------------------- | -------------------------------------- |
| LEU1   | YGL009C         | Glucose   | 0.3  | 0.03       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Ammonia   | 0.3  | -0.22      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Phosphate | 0.3  | -0.07      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Sulfate   | 0.3  | -0.76      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine   | 0.3  | 0.87       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Uracil    | 0.3  | -0.16      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |

 

```R
# 仅显示LEU1和Leucine缺失的数据
filter(ydat, symbol == "LEU1" & nutrient == "Leucine")
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                   | mf                                     |
| ------ | --------------- | -------- | ---- | ---------- | -------------------- | -------------------------------------- |
| LEU1   | YGL009C         | Leucine  | 0.05 | 3.84       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine  | 0.10 | 3.36       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine  | 0.15 | 3.24       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine  | 0.20 | 2.84       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine  | 0.25 | 2.04       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine  | 0.30 | 0.87       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |

 

```R
# 当其他营养缺失时LEU1表达如何？
filter(ydat, symbol == "LEU1" & nutrient == "Glucose")
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                   | mf                                     |
| ------ | --------------- | -------- | ---- | ---------- | -------------------- | -------------------------------------- |
| LEU1   | YGL009C         | Glucose  | 0.05 | -1.12      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Glucose  | 0.10 | -0.77      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Glucose  | 0.15 | -0.67      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Glucose  | 0.20 | -0.59      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Glucose  | 0.25 | -0.20      | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Glucose  | 0.30 | 0.03       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |

 

**让我们用图形显示看看**

 

```R
library(ggplot2)
filter(ydat, symbol == "LEU1") %>%
    ggplot(aes(rate, expression, colour=nutrient)) + geom_line(lwd=1.5)
```

​      

 ![img](data:image/png;base64,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) 

 

**仔细看看，当leucine缺失时LEU1高度表达，因为细胞必须自己合成。**

 

#### EXERCISE 1

1. Display the data where the gene ontology biological process (the bp  variable) is “leucine biosynthesis” (case-sensitive) and the limiting  nutrient was Leucine. (Answer should return a 24-by-7 data frame – 4  genes × 6 growth rates).
2. Gene/rate combinations had high expression (in the top 1% of  expressed genes)? Hint: see ?quantile and try quantile(ydat$expression,  probs=.99) to see the expression value which is higher than 99% of all  the data, then filter() based on that. Try wrapping your answer with a  View() function so you can see the whole thing. What does it look like  those genes are doing? Answer should return a 1971-by-7 data frame.

 

```R
# 习题1
filter(ydat, bp == "leucine biosynthesis" & nutrient == "Leucine")
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                   | mf                                       |
| ------ | --------------- | -------- | ---- | ---------- | -------------------- | ---------------------------------------- |
| LEU9   | YOR108W         | Leucine  | 0.05 | 0.44       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU1   | YGL009C         | Leucine  | 0.05 | 3.84       | leucine biosynthesis | 3-isopropylmalate dehydratase activity   |
| LEU2   | YCL018W         | Leucine  | 0.05 | 1.54       | leucine biosynthesis | 3-isopropylmalate dehydrogenase activity |
| LEU4   | YNL104C         | Leucine  | 0.05 | 1.94       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU9   | YOR108W         | Leucine  | 0.10 | 0.57       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU1   | YGL009C         | Leucine  | 0.10 | 3.36       | leucine biosynthesis | 3-isopropylmalate dehydratase activity   |
| LEU2   | YCL018W         | Leucine  | 0.10 | 1.23       | leucine biosynthesis | 3-isopropylmalate dehydrogenase activity |
| LEU4   | YNL104C         | Leucine  | 0.10 | 1.71       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU9   | YOR108W         | Leucine  | 0.15 | 0.46       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU1   | YGL009C         | Leucine  | 0.15 | 3.24       | leucine biosynthesis | 3-isopropylmalate dehydratase activity   |
| LEU2   | YCL018W         | Leucine  | 0.15 | 0.69       | leucine biosynthesis | 3-isopropylmalate dehydrogenase activity |
| LEU4   | YNL104C         | Leucine  | 0.15 | 1.06       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU9   | YOR108W         | Leucine  | 0.20 | 0.46       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU1   | YGL009C         | Leucine  | 0.20 | 2.84       | leucine biosynthesis | 3-isopropylmalate dehydratase activity   |
| LEU2   | YCL018W         | Leucine  | 0.20 | 0.39       | leucine biosynthesis | 3-isopropylmalate dehydrogenase activity |
| LEU4   | YNL104C         | Leucine  | 0.20 | 0.85       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU9   | YOR108W         | Leucine  | 0.25 | 0.36       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU1   | YGL009C         | Leucine  | 0.25 | 2.04       | leucine biosynthesis | 3-isopropylmalate dehydratase activity   |
| LEU2   | YCL018W         | Leucine  | 0.25 | -0.59      | leucine biosynthesis | 3-isopropylmalate dehydrogenase activity |
| LEU4   | YNL104C         | Leucine  | 0.25 | 0.17       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU9   | YOR108W         | Leucine  | 0.30 | 0.06       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU1   | YGL009C         | Leucine  | 0.30 | 0.87       | leucine biosynthesis | 3-isopropylmalate dehydratase activity   |
| LEU2   | YCL018W         | Leucine  | 0.30 | -1.55      | leucine biosynthesis | 3-isopropylmalate dehydrogenase activity |
| LEU4   | YNL104C         | Leucine  | 0.30 | -0.93      | leucine biosynthesis | 2-isopropylmalate synthase activity      |

 

```R
# 习题2
str(ydat) # 先看看数据
```

​      

```R
Classes 'tbl_df', 'tbl' and 'data.frame':	198430 obs. of  7 variables:
 $ symbol         : chr  "SFB2" NA "QRI7" "CFT2" ...
 $ systematic_name: chr  "YNL049C" "YNL095C" "YDL104C" "YLR115W" ...
 $ nutrient       : chr  "Glucose" "Glucose" "Glucose" "Glucose" ...
 $ rate           : num  0.05 0.05 0.05 0.05 0.05 0.05 0.05 0.05 0.05 0.05 ...
 $ expression     : num  -0.24 0.28 -0.02 -0.33 0.05 -0.69 -0.55 -0.75 -0.24 -0.16 ...
 $ bp             : chr  "ER to Golgi transport" "biological process unknown" "proteolysis and peptidolysis" "mRNA polyadenylylation*" ...
 $ mf             : chr  "molecular function unknown" "molecular function unknown" "metalloendopeptidase activity" "RNA binding" ...
 - attr(*, "spec")=List of 2
  ..$ cols   :List of 7
  .. ..$ symbol         : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ systematic_name: list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ nutrient       : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ rate           : list()
  .. .. ..- attr(*, "class")= chr  "collector_double" "collector"
  .. ..$ expression     : list()
  .. .. ..- attr(*, "class")= chr  "collector_double" "collector"
  .. ..$ bp             : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ mf             : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  ..$ default: list()
  .. ..- attr(*, "class")= chr  "collector_guess" "collector"
  ..- attr(*, "class")= chr "col_spec"
```

 

```R
quantile(ydat$expression, probs=.99)
```

​      

 **99%:** 2.07 

 

```
str(filter(ydat, expression > quantile(ydat$expression, probs=.99)))
```

​      

```R
Classes 'tbl_df', 'tbl' and 'data.frame':	1971 obs. of  7 variables:
 $ symbol         : chr  "ATO3" NA NA NA ...
 $ systematic_name: chr  "YDR384C" "YKL187C" "YGL117W" "YBR047W" ...
 $ nutrient       : chr  "Glucose" "Glucose" "Glucose" "Glucose" ...
 $ rate           : num  0.05 0.05 0.05 0.05 0.05 0.05 0.05 0.05 0.05 0.05 ...
 $ expression     : num  2.27 4.13 2.3 2.14 3.71 2.4 3.22 2.19 2.38 2.54 ...
 $ bp             : chr  "transport*" "biological process unknown" "biological process unknown" "biological process unknown" ...
 $ mf             : chr  "transporter activity" "molecular function unknown" "molecular function unknown" "molecular function unknown" ...
 - attr(*, "spec")=List of 2
  ..$ cols   :List of 7
  .. ..$ symbol         : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ systematic_name: list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ nutrient       : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ rate           : list()
  .. .. ..- attr(*, "class")= chr  "collector_double" "collector"
  .. ..$ expression     : list()
  .. .. ..- attr(*, "class")= chr  "collector_double" "collector"
  .. ..$ bp             : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ mf             : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  ..$ default: list()
  .. ..- attr(*, "class")= chr  "collector_guess" "collector"
  ..- attr(*, "class")= chr "col_spec"
```

 

**可以看到结果跟提示的一致**，这里我没有输出所有结果，比较多，如果你想查看所有的结果，去掉最外层的`str()`函数。

 

#### 旁白：将数据写入文件

既然有`read.csv`对应的`read_csv`函数（我们这里使用的），当然也有`write.csv`对应的`write_csv`函数啦。

下面我将部分数据存储为新对象并写入磁盘文件：

 

```R
# 将结果赋给新对象
leudat = filter(ydat, nutrient == "Leucine" & bp == "leucine biosynthesis")
```

 

```R
str(leudat) # 确保前面的没写错
```

​      

```R
Classes 'tbl_df', 'tbl' and 'data.frame':	24 obs. of  7 variables:
 $ symbol         : chr  "LEU9" "LEU1" "LEU2" "LEU4" ...
 $ systematic_name: chr  "YOR108W" "YGL009C" "YCL018W" "YNL104C" ...
 $ nutrient       : chr  "Leucine" "Leucine" "Leucine" "Leucine" ...
 $ rate           : num  0.05 0.05 0.05 0.05 0.1 0.1 0.1 0.1 0.15 0.15 ...
 $ expression     : num  0.44 3.84 1.54 1.94 0.57 3.36 1.23 1.71 0.46 3.24 ...
 $ bp             : chr  "leucine biosynthesis" "leucine biosynthesis" "leucine biosynthesis" "leucine biosynthesis" ...
 $ mf             : chr  "2-isopropylmalate synthase activity" "3-isopropylmalate dehydratase activity" "3-isopropylmalate dehydrogenase activity" "2-isopropylmalate synthase activity" ...
 - attr(*, "spec")=List of 2
  ..$ cols   :List of 7
  .. ..$ symbol         : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ systematic_name: list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ nutrient       : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ rate           : list()
  .. .. ..- attr(*, "class")= chr  "collector_double" "collector"
  .. ..$ expression     : list()
  .. .. ..- attr(*, "class")= chr  "collector_double" "collector"
  .. ..$ bp             : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ mf             : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  ..$ default: list()
  .. ..- attr(*, "class")= chr  "collector_guess" "collector"
  ..- attr(*, "class")= chr "col_spec"
```

 

```R
# 将结果写入磁盘
write_csv(leudat, "leucinedata.csv")
```

 

### select()

前面我们看到了`filter()`函数用来返回匹配行，`select()`则是用来返回匹配列的。这有点像`subset()`函数的`select`参数吧。

 

```R
# 仅仅选择symbol 和 systematic_name
head(select(ydat, symbol, systematic_name)) # 加了个head防止显示太多
```

​      

| symbol | systematic_name |
| ------ | --------------- |
| SFB2   | YNL049C         |
| NA     | YNL095C         |
| QRI7   | YDL104C         |
| CFT2   | YLR115W         |
| SSO2   | YMR183C         |
| PSP2   | YML017W         |

 

```R
# 可以这样移除列
head(select(ydat, -bp, -mf)) # 加了个head防止显示太多
```

​      

| symbol | systematic_name | nutrient | rate | expression |
| ------ | --------------- | -------- | ---- | ---------- |
| SFB2   | YNL049C         | Glucose  | 0.05 | -0.24      |
| NA     | YNL095C         | Glucose  | 0.05 | 0.28       |
| QRI7   | YDL104C         | Glucose  | 0.05 | -0.02      |
| CFT2   | YLR115W         | Glucose  | 0.05 | -0.33      |
| SSO2   | YMR183C         | Glucose  | 0.05 | 0.05       |
| PSP2   | YML017W         | Glucose  | 0.05 | -0.69      |

 

**注意**进行了上面操作后，原始数据ydat没有改变！相当于先生成了一个拷贝，进拷贝进行了修改。

 

```R
# 如果我们要保存上述数据
nogo = select(ydat, -bp, -mf)
```

 

```R
str(nogo)
```

​      

```R
Classes 'tbl_df', 'tbl' and 'data.frame':	198430 obs. of  5 variables:
 $ symbol         : chr  "SFB2" NA "QRI7" "CFT2" ...
 $ systematic_name: chr  "YNL049C" "YNL095C" "YDL104C" "YLR115W" ...
 $ nutrient       : chr  "Glucose" "Glucose" "Glucose" "Glucose" ...
 $ rate           : num  0.05 0.05 0.05 0.05 0.05 0.05 0.05 0.05 0.05 0.05 ...
 $ expression     : num  -0.24 0.28 -0.02 -0.33 0.05 -0.69 -0.55 -0.75 -0.24 -0.16 ...
 - attr(*, "spec")=List of 2
  ..$ cols   :List of 7
  .. ..$ symbol         : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ systematic_name: list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ nutrient       : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ rate           : list()
  .. .. ..- attr(*, "class")= chr  "collector_double" "collector"
  .. ..$ expression     : list()
  .. .. ..- attr(*, "class")= chr  "collector_double" "collector"
  .. ..$ bp             : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  .. ..$ mf             : list()
  .. .. ..- attr(*, "class")= chr  "collector_character" "collector"
  ..$ default: list()
  .. ..- attr(*, "class")= chr  "collector_guess" "collector"
  ..- attr(*, "class")= chr "col_spec"
```

 

```R
# 我们可以过滤这个新的数据集
filter(nogo, symbol == "LEU1" & rate == .05)
```

​      

| symbol | systematic_name | nutrient  | rate | expression |
| ------ | --------------- | --------- | ---- | ---------- |
| LEU1   | YGL009C         | Glucose   | 0.05 | -1.12      |
| LEU1   | YGL009C         | Ammonia   | 0.05 | -0.76      |
| LEU1   | YGL009C         | Phosphate | 0.05 | -0.81      |
| LEU1   | YGL009C         | Sulfate   | 0.05 | -1.57      |
| LEU1   | YGL009C         | Leucine   | 0.05 | 3.84       |
| LEU1   | YGL009C         | Uracil    | 0.05 | -2.07      |

 

### mutate()

mutate()函数用来给数据添加新的列。**记住，这些函数都不会修改操作的数据。**

 

> The expression level reported here is the log2 of the  sample signal divided by the signal in the reference channel, where the  reference RNA for all samples was taken from the glucose-limited  chemostat grown at a dilution rate of 0.25 h−1. Let’s mutate this data  to add a new variable called “signal” that’s the actual raw signal ratio  instead of the log-transformed signal.

 

意思是让我们添加新的一列，它的数值是表达的原始信号值。

 

```R
head(mutate(nogo, signal = 2^expression))
```

​      

| symbol | systematic_name | nutrient | rate | expression | signal    |
| ------ | --------------- | -------- | ---- | ---------- | --------- |
| SFB2   | YNL049C         | Glucose  | 0.05 | -0.24      | 0.8467453 |
| NA     | YNL095C         | Glucose  | 0.05 | 0.28       | 1.2141949 |
| QRI7   | YDL104C         | Glucose  | 0.05 | -0.02      | 0.9862327 |
| CFT2   | YLR115W         | Glucose  | 0.05 | -0.33      | 0.7955365 |
| SSO2   | YMR183C         | Glucose  | 0.05 | 0.05       | 1.0352649 |
| PSP2   | YML017W         | Glucose  | 0.05 | -0.69      | 0.6198538 |

 

```R
# 我们可以继续往后面加变量
head(mutate(nogo, signal = 2^expression, sigsr = sqrt(signal))) # 注意喔，最后的变量是基于第一个生成的变量，有点意思呢
```

​      

| symbol | systematic_name | nutrient | rate | expression | signal    | sigsr     |
| ------ | --------------- | -------- | ---- | ---------- | --------- | --------- |
| SFB2   | YNL049C         | Glucose  | 0.05 | -0.24      | 0.8467453 | 0.9201877 |
| NA     | YNL095C         | Glucose  | 0.05 | 0.28       | 1.2141949 | 1.1019051 |
| QRI7   | YDL104C         | Glucose  | 0.05 | -0.02      | 0.9862327 | 0.9930925 |
| CFT2   | YLR115W         | Glucose  | 0.05 | -0.33      | 0.7955365 | 0.8919285 |
| SSO2   | YMR183C         | Glucose  | 0.05 | 0.05       | 1.0352649 | 1.0174797 |
| PSP2   | YML017W         | Glucose  | 0.05 | -0.69      | 0.6198538 | 0.7873080 |

 

关于画图，我们后面再讲，这里看看我们为啥用对数化的值来取代原始信号呢？

 

```R
library(tidyr)
mutate(nogo, signal=2^expression, sigsr=sqrt(signal)) %>%
    gather(unit, value, expression:sigsr) %>%
    ggplot(aes(value)) + geom_histogram(bins=100) + facet_wrap(~unit, scales="free")
```

​      

 ![img](data:image/png;base64,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m3B5kPYLzsuK07lknKu2BNL4dXbbIO1nn2THbRyk/ZLzsvx0LhrnqnPXHl726FIb/5Jzkft8kOLAczk6F6TBIwFpzThXLQHpvB+Rhgfsp182cuhhn+EJotcCkM78EWmofTwgye7bBmn8G+PpVy06ncvGuerMtcfROP8yn10Vg4deAKQzyDi//fEzu+7hmwZp4XlZejpXXgZrawFIx5+ow88Eaaz5VkCa/6F7/Jkg6e5bBentoXS/H3t6cfq1Hx/On7nu5dtVdd7aZ1zrh6POfbY0dPiFQTrjUj86ZKj9vvvHhrhBkKI+8andmq9bX2etvT/jS86+zJc0vwxIJ99OzE/tzukOSKvHueqctScna+y79seR54E0+JBwKZBOHjrF4dNDFoA0SuntgHT772w4evPBeOPRAYdLYfhVO/c7GwatRQzjx5/R/ujb1dCrdumELYBEUVdfgERRhgIkijIUIFGUoQCJogwFSBRlKECiKEMBEkUZCpAoylCARFGGAiSKMhQg3UTtOI8XLk7ATRQgXbo4ATdRgHTp4gRcW/3efX27/br77/npr91u/+P5D0h/YHr7+Pv7bvf99wWXvL8CpKurv3Y/Xz7+fOHp391b/ZiBtH+9++tl17yzAqSrq39fyXn+sfv35UHpf8/P/x0gCpD+fj3ix+6fCy96VwVI11df//zrzNdPf/7797c5SF//fPbXBXe8uwKk66t/dk/PT7u/Xz779ue53SlIu93hfuqzirCvr37vvr88cfv9/Px99/Wff38C0haKsK+wvu9+vj1v+/MK3RFIP+OpHfWZReRXWE8vjzZPz6/oPD3//vgZab/73/uffry+2PC/3bdL73lPBUjXWF//vLb9Yzf9GentT3+/fvb77eXv3X+XXvOeCpCusf55fd37+fU53u7b08fTuh/73d/vT/DehIuueG8FSBRlKECiKEMBEkUZCpAoylCARFGGAiSKMhQgUZShAImiDAVIFGWo/wOfXtixNKwTZwAAAABJRU5ErkJggg==) 

 

**很显然第一个更接近正态分布，所以进行分析和检验的效果最好**

 

### arrange()

arrange()函数听名字就知道它要做的事情了。它对感兴趣的列进行排列（或者说排序），第一个参数是数据，后面的是排序变量，可以使用`desc()`函数进行降序排列。

 

```R
# 通过gene symbol排序
head(arrange(ydat, symbol))
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                   | mf                          |
| ------ | --------------- | -------- | ---- | ---------- | -------------------- | --------------------------- |
| AAC1   | YMR056C         | Glucose  | 0.05 | 1.50       | aerobic respiration* | ATP:ADP antiporter activity |
| AAC1   | YMR056C         | Glucose  | 0.10 | 1.54       | aerobic respiration* | ATP:ADP antiporter activity |
| AAC1   | YMR056C         | Glucose  | 0.15 | 1.16       | aerobic respiration* | ATP:ADP antiporter activity |
| AAC1   | YMR056C         | Glucose  | 0.20 | 1.04       | aerobic respiration* | ATP:ADP antiporter activity |
| AAC1   | YMR056C         | Glucose  | 0.25 | 0.84       | aerobic respiration* | ATP:ADP antiporter activity |
| AAC1   | YMR056C         | Glucose  | 0.30 | 0.01       | aerobic respiration* | ATP:ADP antiporter activity |

 

```R
# 按表达值，默认升序排列
head(arrange(ydat, expression))
```

​      

| symbol | systematic_name | nutrient  | rate | expression | bp                | mf                             |
| ------ | --------------- | --------- | ---- | ---------- | ----------------- | ------------------------------ |
| SUL1   | YBR294W         | Phosphate | 0.05 | -6.50      | sulfate transport | sulfate transporter activity   |
| SUL1   | YBR294W         | Phosphate | 0.10 | -6.34      | sulfate transport | sulfate transporter activity   |
| ADH2   | YMR303C         | Phosphate | 0.10 | -6.15      | fermentation*     | alcohol dehydrogenase activity |
| ADH2   | YMR303C         | Phosphate | 0.30 | -6.04      | fermentation*     | alcohol dehydrogenase activity |
| ADH2   | YMR303C         | Phosphate | 0.25 | -5.89      | fermentation*     | alcohol dehydrogenase activity |
| SUL1   | YBR294W         | Uracil    | 0.05 | -5.55      | sulfate transport | sulfate transporter activity   |

 

```R
# 降序
head(arrange(ydat, desc(expression)))
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                    | mf                              |
| ------ | --------------- | -------- | ---- | ---------- | --------------------- | ------------------------------- |
| GAP1   | YKR039W         | Ammonia  | 0.05 | 6.64       | amino acid transport* | L-proline permease activity*    |
| DAL5   | YJR152W         | Ammonia  | 0.05 | 6.64       | allantoate transport  | allantoate transporter activity |
| GAP1   | YKR039W         | Ammonia  | 0.10 | 6.64       | amino acid transport* | L-proline permease activity*    |
| DAL5   | YJR152W         | Ammonia  | 0.10 | 6.64       | allantoate transport  | allantoate transporter activity |
| DAL5   | YJR152W         | Ammonia  | 0.15 | 6.64       | allantoate transport  | allantoate transporter activity |
| DAL5   | YJR152W         | Ammonia  | 0.20 | 6.64       | allantoate transport  | allantoate transporter activity |

 

#### EXERCISE 2

1. First, re-run the command you used above to filter the data for  genes involved in the “leucine biosynthesis” biological process and  where the limiting nutrient is Leucine.
2. Wrap this entire filtered result with a call to arrange() where you’ll arrange the result of #1 by the gene symbol.
3. Wrap this entire result in a View() statement so you can see the entire result.

 

```R
library(tidyverse) # 这是第二次接着学习，我先导入包
```

 

```R
ydat = read_csv(file="./brauer2007_tidy.csv")
```

​      

```R
Parsed with column specification:
cols(
  symbol = col_character(),
  systematic_name = col_character(),
  nutrient = col_character(),
  rate = col_double(),
  expression = col_double(),
  bp = col_character(),
  mf = col_character()
)
```

 

```R
leudat = filter(ydat, nutrient == "Leucine" & bp == "leucine biosynthesis") # 习题1
```

 

```R
head(leudat)
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                   | mf                                       |
| ------ | --------------- | -------- | ---- | ---------- | -------------------- | ---------------------------------------- |
| LEU9   | YOR108W         | Leucine  | 0.05 | 0.44       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU1   | YGL009C         | Leucine  | 0.05 | 3.84       | leucine biosynthesis | 3-isopropylmalate dehydratase activity   |
| LEU2   | YCL018W         | Leucine  | 0.05 | 1.54       | leucine biosynthesis | 3-isopropylmalate dehydrogenase activity |
| LEU4   | YNL104C         | Leucine  | 0.05 | 1.94       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU9   | YOR108W         | Leucine  | 0.10 | 0.57       | leucine biosynthesis | 2-isopropylmalate synthase activity      |
| LEU1   | YGL009C         | Leucine  | 0.10 | 3.36       | leucine biosynthesis | 3-isopropylmalate dehydratase activity   |

 

```R
head(arrange(leudat, symbol)) # 习题2
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                   | mf                                     |
| ------ | --------------- | -------- | ---- | ---------- | -------------------- | -------------------------------------- |
| LEU1   | YGL009C         | Leucine  | 0.05 | 3.84       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine  | 0.10 | 3.36       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine  | 0.15 | 3.24       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine  | 0.20 | 2.84       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine  | 0.25 | 2.04       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |
| LEU1   | YGL009C         | Leucine  | 0.30 | 0.87       | leucine biosynthesis | 3-isopropylmalate dehydratase activity |

 

```R
# 习题3，这里jupyter notebook不支持，需要在rstudio中运行
# View(leudat)
```

 

### summarize()

 

这个函数顾名思义是用来总结数据的。

 

```R
# 获取所有基因的平均表达值
summarize(ydat, mean(expression))
```

​      

| mean(expression) |
| ---------------- |
| 0.003367182      |

 

```R
# 发现列名不是很好用，我们改改
summarize(ydat, meanexp=mean(expression))
```

​      

| meanexp     |
| ----------- |
| 0.003367182 |

 

```R
# 计算生长率和表达的相关性
summarize(ydat, r=cor(rate, expression))
```

​      

| r           |
| ----------- |
| -0.02197312 |

 

```R
# 获取观测值的数目
summarize(ydat, n())
```

​      

| n()    |
| ------ |
| 198430 |

 

```R
# 获取数据中有唯一基因符号的数目（就是有重叠的行都看作一）
summarize(ydat, n_distinct(symbol))
```

​      

| n_distinct(symbol) |
| ------------------ |
| 4211               |

 

### Group_by()

 

从上次操作中我们发现`summarize()`函数本身并不是非常有用。`group_by()`函数也是如此，它将已知数据框转换为分组的数据框。

 

```
head(ydat, 10)
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                                    | mf                                                          |
| ------ | --------------- | -------- | ---- | ---------- | ------------------------------------- | ----------------------------------------------------------- |
| SFB2   | YNL049C         | Glucose  | 0.05 | -0.24      | ER to Golgi transport                 | molecular function unknown                                  |
| NA     | YNL095C         | Glucose  | 0.05 | 0.28       | biological process unknown            | molecular function unknown                                  |
| QRI7   | YDL104C         | Glucose  | 0.05 | -0.02      | proteolysis and peptidolysis          | metalloendopeptidase activity                               |
| CFT2   | YLR115W         | Glucose  | 0.05 | -0.33      | mRNA polyadenylylation*               | RNA binding                                                 |
| SSO2   | YMR183C         | Glucose  | 0.05 | 0.05       | vesicle fusion*                       | t-SNARE activity                                            |
| PSP2   | YML017W         | Glucose  | 0.05 | -0.69      | biological process unknown            | molecular function unknown                                  |
| RIB2   | YOL066C         | Glucose  | 0.05 | -0.55      | riboflavin biosynthesis               | pseudouridylate synthase activity*                          |
| VMA13  | YPR036W         | Glucose  | 0.05 | -0.75      | vacuolar acidification                | hydrogen-transporting ATPase activity, rotational mechanism |
| EDC3   | YEL015W         | Glucose  | 0.05 | -0.24      | deadenylylation-independent decapping | molecular function unknown                                  |
| VPS5   | YOR069W         | Glucose  | 0.05 | -0.16      | protein retention in Golgi*           | protein transporter activity                                |

 

```R
head(group_by(ydat, nutrient),10)
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                                    | mf                                                          |
| ------ | --------------- | -------- | ---- | ---------- | ------------------------------------- | ----------------------------------------------------------- |
| SFB2   | YNL049C         | Glucose  | 0.05 | -0.24      | ER to Golgi transport                 | molecular function unknown                                  |
| NA     | YNL095C         | Glucose  | 0.05 | 0.28       | biological process unknown            | molecular function unknown                                  |
| QRI7   | YDL104C         | Glucose  | 0.05 | -0.02      | proteolysis and peptidolysis          | metalloendopeptidase activity                               |
| CFT2   | YLR115W         | Glucose  | 0.05 | -0.33      | mRNA polyadenylylation*               | RNA binding                                                 |
| SSO2   | YMR183C         | Glucose  | 0.05 | 0.05       | vesicle fusion*                       | t-SNARE activity                                            |
| PSP2   | YML017W         | Glucose  | 0.05 | -0.69      | biological process unknown            | molecular function unknown                                  |
| RIB2   | YOL066C         | Glucose  | 0.05 | -0.55      | riboflavin biosynthesis               | pseudouridylate synthase activity*                          |
| VMA13  | YPR036W         | Glucose  | 0.05 | -0.75      | vacuolar acidification                | hydrogen-transporting ATPase activity, rotational mechanism |
| EDC3   | YEL015W         | Glucose  | 0.05 | -0.24      | deadenylylation-independent decapping | molecular function unknown                                  |
| VPS5   | YOR069W         | Glucose  | 0.05 | -0.16      | protein retention in Golgi*           | protein transporter activity                                |

 

```R
head(group_by(ydat, nutrient, rate),10) # 看起来跟上面结果没差别，这个可能需要多看一些行
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                                    | mf                                                          |
| ------ | --------------- | -------- | ---- | ---------- | ------------------------------------- | ----------------------------------------------------------- |
| SFB2   | YNL049C         | Glucose  | 0.05 | -0.24      | ER to Golgi transport                 | molecular function unknown                                  |
| NA     | YNL095C         | Glucose  | 0.05 | 0.28       | biological process unknown            | molecular function unknown                                  |
| QRI7   | YDL104C         | Glucose  | 0.05 | -0.02      | proteolysis and peptidolysis          | metalloendopeptidase activity                               |
| CFT2   | YLR115W         | Glucose  | 0.05 | -0.33      | mRNA polyadenylylation*               | RNA binding                                                 |
| SSO2   | YMR183C         | Glucose  | 0.05 | 0.05       | vesicle fusion*                       | t-SNARE activity                                            |
| PSP2   | YML017W         | Glucose  | 0.05 | -0.69      | biological process unknown            | molecular function unknown                                  |
| RIB2   | YOL066C         | Glucose  | 0.05 | -0.55      | riboflavin biosynthesis               | pseudouridylate synthase activity*                          |
| VMA13  | YPR036W         | Glucose  | 0.05 | -0.75      | vacuolar acidification                | hydrogen-transporting ATPase activity, rotational mechanism |
| EDC3   | YEL015W         | Glucose  | 0.05 | -0.24      | deadenylylation-independent decapping | molecular function unknown                                  |
| VPS5   | YOR069W         | Glucose  | 0.05 | -0.16      | protein retention in Golgi*           | protein transporter activity                                |

 

**真正非常有用的是将group_by()与summarize结合起来使用。**

 

```R
# 求取每个基因的平均表达值
summarize(group_by(ydat, symbol), meanexp=mean(expression))
```

​      

| symbol | meanexp      |
| ------ | ------------ |
| AAC1   | 0.528888889  |
| AAC3   | -0.216285714 |
| AAD10  | 0.438333333  |
| AAD14  | -0.071666667 |
| AAD16  | 0.241944444  |
| AAD4   | -0.791666667 |
| AAD6   | 0.290277778  |
| AAH1   | 0.046111111  |
| AAP1   | -0.003611111 |
| AAP1'  | -0.421388889 |
| AAT1   | 0.120277778  |
| AAT2   | -0.049444444 |
| ABC1   | -0.127500000 |
| ABD1   | 0.185000000  |
| ABF1   | 0.256388889  |
| ABF2   | 0.488611111  |
| ABP1   | 0.042222222  |
| ABP140 | -0.394722222 |
| ABZ1   | -0.076388889 |
| ACA1   | 0.158888889  |
| ACB1   | 0.179166667  |
| ACC1   | 0.323611111  |
| ACE2   | -0.306111111 |
| ACF2   | 0.180000000  |
| ACF4   | 0.319722222  |
| ACH1   | -0.738333333 |
| ACN9   | -0.382222222 |
| ACO1   | 0.130000000  |
| ACP1   | -0.802777778 |
| ACS1   | -0.078055556 |
| ...    | ...          |
| YSY6   | 0.53222222   |
| YTA12  | -0.28000000  |
| YTA6   | -0.16305556  |
| YTA7   | 0.17194444   |
| YTH1   | 0.22527778   |
| YTM1   | -0.55888889  |
| YTP1   | 0.22416667   |
| YUH1   | 0.21000000   |
| YUR1   | -0.02694444  |
| YVC1   | -0.19657143  |
| YVH1   | -0.38472222  |
| ZAP1   | 0.20416667   |
| ZDS1   | 0.03444444   |
| ZDS2   | -0.03166667  |
| ZEO1   | 0.78000000   |
| ZIM17  | -0.10388889  |
| ZIP1   | 0.10750000   |
| ZMS1   | 0.43111111   |
| ZPR1   | -0.36888889  |
| ZPS1   | 0.65861111   |
| ZRC1   | -0.58416667  |
| ZRG17  | -0.26611111  |
| ZRG8   | 0.24833333   |
| ZRT1   | -0.79833333  |
| ZRT2   | -0.51833333  |
| ZRT3   | -0.22055556  |
| ZTA1   | 0.08777778   |
| ZUO1   | -0.06861111  |
| ZWF1   | -0.30583333  |
| NA     | 0.06314836   |

 

```R
# 获取每一种营养物质生长率与表达值的相关性
summarize(group_by(ydat, nutrient), r=cor(rate, expression))
```

​      

| nutrient  | r           |
| --------- | ----------- |
| Ammonia   | -0.01752025 |
| Glucose   | -0.01119529 |
| Leucine   | -0.03837677 |
| Phosphate | -0.01935061 |
| Sulfate   | -0.01664016 |
| Uracil    | -0.03534036 |

 

## %>%管道

 

### %>%怎么工作

dplyr包从magrittr包导入了一些功能特性，使得我们可以将函数的输出输送到另一个函数的输入，因此我们可以避免使用非常复杂的嵌套函数。它看起来就是`^%>%`，安装好dplyr包导入即可使用。

 

下面是一个简单的例子，两种操作结果一致，我们仔细观察和体会实现的不同。

 

```R
tail(ydat, 5)
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                                         | mf                                  |
| ------ | --------------- | -------- | ---- | ---------- | ------------------------------------------ | ----------------------------------- |
| KRE1   | YNL322C         | Uracil   | 0.3  | 0.28       | cell wall organization and biogenesis      | structural constituent of cell wall |
| MTL1   | YGR023W         | Uracil   | 0.3  | 0.27       | cell wall organization and biogenesis      | molecular function unknown          |
| KRE9   | YJL174W         | Uracil   | 0.3  | 0.43       | cell wall organization and biogenesis*     | molecular function unknown          |
| UTH1   | YKR042W         | Uracil   | 0.3  | 0.19       | mitochondrion organization and biogenesis* | molecular function unknown          |
| NA     | YOL111C         | Uracil   | 0.3  | 0.04       | biological process unknown                 | molecular function unknown          |

 

```R
ydat %>% tail(5)
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                                         | mf                                  |
| ------ | --------------- | -------- | ---- | ---------- | ------------------------------------------ | ----------------------------------- |
| KRE1   | YNL322C         | Uracil   | 0.3  | 0.28       | cell wall organization and biogenesis      | structural constituent of cell wall |
| MTL1   | YGR023W         | Uracil   | 0.3  | 0.27       | cell wall organization and biogenesis      | molecular function unknown          |
| KRE9   | YJL174W         | Uracil   | 0.3  | 0.43       | cell wall organization and biogenesis*     | molecular function unknown          |
| UTH1   | YKR042W         | Uracil   | 0.3  | 0.19       | mitochondrion organization and biogenesis* | molecular function unknown          |
| NA     | YOL111C         | Uracil   | 0.3  | 0.04       | biological process unknown                 | molecular function unknown          |

 

我们使用一个dplyr包动词：

 

```R
ydat %>% filter(nutrient=="Leucine")
```

​      

| symbol | systematic_name | nutrient | rate | expression | bp                                                           | mf                                                           |
| ------ | --------------- | -------- | ---- | ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| SFB2   | YNL049C         | Leucine  | 0.05 | 0.18       | ER to Golgi transport                                        | molecular function unknown                                   |
| NA     | YNL095C         | Leucine  | 0.05 | 0.16       | biological process unknown                                   | molecular function unknown                                   |
| QRI7   | YDL104C         | Leucine  | 0.05 | -0.30      | proteolysis and peptidolysis                                 | metalloendopeptidase activity                                |
| CFT2   | YLR115W         | Leucine  | 0.05 | -0.27      | mRNA polyadenylylation*                                      | RNA binding                                                  |
| SSO2   | YMR183C         | Leucine  | 0.05 | -0.59      | vesicle fusion*                                              | t-SNARE activity                                             |
| PSP2   | YML017W         | Leucine  | 0.05 | -0.17      | biological process unknown                                   | molecular function unknown                                   |
| RIB2   | YOL066C         | Leucine  | 0.05 | -0.02      | riboflavin biosynthesis                                      | pseudouridylate synthase activity*                           |
| VMA13  | YPR036W         | Leucine  | 0.05 | -0.11      | vacuolar acidification                                       | hydrogen-transporting ATPase activity, rotational mechanism  |
| EDC3   | YEL015W         | Leucine  | 0.05 | 0.12       | deadenylylation-independent decapping                        | molecular function unknown                                   |
| VPS5   | YOR069W         | Leucine  | 0.05 | -0.20      | protein retention in Golgi*                                  | protein transporter activity                                 |
| NA     | YOL029C         | Leucine  | 0.05 | -0.10      | biological process unknown                                   | molecular function unknown                                   |
| AMN1   | YBR158W         | Leucine  | 0.05 | -1.09      | negative regulation of exit from mitosis*                    | protein binding                                              |
| SCW11  | YGL028C         | Leucine  | 0.05 | -1.03      | cytokinesis, completion of separation                        | glucan 1,3-beta-glucosidase activity                         |
| DSE2   | YHR143W         | Leucine  | 0.05 | -0.77      | cell wall organization and biogenesis*                       | glucan 1,3-beta-glucosidase activity                         |
| COX15  | YER141W         | Leucine  | 0.05 | -1.02      | cytochrome c oxidase complex assembly*                       | oxidoreductase activity, acting on NADH or NADPH, heme protein as acceptor |
| SPE1   | YKL184W         | Leucine  | 0.05 | -0.11      | pantothenate biosynthesis*                                   | ornithine decarboxylase activity                             |
| MTF1   | YMR228W         | Leucine  | 0.05 | -0.82      | transcription from mitochondrial promoter                    | S-adenosylmethionine-dependent methyltransferase activity*   |
| KSS1   | YGR040W         | Leucine  | 0.05 | -0.60      | invasive growth (sensu Saccharomyces)*                       | MAP kinase activity                                          |
| NA     | YHR036W         | Leucine  | 0.05 | -0.67      | biological process unknown                                   | molecular function unknown                                   |
| NA     | YNL158W         | Leucine  | 0.05 | -0.40      | biological process unknown                                   | molecular function unknown                                   |
| YAP7   | YOL028C         | Leucine  | 0.05 | -0.41      | positive regulation of transcription from RNA polymerase II promoter | RNA polymerase II transcription factor activity              |
| NA     | YBR074W         | Leucine  | 0.05 | -1.61      | proteolysis and peptidolysis                                 | metalloendopeptidase activity                                |
| YVC1   | YOR087W         | Leucine  | 0.05 | -1.51      | cation homeostasis                                           | calcium channel activity*                                    |
| CDC40  | YDR364C         | Leucine  | 0.05 | -0.30      | nuclear mRNA splicing, via spliceosome*                      | RNA splicing factor activity, transesterification mechanism* |
| NA     | YPL162C         | Leucine  | 0.05 | -0.66      | biological process unknown                                   | molecular function unknown                                   |
| RMD1   | YDL001W         | Leucine  | 0.05 | -0.47      | biological process unknown                                   | molecular function unknown                                   |
| PCL6   | YER059W         | Leucine  | 0.05 | -0.61      | regulation of glycogen biosynthesis*                         | cyclin-dependent protein kinase regulator activity           |
| AI4    | Q0065           | Leucine  | 0.05 | -0.58      | RNA splicing*                                                | endonuclease activity                                        |
| GGC1   | YDL198C         | Leucine  | 0.05 | -0.15      | mitochondrial genome maintenance*                            | guanine nucleotide transporter activity                      |
| SUL1   | YBR294W         | Leucine  | 0.05 | -1.83      | sulfate transport                                            | sulfate transporter activity                                 |
| ...    | ...             | ...      | ...  | ...        | ...                                                          | ...                                                          |
| NA     | YNL046W         | Leucine  | 0.3  | -0.03      | biological process unknown                                   | molecular function unknown                                   |
| RPS0B  | YLR048W         | Leucine  | 0.3  | 0.12       | protein biosynthesis*                                        | structural constituent of ribosome                           |
| COS12  | YGL263W         | Leucine  | 0.3  | 1.17       | biological process unknown                                   | molecular function unknown                                   |
| NA     | YNR065C         | Leucine  | 0.3  | -0.11      | biological process unknown                                   | molecular function unknown                                   |
| IZH1   | YDR492W         | Leucine  | 0.3  | -0.51      | lipid metabolism*                                            | metal ion binding                                            |
| NA     | YPR064W         | Leucine  | 0.3  | -0.37      | NA                                                           | NA                                                           |
| IZH4   | YOL101C         | Leucine  | 0.3  | -0.42      | lipid metabolism*                                            | metal ion binding                                            |
| PST1   | YDR055W         | Leucine  | 0.3  | -0.29      | cell wall organization and biogenesis                        | molecular function unknown                                   |
| PRM10  | YJL108C         | Leucine  | 0.3  | -0.08      | conjugation with cellular fusion                             | molecular function unknown                                   |
| NA     | YJL107C         | Leucine  | 0.3  | -0.21      | biological process unknown                                   | molecular function unknown                                   |
| SFA1   | YDL168W         | Leucine  | 0.3  | 0.01       | formaldehyde catabolism                                      | alcohol dehydrogenase activity*                              |
| CAP2   | YIL034C         | Leucine  | 0.3  | -0.05      | filamentous growth*                                          | actin filament binding                                       |
| NA     | YMR122W-A       | Leucine  | 0.3  | 0.05       | biological process unknown                                   | molecular function unknown                                   |
| CIS3   | YJL158C         | Leucine  | 0.3  | 0.03       | cell wall organization and biogenesis                        | structural constituent of cell wall                          |
| NA     | YPR012W         | Leucine  | 0.3  | 0.04       | NA                                                           | NA                                                           |
| RGS2   | YOR107W         | Leucine  | 0.3  | -0.29      | G-protein signaling, coupled to cAMP nucleotide second messenger | GTPase activator activity                                    |
| NA     | YPR117W         | Leucine  | 0.3  | -0.48      | biological process unknown                                   | molecular function unknown                                   |
| NA     | YPR150W         | Leucine  | 0.3  | -0.52      | NA                                                           | NA                                                           |
| CSG2   | YBR036C         | Leucine  | 0.3  | -0.15      | calcium ion homeostasis*                                     | enzyme regulator activity                                    |
| SPO11  | YHL022C         | Leucine  | 0.3  | -0.23      | meiotic DNA double-strand break formation                    | endodeoxyribonuclease activity, producing 3'-phosphomonoesters |
| CHO1   | YER026C         | Leucine  | 0.3  | -0.16      | phosphatidylserine biosynthesis                              | CDP-diacylglycerol-serine O-phosphatidyltransferase activity |
| WSC2   | YNL283C         | Leucine  | 0.3  | -0.20      | cell wall organization and biogenesis*                       | transmembrane receptor activity                              |
| MYO2   | YOR326W         | Leucine  | 0.3  | -0.24      | endocytosis*                                                 | microfilament motor activity                                 |
| NA     | YPL066W         | Leucine  | 0.3  | 0.06       | biological process unknown                                   | molecular function unknown                                   |
| DOA1   | YKL213C         | Leucine  | 0.3  | 0.02       | ubiquitin-dependent protein catabolism*                      | molecular function unknown                                   |
| KRE1   | YNL322C         | Leucine  | 0.3  | 0.15       | cell wall organization and biogenesis                        | structural constituent of cell wall                          |
| MTL1   | YGR023W         | Leucine  | 0.3  | -0.10      | cell wall organization and biogenesis                        | molecular function unknown                                   |
| KRE9   | YJL174W         | Leucine  | 0.3  | -0.01      | cell wall organization and biogenesis*                       | molecular function unknown                                   |
| UTH1   | YKR042W         | Leucine  | 0.3  | 0.18       | mitochondrion organization and biogenesis*                   | molecular function unknown                                   |
| NA     | YOL111C         | Leucine  | 0.3  | -0.06      | biological process unknown                                   | molecular function unknown                                   |

 

### 嵌套比对%>%

 

我们来为一个问题思考一分钟：要是我们想要获得每一种营养条件下涉及白氨酸合成通路中基因表达与生长率的相关性。我们可能会这么做：

 

1. 取得`ydat`数据集
2. 然后使用`filter()`函数取得涉及白氨酸合成通路的基因
3. 然后使用`group_by()`函数处理限定的营养物质
4. 然后`summarize()`总结获得生长率和表达的相关
5. 然后使用`mutate()`将结果保留两位有效数字
6. 然后使用`arrange()`对相关系数进行排序

 

**我们来看看用我们上述的认知去实现它和管道实现它的差别**

 

![img](https://nbviewer.jupyter.org/github/ShixiangWang/masterR/blob/master/Rprogramming/learn/dplyr-pipeline.png)

 

**dplyr的处理方式很干净易懂**

 

```R
ydat %>% 
  filter(bp=="leucine biosynthesis") %>%
  group_by(nutrient) %>% 
  summarize(r=cor(rate, expression)) %>% 
  mutate(r=round(r,2)) %>% 
  arrange(r)
```

​      

| nutrient  | r     |
| --------- | ----- |
| Leucine   | -0.58 |
| Glucose   | -0.04 |
| Ammonia   | 0.16  |
| Sulfate   | 0.33  |
| Phosphate | 0.44  |
| Uracil    | 0.58  |

 

### 管道操作练习

 

下面练习热热身。

 

```R
# 找出ADH2基因的生长率为0.05时的营养和表达值
# Hint: filter and select
# 我们先找基因ADH2再取列值
ydat %>% 
filter(symbol == "ADH2" & rate == 0.05) %>%
select(nutrient, expression)
```

​      

| nutrient  | expression |
| --------- | ---------- |
| Glucose   | 6.28       |
| Ammonia   | 0.55       |
| Phosphate | -4.60      |
| Sulfate   | -1.18      |
| Leucine   | 4.15       |
| Uracil    | 0.63       |

 

```R
# 营养限定为Glucose，生长率限定为0.05时表达值最高的4个基因
# 仅仅显示基因符号，表达值，GO terms
# Hint: filter, arrange, head, select
library(tidyverse)
ydat = read_csv(file="./brauer2007_tidy.csv")
ydat %>%
filter(nutrient == "Glucose" & rate == 0.05) %>%
arrange(desc(expression)) %>%
head(4) %>%
select(symbol, expression, bp, mf)
```

​      

```R
-- Attaching packages --------------------------------------- tidyverse 1.2.1 --
√ ggplot2 2.2.1     √ purrr   0.2.4
√ tibble  1.3.4     √ dplyr   0.7.4
√ tidyr   0.7.2     √ stringr 1.2.0
√ readr   1.1.1     √ forcats 0.2.0
-- Conflicts ------------------------------------------ tidyverse_conflicts() --
x dplyr::filter() masks stats::filter()
x dplyr::lag()    masks stats::lag()
Parsed with column specification:
cols(
  symbol = col_character(),
  systematic_name = col_character(),
  nutrient = col_character(),
  rate = col_double(),
  expression = col_double(),
  bp = col_character(),
  mf = col_character()
)
```

​      

| symbol | expression | bp                  | mf                             |
| ------ | ---------- | ------------------- | ------------------------------ |
| ADH2   | 6.28       | fermentation*       | alcohol dehydrogenase activity |
| HSP26  | 5.86       | response to stress* | unfolded protein binding       |
| MLS1   | 5.64       | glyoxylate cycle    | malate synthase activity       |
| HXT5   | 5.56       | hexose transport    | glucose transporter activity*  |

 

```R
# 当生长率限定为0.05时，每种营养条件下在response to stress通路中每个基因的平均表达值时多少？
# protein biosynthesis通路呢？
# Hint: filter, group_by, summary
ydat %>%
filter(rate == 0.05 & bp == "response to stress") %>%
group_by(nutrient) %>%
summarize(meanexp=mean(expression))
```

​      

| nutrient  | meanexp   |
| --------- | --------- |
| Ammonia   | 0.9426667 |
| Glucose   | 0.7426667 |
| Leucine   | 0.8106667 |
| Phosphate | 0.9806667 |
| Sulfate   | 0.7430769 |
| Uracil    | 0.7313333 |

 

```R
ydat %>%
filter(rate == 0.05 & bp == "protein biosynthesis") %>%
group_by(nutrient) %>%
summarize(meanexp=mean(expression))
```

​      

| nutrient  | meanexp    |
| --------- | ---------- |
| Ammonia   | -1.6133514 |
| Glucose   | -0.6911351 |
| Leucine   | -0.5735676 |
| Phosphate | -0.7496216 |
| Sulfate   | -0.9134807 |
| Uracil    | -0.8799454 |

 

**让我们来试点更有难度的**

 

```R
# 得到10个有最多基因关联的生物学过程？如果改成分子功能呢？
# Hint: group_by summarize n_distinct arrange head
ydat %>%
group_by(bp)%>%
summarize(n = n_distinct(symbol)) %>%
arrange(desc(n)) %>%
head(10)
```

​      

| bp                                                           | n    |
| ------------------------------------------------------------ | ---- |
| biological process unknown                                   | 269  |
| protein biosynthesis                                         | 182  |
| protein amino acid phosphorylation*                          | 78   |
| protein biosynthesis*                                        | 73   |
| cell wall organization and biogenesis*                       | 64   |
| regulation of transcription from RNA polymerase II promoter* | 49   |
| nuclear mRNA splicing, via spliceosome                       | 47   |
| DNA repair*                                                  | 44   |
| aerobic respiration*                                         | 42   |
| ER to Golgi transport*                                       | 42   |

 

```R
ydat %>%
group_by(mf)%>%
summarize(n = n_distinct(symbol)) %>%
arrange(desc(n)) %>%
head(10)
```

​      

| mf                                     | n    |
| -------------------------------------- | ---- |
| molecular function unknown             | 886  |
| structural constituent of ribosome     | 185  |
| protein binding                        | 107  |
| RNA binding                            | 63   |
| protein binding*                       | 53   |
| DNA binding*                           | 44   |
| structural molecule activity           | 43   |
| GTPase activity                        | 40   |
| structural constituent of cytoskeleton | 39   |
| transcription factor activity          | 38   |

 

```R
# 有多少基因我们知道涉及什么过程但我们不知道它是什么
# Hint: filter where bp!="biological process unknown" & mf=="molecular function unknown"
ydat %>%
filter(bp!="biological process unknown" & mf=="molecular function unknown") %>%
select(symbol, bp) %>%
distinct()
```

​      

| symbol | bp                                                           |
| ------ | ------------------------------------------------------------ |
| SFB2   | ER to Golgi transport                                        |
| EDC3   | deadenylylation-independent decapping                        |
| PER1   | response to unfolded protein*                                |
| PEX25  | peroxisome organization and biogenesis*                      |
| BNI5   | cytokinesis*                                                 |
| CSN12  | adaptation to pheromone during conjugation with cellular fusion |
| SEC39  | secretory pathway                                            |
| ABC1   | ubiquinone biosynthesis                                      |
| PRP46  | nuclear mRNA splicing, via spliceosome                       |
| MAM3   | mitochondrion organization and biogenesis*                   |
| AEP1   | protein biosynthesis                                         |
| RSC2   | chromatin remodeling*                                        |
| CWH43  | cell wall organization and biogenesis*                       |
| SWC3   | chromatin remodeling*                                        |
| NA     | karyogamy                                                    |
| BUD14  | cellular morphogenesis during vegetative growth              |
| CYK3   | cytokinesis                                                  |
| PAC1   | nuclear migration, microtubule-mediated                      |
| SUR7   | sporulation (sensu Fungi)                                    |
| RAX1   | bud site selection                                           |
| DFM1   | biological process unknown*                                  |
| ERV29  | ER to Golgi transport                                        |
| SFK1   | actin cytoskeleton organization and biogenesis*              |
| DPH2   | peptidyl-diphthamide biosynthesis from peptidyl-histidine    |
| SAM50  | protein complex assembly*                                    |
| PHB2   | protein folding*                                             |
| GSF2   | protein folding*                                             |
| CYC2   | mitochondrial intermembrane space protein import*            |
| MDM34  | mitochondrion organization and biogenesis                    |
| MDM38  | mitochondrion organization and biogenesis*                   |
| ...    | ...                                                          |
| NCA2   | aerobic respiration*                                         |
| ASR1   | response to ethanol                                          |
| APP1   | actin filament organization*                                 |
| YIP3   | ER to Golgi transport                                        |
| NBP2   | response to heat*                                            |
| ORM2   | response to unfolded protein                                 |
| CWC25  | nuclear mRNA splicing, via spliceosome                       |
| YSC84  | endocytosis*                                                 |
| AEP3   | mRNA metabolism                                              |
| PIN3   | actin cytoskeleton organization and biogenesis               |
| SGF73  | histone acetylation                                          |
| SEL1   | protein secretion*                                           |
| PBP1   | mRNA polyadenylylation                                       |
| PIH1   | rRNA processing*                                             |
| RIM8   | meiosis*                                                     |
| LSB3   | actin filament organization                                  |
| COQ4   | ubiquinone metabolism                                        |
| HAL1   | positive regulation of transcription from RNA polymerase II promoter* |
| OXR1   | response to oxidative stress                                 |
| PRM5   | conjugation with cellular fusion                             |
| GIS3   | intracellular signaling cascade                              |
| BUD7   | bud site selection                                           |
| IES6   | metabolism                                                   |
| CAF40  | regulation of transcription from RNA polymerase II promoter  |
| PST1   | cell wall organization and biogenesis                        |
| PRM10  | conjugation with cellular fusion                             |
| DOA1   | ubiquitin-dependent protein catabolism*                      |
| MTL1   | cell wall organization and biogenesis                        |
| KRE9   | cell wall organization and biogenesis*                       |
| UTH1   | mitochondrion organization and biogenesis*                   |

 

```R
# 限定生长率为0.05，营养为Glucose，哪一个BP通路上调最多？根据上调BP显示一个排序列表，平均表达值为2位有效数字
# Hint: filter, group_by, summarize, mutate, arrange
ydat %>%
filter(rate == 0.05 & nutrient=="Glucose") %>%
group_by(bp) %>%
summarize(meanexp=mean(expression)) %>%
mutate(meanexp=round(meanexp,2)) %>%
arrange(desc(meanexp))
```

​      

| bp                                                           | meanexp |
| ------------------------------------------------------------ | ------- |
| fermentation*                                                | 6.28    |
| glyoxylate cycle                                             | 5.29    |
| oxygen and reactive oxygen species metabolism                | 5.04    |
| fumarate transport*                                          | 5.03    |
| acetyl-CoA biosynthesis*                                     | 4.32    |
| gluconeogenesis                                              | 3.64    |
| fatty acid beta-oxidation                                    | 3.57    |
| lactate transport                                            | 3.48    |
| carnitine metabolism                                         | 3.30    |
| alcohol metabolism*                                          | 3.25    |
| sulfite transport                                            | 3.22    |
| biotin biosynthesis*                                         | 3.10    |
| pyridoxine metabolism*                                       | 3.05    |
| fatty acid transport                                         | 2.87    |
| aromatic amino acid family metabolism                        | 2.69    |
| sodium ion transport                                         | 2.56    |
| arginine catabolism                                          | 2.48    |
| ethanol fermentation*                                        | 2.29    |
| fatty acid beta-oxidation*                                   | 2.21    |
| propionate metabolism                                        | 2.21    |
| carbohydrate metabolism*                                     | 2.10    |
| propionate metabolism*                                       | 2.05    |
| regulation of proteolysis and peptidolysis                   | 2.05    |
| ethanol metabolism                                           | 2.02    |
| pyridoxine metabolism                                        | 1.96    |
| response to copper ion                                       | 1.73    |
| glycine biosynthesis                                         | 1.70    |
| urea transport                                               | 1.61    |
| acetate biosynthesis                                         | 1.42    |
| cellular morphogenesis during conjugation with cellular fusion | 1.42    |
| ...                                                          | ...     |
| translational elongation*                                    | -1.02   |
| thiamin diphosphate biosynthesis                             | -1.04   |
| glycolysis*                                                  | -1.05   |
| intracellular mRNA localization                              | -1.05   |
| serine family amino acid biosynthesis                        | -1.05   |
| translation*                                                 | -1.05   |
| steroid metabolism                                           | -1.11   |
| FAD transport                                                | -1.12   |
| posttranslational protein-membrane targeting*                | -1.14   |
| RNA splicing                                                 | -1.14   |
| peptide pheromone export                                     | -1.16   |
| meiotic DNA recombinase assembly*                            | -1.18   |
| asparagine catabolism                                        | -1.19   |
| protein-ER targeting                                         | -1.19   |
| phosphatidylglycerol biosynthesis*                           | -1.21   |
| regulation of cyclin dependent protein kinase activity*      | -1.23   |
| mating type switching*                                       | -1.26   |
| sucrose catabolism                                           | -1.26   |
| maltose catabolism                                           | -1.32   |
| anaerobic respiration                                        | -1.36   |
| mitochondrial citrate transport                              | -1.36   |
| cell cycle*                                                  | -1.48   |
| ferric-enterobactin transport                                | -1.54   |
| intron homing                                                | -1.62   |
| hypusine biosynthesis from peptidyl-lysine                   | -1.67   |
| iron-siderophore transport                                   | -1.67   |
| low-affinity zinc ion transport                              | -1.78   |
| siderophore transport                                        | -1.89   |
| response to pheromone during conjugation with cellular fusion* | -2.19   |
| high affinity iron ion transport*                            | -2.33   |

 

```R
sessionInfo()
```

​      

```R
R version 3.4.2 (2017-09-28)
Platform: x86_64-w64-mingw32/x64 (64-bit)
Running under: Windows 10 x64 (build 15063)

Matrix products: default

locale:
[1] LC_COLLATE=Chinese (Simplified)_China.936 
[2] LC_CTYPE=Chinese (Simplified)_China.936   
[3] LC_MONETARY=Chinese (Simplified)_China.936
[4] LC_NUMERIC=C                              
[5] LC_TIME=Chinese (Simplified)_China.936    

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base     

other attached packages:
 [1] bindrcpp_0.2    forcats_0.2.0   stringr_1.2.0   dplyr_0.7.4    
 [5] purrr_0.2.4     readr_1.1.1     tidyr_0.7.2     tibble_1.3.4   
 [9] ggplot2_2.2.1   tidyverse_1.2.1

loaded via a namespace (and not attached):
 [1] pbdZMQ_0.2-6     repr_0.12.0      reshape2_1.4.2   haven_1.1.0     
 [5] lattice_0.20-35  colorspace_1.3-2 rlang_0.1.4      foreign_0.8-69  
 [9] glue_1.1.1       modelr_0.1.1     readxl_1.0.0     uuid_0.1-2      
[13] bindr_0.1        plyr_1.8.4       munsell_0.4.3    gtable_0.2.0    
[17] cellranger_1.1.0 rvest_0.3.2      psych_1.7.8      evaluate_0.10.1 
[21] parallel_3.4.2   broom_0.4.2      IRdisplay_0.4.4  Rcpp_0.12.13    
[25] scales_0.5.0     IRkernel_0.8.9   jsonlite_1.5     mnormt_1.5-5    
[29] hms_0.3          digest_0.6.12    stringi_1.1.5    grid_3.4.2      
[33] cli_1.0.0        tools_3.4.2      magrittr_1.5     lazyeval_0.2.0  
[37] crayon_1.3.4     pkgconfig_2.0.1  xml2_1.1.1       lubridate_1.7.1 
[41] assertthat_0.2.0 httr_1.3.1       rstudioapi_0.7   R6_2.2.2        
[45] nlme_3.1-131     compiler_3.4.2  
```

​            