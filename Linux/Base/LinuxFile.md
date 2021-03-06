`目录 start`
 
- [【文件管理】](#文件管理)
    - [Tips](#tips)
        - [设置交换分区](#设置交换分区)
            - [清空交换内存](#清空交换内存)
        - [清除缓存](#清除缓存)
        - [善用*shrc文件](#善用shrc文件)
            - [善用alias](#善用alias)
    - [基本命令](#基本命令)
        - [查找文件](#查找文件)
        - [查看文件](#查看文件)
        - [比较两个文件](#比较两个文件)
            - [不同](#不同)
                - [diff](#diff)
            - [相同](#相同)
        - [文件管理命令](#文件管理命令)
            - [合并文件](#合并文件)
    - [流编辑器](#流编辑器)
        - [sed](#sed)
        - [awk](#awk)
    - [【磁盘管理】](#磁盘管理)
        - [分区介绍](#分区介绍)
        - [设备列表](#设备列表)
        - [常用命令](#常用命令)
            - [dd](#dd)
            - [mount](#mount)
            - [fdisk](#fdisk)
            - [df](#df)
            - [du](#du)
    - [日志](#日志)
        - [用户日志](#用户日志)
        - [系统日志](#系统日志)
        - [应用日志](#应用日志)
    - [在Linux上操作压缩文件的命令](#在linux上操作压缩文件的命令)
        - [tar](#tar)
            - [tar归档和压缩](#tar归档和压缩)
        - [rar](#rar)
        - [zip](#zip)
        - [gz](#gz)
        - [7Z](#7z)
        - [总结](#总结)
    - [常用文件](#常用文件)
        - [划分](#划分)
        - [使用](#使用)
            - [查看发行版](#查看发行版)
            - [查看系统所有用户信息](#查看系统所有用户信息)

`目录 end` |_2018-07-13_| [码云](https://gitee.com/gin9) | [CSDN](http://blog.csdn.net/kcp606) | [OSChina](https://my.oschina.net/kcp1104)
****************************************
# 【文件管理】
> Linux中认为万物皆文件

- 清空文件内容 `true > a.txt ` 
- 安装上传下载文件的工具 `sudo apt install lrzsz`
- `cat ~/.ssh/id_rsa.pub | xsel -b` 将文件复制到剪贴板

## Tips
### 设置交换分区
- 查看内存 `free -h` 
- 创建一个4g 交换文件 `dd if=/dev/zero of=/swapfile bs=1024k count=4096` 
- 格式化成交换文件的格式 `mkswap /swapfile` 
- 启用该文件作为交换分区的文件 ` swapon /swapfile` 
- `/swapfile swap swap defaults 0 0` 写入`/etc/fstab`文件中，让交换分区的设置开机自启
- `sudo sysctl vm.swappiness=15` 临时修改重启注销失效， 查看：`cat /proc/sys/vm/swappiness`
- 永久修改：`/etc/sysctl.conf ` 文件中设置开始使用交换分区的触发值： `vm.swappiness=10`
    - 表示物理内存剩余`10%` 才会开始使用交换分区
- `建议，笔记本的硬盘低于 7200 转的不要设置太高的交换分区使用，大大影响性能，因为交换分区就是在硬盘上，频繁的交换数据`

#### 清空交换内存
- 1.关闭交换分区 `sudo swapoff 交换分区文件`
    - 2.开启交换分区 `sudo swapon 交换分区文件`
- 或者 `swapoff -a && swapon -a`

### 清除缓存
> [参考: 如何在 Linux 中清除缓存（Cache）？](https://linux.cn/article-5627-1.html) `注意要切换到root再运行命令`  
> 仅清除页面缓存（PageCache） `sync; echo 1 > /proc/sys/vm/drop_caches`  
> 清除目录项和inode `sync; echo 2 > /proc/sys/vm/drop_caches`  
> 清除页面缓存，目录项和inode `sync; echo 3 > /proc/sys/vm/drop_caches`  

- 有时候, 因为缓存的问题会引发一些很诡异的问题, 有应用缓存和系统缓存的分别
    - 例如构建工具Maven, 也会因为在一个项目空间下, 多个同名项目的缓存问题 
        - (巨诡异 `if(true){}` 都能不执行, 一个变量的值莫名其妙的时刻被修改 )
    - 然后操作系统的缓存问题也有出现:
        - 例如 启动一个web项目, 运行到某一行, 突然运行不下去了, 前端阻塞在了等待请求这里 重新编译 和请求target目录都没有用, 重启IDEA也没有用
        - 然后找到了上面的博客, 清除了下缓存就OK了

### 善用*shrc文件
> 注意加载顺序 /etc/profile -> ~/.*shrc `各种sh的rc文件` bash zsh ash

#### 善用alias

```sh
    if [ -f ~/.bash_aliases ]; then
        . ~/.bash_aliases
    fi
```
- 在`~/.bashrc`添加这段，然后在 `.bash_aliases` 文件中设置别名
    - 例如 ： `alias Kg.notes='cd ~/Documents/Notes/Code_Notes/'` 
    - 更改文件后，想当前终端就生效就 `source ~/.bashrc` 不执行命令就重启终端即可
_注意_
> 你会发现 当前用户 下 Kg.notes 是正常运行的, 但是 sudo Kg.note 就会报错说找不到命令  
> 神奇的是 配置一个别名 `alias sudo='sudo '` 就可以解决这个问题了 [stackoverflow](https://askubuntu.com/questions/22037/aliases-not-available-when-using-sudo)
_官方说明如下_
```
    The first word of each simple command, if unquoted, is checked to see if it has an alias. If so, that word is replaced by the text of the alias. 
    The characters ‘/’, ‘$’, ‘`’, ‘=’ and any of the shell metacharacters or quoting characters listed above may not appear in an alias name. 
    The replacement text may contain any valid shell input, including shell metacharacters. The first word of the replacement text is tested for aliases, 
    but a word that is identical to an alias being expanded is not expanded a second time. This means that one may alias ls to "ls -F", for instance, 
    and Bash does not try to recursively expand the replacement text. If the last character of the alias value is a space or tab character, 
    then the next command word following the alias is also checked for alias expansion. 
```
- 如[我的配置文件](https://github.com/Kuangcp/Configs/tree/master/Linux/init) `将配置文件分类放`
    - K.h就能显示出每个命令的说明 其实现脚本： [python3脚本](https://github.com/Kuangcp/Script/blob/master/python/show_alias_help.py) 
    - 在别名文件目录时, 建立链接就可以用了 `ln -s `pwd`/.bash_aliases ~/.bash_aliases` 

*************************
`自定义桌面快捷方式文件`
```conf
	[Desktop Entry] #每个desktop文件都以这个标签开始，说明这是一个Desktop Entry文件
	Version = 1.0 #标明Desktop Entry的版本（可选）
	Name = Firefox #程序名称（必须），这里以创建一个Firefox的快捷方式为例
	GenericName = Web Browser #程序通用名称（可选）
	Comment = A Web Browser #程序描述（可选）
	Exec = firefox %u #程序的启动命令（必选），可以带参数运行,当下面的Type为Application，此项有效
	Icon = firefox #设置快捷方式的图标（可选）
	Terminal = false #是否在终端中运行（可选），当Type为Application，此项有效
	Type = Application #desktop的类型（必选），常见值有“Application”和“Link”
	Categories = GNOME;Application;Network; #注明在菜单栏中显示的类别（可选）
```
- [示例文件](https://github.com/Kuangcp/Notes/tree/master/ConfigFiles/Linux/VSCode.desktop)
- 如要将快捷方式放在启动菜单内 将desktop文件放在 `/usr/share/applications/` 目录下即可
- 注意：目录不能有空格 等特殊字符

**************************************
## 基本命令 
### 查找文件
_cd_
- `cd - ` 跳转到上一个目录
- `cd !$` 把上个命令的参数作为cd参数使用。
- `cd //` 系统根目录 

_find_
- `find . -name "*.txt"` 查找当前目录的txt后缀的文件
- `sudo find / -name a.java` 全盘查找
- `find -type f -name README.md` 默认当前目录查找
    - d 文件夹 f 普通文件 l 符号链接文件 b 块设备 c 字符设备 p 管道文件 s 套接字

> 递归删除目录下所有run后缀的文件 `find . -name "*.run"  | xargs rm -f`

> 查找文件内容 `find etc/  |xargs grep -i java`
### 查看文件
_tree_
- 展示目录结构
    - -p 匹配
    - -h 可读的显示文件大小
    - -F 和ls一样
    - -L 目录深度

_ls_
- `参数`
    - `i` 详情 
    - `a` 全部包含隐藏文件 <> `A` 不显示当前目录和上级目录 `.` `..` 
    - `l` 使用较长格式列出信息 详细信息
    - `h` 人类可阅读
    - `F` 标明文件夹,文件,可执行文件 
    - `w` 100 限制输出每行的字符长度 0 则是无限制 和 l 共用就则无视该限制
    - `R` 递归显示所有子文件夹
    - `r` 逆序
    - `B` 不列出以〜结尾的隐含条目
    - `t` 按修改时间从顶至下,一般不单用,和 g|l 结合一起用
    - `c` 按ctime(创建时间)一般是文件夹,文件则是修改时间排列
        - 和 lt|gt 一起用 即 `ls -clt` 同上的排列顺序
- `执行ls -l 命令后的输出`
    1. 输出类型：d 目录 l 软链接 b 块设备 c 字符设备 s socket p 管道 - 普通文件
    1. 输出权限信息：r 读权限 w 写权限 x 执行权限
        - rwx有三个，是因为 `拥有者，所属用户组 其他用户` 代表的rwx权限
        - ![权限输出图](https://dn-anything-about-doc.qbox.me/linux_base/3-10.png/logoblackfont)
        - ![权限计算图](https://dn-anything-about-doc.qbox.me/linux_base/3-14.png/logoblackfont)
        - `chmod 700 文件` 就是只设置拥有者具有读写权限
        - 加减权限操作 `chmod go-rw 文件` `g group` `o others` `u user` `+- 增减权限`
    1. 硬链接数 一般你可以理解成子目录数（对于普通文件，总是1，对于目录来说，为对应目录的 _下一级子目录的个数_ +2 (+2是由于 . 和 .. 的原因））
    1. 最后是归属用户和用户组, 大小, 最后修改日期

- `ls -lFh` 列出所有文件的详细信息, 并且文件大小是人类可阅读的

_file_
- file a.txt 查看文件类型
    - -i 输出文件的MIME类型
    - -F "#" 修改输出分隔符

_wc_
- `wc [-lmw] ` 参数说明： -l :多少行-m:多少字符 -w:多少字
- cat mul.sh | wc -l
- wc -l mul.sh

_cat_
> 类似的还有 nl more less

带行号输出 `cat -n file` 或者 `nl file`但是空行不会编号, 除非这样: `nl -b a file`

_nl_
- [参考博客: 每天一个linux命令(11)：nl命令](http://www.cnblogs.com/peida/archive/2012/11/01/2749048.html#/)

_less_
> 该命令的导航是和Vi体系一样的, 建议打开大文件使用less或者more 如果用vim,文件全加载到内存了  
> 诸多软件使用到了分页, 怀疑就是借助less实现的, 因为快捷键一模一样, 例如 man命令, 各个软件的-h, git的log 等等..优点很多

- [ less命令简介](https://blog.csdn.net/caihaijiang/article/details/6113419)
- z/b 上下翻页 g/G 文件首/尾 
- F 监听文件


_tail_
> tail命令用于输入文件中的尾部内容。tail命令默认在屏幕上显示指定文件的末尾10行。 来自: http://man.linuxde.net/tail

- `--retry`：即是在tail命令启动时，文件不可访问或者文件稍后变得不可访问，都始终尝试打开文件。此选项需要与选项“——follow=name”连用； 
- `-c或——bytes=`：输出文件尾部的N（N为整数）个字节内容； 
- `-f或；--follow`：显示文件最新追加的内容。“name”表示以文件名的方式监视文件的变化。“-f”与“-fdescriptor”等效； 
- `-F`：与选项“-follow=name”和“--retry"连用时功能相同； 
- `-n或——line=`：输出文件的尾部N（N位数字）行内容。 
- `--pid=<进程号>`：与“-f”选项连用，当指定的进程号的进程终止后，自动退出tail命令； 
- `-q或——quiet或——silent`：当有多个文件参数时，不输出各个文件名； 
- `-s<秒数>或——sleep-interal=<秒数>`：与“-f”选项连用，指定监视文件变化时间隔的秒数；
- `-v或——verbose`：当有多个文件参数时，总是输出各个文件名；

```
    tail file （显示文件file的最后10行） 
    tail +20 file （显示文件file的内容，从第20行至文件末尾） 
    tail -c 10 file （显示文件file的最后10个字符）
```
_head_
> 查看文件头部, 前十行

### 比较两个文件
#### 不同
- `grep -vwf 文件1 文件2`
##### diff
> [参考博客](http://www.cnblogs.com/chenjianhong/archive/2012/09/26/4144940.html)

#### 相同

### 文件管理命令
_rename_
`rename命令的使用(基于perl)`
- `rename "s/.html/.php/" * ` //把.html 后缀的改成 .php后缀
- `rename "s/$/.txt/" *  `   //把所有的文件名都以txt结尾
- `rename "s/.txt//" *  `   //把所有以.txt结尾的文件名的.txt删掉
- `rename "s/AA/aa/" * `  //把文件名中的AA替换成aa
- `rename "s/ - 副本/_bak/" *` 将文件`-副本`结尾改成`_bak`结尾

_chown_
- `chown [-R] 账号名称 文件或目录`
- `chown [-R] 账号名称:用户组名称 文件或目录`

_chgrp_
- 更改文件所属用户组 `chgrp group file`
    - -R 递归子目录
    
_ln_
- `ln -s 源文件或目录 目标绝对路径` 生成软链接（快捷方式）
```sh
    ln -s `pwd`/a.md ~/a.md 
```

_cp_
- cp   `cp -ri 目录或正则 目录` 目录所有文件复制过去
    - a 该选项通常在拷贝目录时使用。它保留链接、文件属性，并递归地拷贝目录，其作用等于dpR选项的组合。
    - d 拷贝时保留链接。
    - f 删除已经存在的目标文件而不提示。
    - i 和f选项相反，在覆盖目标文件之前将给出提示要求用户确认。回答y时目标文件将被覆盖，是交互式拷贝。
    - p 此时cp除复制源文件的内容外，还将把其修改时间和访问权限也复制到新文件中。
    - r 若给出的源文件是一目录文件，此时cp将递归复制该目录下所有的子目录和文件。此时目标文件必须为一个目录名。
    - l 不作拷贝，只是链接文件。

_rm_
- rm  `rm -rf 目录` 不提示性删除
    - f 忽略不存在的文件，从不给出提示。
    - r 指示rm将参数中列出的全部目录和子目录均递归地删除。
    - i 进行交互式删除(y/n的询问)。 

> 特别注意 rm -rf link 文件时, 如果只是想删除link文件 那么就不要在link文件后加上 / 例如:  
> `rm -rf linkDir/ ` 这个命令是将 link到的目录下的文件全部删除而不是 删除link文件本身 

_wc_
- `wc [-lmw] ` 参数说明： -l :多少行-m:多少字符 -w:多少字
- cat mul.sh | wc -l
- wc -l mul.sh

_mv_
- mv `mv 目录或正则 目录` 移动
    - I 交互方式操作。如果mv操作将导致对已存在的目标文件的覆盖，此时系统询问要求用户回答y或n，这样可以避免误覆盖文件。
    - f 禁止交互操作。在mv操作要覆盖某已有的目标文件时不给任何指示，指定此选项后，i选项将不再起作用。

#### 合并文件
1. 最简单就是 `cat file1 file2 > result`

*****************
## 流编辑器
### sed
> 参数 命令 文件

- `参数`
    - `-n` 直接在控制台输出的操作的结果，源文件不变 
    - `-i` 在源文件中进行修改
- `命令`
    - p 打印 `sed -n Np 文件名`
    - a 新增 在下一行
    - i 插入 在上一行 将hello插入到第4行：`sed -in "4i hello" test.md`
    - c 替换 整行
    - s 替换 字符串的替换
    - d 删除 行级别, 删除2-4行 `sed -i "2,4d" test.md`

> 1. 截取指定行数到新文件 `sed -n ‘开始行数，结束行数p’ info.log > newFile.log`
> 2. 修改配置文件中name的值为123 `sed -i "s/name=.*/name=123/g" config.conf`

### awk
> awk有3个不同版本: awk、nawk和gawk，未作特别说明，一般指gawk，gawk 是 AWK 的 GNU 版本。

*****************************
## 【磁盘管理】
### 分区介绍
- /root 系统用户 root 的主目录
- /home 其他用户的主目录分配路径
- /boot 系统引导目录

### 设备列表
- /dev/random 产生随机数的设备

### 常用命令
- 创建一个空文件 `dd if=/dev/zero of=virtual.img bs=1M count=256` 查看大小 `du -h virtual.img`
- 将虚拟磁盘镜像格式化为指定的文件系统 `sudo mkfs.ext4 virtual.img`
- 查看支持的文件系统 `ls -l /lib/modules/$(uname -r)/kernel/fs`
- 挂载这个镜像到`/mnt`：`mount -o loop -t ext4 virtual.img /mnt`
- 只读方式挂载 `mount -o loop --ro virtual.img /mnt`
- 卸载挂载的磁盘 `sudo umount /mnt`

#### dd
> [使用 dd 命令进行硬盘 I/O 性能检测 ](https://linux.cn/article-6104-1.html)

#### mount
- `mount [options] [source] [directory] `
- `mount [-o [操作选项]] [-t 文件系统类型] [-w|--rw|--ro] [文件系统源] [挂载点]`
- 查看已挂载信息 `mount`

#### fdisk
- 查看磁盘分区表信息 ：`sudo fdisk -l `

#### df 
- `df -h ` 查看挂载文件系统信息

#### du
- `du -sh 目录` 查看磁盘占用总大小 h 自动搭配单位（human read ）
- `du --max-depth` 一级子目录使用情况
- du -sm * | sort -n //统计当前目录大小 并安大小 排序
- du -sk * | sort -n
- du -sk * | grep guojf //看一个人的大小
- du -m | cut -d "/" -f 2 //看第二个/ 字符前的文字

*********************************
## 日志
> 基本都在 `/var/log` 下

### 用户日志
- last 查看用户最后登录时间

### 系统日志
- 查看系统启动日志 `less /var/log/boot.log` 
- 查看系统消息日志 `less /var/log/messages`

### 应用日志
> [处理Apache日志的Bash脚本](http://www.ruanyifeng.com/blog/2012/01/a_bash_script_of_apache_log_analysis.html)

************************************
## 在Linux上操作压缩文件的命令
> Linux默认自带ZIP压缩，最大支持4GB压缩，RAR的压缩比大于4GB.

### tar

1  `这五个是独立的参数, 五个参数之间互斥`
- c : 打包 压缩
- x : 解压
- t : 查看内容 不解压
- r : 向压缩归档文件末尾追加文件
- u : 更新原压缩包中的文件

2 `可选参数`  
2.1 `下面的参数是根据需要在压缩或解压档案时可选的。`
- z：有gzip属性的
- j：有bz2属性的
- Z：有compress属性的
- v：显示所有过程
- O：将文件解开到标准输出

2.2 `其他可选参数`
- `-p` 保留绝对路径符
- `-v` 将压缩或解压的过程输出

3 `最后`
- `-f 是必须的,-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接文件或目录`

> 以上则组合出了 tar 的所有使用场景

***************************
**示例 :**
- `tar -rf all.tar *.gif`这条命令是将所有.gif的文件增加到all.tar的包里面去。
- `tar -uf all.tar logo.gif`这条命令是更新原来tar包all.tar中logo.gif文件，
- `tar -tf all.tar` 这条命令是列出all.tar包中所有文件，
- `tar -xf all.tar` 这条命令是解出all.tar包中所有文件，

- 保留文件属性和跟随链接， -p 保留属性 -h 备份的源文件而不是链接本身
    - `tar -cphf etc.tar /etc`

********************
#### tar归档和压缩
_压缩_
- tar -cf a.tar *.txt **仅仅归档,没有压缩**
    1. `-czf` tar.gz **gz压缩**
    1. `-cjf` tar.bz2 
    1. `-cZf` tar.Z
    1. `-cJf` tar.xz

_解压_
- `tar -xf file.tar`      // 解压 tar
- `tar -xzf file.tar.gz`  // 解压 tar.gz
- `tar -xjf file.tar.bz2` // 解压 tar.bz2
- `tar -xZf file.tar.Z `  // 解压 tar.Z

****************
### rar
_压缩_
- `rar a jpg.rar *.jpg`    // rar格式的压缩

_解压_
- `unrar x file.rar`       // 解压 rar
    - `e` 不保留目录结构,平铺解压

********************
### zip
_压缩_
- `zip images.zip *.jpg` //zip格式的压缩
- `zip -r file.zip code/*` 压缩code目录下所有文件
    - `zip -r ./a.zip ./*` 压缩当前目录所有文件
    - `-q`安静模式, 终端不输出
    - `-o` 输出文件`
    - `-r` 表示递归
    - `-l` 兼容Windows的换行符
    - `-e` 加密
    - `-d filename` 在zip中删除某文件 删除某目录`dir/*`
        - _注意_: 所有的文件和目录都是相对于zip的根目录的完整路径

_解压_
- `unzip file.zip `//解压zip
    - -q 终端不输出 
    - -d 指定解压目录 
    - -l 不解压,查看所有文件 
    - -O 指定编码

***************************
### gz
> gunzip 

- [ ] gunzip 命令的学习
***************************
### 7Z
> 安装 apt install p7zip-full 或者 p7zip 
> man 7z 查看帮助文档  
-  `7z <command> [<switches>... ] <archive_name> [<file_names>... ] [<@listfiles>... ]`
    - b benchmark 评测分数 [个人电脑评测](https://gitee.com/kcp1104/codes/0r72axdcp1yewmnljhi8g38)
    
_压缩_ 
- a 压缩包名 文件名 

_解压_
- 7z x file
    - -o 路径

- [ ] 7z命令的 学习使用

***************
### 总结
| 文件名模式 | 解压方式 |
|:----|:----|
| *.tar |  用 tar -xvf 解压 |
| *.gz|用 gzip -d或者gunzip 解压|
|*.tar.gz和*.tgz| 用 tar -xzf 解压|
|*.bz2|用 bzip2 -d或者用bunzip2 解压|
|*.tar.bz2|用tar -xjf 解压|
|*.Z|用 uncompress 解压|
|*.tar.Z| 用tar -xZf 解压|
|*.rar|用 unrar e 解压|
|*.zip|用 unzip 解压|

**************************
## 常用文件
### 划分
> Linux 系统配置文件的大致分布

- [ ] 学习
### 使用
> 具体配置文件的使用

#### 查看发行版
`cat /etc/issue` 通用
`cat /etc/redhat-release` redhat系

_查看内核版本_
`cat /proc/version`
`uname -a`

#### 查看系统所有用户信息
> /etc/passwd 包含了用户,用户组,用户home目录 shell类型等信息  
> 看第三个参数:500以上的,就是后面建的用户了.其它则为系统的用户.
