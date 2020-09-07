### linux notebook

#### 系统介绍

```shell

Linux 命令记录
nohup python3 -u test.py >test.log 2>&1 &
1. 最后一个“&”表示后台运行程序,即使关闭当前的终端也可以运行
2. “nohup” 表示程序不被挂起
3. “python”表示执行python代码
4. “-u”表示不启用缓存，实时输出打印信息到日志文件（如果不加-u，则会导致日志文件不会实时刷新代码中的print函数的信息）
5. “test.py”表示python的源代码文件
6. “test.log”表示输出的日志文件
7. “>”表示将打印信息重定向到日志文件
8. “2>&1”表示将标准错误输出转变化标准输出，可以将错误信息也输出到日志文件中（0-> stdin, 1->stdout, 2->stderr）


2、查看当前后台运行的命令

有两个命令可以用，jobs和ps,区别是jobs用于查看当前终端后台运行的任务，换了终端就看不到了。而ps命令用于查看瞬间进程的动态，可以看到别的终端运行的后台进程。

（1）jobs命令

        功能：查看当前终端后台运行的任务

       

       jobs -l选项可显示当前终端所有任务的PID，jobs的状态可以是running，stopped，Terminated。+ 号表示当前任务，- 号表示后一个任务。

（2）ps命令

          功能：查看当前的所有进程

          

         ps -aux | grep "test.sh"    #a:显示所有程序  u:以用户为主的格式来显示   x:显示所有程序，不以终端机来区分


3、关闭当前后台运行的命令

      kill命令：结束进程

     （1）通过jobs命令查看jobnum，然后执行   kill %jobnum

     （2）通过ps命令查看进程号PID，然后执行  kill %PID

       如果是前台进程的话，直接执行 Ctrl+c 就可以终止了


4、前后台进程的切换与控制

     （1）fg命令

       功能：将后台中的命令调至前台继续运行

       如果后台中有多个命令，可以先用jobs查看jobnun，然后用 fg %jobnum 将选中的命令调出。

     （2）Ctrl + z 命令

       功能：将一个正在前台执行的命令放到后台，并且处于暂停状态

     （3）bg命令

       功能：将一个在后台暂停的命令，变成在后台继续执行

       如果后台中有多个命令，可以先用jobs查看jobnum，然后用 bg %jobnum 将选中的命令调出继续执行。

5、取文件的并集、交集
        1. 取出两个文件的并集（重复的行只保留一份）

  cat file1 file2 | sort | uniq > file3

        2. 取出两个文件的交集（只留下同时存在于两个文件中的文件）

  cat file1 file2 | sort | uniq -d > file3

        3. 删除交集，留下其他的行

  cat file1 file2 | sort | uniq -u > file3

6、文件合并

        1. 一个文件在上，一个文件在下

  cat file1 file2 > file3

        2. 一个文件在左，一个文件在右

  paste -d "\t" file1 file2 > file3 ##-d指定分隔符

7、一个文件去掉重复的行

        1. 重复的多行记为一行

  sort file |uniq

        2. 重复的行全部去掉

  sort file |uniq -u

8、文件拆分

        1.split 命令

使用方式：split [OPTION] [INPUT [PREFIX]

 OPTION 有

　　- l<行数>或-l<行数> 　指定每多少行就要切成一个小文件。

　　-b<字节> 　指定每多少字就要切成一个小文件。支持单位:m,k

　　-C<字节> 　与-b参数类似，但切割时尽量维持每行的完整性。

如：

a>按照 指定分割后文件行数

   对于文本文件，可以通过指定分割后文件的行数来进行文件分割。

   命令：split -l 300 large_file.log new_file_prefi

b>按照 指定分割后文件大小

   对于文本文件，可以通过指定分割后文件的行数来进行文件分割。

   命令：split -l 300 large_file.log new_file_prefix


9、awk
	
ARGC               命令行参数个数
ARGV               命令行参数排列
ENVIRON            支持队列中系统环境变量的使用
FILENAME           awk浏览的文件名
FNR                浏览文件的记录数
FS                 设置输入域分隔符，等价于命令行 -F选项
NF                 浏览记录的域的个数,*列数*
NR                 已读的记录数，*行数*
OFS                输出域分隔符
ORS                输出记录分隔符
RS                 控制记录分隔符
$0变量是指整条记录。$1表示当前行的第一个域,$2表示当前行的第二个域,......以此类推。
$NF是number finally,表示最后一列的信息，跟变量NF是有区别的，变量NF统计的是每行列的总数
        
	1.命令行方式
awk [-F  field-separator]  'commands'  input-file(s)

其中，commands 是真正awk命令，[-F域分隔符]是可选的。 input-file(s) 是待处理的文件。
在awk中，文件的每一行中，由域分隔符分开的每一项称为一个域。通常，在不指名-F域分隔符的情况下，默认的域分隔符是空格。
awk工作流程是这样的：读入有'\n'换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,
$n表示第n个域。默认域分隔符是"空白键" 或 "[tab]键"

awk -F: '{print $1}' /etc/passwd 
分隔符为：并输出输入文件的第一个域

awk -F: '{print $NF}' /etc/passwd
分隔符为：并输出文件的最后一列
awk '{print NF}' 
输出文件域的个数

cat /etc/passwd |awk  -F ':'  'BEGIN {print "name,shell"}  {print $1","$7} END {print "blue,/bin/nosh"}'
awk工作流程是这样的：先执行BEGING，然后读取文件，读入有/n换行符分割的一条记录，然后将记录按指定的域分隔符划分域，填充域，$0则表示所有域,$1表示第一个域,$n表示第n个域,随后开始执行模式所对应的动作action。接着开始读入第二条记录······直到所有的记录都读完，最后执行END操作。

awk -F: '/root/{print $7}' /etc/passwd 
查找输入文件带有root关键字所有的行，并输出文件的第7个域。

awk -F: '{$1=""; print $0}' /etc/passwd
令第一列为空，然后输出剩余的所有列

awk -F: '{$1=null;print $0}' /etc/passwd
删除文件的第一列并输出剩余的所有列

awk  -F ':'  '{print "filename:" FILENAME ",linenumber:" NR ",columns:" NF ",linecontent:"$0}' /etc/passwd
指定分割符为冒号，输出文件名，对应的记录行数，列数，以及改行的所有内容

awk -F: '{NR==2；print $0}' /etc/passwd
输出第二行的所有内容

#切换分割符#
awk -v FS=':' -v OFS='\t' '{print $1, $2, $3}' /etc/passwd
awk 'BEGIN{FS=":";OFS="\t"}{print $1,$2,$3}' /etc/passwd
指定输入文件分割符为冒号；输出文件分割符为tab键；-v也是options的一种，用于设置变量的值。


awk -F"," '{for(i=2;i<=NF;i++){printf "%s ", $i}; printf "\n"}'
##删除第一列，并切换分隔符

awk -F "," '{for(i=1;i<=10;i=i+1){printf$i"\t"};print"\n"}' file.txt > fileout.txt
##打印前10列并用"\t"分隔

10、tr
 tr 命令用于转换或删除文件中的字符。
 tr [-cdst][--help][--version][第一字符集][第二字符集]  
 
 cat testfile |tr a-z A-Z 
 将testfile文件中的小写字母改成大写字母
 
 cat testfile|tr "," "\t" > testfile1
 将testfile文件中的，分隔符改成tab分割符
 

11、cut
cut -d ":" -f 1-2,4,5 /etc/passwd
指定分割符为冒号；输出1，2，4，5列



12、vim
一般指令模式（删除、复制）
编辑模式
指令列命令模式（保存、查找）

一般指令模式命令：
ctrl + f 向下翻页
ctrl + b 向上翻页
0 移到当前行第一个字符处
$ 移到当前行最后一个字符处
H 移到屏幕第一行的第一个字符处
M 移到屏幕中间那行的第一个字符处
L 移到屏幕最后一行的第一个字符处
G 移到整个文件的最后一行
gg 移到整个文件的第一行
nG 跳到这个文件的第n行
n+enter 跳n行

##查找
/word 向光标之上查找word
?word 向光标之下查找word

##删除
dd 删除光标所在行
ndd 删除光标以下的n行
d1G 删除光标到第一行的所有行
dG 删除光标以下的所有行

##复制
yy 复制光标所在行
nyy 向下复制n行
y1G 复制光标到第一行的所有数据
yG 复制光标到最后一行的所有数据

##复原，撤销
u 撤销上次命令的结果
ctrl + r 重复上次命令


13、命令别名设定功能
alias lm='ls -a' ##lm相当于ls -a 的效果
unalias lm ##取消别名

14、变量
##设置变量
a='covariate'
##显示变量
echo $a
##把自定义变量输出到环境变量中
export a
##删除变量
unset a
env ##观察环境变量
set ##显示所有变量

[dmtsai@study ~]$ declare [-aixr] variable
选项与参数：
-a ：将后面名为 variable 的变量定义成为数组 (array) 类型
-i ：将后面名为 variable 的变量定义成为整数数字 (integer) 类型
-x ：用法与 export 一样，就是将后面的 variable 变成环境变量；
-r ：将变量设定成为 readonly 类型，该变量不可被更改内容，也不能 unset
范例一：让变量 sum 进行 100+300+50 的加总结果
[dmtsai@study ~]$ sum=100+300+50
[dmtsai@study ~]$ echo ${sum}
100+300+50 <==咦！怎么没有帮我计算加总？因为这是文字型态的变量属性啊！
[dmtsai@study ~]$ declare -i sum=100+300+50\ echo $sum

ping -C 4 www.baidu.com ##ping用来检测网速
man ls ##帮助文档

##Linux的目录结构
/bin ##bin是binary的缩写，该目录底下存放的是命令
/boot ##该目录下存放的是启动Linux使用的一些核心软件
/dev ##该目录下存放的是Linux的外部设备
/etc ##该目录下存放的是所有系统管理所需要的配置文件和子目录
/home ##家目录
/lib和/lib64 ##这两个目录下存放的是系统最基本的动态连接共享库，几乎所有的应用程序都需要用到这些共享库
/media ##系统会自动识别一些设备，如u盘等，当识别后，Linux会把这些设备挂载到该目录下
/mnt ##系统提供该目录是为了让用户临时挂载别的文件系统
/opt ##这是给主机额外安装软件所设置的目录，该目录默认为空
/proc ##该目录是一个虚拟目录，是系统内存的映射
/root ##该目录是系统管理员的用户家目录
/run ##这个目录和/var/run是同一个目录，这里面存放的是一些服务的pid
/sbin ##super bin，该目录存放的是系统管理员使用的系统管理命令/程序
/srv ##该目录存放的是一些服务器启动之后需要提取的数据
/sys ##该目录存放的是与硬件驱动程序相关的信息
/tmp ##该目录存放一些临时文件
/usr ##类似于windows下的program files目录，用户很多应用程序和文件都存放在该目录下
/usr/bin ##该目录存放的是系统用户使用的应用程序
/usr/sbin ##该目录存放超级用户使用的比较高级的管理程序和守护程序
/var ##该目录存放的是不断扩充且经常修改的目录，系统上运行各个程序时所产生的日志
```

