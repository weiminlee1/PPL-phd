# lme4 - linear mixed-effects models(线性混合效应模型)

### Mixed-model formulas(混合效应模型公式)

```html
<html>
<body>
    

(y|B=b) ~ N(Xβ + Zb + o, σ <sup>2</sup>W<sup>-1</sup>


rsep ~ FEexpr + (REexpr1 | factor1) + (REexpr2 | factor2) + ...

此公式指定模型，数据
rsep 决定响应变量
expr 指定模型矩阵列
FEexpr 决定固定效应模型矩阵列(X)的表达式 
REexpr|factor 决定随机效应模型矩阵(Z)和相关协变量的表达式

</body>  
</html>
```

### Understanding mixed-model formulas(混合效应模型公式的理解)

```R
expr | factor
expr 线性模型公式，根据标准的线形模型或者广义线性模型规则产生一个模型矩阵
factor R因子，
|可以理解为模型矩阵和分组因子间有特定方式的interaction
interaction表示在分组因子的每一水平上，模型矩阵有不同的效应
这种特定方式的interaction表示这种效应是以未观察到的随机变量，而不是未知的固定参数来建模的
```

| Formula            | Alternative         | Meaning                                     |
| ------------------ | ------------------- | ------------------------------------------- |
| (1 \| g)           | 1 + (1 \| g)        | Random intercept with fixed mean            |
| 0+offset(o)+(1\|g) | -1+offset(o)+(1\|g) | Random intercept with a priori means        |
| (1\|g1/g2)         | (1\|g1)+(1\|g1:g2)  | Intercept varying among g1 and g2 within g1 |
| (1\|g1)+(1\|g2)    | 1+(1\|g1)+(1\|g2)   | Intercept varying among g1 and g2           |
| x+(x\|g)           | 1+x+(1+x\|g)        | Correlated random intercept and slope       |
| x+(x\|\|g)         | 1+x+(1\|g)+(0+x\|g) | Uncorrelated random intercept and slope     |

* (1 | g)是最简单的混合线性模型公式，每一组的水平都有自己的随机截距，这些截距的均值和标准差是要被评估的参数，可以被描述为：这个模型是包含了任一随机效应的非零均值作为固定效应参数

* 如果想要指定一个随机截距有先验已知的均值，可以使用offset函数，这种模型包含非固定效应，或者更加准确的固定效应模型矩阵，X，0列和β长度为0

* 在有多个分组因子存在的情况下，比如，如果观察项可以被g2分组，g2嵌合在g1内，那么(1|g1/g2)可以用来解释这种情况下截距的变化

* 然而，lme4最实用的原因在于可以被用来拟合随机效应关联非嵌合的分组因素，比如，假设一组数据可以通过两个交叉分组因素g1和g2来进行分组，那么就可以用(1|g1)+(1|g2)公式

* **除了截距不同，斜率也可以不同，x+(x|g)公式可以用来解释在不同的分组因素水平下，截距和斜率也不同的情况 **

* 指定不相关的随机效应
  * lme4在默认的情况下，假设所有回归系数与相同的随机效应项相关，指定一个不相关的斜率和截距，可以通过(x\|\|g)来表示，
  * 比如，在sleepstudy这个数据集中，评估Days斜率和截距间的相关性非常低（0.066），所以可以用下面的表达式来拟合：
    * fm2  <- lmer(Reaction ~ Days + (Days || Subject), sleepstudy)



