

+++

title = "Linux"
description = "Linux"
tags = ["it", "sys"]

+++



# Linux

- Linux
  - Linus Benedict Torvalds（Linux之父）编写的开源操作系统的内核
  - 广义上的基于 Linux内核 的 Linux操作系统
- 内核版本：https://www.kernel.org/
  - 内核版本分为三个部分：主版本号、次版本号、末版本号
  - 次版本号是奇数为开发版，偶数为稳定版。但是实际上在2.6以后不这么区分了
- 发行版本：RedHat Enterprise Linux、Fedora、CentOS、Debian、Ubuntu...
- 终端：图形终端，命令行终端，远程终端（SSH、VNC）



## hello

- 镜像
  - CentOS：http://mirrors.aliyun.com/centos/ ，一般下载dvd版本即可
  - Ubuntu：http://mirrors.aliyun.com/ubuntu-releases/ ，20.04/ubuntu-20.04.1-live-server-amd64.iso

### 环境

#### VirtualBox

1. 新建

2. 名称：centos7 → 文件夹：D:\it\virtual_machine\VirtualBoxVM → 类型：Linux → 版本：Red Hat (64-bit)

3. 内存大小：2048mb

4. 现在创建虚拟硬盘 → VDI (VirtualBox 磁盘映像) → 动态分配 → 硬盘大小：20G

5. 设置 → 存储 → 控制器: IDE → 右侧光盘图标选择虚拟盘 → OK

6. 设置 → 网络 → 选择桥接网卡

7. 启动 → Install CentOS Linux 7 → English

8. DATE & TIME → Asia Shanghai

9. SOFTWARE SELECTION → Minimal Install （按需选择）

10. INSTALL ATION DESTINATION → 直接点done即可

11. NETWORK & HOST NAME → Ethernet (enp0s3)：ON

12. 开始安装 → 设置 ROOT PASSWORD → 安装成功后 reboot

13. 虚拟机与宿主机 共享粘贴板、拖拽文件

    1. 设置 - 常规 - 高级 - 共享粘贴板 双向 & 拖放 双向

    2. 设置 - 存储 - 控制器：SATA - √ 使用主机输入输出（I/O）缓存

    3. 设置 - 存储 - 控制器：SATA - .vdi - √ 固态驱动器(s)

    4. kernel-devel

       ```sh
       # 安装 kernel-devel 和 gcc
       yum install -y kernel-devel gcc
       # 更新 kernel 和 kernel-devel 到最新版本
       yum upgrade kernel kernel-devel -y
       # 重启
       reboot
       ```

    5. 虚拟机窗口上方菜单栏 - 设备 - 安装增强功能（会挂载一个光盘，重新安装要先弹出iso，否则报"未能加载虚拟光盘"）

       ```shell
       # centos8上遇到问题
       #提示 please install the gcc make perl packages from your modules，执行后再重新安装增强功能，
       yum install gcc perl make -y
       # 然后查看日志又提示 please install libelf-dev, libelf-devel or elfutils-libelf-devel
       yum -y install elfutils-libelf-devel
       # 还是不行，尝试其它可能影响的内容
       yum -y install gcc-c++
       # 若是以上相关软件或内核更新后，增强功能还无法正常使用，使用一次性更新所有软件（这次可以了）
       yum update
       ```

       



#### VMware

1. 新建虚拟机，自定义，稍后安装操作系统，2cpu2核心以上，内存2048以上，NAT网络，LSI Logic，SCSI
2. 创建新虚拟磁盘，磁盘大小20G以上，将虚拟磁盘存储为单个文件

- CentOS 修改`/etc/sysconfig/network-scripts/ifcfg-ens33`中`NOBOOT=yes`，表示系统启动时激活网卡

### 目录

- 目录
  - /：根目录
  - /root：root用户的家目录
  - /home/username：普通用户的家目录
  - /etc：配置文件目录
  - /bin：命令目录
  - /sbin：管理命令目录
  - /usr/bin、/usr/sbin：系统预装的其他命令





## 命令行

- 命令（command）：通过使用命令调用对应的命令程序文件，如使用`ls`命令将调用`/bin/ls`程序文件
- 参数（option选项）：

### 语法