#### 关机、重启

```shell
ps -aux ##查看是否还有后台进程运行
who ##查看还有其他人在登录
shutdown -h 10 ##计算机将在10分钟关机，所有登录用户可以看到
shutdown -h now
shutdown -h 20:25 ##系统会在八点二十五关机
shutdown -r now ##立即重启
shutdown -r 10+ ##十分钟后重启
reboot ##等同于 shutdown -r now
halt ##等同于 shutdown -h now
```

#### 忘记root密码

```shell l
##无需重装系统，只需要进入emergency mode更改root 密码即可

1重启系统
2进入emergency模式
3修改root密码

uname -a ##查看系统的版本

```

#### 创建和删除目录

```shell
mkdir -m -p /tmp/test/123
-m 设置目录权限
-p 连续创建
rmdir /tmp/test ##只能删除空目录
rm -r /tmp/test/ ##删除目录
rm -rf /tmp/test ##强制删除


```

#### 环境变量

```shell
##更改内置变量的别名
alias la='ls -a'
unalias la

##设置变量
a='linux_file'
echo $a ##显示环境变量
export a ##把自定义变量输出到环境变量中
unset a ##取消环境变量


```

#### 常见命令

```shell
##复制
cp [选项] [源文件] [目的文件]
cp -r dir1 dir2 ##复制dir1为dir2
cp file1 file2 ##将文件file1复制为file2

##移动和重命名
mv file1 ./dir ##将file1移动到dir中
mv file1 file2 ##将file1更名为file2

##创建文件
touch file1

##显示文件内容
cat file
cat -n file ##查看文件时输出行号
cat -A file ##显示所有的内容，包括特殊字符

more file
less file 

head -n file ##显示文件的前n行
tail -10 file ##显示文件的后10行
tail -f file ##动态显示文件

##搜索文件
##which 命令查找可执行文件的绝对路径
which vi
whereis ls
find [路径] [参数]
find -mtime +n/-n ##表示写入时间大于或小于n天的文件
find ./dir -mtime -1 ##从dir目录中查找写入时间小于1天的文件
find ./ -name test2 ##在当前目录下查找test2的文件
find ./ -type d ##通过文件类型查找文件

##创建链接
ln -s [来源文件] [目的文件] ##-s表示软链接
ln -s passwd passwd-soft
ln passwd passwd-hard



```

