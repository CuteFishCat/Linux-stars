## 第三章  Linux文件管理

### 第一节 文件系统
#### 3.1.1 使用lsblk命令查看当前Linux系统的磁盘分区块设备信息   
>CentOS6.10:  
```bash
[root@localhost ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk 
├─sda1   8:1    0    1G  0 part /boot
├─sda2   8:2    0    2G  0 part [SWAP]
└─sda3   8:3    0   17G  0 part /
sr0     11:0    1  3.7G  0 rom  /media/CentOS_6.10_Final
[root@localhost ~]#
```
>CentOS7.6
```bash
[root@localhost ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   20G  0 disk 
├─sda1   8:1    0    1G  0 part /boot
├─sda2   8:2    0   15G  0 part /
└─sda3   8:3    0    4G  0 part [SWAP]
sr0     11:0    1  4.3G  0 rom  /run/media/root/CentOS 7 x86_64
[root@localhost ~]#
```

>Ubuntu1804:
```bash
gordon@gordon:~$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0    7:0    0   91M  1 loop /snap/core/6350
loop1    7:1    0   91M  1 loop /snap/core/6405
sda      8:0    0   20G  0 disk 
├─sda1   8:1    0    1M  0 part 
├─sda2   8:2    0    1G  0 part /boot
├─sda3   8:3    0    2G  0 part [SWAP]
└─sda4   8:4    0   17G  0 part /
sr0     11:0    1  834M  0 rom  
gordon@gordon:~$
```


#### 3.1.2 使用 ls 命令查看 `/` 下的一级目录结构，了解Linux文档目录结构组成
>CentOS6.10
```bash
[root@localhost ~]# cd /
[root@localhost /]# ls
bin   dev  home  lib64       media  mnt  opt   root  selinux  sys  usr
boot  etc  lib   lost+found  misc   net  proc  sbin  srv      tmp  var
[root@localhost /]#
```

>CentOS7.6
```bash
[root@localhost ~]# cd /
[root@localhost /]# ls
bin   dev  home  lib64       media  opt   root  sbin  sys  usr
boot  etc  lib   lost+found  mnt    proc  run   srv   tmp  var
[root@localhost /]#
```

>Ubuntu1804
```bash
gordon@gordon:~$ cd /
gordon@gordon:/$ ls
bin   etc         initrd.img.old  lost+found  opt   run   srv  usr      vmlinuz.old
boot  home        lib             media       proc  sbin  sys  var
dev   initrd.img  lib64           mnt         root  snap  tmp  vmlinuz
gordon@gordon:/$
```

通过上面 ls 命令的执行结果，我们了解到 Linux 文件系统被组织成一个单根倒置树状结构，最顶层的 **根** 用 `/` 来表示。

