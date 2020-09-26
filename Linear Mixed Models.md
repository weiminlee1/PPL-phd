**前言**

**为什么要用混合线性模型**：比如测量了不同收入水平的人群的收入和幸福感，但每个群体内收入水平是不同的，幸福感也不同，两者之间的关系也是不同的， 如果直接用一般线性模型，会造成错误的结论，这个时候要考察的是可以推广到不同收入群体的收入和幸福感之间的关系 （即考察的关系不仅可以应用于当前的收入群体，还可以应用到其他的群体）。这时候需要用到混合线性模型（或者层次线性模型）。

## `R`

`R` 中混合线性模型要依靠`lme4`或者`lmerTest`数据包（强烈推荐后者，因为会输出显著性）

```R
library(lmerTest)
```



**基本表达式**![640?wx_fmt=gif](https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_gif/y2fhgP4leTh3Vmv0CltGnMU46LCgYpqicWtCj4JRpm5rEhL01E2TawKfRX8ClGytzNsakJvQdic4IdJxQufjZ6Xw/640?wx_fmt=gif)

- `data` - 要处理的数据集；
- `formula` - 表达式；
- `DV` - 因变量；
- `Fixed_Factor` - 固定因子，即考察的自变量；
- `Random_intercept` - 随机截距，即认为不同群体的因变量的分布不同 （可以理解成有些人出生就在终点，而你是在起点......）；
- `Random_Slope` - 随机斜率，即认为不同群体受固定因子的影响是不同的 （可以理解成别人花两个小时能赚10000元，而你只能挣个被试费......）；
- `Random_Factor` - 随机因子；

**数据整理形式**

数据整理可参考politeness

该数据收集了若干被试（`subject`）的性别（`gender`），以及用不同的态度（`attitude`）在不同场合（`scenario`）下说话的音高（`frequency`）。 这是一个典型的被试内设计（7 * 2设计）。



**结果查看**

以politeness 数据为例，首先加载数据包并将`scenario`变量变为因子变量。

```R
politeness = readr::read_csv('https://raw.githubusercontent.com/usplos/self-programming/master/politeness_data.csv')politeness$scenario = factor(politeness$scenario)library(lmerTest)
```



建立模型，用`summary()`函数查看结果， 这里需要注意：如果设置随机效应，模型可能无法收敛或者自由度溢出，这个时候需要调整或者取消随机效应：

```R
fit1 = lmer(frequency ~ scenario * attitude + (1+attitude|subject) + (1+attitude|gender), data = politeness)summary(fit1)
```



结果为：

```R
Linear mixed model fit by REML t-tests use Satterthwaite approximations to degrees of  freedom [lmerMod]Formula: frequency ~ scenario * attitude + (1 + attitude | subject) +      (1 + attitude | gender)   Data: politeness REML criterion at convergence: 680.1 Scaled residuals:      Min       1Q   Median       3Q      Max -1.65355 -0.68642 -0.03673  0.50259  2.85443  Random effects: Groups   Name        Variance  Std.Dev. Corr  subject  (Intercept) 6.037e+02 24.5698                 attitudepol 1.086e-02  0.1042  1.00  gender   (Intercept) 6.520e+03 80.7494                 attitudepol 1.127e+02 10.6159  -1.00 Residual             6.100e+02 24.6985       Number of obs: 83, groups:  subject, 6; gender, 2 Fixed effects:                      Estimate Std. Error      df t value Pr(>|t|)   (Intercept)            180.767     58.843   1.050   3.072  0.19030   scenario2               17.450     14.260  64.000   1.224  0.22554   scenario3               46.667     14.260  64.000   3.273  0.00172 **scenario4               44.833     14.260  64.000   3.144  0.00253 **scenario5               16.800     14.260  64.000   1.178  0.24310   scenario6                8.867     14.260  64.000   0.622  0.53628   scenario7               18.133     14.260  64.000   1.272  0.20810   attitudepol             -9.717     16.115   9.430  -0.603  0.56075   scenario2:attitudepol   15.133     20.166  64.000   0.750  0.45575   scenario3:attitudepol  -31.283     20.166  64.000  -1.551  0.12577   scenario4:attitudepol   -4.650     20.166  64.000  -0.231  0.81837   scenario5:attitudepol   -4.783     20.166  64.000  -0.237  0.81326   scenario6:attitudepol  -14.703     20.699  64.030  -0.710  0.48008   scenario7:attitudepol  -30.033     20.166  64.000  -1.489  0.14132   ---Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
```