- 命令 <必选参数1 | 必选参数2> [-option {必选参数1 | 必选参数2 | 必选参数3}] [可选参数...] {(默认参数) | 参数 | 参数}
- 命令行语法符号
  - 方括号**[ ]**：可选参数，在命令中根据需要加以取舍
  - 尖括号**< >**：必选参数，实际使用时应将其替换为所需要的参数
  - 大括号**{ }**：必选参数, 内部使用, 包含此处允许使用的参数
  - 小括号**()**：指明参数的默认值, 只用于{ }中
  - 管道符（竖线）**|**：分隔多个互斥参数, 含义为"或", 使用时只能选择一个
  - 省略号**...**：多个参数
  - 分号**;**：分割多个命令，命令将按顺序执行
- 注意：命令行语法（包括在 UNIX 和 Linux 平台中使用的用户名、密码和文件名）是区分大小写的，如commandline、CommandLine、COMMANDLINE 是不一样的

### 简写

- 多命令：命令之间`;`分隔，顺序执行

```sh
cd / ; ls
```

- 多参数简写

```shell
ls -l -r -t -R
ls -lrtR #简写
```

- 当前目录简写

```shell
ls ./
ls .   #简写。具体到文件（或目录）时无法使用"."作简写
ls     #简写

cd ../
cd ..  #简写

cat ./config.yml
cat config.yml #简写
```

### 用户

```shell
clear     #清屏。或者快捷键CTRL+L
su - root #切换到root用户
exit      #退出当前系统用户
init 0    #关机
```

### 帮助命令

- 为什么要学习帮助命令：Linux的基本操作方式是命令行，海量的命令不适合“死记硬背”，你要升级你的大脑
- 使用网络资源（搜索引擎和官方文档)

###### help

```shell
help cd   #查看内部命令帮助
type cd   #查看命令类型。shell（命令解释器）自带的命令称为内部命令，其他的是外部命令
ls --help #查看外部命令帮助
```

###### info

```shell
info ls #查看ls命令的信息。info比help更详细，作为help的补充
```

###### man

- `man [(1)|2|3|4|5|6|7|8|9] [文件名]`：查看指定文件（程序）对应的手册（如果有的话）。这里1-9指定要查看手册的类型，因为系统中很多重名文件，所以需要分类来区分
- 按`Q`退出手册

```shell
man man   #查看man命令的手册。可以看到默认的选项"1"，表示查看 可执行程序或shell命令 这一类型的文件的手册

man ls    #查看ls命令的手册。选项"1"是默认选项，可以省略
man 1 ls
man -a ls #查看所有名为ls的文件的手册
```



### 文件操作

- 一切皆文件
- TAB快捷键补全文件（目录）名
- 路径：
  - /
  - ./
  - ../



##### pwd

```shell
pwd #显示当前的目录名称
```

##### cd

- 更改当前的操作目录
  - `cd /path/to/...`：绝对路径
  - `cd ./path/to/...`：相对路径
  - `cd ../path/to/...`：相对路径

```shell
man cd #将查看/bin/bash（一个命令解释器shell的实现）的手册，因为cd是bash的内置命令

cd ../ #回到上一级目录
cd ..  #简写

cd - #回到上一次使用的目录。即可以实现两个目录来回切换的效果
```

##### ls

- `ls [option]... [path]...`：查看指定目录下的文件：ls / /root，可以同时显示/和/root下的目录
- 参数（常用）
  - `-l`：
    - 长格式显示文件：文件类型和权限，文件个数，创建用户，创建用户所属用户组，文件大小，最后修改时间，文件名
    - 文件类型：文件为-，目录为d
  - `-a`：显示隐藏文件。文件名以"."开头即隐藏文件
  - `-r`：逆序显示（根据文件名）
  - `-h`：文件大小将使用单位m、g、t等（根据文件大小自判定的），一般配合`-l`使用
  - `-t`：按照时间顺序显示
  - `-R`：递归显示所有文件

```shell
ls -lrt / /root #列出 / 和 /root 下的所有文件
ls -lh
```



##### touch

- `touch <路径文件名>`：新建文件



##### mkdir

- `mkdir [option]... [dirName]...`：新建文件夹

```shell
mkdir ./a/1 ./a/2 #指定路径新建文件夹
mkdir a #在当前目录新建文件夹可以简写
mkdir -p /a/b/c/d/e #-p忽略报错，已存在的不会提示报错，路径上不存在的目录也都将被新建
```