为了规范和统一不同人员使用Linux系统，诞生了FHS [http://refspecs.linuxbase.org/fhs.shtml](http://refspecs.linuxbase.org/fhs.shtml)，他约定了根下不同目录的用途。


我们可以使用tree命令查看根的树状结构:
tree 命令默认将显示全部路径结构，这里我们使用 `-L 1` 指定只显示一层目录结构：
>CentOS7.6
```bash
[root@localhost /]# tree -L 1
.
├── bin -> usr/bin
├── boot
├── dev
├── etc
├── home
├── lib -> usr/lib
├── lib64 -> usr/lib64
├── lost+found
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin -> usr/sbin
├── srv
├── sys
├── tmp
├── usr
└── var

20 directories, 0 files
[root@localhost /]#
```
下面介绍几个重要目录的用途，详情请参考FHS：  
* /bin      所有用户使用的基本命令;不能关联至独立分区，OS启动即会用到的程序  
* /boot     引导文件存放目录，内核文件(vmlinuz)、引导加载器(bootloader, grub) 都存放于此目录
* /dev      设备文件及特殊文件存储位置
* /etc      存放各种配置文件的目录
* /home     普通用户家目录的位置
* /lib      启动时程序依赖的基本共享库文件以及内核模块文件(/lib/modules)
* /lib64    专用于x86_64系统上的辅助共享库文件存放位置
* /mnt      临时文件系统挂载点
* /opt      第三方应用程序的安装位置
* /proc     用于输出内核与进程信息相关的虚拟文件系统
* /root     超级管理员的家目录
* /sbin     管理类的基本命令;不能关联至独立分区，OS启动即会用到的程序
* /srv      系统上运行的服务用到的数据
* /sys      用于输出当前系统上硬件设备相关信息虚拟文件系统
* /tmp      临时文件存储位置
* /usr      目录包含所有的命令、程序库、文档和其它文件
* /var      可变数据存放目录


#### 3.1.3  Linux文件名规则  
Linux系统中文件由 元数据(metadata) 和 数据(data) 构成。  
文件名在元数据中存储，最长为255个字节，文件名称区分大小写，通常建议文件名不要过长，不要使用特殊字符，使用大小写字母 数字 点`.`  
下划线`_`  减号`-`即可满足需要，若文件名首字符为 点`.` 通常约定表示为隐藏文件。  
要想指定一个文件可以使用绝对路径来表示，因此此路径必须从 根 / 开始，比如使用`ls -al` 命令查看 `/usr/bin/date` 文件，整个路  
径名称不能超过4095个字节:

>CentOS7.6
```bash
[root@localhost /]# ls -al /usr/bin/date
-rwxr-xr-x. 1 root root 62296 Oct 31 03:16 /usr/bin/date
[root@localhost /]#
```

#### 3.1.4  Linux文件类型分类  
|文件类型(7种)  | 简写符号 |
|---------|--------|
|普通文件       |-  |
|目录文件       |d  |
|块设备         |b  |
|字符设备       |c  |
|符号链接文件   |l  |
|管道文件pipe   |p  |
|套接字文件socket|s |

简写符号可以使用ls -l 命令查看，我们查看一下root 家目录:
>CentOS7.6
```bash
[root@localhost ~]# ll
total 40
-rw-------. 1 root root 1863 Feb 14 12:05 anaconda-ks.cfg
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Desktop
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Documents
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Downloads
-rw-r--r--. 1 root root 1894 Feb 14 12:06 initial-setup-ks.cfg
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Music
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Pictures
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Public
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Templates
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Videos
[root@localhost ~]
```
ls显示结果中每行最左边第一个字符代表文件类型，上图中发现有 - 和 d 两种类型文件。下面我们查看一下其他几种文件类型:
>CentOS7.6
```bash
[root@localhost ~]# ls -l /dev/sda
brw-rw----. 1 root disk 8, 0 Mar 10 12:33 /dev/sda
[root@localhost ~]# ls -l /bin
lrwxrwxrwx. 1 root root 7 Feb 14 11:57 /bin -> usr/bin
[root@localhost ~]# ls -l /dev/tty
crw-rw-rw-. 1 root tty 5, 0 Mar 10 12:33 /dev/tty
[root@localhost ~]# ls -l /dev/log
srw-rw-rw-. 1 root root 0 Mar 10 12:33 /dev/log
[root@localhost ~]# ls -l /run/dme*
prw-------. 1 root root 0 Mar 10 12:33 /run/dmeventd-client
prw-------. 1 root root 0 Mar 10 12:33 /run/dmeventd-server
[root@localhost ~]#
```
除了使用 ls 之外，Linux中还可以使用 file 和 stat 命令查看文件类型:
>CentOS7.6
```bash
[root@localhost ~]# file /dev/log
/dev/log: socket
[root@localhost ~]# stat /dev/log
  File: ‘/dev/log’
  Size: 0               Blocks: 0          IO Block: 4096   socket
Device: 5h/5d   Inode: 1343        Links: 1
Access: (0666/srw-rw-rw-)  Uid: (    0/    root)   Gid: (    0/    root)
Context: system_u:object_r:devlog_t:s0
Access: 2019-03-10 20:58:25.518622488 +0800
Modify: 2019-03-10 12:33:15.556000001 +0800
Change: 2019-03-10 12:33:15.556000001 +0800
 Birth: -
[root@localhost ~]#
```

上图中stat命令的返回结果包含三个时间:  

|时间类型|代表含义|
|-------|------|
|Access 时间|读取文件内容时间|
|Modify 时间|修改文件内容的时间|
|Change 时间|修改文件元数据的时间|


#### 3.1.5 CentOS6 和 CentOS7 目录结构变化
主要是bin sbin 和 lib lib64 的变化,CentOS7 将他们存放到/usr下，并创建了软连接和CentOS6进行兼容。  
>CentOS7.6
```bash
[root@localhost /]# tree -L 1
.
├── bin -> usr/bin
├── boot
├── dev
├── etc
├── home
├── lib -> usr/lib
├── lib64 -> usr/lib64
├── lost+found
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin -> usr/sbin
├── srv
├── sys
├── tmp
├── usr
└── var

20 directories, 0 files
[root@localhost /]#
```

#### 3.1.6 当前工作路径、绝对路径、相对路径
Linux系统中表示一个文件要使用完整的路径，即从 / 目录开始，我们把这样的路径成为绝对路径。  
但是绝对路径比较长，使用起来不方便，因此Linux中定义了当前工作目录(PWD)这个系统变量，比  
如root用户登录后默认的当前工作目录为 /root  
>CentOS7.6
```bash
[root@localhost ~]# echo $PWD
/root
[root@localhost ~]#
```
当我指定文件操作时，如果文件路径不是以 / 开始的，则默认将文件名前面添加上当前工作路径变  
量，拼接成绝对路径，如下演示:  
>CentOS7.6
```bash
[root@localhost ~]# ls -l anaconda-ks.cfg 
-rw-------. 1 root root 1863 Feb 14 12:05 anaconda-ks.cfg
[root@localhost ~]# ls -l /root/anaconda-ks.cfg  
-rw-------. 1 root root 1863 Feb 14 12:05 /root/anaconda-ks.cfg
[root@localhost ~]#
```
我们把上面的不以 / 开头的文件路径表示方式成为相对路径，相对的参考内容为 当前工作路径。

文件目录层级使用/进行分割，最后一个分割/后面的内容为文件基名，之前的内容为目录名，我们可以  
使用 basename 和 dirname 分析一个文件的这两个信息:
>CentOS7.6
```bash
[root@localhost ~]# dirname anaconda-ks.cfg 
.
[root@localhost ~]# basename anaconda-ks.cfg 
anaconda-ks.cfg
[root@localhost ~]# dirname /root/anaconda-ks.cfg 
/root
[root@localhost ~]# basename /root/anaconda-ks.cfg    
anaconda-ks.cfg
[root@localhost ~]#
```

上面讲解的当前工作目录非常重要，我们可以根据使用需要使用 cd 命令修改他，修改后系统变量中还保  
存着上次的工作目录成为OLDPWD，可以使用 cd - 在新老工作目录直接切换，比较方便:
>CentOS7.6
```bash
[root@localhost ~]# cd /
[root@localhost /]# echo $PWD
/
[root@localhost /]# echo $OLDPWD
/root
[root@localhost /]# cd -
/root
[root@localhost ~]# echo $PWD
/root
[root@localhost ~]# echo $OLDPWD
/
[root@localhost ~]#
```

由于用户家目录使用频率比较高，cd命令后不加任何参数，可以直接修改当前工作目录为用  
户家目录，使用cd ~ 也可以实现相同效果，CDPATH系统变量给cd命令使用:  
>CentOS7.6
```bash
[root@localhost ~]# cd /
[root@localhost /]# cd
[root@localhost ~]# cd /
[root@localhost /]# cd ~
[root@localhost ~]# echo $PWD
/root
[root@localhost ~]#
```

cd 命令后使用 -P选项后，当前工作目录的系统变量会使用绝对路径:
>CentOS7.6
```bash
[root@localhost ~]# cd /
[root@localhost /]# cd bin
[root@localhost bin]# echo $PWD
/bin
[root@localhost bin]# cd
[root@localhost ~]# cd /usr/bin
[root@localhost bin]# echo $PWD
/usr/bin
[root@localhost bin]# cd
[root@localhost ~]# cd /
[root@localhost /]# cd -P bin
[root@localhost bin]# echo $PWD
/usr/bin
[root@localhost bin]#
```
在执行

之前要想查看当前工作目录的具体内容，需要使用echo打印系统变量的方式，shell中给我们  
提供了 pwd 命令 用以显示当前工作目录:
>CentOS7.6
```bash
[root@localhost ~]# pwd
/root
[root@localhost ~]# cd /
[root@localhost /]# pwd
/
[root@localhost /]#
```

pwd 命令还有两个常用选项，-P 显示真实物理路径，—L 显示连接路径，默认使用 -L 参数:
>CentOS7.6
```bash
[root@localhost ~]# cd /bin
[root@localhost bin]# pwd
/bin
[root@localhost bin]# pwd -P
/usr/bin
[root@localhost bin]#
```

#### 3.1.7 ls 常见用法

ls 命令的功能为列出目录内容，使用方法为 ls  选项   文件， 比如:
>CentOS7.6
```bash
[root@localhost ~]# ls -l anaconda-ks.cfg 
-rw-------. 1 root root 1863 Feb 14 12:05 anaconda-ks.cfg
[root@localhost ~]#
```
常用选项如下表:  

|选项   |选项用途|
|------|-------|
|-a 或者 --all        | 显示结果中包含以 . 开头的文件    |
|-A 或者 --almost-all | 显示结果中去除 . 和 .. 这两个文件|
|-B                   |显示结果中去除以~结尾的文件，通常代表备份文件|
|-c                   |以文件最后修改时间排序显示文件，最后修改的最先显示|
|-cl                  |显示文件最后修改时间，并按文件名称排序显示|
|-clt                 |以文件最后修改时间排序显示文件 |
|-d                   |仅显示出目录文件自身信息不显示其内部包含的文件信息|
|-f                   |不排序，按照目录内文件顺序显示并显示出.开头文件，同时去除显示颜色效果|
|--format=word        |根据不同单词，使用不同风格显示across, commas, horizontal, long, single-column, verbose, vertical|
|--full-time          |使用完整时间格式显示文件时间，效果等同于like -l --time-style=full-iso  |
|-g                   | 效果类似于 -l 选项，但是不显示文件的所有者信息                        |
|--group-directories-first| 显示目录内容时，先显示目录，然后再显示其他文件|
|-G 或者 --no-group        |在-l等长选项显示时，不显示文件所属组信息|
|-h                       |在-l等长格式显示文件时，文件大小以K,M,G为单位方便查看文件大小|
|--hide=PATTERN       |不显示符合PATTERN的对应文件              |
|-i 或者 --inode      |打印出每个文件的inode文件|
|-I 或者 --ignore=PATTERN| 和 --hide 选项类似，不显示出PATTERN匹配到的文件 |
|-l                   |使用格式显示文件内容信息|
|-L 或者 --dereference |使用L选项时，如果使用软连接时显示连接指定的文件信息而非连接文件本身|
|-m                   |在显示文件是使用逗号分隔符将文件隔开显示|
|-n 或者 --numeric-uid-gid |和 -l 选项类似，但是文件的属主和文件的所属用户组使用数字表示|
|-o                   | 和 -l 选项类似，但是不显示组信息|
|-r 或者 --reverse     | 逆序输出之前排序后的文件       |
|-R 或者 --recursive   | 递归显示所有文件夹下的文件内容  |
|-s                   |显示文件所占用的块文件个数|
|-S                   |以文件大小对文件排序进行显示|
|--sort=word          |根据word代表的排序条件对输出内容进行排序, 条件有 none, size,  time, version, extension |
| --time=word     |在 -l 等详细文件信息显示格式下，默认显示文件的修改时间，可以指定word为atime或者access |
|--time-style=SYTLE |在 -l 等详细文件信息显示格式下，可以使用指定时间格式显示:full-iso, long-iso, iso, locale, 还有一种指定时间格式和date命令类似,可以使用+FORMAT2来定义|
|-t               |以文件修改时间排序，最后修改的文件最先显示|
|-U               |不进行排序，默认按目录内的文件顺序显示|
|-v               |按数字和字符顺序排序               |
|-x               |取消以行为单位展示文件             |
|-X               |以文件的扩展名为单位，按照英文字母排序|
|-1               |只有文件名时，按行显示             |
> 以上 ls 命令的使用方法，可以逐个测试验证一下。具体使用时可以快速查询帮助文档即可。


#### 3.1.8 文件名通配符
当一个命令中要求输入文件名称时，除了指明具体的文件名称外，还可以使用文件通配符。
使用 man 7 glob 查看具体规范定义，常见的文件名通配符有：  
|通配符号     |代表的含义|
|------------|--------|
|*           |匹配0到任意个字符|
|？           |代指任意一个字符|
|[0-9]       |匹配数字范围|
|[a-z]       |匹配小写字母|
|[A-Z]        |匹配大小字母|
|[magedu]     |匹配maged5个字符中的任意一个|
|[^magedu]    |匹配magedu5个字符以外的任意一个字符|
|[:digit:]    |任意数字，相当于0-9 |
|[:lower:]    |任意小写字母 |
|[:upper:]    |任意大写字母 |
|[:alpha:]    |任意大小写字母|
|[:alnum:]    |任意数字或字母|
|[:blank:]    |水平空白字符 | 
|[:space:]    |水平或垂直空白字符| 
|[:punct:]    |标点符号 |
|[:print:]    |可打印字符|
|[:cntrl:]    |控制(非打印)字符|
|[:graph:]    |图形字符 |
|[:xdigit:]   |十六进制字符|

### 第二节  文件操作

#### 3.2.1 touch命令
使用 touch 命令可以创建新文件，如果文件已经存在则更新文件的元数据时间。
>CentOS7.6
```bash
[root@localhost ~]# stat anaconda-ks.cfg 
  File: ‘anaconda-ks.cfg’
  Size: 1863            Blocks: 8          IO Block: 4096   regular file
Device: 802h/2050d      Inode: 542218      Links: 1
Access: (0600/-rw-------)  Uid: (    0/    root)   Gid: (    0/    root)
Context: system_u:object_r:admin_home_t:s0
Access: 2019-03-10 20:57:12.246620983 +0800
Modify: 2019-02-14 12:05:57.500905137 +0800
Change: 2019-02-14 12:05:57.500905137 +0800
 Birth: -
[root@localhost ~]# touch anaconda-ks.cfg 
[root@localhost ~]# stat anaconda-ks.cfg 
  File: ‘anaconda-ks.cfg’
  Size: 1863            Blocks: 8          IO Block: 4096   regular file
Device: 802h/2050d      Inode: 542218      Links: 1
Access: (0600/-rw-------)  Uid: (    0/    root)   Gid: (    0/    root)
Context: system_u:object_r:admin_home_t:s0
Access: 2019-03-11 15:27:35.109989242 +0800
Modify: 2019-03-11 15:27:35.109989242 +0800
Change: 2019-03-11 15:27:35.109989242 +0800
 Birth: -
[root@localhost ~]#
```

touch 命令的常用选项:  
|选项|作用|
|-----|------|
|-c   |使用该选项，如果touch的文件不存在，则不创建|
|-a   |touch 的文件存在时，只更新atime 和 ctime|
|-m   |touch 的文件存在时，只根下mtime 和 ctime|
|-t   |使用指定时间修改文件的三个时间,格式[[CC]YY]MMDDhhmm[.ss]|
>CentOS7.6
```bash
[root@localhost ~]# stat anaconda-ks.cfg 
  File: ‘anaconda-ks.cfg’
  Size: 1863            Blocks: 8          IO Block: 4096   regular file
Device: 802h/2050d      Inode: 542218      Links: 1
Access: (0600/-rw-------)  Uid: (    0/    root)   Gid: (    0/    root)
Context: system_u:object_r:admin_home_t:s0
Access: 2019-03-11 16:05:27.118035903 +0800
Modify: 2019-03-11 16:05:27.118035903 +0800
Change: 2019-03-11 16:05:27.118035903 +0800
 Birth: -
[root@localhost ~]# touch -t 201901010000.00 anaconda-ks.cfg  
[root@localhost ~]# stat anaconda-ks.cfg 
  File: ‘anaconda-ks.cfg’
  Size: 1863            Blocks: 8          IO Block: 4096   regular file
Device: 802h/2050d      Inode: 542218      Links: 1
Access: (0600/-rw-------)  Uid: (    0/    root)   Gid: (    0/    root)
Context: system_u:object_r:admin_home_t:s0
Access: 2019-01-01 00:00:00.000000000 +0800
Modify: 2019-01-01 00:00:00.000000000 +0800
Change: 2019-03-11 16:05:53.079036436 +0800
 Birth: -
[root@localhost ~]#
```


#### 3.2.2 cp 命令
Linux系统中我们可以使用cp命令完成目录和文件的拷贝复制工作。  
命令 格式:  
cp [OPTION]... SOURCE DEST   将源文件复制为目标文件   
cp [OPTION]... SOURCE... DIRECTORY  将多个文件拷贝到指定目录中  
cp [OPTION]... -t DIRECTORY SOURCE...  -t选项 将目录写在文件之前的用法

下图指定了文件拷贝的使用场景和情况:
| 目标文件       |               |不存在         |存在且为文件     |存在且为目录                  |
|---------------|---------------|--------------|---------------|---------------------------|
|源文件为一个文件  |               |直接复制源文件  |覆盖目标文件     |将原文件复制到目标文件目录下    |
|源文件为多个文件  |               |提示错误操作    | 提示为错误操作  |将多个源文件复制到目标文件目录下 |
|源文件为目录文件  |必须使用 -r 选项 |直接递归复制源目录|提示错误       |将源目录复制到目标文件目录下    |

下面逐一演示一下上面的各种拷贝场景：
>CentOS7.6 源文件为一个文件
```bash
[root@localhost ~]# ls ss
ls: cannot access ss: No such file or directory
[root@localhost ~]# cp /etc/passwd ss
[root@localhost ~]# ls -al ss
-rw-r--r--. 1 root root 2310 Mar 11 17:15 ss
[root@localhost ~]# 
[root@localhost ~]# cp /etc/passwd ss
cp: overwrite ‘ss’? y
[root@localhost ~]# 
[root@localhost ~]# cp /etc/passwd ./
[root@localhost ~]# ls -al passwd 
-rw-r--r--. 1 root root 2310 Mar 11 17:16 passwd
[root@localhost ~]#
```

>CentOS7.6 源文件为多个文件
```bash
[root@localhost ~]# ls -al ./ss
ls: cannot access ./ss: No such file or directory
[root@localhost ~]#
[root@localhost ~]# cp /etc/passwd /etc/shadow ./ss
cp: target ‘./ss’ is not a directory
[root@localhost ~]# 
[root@localhost ~]# cp /etc/passwd /etc/shadow anaconda-ks.cfg 
cp: target ‘anaconda-ks.cfg’ is not a directory
[root@localhost ~]#
[root@localhost ~]# cp /etc/passwd /etc/shadow ./
[root@localhost ~]# ls -al passwd shadow 
-rw-r--r--. 1 root root 2310 Mar 11 17:03 passwd
----------. 1 root root 1265 Mar 11 17:03 shadow
[root@localhost ~]# 
```

>CentOS7.6 源文件为目录文件
```bash
[root@localhost ~]# ls -al ss
ls: cannot access ss: No such file or directory
[root@localhost ~]# cp -r /etc/sysconfig/network-scripts/ ss
[root@localhost ~]# ls
anaconda-ks.cfg  Documents  initial-setup-ks.cfg  Pictures  ss         Videos
Desktop          Downloads  Music                 Public    Templates
[root@localhost ~]# 
[root@localhost ~]# touch tt
[root@localhost ~]# cp -r /etc/sysconfig/network-scripts/ tt
cp: cannot overwrite non-directory ‘tt’ with directory ‘/etc/sysconfig/network-scripts/’
[root@localhost ~]#
[root@localhost ~]# cp -r /etc/sysconfig/network-scripts/ Downloads/
[root@localhost ~]# ls Downloads/
network-scripts
[root@localhost ~]#
```

下面介绍了常用的 cp 命令选项:

|选项     |作用                  |
|--------|---------------------|
|-a      |递归复制，并且保留源文件的所有属性信息，相当于-dR --preserve=all|
|--attributes-only|复制时只复制文件元数据，数据内容为空|
|--backup=contral|当要覆盖文件时，先对源文件进行备份，支持的参数有：none, off, simple, never, existing, nil, numbered, t|
|-b      |和 --backup功能相同，但是不用指定参数,备份生成的文件已~结尾|
|-d|不复制链接内容，只复制链接文件本身 相当于-no-dereference --preserv=links|
|-i 或者 --interactive   |当覆盖文件时提示确认信息 |
|-L| 复制文件时复制链接文件指向的内容，而非链接文件本身，和 -d 命令效果相反|
|-n      |文件拷贝时禁止进行覆盖操作|
|-p      |复制文件的mode,文件所属主和组信息,时间信息|
|--preserve[=ATTR_LIST]|指定复制文件时要复制的属性，属性有:mode,ownership,timestamps,context, links, xattr, all|
|--no-preserve=ATTR_LIST| 和 --preserve=ATTR_LIST 相反, 不指定要复制的元数据信息|
|-r -R        |递归复制目录内容必须的选项 |
|-s     |通过创建符号链接的方式来实现复制的效果|
|-S     |指定特定的备份文件后缀名,可以和-b选项组合使用|
|-t     |复制文件到目录时可以指定目录|
|-T     |复制时指明目标文件不是目录，为普通文件|
|-u 或者 --update |如果源文件比目标文件新，或者目标文件不存在时才进行复制|
| -v, --verbose| 显示复制的过程|
> 以上 cp 命令的使用方法，可以逐个测试验证一下。配合 man 帮助理解具体含义。



#### 3.2.3 mv 命令
移动或者重命名文件，mv命令可以实现文件的移动一个文件到另一个路径下，若移动的位置还在源目录则实现了修改文件名称的功能。

命令使用风格和 cp 命令类似:  
mv [OPTION]... [-T] SOURCE DEST   
mv [OPTION]... SOURCE... DIRECTORY  
mv [OPTION]... -t DIRECTORY SOURCE...  
|选项     |作用                  |
|--------|---------------------|
|--backup=contral|当要覆盖文件时，先对源文件进行备份，支持的参数有：none, off, simple, never, existing, nil, numbered, t|
|-b      |和 --backup功能相同，但是不用指定参数,备份生成的文件已~结尾|
|-f 或者 --force | 当目标文件存在时,强制覆盖,不进行提示|
|-i 或者  --interactive| 当目标文件存在时，提示是否进行覆盖操作|
|-n 或者 --no-clobber|当目标文件存在时不进行覆盖|
|-S 或者 --suffix=SUFFIX|和 需要覆盖备份时，可以指定备份文件的后缀名|
|-t     |移动文件到目录时 可以指定目录|
|-T     |移动时指明目标文件不是目录，为普通文件|
|-u 或者 --update |移动源文件比目标文件新，或者目标文件不存在时才进行移动|
| -v, --verbose| 显示复制的过程|



#### 3.2.4 rm 命令
使用 rm 命令删除文件或者目录，删除的文件不可恢复，日常应 **慎重** 进行删除操作。  
基本使用:
>CentOS7.6
```bash
[root@localhost ~]# ls -adl file*
-rw-r--r--. 1 root root    0 Mar 15 21:29 file11
-rw-r--r--. 1 root root    0 Mar 15 21:29 file12
-rw-r--r--. 1 root root    0 Mar 15 21:29 file13
-rw-r--r--. 1 root root    0 Mar 15 21:30 file14
-rw-r--r--. 1 root root    0 Mar 15 21:30 file15
drwxr-xr-x. 2 root root 4096 Mar 15 21:31 filedir
[root@localhost ~]# rm file15
rm: remove regular empty file ‘file15’? y
[root@localhost ~]# rm file14 file13 file12 file11 
rm: remove regular empty file ‘file14’? y
rm: remove regular empty file ‘file13’? y
rm: remove regular empty file ‘file12’? y
rm: remove regular empty file ‘file11’? y
[root@localhost ~]#
```
默认无法删除目录，请使用 -r 选项删除非空目录和其内部的文件，文件名以 - 开头时可以使用 -- 选项或者使用./文件名方式进行删除。  
>CentOS7.6
```bash

```

rm 命令使用选项:  
|选项     |作用                  |
|--------|---------------------|
|-f      |忽略不存在的文件或参数，不进行提示是否确定删除文件|
|-i      |每次删除前都进行提示是否确认删除文件|
|-I       |同时删除文件数大于3，或者带有递归删除选项时，只进行一次确认删除提示|
|--interactive=when|当没有when或者when为always时，每个文件删除时都进行确认提示，等同于 -i 选项；when为once时等同于 -I 选项，when为never时从不进行删除确认提示|
|-r, -R, --recursive|递归删除目录和其内部的文件|
|-d 或者 --dir| 删除空目录文件|
|-v 或者 --verbose|显示删除的文件名称|

#### 3.2.5 目录操作命令

tree 命令:  
使用 tree 命令可以方便查看目录层级结构关系，常见的选项有:  
|选项  |作用    |
|-----|-------|
|-d         |只显示目录|
|-L level   |指定显然目录的层级深度，默认显示所有内容|
|-P pattern |只显示由指定的pattern匹配到的路径     |

>CentOS7.6
```bash
[root@localhost ~]# ll
total 44
-rw-------. 1 root root 1863 Feb 14 12:05 anaconda-ks.cfg
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Desktop
drwxr-xr-x. 2 root root 4096 Mar 15 21:30 dir
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Documents
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Downloads
-rw-r--r--. 1 root root    0 Mar 15 22:01 file
drwx------. 5 root root 4096 Mar 12 19:18 gordon
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Music
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Pictures
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Public
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Templates
drwxr-xr-x. 2 root root 4096 Feb 14 12:07 Videos
[root@localhost ~]# tree -d
.
├── Desktop
├── dir
├── Documents
├── Downloads
├── gordon
├── Music
├── Pictures
├── Public
├── Templates
└── Videos

10 directories
[root@localhost ~]#
[root@localhost ~]# tree P*
Pictures
Public

0 directories, 0 files
[root@localhost ~]# tree D*
Desktop
Documents
Downloads

0 directories, 0 files
[root@localhost ~]#
[root@localhost ~]# tree -L 1 /
/
├── bin -> usr/bin
├── boot
├── dev
├── etc
├── home
├── lib -> usr/lib
├── lib64 -> usr/lib64
├── lost+found
├── media
├── mnt
├── opt
├── proc
├── root
├── run
├── sbin -> usr/sbin
├── srv
├── sys
├── tmp
├── usr
└── var

20 directories, 0 files
[root@localhost ~]#
```


mkdir 命令  
使用 mkdir 命令可以创建文件夹，常见选项如下：  
|选项     |作用     |
|--------|---------|
|-p      |创建文件夹时，如果父目录不存在，则默认自动创建| 
|-v      |显示目录的创建过程信息|
|-m      |指定目录创建时的文件夹权限|
>CentOS7.6
```bash
[root@localhost ~]# mkdir -p /data/newdir
[root@localhost ~]# mkdir -pv /data/olddir
mkdir: created directory ‘/data/olddir’
[root@localhost ~]#
[root@localhost ~]# mkdir magedu
[root@localhost ~]# ls -ald magedu
drwxr-xr-x. 2 root root 4096 Mar 18 17:45 magedu
[root@localhost ~]# mkdir -m 777 magedu2
[root@localhost ~]# ls -ald magedu*
drwxr-xr-x. 2 root root 4096 Mar 18 17:45 magedu
drwxrwxrwx. 2 root root 4096 Mar 18 17:45 magedu2
[root@localhost ~]#
```

rmdir 命令  
使用rmdir命令可以删除空文件夹，常见选项如下：  
|选项     |作用   |
|--------|-------|
|-p      |删除目录和其上级的空父目录|
|-v      |显示删除文件夹的信息|
>CentOS7.6
```bash
[root@localhost ~]# mkdir -pv magedu/newdir/a
mkdir: created directory ‘magedu’
mkdir: created directory ‘magedu/newdir’
mkdir: created directory ‘magedu/newdir/a’
[root@localhost ~]# rmdir magedu/newdir/a
[root@localhost ~]# rmdir -pv magedu/newdir/
rmdir: removing directory, ‘magedu/newdir/’
rmdir: removing directory, ‘magedu’
[root@localhost ~]#
```



### 第三节 文件系统结构

#### 3.3.1 硬盘

硬盘是存储数据的媒介，计算机通过操作系统将数据写入磁盘，实现数据存储。硬盘内的磁盘存储数据的最小单位是一个扇区大小为512Byte。

#### 3.3.2 文件系统

通常用户不能直接访问硬盘，我们都是在硬盘上进行分区，然后创建特定的文件系统，通过使用文件系统
来操作硬盘。常见的文件系统有，windows的FAT32，NTFS，Linux的ext4，xfs，Btrfs等。文件系统
以块为单位来使用磁盘，目前主流使用8个连续的磁盘扇区构成一个块，大小为4kB，如果一个文件实际大小不到4kB，则也占用这一个完整的块。

现代文件系统通常包含以下几个重要组成部分：

* 引导块：  
作为文件系统的第一个块，不被文件系统使用，包含引导文件系统相关的信息。

* 超级块：  
在引导块之后，用以存储文件系统有关的参数信息。

* inode节点：  
用以存储文件元数据信息，多个文件名指向同一个inode时，即为 hard link （硬链接);  
当文件数据块比较小时，inode节点中直接保存数据块位置，若文件数据块比较大时，inode节点中保存间接  
数据块地址，然后间接数据块最终保存实际文件的数据块地址。间接数据块还可以再指向间接数据块，一次来实  
现更大数块信息的保存。inode编号为0的块为间接数据块，文件系统根目录的inode编号为2。
>CentOS6.7
```bash
[root@localhost /]# ll -di / 
2 dr-xr-xr-x. 18 root root 4096 Jan  4 22:23 /
[root@localhost /]# ls -di /
2 /
[root@localhost /]# ls -i / 
    17 bin   655361 etc       15 lib64       786435 mnt   262146 root  393220 srv  131074 usr
     2 boot  786434 home      11 lost+found      16 opt     8572 run        1 sys  524290 var
     3 dev       13 lib   262147 media            1 proc      18 sbin  393218 tmp
[root@localhost /]#
```