其中，随机效应的结果如下，可以看出确实对不同的被试或者性别来说，态度的影响是不同的，而且音高本身也是有差别的。

```R
Random effects: Groups   Name        Variance  Std.Dev. Corr  subject  (Intercept) 6.037e+02 24.5698                 attitudepol 1.086e-02  0.1042  1.00  gender   (Intercept) 6.520e+03 80.7494                 attitudepol 1.127e+02 10.6159  -1.00 Residual             6.100e+02 24.6985       
```



固定效应的结果如下，这里是把`scenario`的第一个水平作为基线，其他水平和他比较的结果，看出第3、4类场景显著高于第一类场景。

```R
Fixed effects:                      Estimate Std. Error      df t value Pr(>|t|)   (Intercept)            180.767     58.843   1.050   3.072  0.19030   scenario2               17.450     14.260  64.000   1.224  0.22554   scenario3               46.667     14.260  64.000   3.273  0.00172 **scenario4               44.833     14.260  64.000   3.144  0.00253 **scenario5               16.800     14.260  64.000   1.178  0.24310   scenario6                8.867     14.260  64.000   0.622  0.53628   scenario7               18.133     14.260  64.000   1.272  0.20810   attitudepol             -9.717     16.115   9.430  -0.603  0.56075   scenario2:attitudepol   15.133     20.166  64.000   0.750  0.45575   scenario3:attitudepol  -31.283     20.166  64.000  -1.551  0.12577   scenario4:attitudepol   -4.650     20.166  64.000  -0.231  0.81837   scenario5:attitudepol   -4.783     20.166  64.000  -0.237  0.81326   scenario6:attitudepol  -14.703     20.699  64.030  -0.710  0.48008   scenario7:attitudepol  -30.033     20.166  64.000  -1.489  0.14132   ---Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
```



注意，这里的固定效应不是主效应和交互作用，要查看主效应和交互作用需要用`anova()`函数:

```R
anova(fit1)
```



结果如下，可看出只有场景的主效应显著，态度的主效应和交互作用都不显著。

```R
Analysis of Variance Table of type III  with  Satterthwaite approximation for degrees of freedom                   Sum Sq Mean Sq NumDF  DenDF F.value    Pr(>F)    scenario          19400.2  3233.4     6 64.011  5.3005 0.0001728 ***attitude           2775.3  2775.3     1  1.131  4.5496 0.2556737    scenario:attitude  4985.4   830.9     6 64.011  1.3621 0.2435039    ---Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
```



**随机斜率的取舍**

在上面建立的模型中，包含随机斜率和随机截距，但是注意到，有两个固定效应，是把两个固定效应及其交互作用全都作为随机效应，还是选其中部分作为随机效应呢？这里我们课题组的标准是：首先考虑全模型，即如下命令：

```R
fitAll = lmer(frequency ~ scenario * attitude + (1+attitude * scenario|subject) + (1+attitude * scenario|gender), data = politeness)
```



结果如下，模型出现问题，观测量的个数小于随机因子的个数。

```R
错误: number of observations (=83) <= number of random effects (=84) for term (1 + attitude * scenario | subject); the random-effects parameters and the residual variance (or scale parameter) are probably unidentifiable
```



移除交互作用后，模型如下：

```R
fitAll2 = lmer(frequency ~ scenario * attitude + (1+attitude + scenario|subject) + (1+attitude + scenario|gender), data = politeness)
```



结果如下，模型无法收敛，这说明还需要调整模型：

```R
Warning messages:1: In commonArgs(par, fn, control, environment()) :  maxfun < 10 * length(par)^2 is not recommended.2: In optwrap(optimizer, devfun, getStart(start, rho$lower, rho$pp),  :  convergence code 1 from bobyqa: bobyqa -- maximum number of function evaluations exceeded
```



下一步，要选择移除哪一个随机因子，以及移除`subject`还是`gender`上的随机因子，于是有四种模型：

```R
fitAll3_1 = lmer(frequency ~ scenario * attitude + (1+attitude|subject) + (1+attitude + scenario|gender), data = politeness);fitAll3_2 = lmer(frequency ~ scenario * attitude + (1+scenario|subject) + (1+attitude + scenario|gender), data = politeness);fitAll3_3 = lmer(frequency ~ scenario * attitude + (1+attitude+ scenario|subject) + (1+attitude|gender), data = politeness);fitAll3_4 = lmer(frequency ~ scenario * attitude + (1+attitude + scenario|subject) + (1+scenario|gender), data = politeness)
```