##### rm

```shell
rmdir /a #只能删除空的目录
rm -r /a #可以删除非空目录，但是会进行递归提示询问
rm -r -f /a #参数f使删除时不进行递归提示询问
rm -rf / a  #参数f风险很大，一定要检查好参数，如果像这样/和a之间多了个空格，即删除整个根目录和当前目录下的a目录中的所有文件，gg
```



##### cp

- `cp <源文件名> <目标文件名>`：复制文件
- `-r`：可以复制目录
- `-v`：显示复制进度
- `-p`：保留文件原来的时间属性
- `-a`：保留文件原来的所有属性（`ls -l`能看到）

```shell
cp /a/
```

##### mv

- `mv <源> <目标>`：移动。在同一个目录内移动即实现改名

```shell
mv ./a1/b ./a2 #移动b到a2目录下
mv ./a1/b ./a2/bre #移动并重命名
mv file* ./a #*匹配1或多个字符，所有对应的文件都可以被移动
```



- 通配符
  - 定义: shell  内建的符号
  - 用途:操作多个相似（有简单规律）的文件
  - 常用通配符
    - *：匹配任何字符串
    - ?：匹配1个字符串
    - [xyz]：匹配xyz任意一个字符
  - [a-z]：匹配一个范围
  - [!xyz]、\[^xyz]：不匹配
- 
  - *：匹配一或多个字符
  - ?：匹配一个字符



##### dd

> 文件空洞：文件从开头到结尾的扇区所占用的磁盘空间中，未存储任何数据的部分被称为空洞。比如给linux创建一个虚拟机的磁盘空间1T，但是实际只存放1M的数据，其它都是空洞，用`du`查看将是1M，`ls -lh`查看将是1T

- `dd`：复制文件，可以制造文件空洞。`cp`则不能制造文件空洞
- 参数
  - `if`：输入文件
  - `of`：输出文件
  - `bs`：块大小，即指定多少空间作为一个块进行读写
  - `count`：读写次数（以块为单位的）
  - `seek`：跳过块数

```sh
# 从/dev/zero可以读取无穷多个0，用以测试dd。跳过5个块，即有10M不写入数据，将成为文件的空洞
dd if=/dev/zero of=testfile bs=2M count=10 seek=5
```



##### du

- `du`：查看文件实际占用磁盘空间，不计算文件空洞，默认单位byte。`ls -lh`中显示的文件大小有些不同，是记录文件开头到结尾的磁盘空间，包括范围内的文件空洞

```sh
# 查看：ls -lh和du分别查看afile都是20m
dd if=/dev/zero bs=2M count=10 of=afile
ls -lh afile
du afile

# 查看：ls -lh和du分别查看bfile分别为30、20M
dd if=/dev/zero bs=2M count=10 seek=5 of=bfile
ls -lh bfile
du bfile
```





### 文本

###### cat

- cat：文本内容显示到终端

###### head

- head：查看文件开头。默认显示10行，-5参数显示5行

###### tail

- tail：查看文件结尾。默认显示10行，-5参数显示5行
  - 常用参数-f 文件内容更新后，显示信息同步更新，ctrl+c停止

###### wc

- wc：统计文件内容信息。-l参数显示文件行数

###### more

- more：将分页显示
- less more



### 打包和压缩

- Linux的备份压缩
  - 最早的Linux备份介质是磁带，使用的命令是tar，即打包
  - 可以对打包后的磁带文件进行压缩储存，压缩的命令是gzip和bzip2
  - 经常使用的扩展名是.tar.gz.tar.bz2.tgz
- /etc一般是保存配置文件的目录，属于重点备份的文件，以etc为例进行打包和压缩
- `tar [option]... <目标.tar.压缩方式> <源>`：可以采用双扩展名以方便知道是那种方式打包的

```shell
#打包
tar cf /tmp/etc-bk.tar /etc #c表示打包，f指定文件，tar的选项没有-或者--作选项引导符；
#解包
tar xf /tmp/etc-bk.tar -C /tmp #x表示解包，f指定文件，-C指定要存放的位置，不指定则存放在同目录下

#打包并压缩。可以单独使用gzip或bzip2命令进行压缩，但是tar已经集成了它们的功能。gzip压缩速度更快，bzip2压缩比例更小
tar zcf /tmp/etc-bk.tar.gz /etc #z即使用gzip进行压缩，也可以单后缀.tgz
tar jcf /tmp/etc-bk.tar.bz2 /etc #j即使用bzip2进行压缩，也可以但后缀.tbz2

#解压缩并解包，即zxf或jxf
tar x
```