* 数据块：
实际存在文件数据的位置。

* 日志功能：通过存储数据跟新事务日志信息，加快系统崩溃后数据一致性检查时间。
> ****可是使用dumpe2fs查看文件系统相关信息 比如：`[root@localhost ~]# dumpe2fs /dev/sda1`

>系统访问文件方式:   
1.在文件的所在文件夹中找到文件名指定的inode编号  
2.通过inode编号找到文件元数据  
3.通过文件元数据中保存的文件数据块位置信息找到文件最终数据。 

#### 3.3.3 硬链接和符号链接

* 硬链接：不同文件名指向同一个inode节点，文件类型通常不可以是目录文件；不能夸磁盘分区；目录内的`. ..`文件就是硬链接。
* 软连接：翻译名称为符号链接，对应数据块中存储指向文件的文件路径，如果文件指向的路径是相对路径，则相对路径的参照文件为  
符号链接文件本身；可以跨分区；对应的指向文件可以不存在。符号链接可以指向符号链接，Linux内核一般限定最多40次的连续符号  
链接引用。

#### 3.3.4 虚拟文件系统
由于每种文件系统具体实现细节各不相同，Linux中通过VFS（虚拟文件系统）来适配各种具体的文件系统，通过接口统一了对文件系统的访问操作。