同时要建立一个零模型（只有随机截距）：

```R
fitZero = lmer(frequency ~ scenario * attitude + (1|subject) + (1|gender), data = politeness)
```



接下来依次比较零模型和以上的四个模型，用到的是`anova()`函数：

```R
anova(fitZero, fitAll3_1) # ResultData: politenessModels:object: frequency ~ scenario * attitude + (1 | subject) + (1 | gender)..1: frequency ~ scenario * attitude + (1 + attitude | subject) + ..1:     (1 + attitude + scenario | gender)       Df    AIC     BIC  logLik deviance  Chisq Chi Df Pr(>Chisq)object 17 811.33  852.45 -388.66   777.33                         ..1    54 872.74 1003.35 -382.37   764.74 12.592     37     0.9999anova(fitZero, fitAll3_2) # ResultData: politenessModels:object: frequency ~ scenario * attitude + (1 | subject) + (1 | gender)..1: frequency ~ scenario * attitude + (1 + scenario | subject) + ..1:     (1 + attitude + scenario | gender)       Df    AIC     BIC  logLik deviance  Chisq Chi Df Pr(>Chisq)object 17 811.33  852.45 -388.66   777.33                         ..1    79 910.46 1101.55 -376.23   752.46 24.863     62          1anova(fitZero, fitAll3_3) # ResultData: politenessModels:object: frequency ~ scenario * attitude + (1 | subject) + (1 | gender)..1: frequency ~ scenario * attitude + (1 + attitude + scenario | ..1:     subject) + (1 + attitude | gender)       Df    AIC    BIC  logLik deviance  Chisq Chi Df Pr(>Chisq)object 17 811.33 852.45 -388.66   777.33                         ..1    54 867.53 998.14 -379.76   759.53 17.802     37     0.9968anova(fitZero, fitAll3_4) #ResultData: politenessModels:object: frequency ~ scenario * attitude + (1 | subject) + (1 | gender)..1: frequency ~ scenario * attitude + (1 + attitude + scenario | ..1:     subject) + (1 + scenario | gender)       Df    AIC     BIC  logLik deviance  Chisq Chi Df Pr(>Chisq)object 17 811.33  852.45 -388.66   777.33                         ..1    79 913.52 1104.61 -377.76   755.52 21.806     62          1
```



结果四个模型和零模型都不显著，这种情况下，选取p值最小的模型，即`fitAll3_3`，虽然包含随机斜率的模型和不包含随机斜率的模型无明显差异，但是仍然要考虑随机斜率，因为实验中的被试只是个样本，不能保证样本之外的个体随机斜率也没有影响。

目前暂定的标准（待定）是：

- 优先考虑随机斜率
- 优先考虑全模型
- 舍弃或削减模型的标准是该模型不能收敛，或者自由度溢出
- 优先削减交互作用的随机效应
- 当遇到需削减同水平位置的随机因子时（比如两个随机因子需要舍弃一个时），应考虑所有情况，并将其和零模型作比较；优先保留与零模型有显著差异的模型；当比较都不显著时，优先考虑保留p较小的模型。





**调整固定因子比较基线**

上面的固定效应的结果中，是以`scenario`的第一个水平作为基线，如果想人为设置比较基线，最稳定的方法是用`factor(..., levels = )`重新编码因子， 并设置水平的顺序，这里将最后一个水平设为基线：

```R
politeness$scenario = factor(politeness$scenario, levels = c(7,1:6))fit1 = lmer(frequency ~ scenario * attitude + (1+attitude|subject) + (1+attitude|gender), data = politeness)summary(fit1)
```



结果如下，看到固定因子的基线及其相应的数值都变化了：

