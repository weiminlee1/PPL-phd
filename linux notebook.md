### linux notebook

#### 系统介绍

```shell
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






```

