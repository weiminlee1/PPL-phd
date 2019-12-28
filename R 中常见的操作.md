### R 中常见的操作

```R
##字符串转换为变量，get()函数

mylist <- c('China', 'America', 'British', 'French')

for (i in seq(1,4,1)){
  item <- mylist[i]
  cat(get('item'), '\n')
}


Linux下运行R脚本

第一步

打开编辑器第一行输入
#! /usr/lib/R/bin/Rscript --vanilla
代码

#options：默认–restore – save --no-readline；–help 查看帮助信息；–version 查看R版本；–slave只打印R脚本的输出，而不显示脚本具体执行情况；–no-timing 去除输出文档结束的运行时间输出。
输入代码
第二步

终端赋予执行权限
$ chmod +x test.r
第三步

终端执行脚本
Rscript test.r
如果需要后台挂起可以使用nohup命令
nohup Rscript ./test.r &
————————————————

```