### 文本编辑



###### vi

- vi是多模式文本编辑器
- 多模式产生的原因
- 四种模式，通过模式的切换，就可以无需鼠标仅使用键盘进行各种各样的文本操作
  - 正常模式(Normal-mode)：进入编辑器界面时的初始模式，显示文本内容，有光标，光标可以移动。该模式下所有键盘输入的按键都是对编辑器所下的命令。
  - 插入模式(Insert-mode)：
    - Normal模式下，使用快捷键 i 进入insert模式，可以输入文本内容
    - 
  - 命令模式(Command-mode)
  - 可视模式(Visual-mode)
- `vi`：进入编辑器，默认进入的是vim的版本，是原始vi编辑器的扩展，是一个同vi向上兼容的文本编辑器
- `vim`：进入vim编辑器，或者`vim <filename>`以vim编辑器打开某个文件
  - esc回到Normal模式，有光标
  - 正常模式
    - h、j、k、l：左、下、上、右，移动光标。在图形界面或远程终端上，如果有 左、下、上、右（箭头）方向键，效果是一样的，但是如果是字符终端，可能会出现乱码
    - 复制
      - 一行：Normal模式下，按yy可以复制一行内容
      - 多行：Normal模式下，按3yy即可复制当前光标所在行开始的3行
      - 光标到行尾：y$
    - 剪切：dd、d$，与y类似
    - 粘贴：按p键，可以在光标所在行的下一行粘贴，继续按p键可以粘贴多次
    - 撤销：u键
    - 重做：把撤销的内容恢复，ctrl+R
    - 删除单个字符：x
    - 替换单个字符：按r键，在输入新的字符
    - 显示行数：`:set nu`
    - 移动到指定行：5G移动到第5行，g移动到第一行，G移动到最后一行
    - ^，或者说shift+6：移动光标到一行开头
    - $，或者或shift+4：移动光标到一行末尾
  - insert
    - Normal模式下，使用快捷键 i 进入insert模式，光标位置不变
    - 大写的`I`，进入插入模式，光标将从，光标将移动到光标所在行的开头
    - 小写a：光标右移一位，
    - 大写A：光标行最右
    - o：光标到下一行，并且是新开空行
    - O：光标到上一行，并且是新开空行
  - 命令模式：
    - `:`：进入命令模式，窗口末尾显示`:`时即表示处于命令模式了，可以键入命令
      - 按esc退出命令模式
      - `w <filename>`：保存文件到目标，如`w /tmp/a.txt`，如果是通过文件名打开的vim编辑器，只需要`w`命令即可保存修改
      - `q`：退出vim编辑器
      - `wq`、`wq <filename>`：连用，即保存并退出
      - `q!`：不保存并退出
      - `!<命令>`：如果想要临时执行一些系统命令，查看系统命令的结果，可以通过该命令来进行，如`!ip addr`查看ip地址，按`回车`即可重新回到编辑器界面
    - `/字符串`：查找指定字符串，按n跳到下一个，shift+n跳到上一个
    - `:s/旧串/新串`：替换光标所在的行的匹配的字符
    - `:%s/旧串/新串`：替换全文匹配的字符第一个字符
    - `:%s/旧串/新串/g`：替换全文匹配的字符所有
    - `:set nu`：显示行数。只单次生效，需要修改vim配置以长期生效
      - `vim /etc/vimrc`：移动到最后一行插入内容`set nu`，保存退出即可
    - `:set nonu`：不显示行数
  - Visual：可视模式 
    - v：字符可视模式
    - V：行可视模式
    - ctrl+v：块可视模式（类似于列模式）
      - `d`：删除选中块
      - `ctrl+i`：进入插入模式，输入要插入的内容，按2次esc，可以批量给选中的每一行前面添加相同内容

### 环境变量

- PATH：指定命令的搜索路径
  - PATH=$PAHT:<PATH 1>:<PATH 2>:--------:< PATH n >
  - export PATH