```R
Linear mixed model fit by REML t-tests use Satterthwaite approximations to degrees of  freedom [lmerMod]Formula: frequency ~ scenario * attitude + (1 + attitude | subject) +      (1 + attitude | gender)   Data: politeness REML criterion at convergence: 680.1 Scaled residuals:      Min       1Q   Median       3Q      Max -1.65355 -0.68642 -0.03673  0.50259  2.85443  Random effects: Groups   Name        Variance  Std.Dev. Corr  subject  (Intercept) 6.037e+02 24.5698                 attitudepol 1.086e-02  0.1042  1.00  gender   (Intercept) 6.520e+03 80.7494                 attitudepol 1.127e+02 10.6159  -1.00 Residual             6.100e+02 24.6985       Number of obs: 83, groups:  subject, 6; gender, 2 Fixed effects:                      Estimate Std. Error       df t value Pr(>|t|)  (Intercept)           198.9000    58.8431   1.0500   3.380   0.1731  scenario1             -18.1333    14.2597  64.0100  -1.272   0.2081  scenario2              -0.6833    14.2597  64.0100  -0.048   0.9619  scenario3              28.5333    14.2597  64.0100   2.001   0.0496 *scenario4              26.7000    14.2597  64.0100   1.872   0.0657 .scenario5              -1.3333    14.2597  64.0100  -0.094   0.9258  scenario6              -9.2667    14.2597  64.0100  -0.650   0.5181  attitudepol           -39.7500    16.1148   9.4300  -2.467   0.0346 *scenario1:attitudepol  30.0333    20.1662  64.0100   1.489   0.1413  scenario2:attitudepol  45.1667    20.1662  64.0100   2.240   0.0286 *scenario3:attitudepol  -1.2500    20.1662  64.0100  -0.062   0.9508  scenario4:attitudepol  25.3833    20.1662  64.0100   1.259   0.2127  scenario5:attitudepol  25.2500    20.1662  64.0100   1.252   0.2151  scenario6:attitudepol  15.3300    20.6995  64.0400   0.741   0.4616  ---Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
```



这种比较（和基线比较）只是`R`里的其中一种比较方式，`R`里的比较方式共有如下五种：

- `contrasts = contr.helmert`：第二个水平对照第一个水平，第三个水平对照前两个的均值，第四个水平对照前三个的均 值，以此类推
- `contrasts = contr.poly`：基于正交多项式的对照，用于趋势分析（线性、二次、三次等）和等距水平的有序因子
- `contrasts = contr.sum`：对照变量之和限制为0。也称作偏差找对，对各水平的均值与所有水平的均值进行比较
- `contrasts = contr.treatment`：各水平对照基线水平（默认第一个水平）。也称作虚拟编码。（这个是无序因子常用的编码形式，也是LMM常用的和默认的）
- `contrasts = contr.SAS`：类似于`contr.treatment`，只是基线水平变成了最后一个水平。

比如不想和基线比较，而是想看每种水平和总体的均值的偏差程度，需要设置成`contr.sum`的格式，可以用`options(contrasts = )`来调节（注意，`option()`设置的时候需要同时设置无序因子和有序因子的比较方法）：

```R
options(contrasts = c('contr.sum','contr.poly')) fit1 = lmer(frequency ~ scenario * attitude + (1|subject) + (1|gender), data = politeness)summary(fit1)
```



结果如下，看出第1、2、4、5类场合和均值有显著差别，其它的比较方法读者可自行尝试。

```R
Linear mixed model fit by REML t-tests use Satterthwaite approximations to degrees of  freedom [lmerMod]Formula: frequency ~ scenario * attitude + (1 | subject) + (1 | gender)   Data: politeness REML criterion at convergence: 699.4 Scaled residuals:      Min       1Q   Median       3Q      Max -1.77989 -0.64324 -0.04626  0.55751  2.97078  Random effects: Groups   Name        Variance Std.Dev. subject  (Intercept)  615.6   24.81    gender   (Intercept) 5684.8   75.40    Residual              628.3   25.07   Number of obs: 83, groups:  subject, 6; gender, 2 Fixed effects:                    Estimate Std. Error      df t value Pr(>|t|)    (Intercept)          192.650     54.338   1.000   3.545 0.174938    scenario1            -13.625      6.708  64.010  -2.031 0.046378 *  scenario2            -16.742      6.708  64.010  -2.496 0.015148 *  scenario3              8.275      6.708  64.010   1.234 0.221847    scenario4             14.283      6.708  64.010   2.129 0.037073 *  scenario5             25.766      6.708  64.010   3.841 0.000283 ***scenario6             -2.334      6.708  64.010  -0.348 0.729051    attitude1              9.938      2.756  64.020   3.606 0.000609 ***scenario1:attitude1    9.937      6.708  64.010   1.481 0.143382    scenario2:attitude1   -5.079      6.708  64.010  -0.757 0.451662    scenario3:attitude1  -12.646      6.708  64.010  -1.885 0.063921 .  scenario4:attitude1   10.562      6.708  64.010   1.575 0.120263    scenario5:attitude1   -2.754      6.708  64.010  -0.411 0.682698    scenario6:attitude1   -2.688      6.708  64.010  -0.401 0.689965    ---Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1
```