#### 用户组管理

```shell
##新增组的命令
groupadd [-g GID] groupname
groupadd grptest1 ##tail -n1 /etc/group

##删除组
groupdel grptest1

##增加用户
useradd [-u UID] [-g GID] [-d HOME] [-M] [-s]
-u 表示自定义UID
-g 表示使新增用户属于已经存在的某个组，后面可以跟组ID，也可以跟组名
-d 表示自定义用户的家目录
-M 表示不建立家目录
-s 表示自定义shell

##删除用户
userdel [-r] username
-r 表示当删除用户时候，一并删除该用户的家目录

##用户身份切换
su [-] username ##普通用户命令不加username时，就是切换到root用户
##不加-时，切换到另一账户时，当前目录没有变化；而加上-切换到另一账户的家目录

##查找IP地址
ip address


##查看磁盘或者目录的容量
df ##disk filesystem的简写，用于查看已挂载磁盘的总容量、使用容量、剩余容量
df -i ##查看inodes的使用情况
df -h ##表示使用合适的单位显示-m
df -k/-m

##查看某个目录或文件所占空间的大小
du -a ##表示全部文件和目录的大小都列出来
du -b ##表示列车的值以B为单位输出
du -h
du -s ##表示只列出总和
du -sh dir

##fdisk是硬盘的分区工具，是一个非常实用的命令，但是fdisk只能划分小于2tb的分区

fdisk -l [设备名称] ##列出该设备的分区表


##压缩工具
gzip [-d#] filename ##其中#为1-9的数字
-d 该参数在解压缩时使用
-# 表示压缩等级，1为最差，9为最好，6为默认
gzip filename
gzip -d filename.gz

bzip2 [-dz] filename ##它只有-z压缩和-d解压缩两个常用选项。压缩级别有1-9
bzip2 filename
bzip2 -d filename.bz2

##打包工具
tar [-zjxcvfpP] filename tar
-z 表示同时使用gzip压缩
-j 表示同时用bzip2压缩
-J 表示同时用xz压缩
-x 表示解包或者解压缩
-t 查看tar包里的文件
-c 建立一个tar包或者压缩文件包
-v 可视化
-f 后面跟文件名

#macbook terminal install tree
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)" < /dev/null 2> /dev/null

brew install tree
tree -a 显示所有文件
tree -d 只显示文件夹
tree -L n 显示项目的层级。n表示层级数
tree - I pattern 用于过滤不想要显示的文件或者文件夹

brew install wget

#zcat, bzcat查看压缩文档
zcat 11.txt.gz
bzcat 22.txt.bz2

##安装源码包
##安装源码包，需要我们把源代码编译成可执行的二进制文件，源码包的编译用到了Linux系统里的编译器。常见的源码包一般都是用C语言开发的，因为C语言是Linux上最标准的的程序语言。Linux上的C语言编译器称为gcc，利用它可以把C语言编译成可执行的二进制文件。

##安装gcc
yum install -y gcc

##源码包安装的步骤
./configure --prefix=="path" ##指定安装路径
make
make install

##通配符
##可以使用*来匹配零个或多个字符，用？匹配一个字符，\脱义字符

##输入/输出重定向
输入重定向的命令是 <
输出重定向的命令是 >
追加重定向的命令 >>

##管道符|，用于将前一个指令的输出作为后一个指令的输入
cat /etc/passwd|wc -l

##作业控制
ctrl + z 暂停
fg 恢复
bg 后台运行

##使用env命令，可列出系统预设的全部系统变量
env

.bashrc_profile ##该文件定义了用户的个人化路径与环境变量的文件名称。每个用户都可以使用该文件输入专属于自己的shell的信息，当用户登录时，该文件仅仅执行一次。
.bashrc ##该文件包含专属于自己的shell的bash信息，当登录或每次打开新的shell时，该文件会被读取。例如，你可以将用户自定义的别名或者自定义变量写到这个文件中。
.bash_history ##该文件用于记录命令历史
.bash_logout ##当退出shell时，会执行该文件，可以将一些清理的工作放到这个文件中。

##cut命令
cut -d '分隔字符' [-cf] n
-d:后面跟分隔字符，分隔字符要用单引号括起来
-c:后面接的是第几个字符；-c n1-n2截取一个区间
-f:后面接的是第几个区块；-f 1表示截取第一段


##shell 
##shell是系统跟计算机硬件交互时使用的中间介质，它只是系统的一个工具。在shell和计算机硬件之间还有一层东西-系统内核。系统内核就是人的大脑，用户把指令告诉shell，然后shell再传输给系统内核，接着内核再去支配计算机硬件去执行各种操作。


# ！是与命令历史有关的一个特殊字符，该字符常用的应用有3个：
!! #表示执行上一条指令

pwd
!! ##重复pwd

!n ##n代表数字，表示执行命令历史中的第n条指令

history |grep 1002
!1002 #表示执行1002条命令

!string ##表示执行带string开头的命令
!pw ##表示执行命令历史最近一次以pw开头的命令

##系统环境变量与个人环境变量的配置文件

/etc/profile ##这个文件预设了几个重要的变量，例如PATH, USER, LOGNAME, MAIL, INPUTRC, HOSTNAME, HISTSIZE, umask等

/etc/bashrc ##这个文件主要预设umask以及PS1。这个PS1就是我们在输入命令时前面的那串字符。
echo $PS1

.bash_profile ##该文件定义了用户的个人化路径与环境变量的文件名称。每个用户都可以使用该文件输入专属于自己的shell信息，当用户登录时，该文件仅仅执行一次

.bashrc ##该文件包含专属于自己的shell的bash信息，当登录或每次打开新的shell时，该文件会被读取

.bash_history ##该文件用于记录命令历史

.bash_logout ##当退出shell时，会执行该文件，你可以将一些清理的工作放到这个文件夹中



##sort命令
sort [-t 分隔符] [-kn1,n2] [-nru] ##这里的n1和n2指的是数字
-t 后面跟分隔字符，作用跟cut的-d选项一样
-n 表示使用纯数字排序
-r 表示方向排序
-u 表示去重复
-kn1,n2 表示由n1区间排序到n2区间，可以只写-kn1，即对n1字段排序

-t选项后面跟分隔符，-k选项后面跟单个数字表示对第几个区域的字符串排序，-n选项则表示使用纯数字排序

head -n5 /etc/passwd | sort -t: -k3 -n
head -n5 /etc/passwd | sort -t: -k3,5 -r ##-k选项后面跟数字n1和n2表示对第n1和n2区域内的字符串排序，-r选项则表示方向排序

去除重复行
sort file |uniq

查找非重复行
sort file |uniq -u

查找重复行
sort file |uniq -d

统计
sort file | uniq -c
##wc命令
##wc命令用于统计文档的行数、字符串或词数。该命令的常用选项有-l（统计行数）、-m（统计字符数）和-w（统计词数）

wc /etc/passwd  ##把行数，词数和字符数依次输出
wc -l /etc/passwd
wc -m /etc/passwd
wc -w /etc/passwd 


##uniq命令
##uniq命令用来删除重复的行，该命令只有-c选项比较常用，它表示统计重复的行数，并且把行数写在前面

##使用uniq前，必须先给文件排序，否则不管用
sort test.txt | uniq -c

##tee命令
##tee命令后面跟文件名，起作用类似于重定向>，但它比重定向多一个功能，即把文件写入后面所跟的文件时，还显示在屏幕上。该命令常用于管道符
echo "aaaaaaa" | tee test.txt ##可以覆盖内容

##tr命令
##tr命令用于替换字符，常用于处理文档中出现的特殊符号
-d ##表示删除某个字符，后面跟要删除的字符
-s ##表示删除重复的字符

head -n2 /etc/passwd | tr '[a-z]' '[A-Z]' ##把小写字母变成大写字母

grep 'root' /etc/passwd | tr 'r' 'R' ##替换一个字符

##split 命令
##split命令用于切割文档，常用的选项为-b和-l
-b ##表示依据大小来分割文档，单位为byte
split -b 500 passwd

-l ##表示依据行数来分割文档
split -l 10 passwd

##note:如果split不指定目标文件名，则会以xaa、xab这样的文件名来存取切割后的文件

##$特殊符号
##符号$可以用作变量前面的标识符，还可以和！结合起来使用
ls !$
!$ ##表示上条命令中的最后一个变量

##特殊符号；
##通常，我们都是在一行中输入一个命令，然后回车就运行了。如果想在一行中运行两个或两个以上的命令，需要在命令之间加符号；
mkdir testdir; touch test1.txt; touch test2.txt; ls -d test*

##特殊符号～
##符号～表示用户的家目录，root用户的家目录是/root，普通用户则是/home/username

##特殊符号&
##如果想把一条命令放到后台执行，则需要加上符号&，它通常用于命令运行时间较长的情况。
sleep 30 &
jobs


##重定向符号>、>>、2>、和2>>
## >和>>，他们分别表示取代和追加的意思。当我们运行一个命令报错时，报错信息会输出到当前屏幕。如果想重定向到一个文本，则需要用重定向符号2>或者2>>，他们分别表示错误重定向和错误追加重定向


##中括号[]
##中括号内为字符组合，代表字符组合中的任意一个，可以是一个范围（1-3，a-z）
ls -l test[12b].txt
##test1.txt test2.txt testb.txt

##特殊符号&&和||
## command1; command2 ##使用;时，不管command1是否执行成功，都会执行command2

##command1 && command2 ##使用&&时，只有command1执行成功后，command2才会执行，否则command2不执行

##command1 || command2 ##使用||时，command1执行成功后则command2不执行，否则执行command2，即执行一条
```