临时：仅对当前 shell(bash) 或其子 shell(bash) 生效

```sh
export PATH=$PATH:/usr/local/go/bin
```

用户

```sh
# 编辑文件 ~/.bash_profile
vim ~/.bash_profile
# 写入环境变量
export PATH=$PATH:/usr/local/go/bin
# 刷新环境变量使生效
source ~/.bash_profile
```

全局

```sh
# 编辑文件 /etc/profile
vim /etc/profile
# 写入环境变量
export PATH=$PATH:/usr/local/go/bin
# 刷新环境变量使生效
source /etc/profile
```





## grub配置

- grub：centos7使用的是grub2，此前是grub1，grub1中所有的配置文件需要手动去编辑，要像网卡配置文件一样记住每个项是什么功能，grub2则可以用命令即可进行修改

- grub配置文件：/boot/grub2/grub.cfg，一般不要直接编辑该文件，而是修改/etc/default/grub文件（一些基本配置），如果还有更详细的配置，可以修改/etc/grub.d/下的文件
  - grub2-mkconfig -o /boot/grub2/grub.cfg：产生新的grub配置文件
  - /etc/default/grub
    
    - GRUB_DEFAULT=saved。表示系统默认引导的版本内核。通过命令grub2-editenv list查看默认引导的版本内核
    
      - grep ^menu /boot/grub2/grub.cfg：找到文本文件当中包含关键字的一行，^表示以什么开头，这里即在grub配置文件去找到以menu开头的，能够看到内核的列表，按索引以0开始
      - grub2-set-default 0：选择第一个内核作为linux启动时的默认引导
      - grub2-editenv list：查看当前默认引导内核已经改变了
    
    - GRUB_CMDLINE_LINUX：确认引导时对linux内核增加什么参数
    
      - rhgb：引导时为图形界面
    
      - quiet：静默模式，引导时只打印必要的消息，启动出现异常时可以去除quiet和rhgb以显示更多信息
    
      - readhat7重置root密码
    
        - 引导界面时，选择要引导的内核，按E进入设置信息，找到linux16开头的一段，可以发现刚刚quiet、rhgb等信息，可以直接键盘输入添加更多项，在该行末尾添加rd.break，ctrl+x启动
    
        - 进入后是内存中的虚拟的一个文件系统，而真正的根目录是/sysroot（输入命令mount可以发现根为/sysroot），在这里所做的操作是不会进行保存的，并且是只读方式的挂载，不能写，防止修复时损坏原有文件
    
        - `mount -o remount,rw /sysroot`，重新挂载到根目录并且是要可读写的，之后mount，发现有了r,w权限
    
        - chroot /sysroot 选择根，即设置根为/sysroot目录；
    
          - echo 123456|passwd --stdin root 修改root密码为redhat，echo正常情况是打印到终端，这里通过管道符发送给password命令，--stdin是password命令的参数，正常情况是通过终端输入，这里即表示通过标准输入进行输入，并传递给root用户。或者password 123456；或者输入passwd，交互修改；
    
          - SELinux安全组件，叫做强制访问控制，会对etc/password和/etc/shadow进行校验，如果这两个文件不是在系统进行标准修改的，会导致无法进入系统。`vim /etc/selinux/config`中可以通过设置SELINUX=disabled关闭SELinux，即使生产环境也多半会关掉它
          - 或者修改/etc/shadow文件，touch /.autorelabel，这句是为了selinux生效
          - 注意备份
    
        - exit退出根回到虚拟的root中，然后reboot



## 快捷键

- TAB快捷键补全文件（目录）名
- CTRL+L清屏
- CTRL+C终止（命令）程序

## 添加执行权限

chmod a+x 文件名，a表示所有，x表示执行

## 重定向符号

https://blog.csdn.net/hellozpc/article/details/46721811

- 标准输入输出重定向：即指定命令产生的 stdout（标准输出信息）、stdin（标准输入信息） 写入哪个文件
  - `>`：输出重定向到一个文件或设备，覆盖原来的文件。
    - `goland.sh >/dev/null` 表示将 goland 运行产生的 stdout 的输出到 /dev/null 中
  - `>!`：输出重定向到一个文件或设备 强制覆盖原来的文件
  - `>>`：输出重定向到一个文件或设备 追加原来的文件
  - `<`：输入重定向到一个程序