**简单效应分析**

注意，固定效应(Fixed Efeects)，主效应(Main Effects) 和 简单效应(Simple Effects) 是三个不同的概念。 虽然这里交互作用并不显著，但是还是要演示一下如何进行简单效应分析：

```
library(emmeans)emmeans(fit1, pairwise~attitude|scenario) # 比较不同场合下不同态度之间的差别
```



结果分为两部分，第一部分输出不同scenario水平下，不同attitude的均值、标准误自由度等信息，如下：

```R
$emmeansscenario = 7: attitude   emmean       SE   df  lower.CL upper.CL inf      198.9000 58.84312 1.05 -466.4451 864.2451 pol      159.1500 51.59934 1.07 -405.1260 723.4260 scenario = 1: attitude   emmean       SE   df  lower.CL upper.CL inf      180.7667 58.84312 1.05 -484.5784 846.1117 pol      171.0500 51.59934 1.07 -393.2260 735.3260 scenario = 2: attitude   emmean       SE   df  lower.CL upper.CL inf      198.2167 58.84312 1.05 -467.1284 863.5617 pol      203.6333 51.59934 1.07 -360.6426 767.9093 scenario = 3: attitude   emmean       SE   df  lower.CL upper.CL inf      227.4333 58.84312 1.05 -437.9117 892.7784 pol      186.4333 51.59934 1.07 -377.8426 750.7093 scenario = 4: attitude   emmean       SE   df  lower.CL upper.CL inf      225.6000 58.84312 1.05 -439.7451 890.9451 pol      211.2333 51.59934 1.07 -353.0426 775.5093 scenario = 5: attitude   emmean       SE   df  lower.CL upper.CL inf      197.5667 58.84312 1.05 -467.7784 862.9117 pol      183.0667 51.59934 1.07 -381.2093 747.3426 scenario = 6: attitude   emmean       SE   df  lower.CL upper.CL inf      189.6333 58.84312 1.05 -475.7117 854.9784 pol      165.2133 51.85221 1.09 -382.3007 712.7273 Degrees-of-freedom method: kenward-roger Confidence level used: 0.95 
```



第二部分是显著性检验的结果，如下：

```R
$contrastsscenario = 7: contrast   estimate       SE   df t.ratio p.value inf - pol 39.750000 16.11485 8.56   2.467  0.0371 scenario = 1: contrast   estimate       SE   df t.ratio p.value inf - pol  9.716667 16.11485 8.56   0.603  0.5622 scenario = 2: contrast   estimate       SE   df t.ratio p.value inf - pol -5.416667 16.11485 8.56  -0.336  0.7449 scenario = 3: contrast   estimate       SE   df t.ratio p.value inf - pol 41.000000 16.11485 8.56   2.544  0.0327 scenario = 4: contrast   estimate       SE   df t.ratio p.value inf - pol 14.366667 16.11485 8.56   0.892  0.3970 scenario = 5: contrast   estimate       SE   df t.ratio p.value inf - pol 14.500000 16.11485 8.56   0.900  0.3929 scenario = 6: contrast   estimate       SE   df t.ratio p.value inf - pol 24.420042 16.90704 9.87   1.444  0.1796
```



看出在第7、3类场合下，不同态度的音高不同，其余的没有显著差别。



**广义混合线性模型**

如果因变量不是连续变量，则需要用广义线性模型。

`R`中做GLMM(Genaralized Linear Mixed Model)用到的函数是:

```R
glmer(data = , formula = , family = ,...)
```



其中 `family = `有多种不同的选择(注意是字符型的)，分别如下:

- `binomial` - `link = “logit”`;
- `gaussian` - `link = "identity"`;
- `gamma` - `link = "inverse"`;
- `inverse.gaussian` - `link = "1/mu^2"`;
- `poisson` - `link = "log"`;
- `quasi` - `link = "identity", variance = "constant"`;
- `quasibinomial` - `link = "logit"`;
- `quasipoisson` - `link = "log"`;

其他的设置和`lmer()`是一样的。