### 正则表达式

```shell
##grep/egrep 工具的使用
grep [-cinvABC] 'word' filename 
-c ##表示打印符合要求的行数
-i ##表示忽略大小写
-n ##表示输出符合要求的行及其行号
-v ##表示打印不符合要求的行
-A ##后面跟一个数字，例如-A2表示打印符合要求的行以及下面的两行
-B ##后面跟一个数字、例如-B2表示打印符合要求的行以及上面两行
-C ##后面跟一个数字，例如-C2表示打印符合要求的行以及上下各两行

grep -B2 'halt' /etc/passwd
##打印含有'halt'行及其上面两行

grep -n 'root' /etc/passwd ##过滤出带有某个关键词的行，并数次饭行号
grep -nv 'nologin' /etc/passwd ##过滤出不带有某个关键词的行，并输出行号

grep -v '[0-9]' /etc/passwd ##过滤出所有不包含数字的行

grep -v '^#' /etc/sos.conf ##过滤掉所有以#开头的行


```

### awk

```shell
#使用awk取某一行数据中的倒数第N列：$(NF-(n-1))
#比如取/etc/passwd文件中的第2列、倒数第1、倒数第2、倒数第4列（以冒号为分隔符）。（$NF表示倒数第一列，$(NF-1)表示倒数第二列）

~ 匹配正则
!~ 不匹配正则
== 等于
!= 不等于

1) 打印上面test文件中第二列匹配80开头并以80结束的行
 awk '{if($2~/^80$/)print}' test.txt
2）打印上面test文件中第二列中不匹配80开头并以80结束的行
 awk '{if($2!~/^80$/)print}' test.txt
 
3) 打印第一列后面所有的列，原理把不需要的列赋为空
	awk '{ $1=""; print $0 }' ur_file 
	awk '{for(i=1; i<=2; i++){ $i="" }; print $0 }' urfile ##跳过前两列
	awk '{for(i=3;i<=NF;i++){$i=""};print$0}' urfile ##只输出前两列
	
4)删除行
去掉第一行
awk 'NR>1' AIRA_GPS.aload > ccc
awk 'NR>2{print p}{p=$0}' urfile
第一行时, NR=1, 不执行print, p=第一行的内容
第二行时, NR=2, 不执行print, p=第二行的内容
第三行时, NR=3, 执行print p,此时p=第二行的内容, 即打印第二行, 然后p=第三行
5）有条件删除行
awk 'BEGIN{printf "geneid\ta1\ta2\tb1\tb2\n"}{if($2+$3+$4+$5>0)print $0}' output.matrix > deseq2_input.txt
6）引用外部变量，跳过首行
for i in $(seq 1 22); do awk -F "\t" '{if(NR>2)print"'$i'""\t"$1}' chr$i.info > chr$i.new.snpID.txt

```

### 行列转换

```shell
awk '{for(i=1;i<=NF;i=i+1){a[NR,i]=$i}}END{for(j=1;j<=NF;j++){str=a[1,j];for(i=2;i<=NR;i++){str=str " " a[i,j]}print str}}' test

```

### sed

```shell
1）在原文件上删除某行
sed '1d' test              #删除第一行 
sed '$d' test              #删除最后一行
sed '1,2d' test           #删除第一行到第二行
sed '2,$d' test           #删除第二行到最后一行
sed -i 'N,Md' filename					# file的[N,M]行都被删除
sed -i '/xxx/d' filename	#删除包含"xxx"的行
sed 's/原字符串/替换字符串/'

```

### 取消等待的任务

```
qstat -u liweimin|grep "Q"|awk -F "." '{if(NR>1)print$1}'|xargs qdel
```