- 标准错误重定向：即指定命令产生的 stderr（标准错误信息） 写入哪个文件
  - `2>`：将一个标准错误输出重定向到一个文件或设备 覆盖原来的文件 b-shell
    - `goland.sh >/dev/null` 表示将 goland 运行产生的 stdout 的输出到 /dev/null 中
  - `2>>`：将一个标准错误输出重定向到一个文件或设备 追加到原来的文件
  - `2>&1`：将一个标准错误输出重定向到标准输出 注释:1 可能就是代表 标准输出
  - `>&`：将一个标准错误输出重定向到一个文件或设备 覆盖原来的文件 c-shell
  - `|&`：将一个标准错误 管道 输送 到另一个命令作为输入

## 别名



## sysctl

可以查看和修改系统参数



## grubby

可以修改内核参数



## 图形界面黑屏

centos7上，使用root用户登录，会出现图形界面黑屏问题，在输入用户名密码之前是有图形界面的，但是输入用户名密码之后桌面出现一秒钟之后转为黑屏

解决问题：通过 ctrl+alt+F2 切换到命令行界面，或者通过其它终端连接该机器，通过 `startx` 命令重启图形界面，可以发现缺少（损失）了某些文件，因为系统启动时会检查用户家目录下所有文件，因缺少相应文件导致图形界面不可用，通过 `startx` 重启图形界面过程中系统会自动重新创建 /root/.Xauthority 文件，图形界面就可以正常使用了。但是重启发现还是会黑屏，仍然需要通过 `startx` 重开图形界面，但是仅提示 `/root/.serverauth.xxx` 文件不存在，

```sh
$ startx
xauth:  file /root/.serverauth.3218 does not exist
xauth:  file /root/.Xauthority does not exist
xauth:  file /root/.Xauthority does not exist


X.Org X Server 1.20.4
X Protocol Version 11, Revision 0
Build Operating System:  3.10.0-957.1.3.el7.x86_64 
Current Operating System: Linux MiWiFi-R3A-srv 3.10.0-1160.24.1.el7.x86_64 #1 SMP Thu Apr 8 19:51:47 UTC 2021 x86_64
Kernel command line: BOOT_IMAGE=/vmlinuz-3.10.0-1160.24.1.el7.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet LANG=zh_CN.UTF-8 user_namespace.enable=1
Build Date: 24 February 2021  09:09:20PM
Build ID: xorg-x11-server 1.20.4-15.el7_9 
Current version of pixman: 0.34.0
        Before reporting problems, check http://wiki.x.org
        to make sure that you have the latest version.
Markers: (--) probed, (**) from config file, (==) default setting,
        (++) from command line, (!!) notice, (II) informational,
        (WW) warning, (EE) error, (NI) not implemented, (??) unknown.
(==) Log file: "/var/log/Xorg.1.log", Time: Wed Apr 21 00:16:03 2021
(==) Using config directory: "/etc/X11/xorg.conf.d"
(==) Using system config directory "/usr/share/X11/xorg.conf.d"
VMware: No 3D enabled (0, Success).
```



## sh -c

这个命令将权限不够，因为重定向符号 “>” 和 ">>" 也是 bash 的命令。我们使用 sudo 只是让 echo 命令具有了 root 权限

```sh
sudo echo "hahah" >> test.csv`
```

 `sh -c` 命令，它可以让 bash 将一个字串作为完整的命令来执行，这样就可以将 sudo 的影响范围扩展到整条命令

```sh
sudo sh -c echo "hahah" >> test.csv`
```





## 关闭swap分区

```sh
# 删除 swap 区所有内容，即临时关闭
swapoff -a
# 删除 swap 挂载，注释 swap 相关行即可，这样系统下次启动不会再挂载 swap
vim /etc/fstab
# 重启
reboot
# 查看。swap 一行应该全部是 0
free -h
```



## 修改静态主机名hostname

```sh
vim /etc/hostname
# 或者
hostnamectl set-hostname xxx
# 查看
hostname
hostnamectl
```



## ubuntu查看内核

```shell
# x86_64,x64,AMD64基本上是同一个东西
arch
```

## ubuntu18.04配置apt源

`/etc/apt/sources.list`

```
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
```



