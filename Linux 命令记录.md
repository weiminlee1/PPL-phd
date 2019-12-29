### Linux 命令记录

```shell
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

  paste file1 file2 > file3

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



```

