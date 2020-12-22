+++

title = "Linux"
date = "2020-11-11"
description = "linux"
tags = ["sys", "linux"]

+++



# linux

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
  - CentOS：http://mirrors.aliyun.com/centos/ ，7.8.2003/isos/x86_64/CentOS-7-x86_64-DVD-2003.iso
  - Ubuntu：http://mirrors.aliyun.com/ubuntu-releases/ ，20.04/ubuntu-20.04.1-live-server-amd64.iso

### 环境

#### VirtualBox

1. 新建
2. 名称：centos7 → 文件夹：D:\it\virtual_machine\VirtualBoxVM → 类型：Linux → 版本：Red Hat (64-bit)
3. 内存大小：2048mb
4. 现在创建虚拟硬盘 → VDI (VirtualBox 磁盘映像) → 动态分配 → 硬盘大小：20G
5. 设置 → 存储 → 右侧光盘图标选择虚拟盘 → OK
6. 启动 → Install CentOS Linux 7 → English
7. DATE & TIME → Asia Shanghai
8. SOFTWARE SELECTION → Minimal Install
9. INSTALL ATION DESTINATION → 直接点done即可
10. NETWORK & HOST NAME → Ethernet (enp0s3)：ON
11. 开始安装 → 设置ROOT PASSWORD → 安装成功后reboot

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

## 用户



```shell
useradd user1 #添加新用户
#创建用户后会在/home中创建用户对应的目录，目录名与用户名一致
ls -a /home/yuanya #会有一些隐藏文件
tail -10 /etc/passwd #会有用户相关内容
tail -10 /etc/shaow #会有用户相关内容，密码相关

id #查看当前用户
id root #查看指定用户。可以借此验证是否存在指定用户
#用户会有唯一id，系统是通过id识别不同用户的。root用户uid=0，如果把普通用户的uid改为0，则也会被系统当作root用户对待
#用户有用户组，创建时不指定用户组则属于其同名组，即用户组名与用户名一致，只有该用户一人
groupadd group1 #新建用户组
groupdel group1 #删除用户组
useradd user2 group1 #添加新用户并指定用户组

passwd #为当前用户自己设置密码
passwd yuanya #为指定用户设置密码

userdel yuanya#删除用户，用户的家目录/home/yuanya会被保留，文件所属用户变为数字，只有root用户可以使用，加-r则不会保留家目录。/etc/passwd、/etc/shaow中相关用户信息都将被删除

man usermod #usermod用于修改用户属性
usermod /home/yuanya yuanyatianchi #修改家目录/home/yuanya的名字为yuanyatianchi，相当于搬家
usernod -g group1 yuanya #将指定用户分配到指定用户组

man chage #更改用户密码过期信息

#root用户切换普通用户无需密码，普通用户切换其它用户需要密码
su - user1 #切换用户，"-"表示用户及用户运行环境的完全切换，这里运行环境切换指自动进入到user1的家目录/home/user1，root的家目录是/root
su user1 #不完全切换，仍在之前用户的家目录中，但是已经没有ls读取权限了，需要手动切换user1自己的家目录
```

## 网络配置

##### 网卡

查看

```shell
ip addr #查看网卡及相关信息。简写，等价于ip addr ls、ip addr show
ip addr ls ens33 #查看指定网卡。不能省略ls或show
ip addr add 10.0.0.1/24 dev ens33 #设置ip

ip link #查看网卡物理连接情况。ip link ls、ip link show等都是一样的
ip link ens33 #查看指定网卡
ip link set dev ens33 up #网卡启动
ip link set dev ens33 down #网卡停止
```

修改

- 网卡接口命名修改：默认为`ens33`或`enp0s3`之类的。没有特殊需求默认即可
  - `vim /etc/default/grub`编辑grub，在`GRUB_CMDLINE_LINUX`参数内容的末尾`rhgb quiet`的后面添加` biosdevname=0 net.ifnames=0`两项，网卡命名受这两个参数影响（如下表）
  - `grub2-mkconfig -o /boot/grub2/grub.cfg`更新到真正的启动配置文件
  - `reboot`重启

|       | biosdevname | net.ifnames | 网卡名           |
| ----- | ----------- | ----------- | ---------------- |
| 默认  | 0           | 1           | ens33或enp0s3... |
| 组合1 | 1           | 0           | em1              |
| 组合2 | 0           | 0           | eth0             |



##### 网关

```shell
ip route #查看网关信息。ip route ls、ip route show等都是一样的
ip route | column -t #格式化一下

#设置路由
ip route add default gw <网关ip> #设置默认路由
ip route add -host<指定ip> gw<网关ip> #为目的地址设置指定网关
ip route add -net<指定网段> netmask <子网掩码> gw<网关ip> #为目的网段设置指定网关
ip route add 192.168.56.0/24 via 192.168.56.2 dev ens33 #设置静态路由

#删除路由
ip route del 192.168.56.0/24 #删除静态路由
```



##### 网络故障排除命令

- 从上到下逐步分析，基本能够解决大部分网络问题了
- ping www.baidu.com：到目标主机是否畅通
- traceroute www.baidu.com：追踪路由。centos7没有原装
- mtr www.baidu.com：检查到目标主机之间是否有数据丢失。centos7没有原装
- nslookup www.baidu.com：搜索域名的ip。centos7没有原装
- telnet www.baidu.com：检查端口连接状态，键入^]然后quit退出telnet。centos7没有原装
- tcpdump -i any -n host 10.0.0.1 and port 80 -w /tmp/filename：更细致的分析数据包，抓包工具，后面详细讲。centos7没有原装。centos7没有原装
- netstat -ntpl：查看本机服务监听地址。n显示ip地址而不是域名，t以tcp截取显示的内容（udp等就不显示了），p显示端口对应的进程，l即listen表示处于监听状态的服务。centos7没有原装，通过ss -ntpl代替
- ss -ntpl：查看本机服务监听地址。n显示ip地址而不是域名，t以tcp截取显示的内容（udp等就不显示了），p显示端口对应的进程，l即listen表示处于监听状态的服务

##### 网络服务管理

- 前面的很多配置都是临时的，网卡重启后就重置了，通过配置文件修改才能实现持久配置

- 网络服务管理程序分为两种，分别为SysV和systemd（centos7中新的），最好不要同时使用两套网络管理系统，可以选择关闭其一

  - SysV
    - service network start|stop|restart|status：service的操作
    - chkconfig -list network：查看SysV服务在不同运行级别中的启用状况
    - `chkconfig --level 2345 network <on|off>`：打开或关闭运行级别为2、3、4、5的SysV服务。这样就将网络管理都交给systemd的NetworkManager了

  - systemd的NetworkManager额外功能在于：比如插入网线（或者连接无线）可以识别网卡激活状态自动进行一些网络激活，对个人电脑来说颇有用处，对服务器来说稍显鸡肋。如果都是新写一些脚本，可以使用systemd；如果有一些SysV的脚本需要延用，为了方便可以继续使用SysV
    - systemctl list-unit-files NetworkManager.service：查看是否开启
    - systemctl start|stop|restart NetworkManger.service：启动、停止、重启NetworkManger
    - systemctl enable|disable NetworkManger：启用、禁用NetworkManger


##### 网络配置相关文件

- /etc/sysconfig/network-scripts/ifcfg-ens33（根据网卡名有变）

  - BOOTPROTO="dhcp"：遵循dhcp协议的动态ip，可以改为"none"表示静态ip

  - ONBOOT="yes"：是否开机启动，"no"时则需要手动启动网卡

  - 静态内容

    ```shell
    TYPE=Ethernet
    UUID=045d35e8-49bc-4865-b0
    NAME=eth0
    DEVICE=eth0
    ONBOOT=yes
    B0OTPROTO=none #静态
    IPADDR=10.211.55.3 #地址
    NETMASK=255.255.255.0 #子网掩码
    GATEWAY=10.211.55.1 #网关
    DNS1=114.114.114.114 #nds，可以有3个，NDS1、NDS2、NDS3
    ```

  - service network restart或systemctl restart NetworkManger.service重启网卡即可生效

- 主机

  - `hostname`查看
  
  - `hostname 临时主机名`
  
  - `hostname set-hostname 永久主机名`
  
  - hosts配置文件：/etc/hosts。因为有些服务绑定的可能是主机名，所以记得在hosts文件的最后面中配置映射。否则可能启动时会卡住某些服务直到超时，会很慢
  
    ```shell
    127.0.0.1 主机名如yuanya.tianchi
    ```
  
  - reboot重启主机以验证
  
    

## 软件管理

> 包管理器是方便软件安装、卸载，解决软件依赖关系的重要工具

- Centos、RedHat使用yum包管理器，软件安装包格式为rpm（RedhatPackageManager）
- Debian、Ubuntu使用apt包管理器，软件安装包格式为deb（）

软件包管理器
rpm包和rpm命令
yum仓库
源代码编译安装
内核升级
grub配置文件

###### rpm

- rpm包格式
  - vim-common-7.4.10-5.el7.x86_64.rpm
  - 软件名称-软件版本.系统版本.平台.rpm
- 参数
  - `-q`：查询软件包
  - `-i`：安装软件包
  - `-e`：卸载软件包
- ls -l查看/dev，这里面都是设备文件，发现c和b的文件类型，c表示字符设备，b表示块设备，把光盘加载到虚拟机即加载到/dev/sr0中的，`dd if=/dev/sr0 of=/xxx/xxx.iso`就可以把真的光盘做成光盘镜像，块设备不能通过cp等命令直接进行操作，需要挂载（类似于windows中插入光盘后会自动挂载弹出新盘符，linux需要自行手动操作），`mount /dev/sr0 /mnt `，linux下推荐挂载到/mnt目录，-t可以指定挂载类型，默认则会是自动识别。之后可以发现文件是只读的，但是可以拷贝
- `rmp -qa | more`：a是查询所有的意思，可以查询所有系统安装的软件包，软件包很多，通过管道符` | more`分屏显示，按空格下一页，按q退出
- `rmp -q vim-common`：根据软件名查询软件包名
- `rmp -e vim-enhanced`：卸载软件
- `rmp -i vim-enhanced-7.4.160-5.el7.x86_64.rpm`：安装软件包。vim-enhanced依赖vim-common，如果vim-common没有被安装，将安装失败，所以需要先安装vim-common，如果把两个软件包都放在同一个目录，也可以自动安装依赖

###### yum

- rpm问题很明显了，如果是一个庞大的依赖树，将会非常恐怖，难以人工安装，所以就有了yum仓库（包管理器），用于实现rmp安装自动依赖；还有如果版本不符合要求，还需要通过源代码编译安装软件包
- rpm包的问题：需要自己解决依赖关系，软件包来源不可靠
- Centos yum源：http://mirror.centos.org/centos/7/
- 国内镜像：https://developer.aliyun.com/mirror/

- 配置yum源

```shell
#备份
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bk
#下载ailiyun的yum源配置文件，无wget用curl亦可
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
#安装epel，用于扩展yum仓库可以安装的软件包，比如最新的linux内核等
install epel-release -y
#epel的aliyun镜像配置
wget http://mirrors.aliyun.com/repo/epel-7.repo -O /etc/yum.repos.d/epel.repo
#清空并刷新缓存
yum clean all && yum makecache
```

###### apt

替换为阿里源：https://blog.csdn.net/wangyijieonline/article/details/105360138

```shell
lsb_release -a #查看代号codename
```

到阿里源看下对应代号的源是否存在：http://archive.ubuntu.com/ubuntu/dists/ ，存在则可以根据模板进行替换

```sh
#把所有的TODO替换成系统的codename
deb http://mirrors.aliyun.com/ubuntu/ TODO main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ TODO main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ TODO-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ TODO-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ TODO-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ TODO-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ TODO-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ TODO-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ TODO-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ TODO-backports main restricted universe multiverse
```

```sh
#以codename=focal为例
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```



###### wget

```shell
yum install wget -y
```

- yum常用选项：-y：安装、更新过程中选yes
  - `yum install [软件名]`：安装软件包
  - `yum remove [软件名]`：卸载软件包
  - `yum list`查看软件列表，listl grouplist查看软件包
  - `yum update`检查yum仓库并更新所有可更新软件包，`yum update [软件名]`更新指定软件包
- 如果要使用yum中没有的或者还未更新的包，可以使用使用源代码编译安装的方式安装软件包



###### 二进制安装

即类似与windows上的大部分软件安装方式，安装过程也需要授权各种协议，也非常麻烦



###### 源代码编译安装

- 将源代码编译成可以执行程序，copy到指定目录使用即可
- 源代码编译安装
  - 获取压缩包：wget https://openresty.org/download/openresty-1.15.8.1.tar.gz
  - 解压：tar -zxf openresty-VERSION.tar.gz
  - 进入目录：cd openresty-VERSION/
  - ./configure --prefix=/usr/local/openresty：当前的系统环境已经预设在源代码中了，但是没有与真正的系统环境匹配，运行可执行文件configure使其配置（没有的话看看有没有README等指导），--prefix指定程序目录
  - make -j2：编译。j2表示用2个逻辑上的cpu进行编译，如果代码之间没有上下文依赖关系则可以提高编译速度
  - make install：把编译好的程序安装到--prefix指定的目录
- 源代码编译安装可能会遇到各种错误，比如依赖问题，需要根据提示逐一解决，非常麻烦，不到迫不得已还是yum吧



- linux内核升级
  - rpm格式内核
    - uname -r：查看内核版本
    - yum install kernel-3.10.0：升级指定内核版本
    - yum update：升级已安装的软件包（包括linux内核）和补丁
  - 源代码编译安装内核
    - yum install gcc gcc-C++ make ncurses-devel openssl-develeurutls-Deu-uevet：安装依赖包，可以看到有这么多依赖包，如果提前不知道，直接安装的话就需要逐个查看报错并安装解决
    - 下载并解压缩内核并解压：https://www.kernel.org ，生成环境一定要用稳定stable或者长期支持langterm版本，tar xvf linux-5.1..10.tar.xz -C /usr/src/kernels解压（tar已经直接支持xz格式了）
    - 配置内核编译参数
      - cd /usr/src/kernels/linux-5.1.10/
      - make menuconfig | allyesconfig | allnoconfig。选择menuconfig弹出会出来界面菜单自行配置，allyesconfig则全部配置yes，allnoconfig则全部配置no（甚至可能无法启动）
        - menuconfig配置支持NTFS文件系统：找到Filesytems，找到NTFS file system support，空格进行选择，默认为空[ ]表示no，变为[M]表示作为内核的模块，将编译进内核使用模块的方式加载，模块意味着可以移除，以减小内核体积，[*]表示固化到内核中，不能被移除。选中NTFS后还出现了子选项，比如NTFS write support表示写入支持，子选项的[\*]表示固化到父选项，而不是内核。
        - 其实这个未安装的内核，其所有配置都在内核目录下的.config文件中，启用的项等于m或者y，未启用的则被#注释掉了，非常熟悉的话甚至可以直接修改配置文件
        - 当前使用中的内核的配置文件，在/boot目录下，如果想要沿用当前系统内核配置，拷贝配置文件即可：cp /boot/config-kernelversion.platform /usr/src/kernels/linux-5.1.10/.config
        - 当然仍然可以再make menuconfig进入菜单进行更多修改
    - 编译：make -j2 all，通过lscpu查看CPU信息
    - 安装内核：注意保证磁盘空间充足，df -h可以查看磁盘分区信息
      - 先安装内核所支持的模块：make modules_install
      - 再安装内核：make install
    - reboot重启进入系统引导界面，可以发现新安装的内核可以选择了

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

## 进程管理

- 进程：进行中的程序。运行中的程序，从程序开始运行到终止的整个生命周期是可管理的。C程序的启动是从main函数`int main(int agrc, char*argv[])`开始的，终止的方式分为正常终止、异常终止，正常终止有从main返回、调用exit等方式，异常终止有调用abort、接收信号等方式。计算机资源不足时等情况进行，进程和权限有着密不可分的关系。


### 进程查看

##### ps

```shell
ps -efL
```

- ps：process status，进程状态
  - 参数
  - -e：可以查看更多进程，类似于win中的系统进程
    - -f：额外的信息UID、PPID
    - -L：Light，表示轻量级，轻量级进程，实际上即线程
  - 内容
    - PID：唯一标识进程（不同用户使用同样的程序也是不同的进程）
    - TTY：终端的次要装置号码（minor device number of tty）。即表示当前执行程序的终端
    - UID：效用户id。表示进程是由哪个用户启动的信息（可修改），默认显示为启动进程的用户
    - PPID：进程的父进程id。（linux的 0号进程 和 1 号进程：https://www.cnblogs.com/alantu2018/p/8526970.html ，注意centos7的systemd即以前的init）
- pstree：查看进程的树形结构，父子进程关系清晰，但是是静态查看的，ps是动态刷新的。非默认安装的程序，需要自行下载

##### top

- top：系统状态，包括进程状态，是动态更新的
  - 参数
    - -p 1：指定pid查看进程
  - 内容

    - 系统状态
      - up：运行时长
      - users：当前登录用户数量
      - load average：平均负载，系统进行采用对不同时间内的系统负载进行的计算，3个数值分别是1、5、15分钟内的负载，1即满负载
      - Tasks：任务状态
        - total：当前运行的总进程数
        - running：运行进程数
        - sleeping：睡眠进程数
        - stopped：停止进程数
        - zombie：僵尸进程数
      - %Cpu(s)：cpu使用情况（平均值）
        - us：用户空间使用cpu占比
        - sy：内核空间使用cpu占比
        - ni：用户进程空间内改变过优先级的进程使用cpu占比
        - id：空闲cpu占比
        - wa：等待输入输出的CPU时间百分比
        - hi：硬件CPU中断占用百分比
        - si：软中断占用百分比
        - st：虚拟机占用百分比
      - KiB Mem：内存状态
        - total：物理内存总量
        - used：使用内存量
        - free：空闲内存量
        - buffers：用作内核缓存的内存量
      - KiB Swap：交换区状态
        - total：交换区总量
        - used：使用交换区量
        - free：空闲交换区量
        - buffers：缓冲的交换区量。内存中的内容被换出到交换区，而后又被换入到内存，但使用过的交换区尚未被覆盖，该数值即为这些内容已存在于内存中的交换区的大小，相应的内存再次被换出时可不必再对交换区写入
    - 进程状态：
      - PID：进程id
      - USER：进程所有者的用户名
      - PR：优先级
      - NI：nice值。负值表示高优先级，正值表示低优先级
      - VIRT：进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
      - RES：进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
      - SHR：共享内存大小，单位kb
      - S：进程状态(D=不可中断的睡眠状态,R=运行,S=睡眠,T=跟踪/停止,Z=僵尸进程
      - %CPU：上次更新到现在的CPU时间占用百分比
      - %MEM：进程使用的物理内存百分比
      - TIME+：进程使用的CPU时间总计，单位1/100秒
      - COMMAND：命令名/命令行
  - 操作
    - `s`：按s可以输入数字更改状态刷新间隔（默认3秒/次），回车确认



### 控制命令

#### 优先级

##### nice

- nice：优先级调整。优先级值从-20到19，值越小优先级越高，抢占资源就越多。写一个无限循环的脚本demo.sh，`./demo.sh`运行
  - -n 10 ./demo.sh：设置demo.sh的优先级为10
- renice：重新设置优先级
  -  -n 15：重新设置优先级值为15

#### 作业控制

##### jobs

- `jobs`：查看后台作业的工作程序
  - 内容
    - 
  - `bg 1`指定程序序号使其到后台运行
  - `fg 1`指定程序序号使其到前台运行，通过CTRL+Z可以再入后台并暂停挂起，可以通过bg或fg再运行

##### &

- `&`：使程序后台运行

```shell
./demo.sh &
```



### 进程的通信方式—信号

- 信号是进程间通信方式之一，典型用法是:终端用户输入中断命令，通过信号机制停止一个程序的运行。
- 使用信号的常用快捷键和命令
  - kill -l：查看所有信号
    - SIGINT：2号信号，通知前台进程组终止进程，可以被程序处理，快捷键CTRL+C
    - SIGKILL：9号信号，立即强制结束程序，不能被阻塞和处理，kill -9 [pid]

- nohup：一般与&符号配合运行一个命令

  - nohup命令使进程忽略hangup（挂起）信号：比如一个进程正在前台运行，关闭该终端则将发起hangup信号，会使该进程被关掉，如果使用nohup则忽略该信号，即使关闭终端也不会被关闭，但是会变成一个孤儿进程，因为终端关闭了，其父进程没了，但是会被新终端的1号进程作为父进程，nohup启动的进程仍然是用户有关的，会随着用户终端而改变

- 守护（Daemon）进程：随着开机启动，是用户无关的、在用户之前启动的进程，不需要用户终端的，因为没有终端打印日志等信息，所以一般是以文件的形式记录日志信息

  - 守护进程会将其使用的目录切换为根目录，这是什么意思呢，比如windows下，如果使用一个软件的时候，你要删除这个软件所在的目录，会提示目录被使用无法删除，实际上进程运行是基于所在目录的，而根目录只有在关机或重启时才会被卸载

- 系统日志：/proc，这个目录下所有的内容在硬盘中默认是不存在的，它是操作系统去内存中读取信息以文件的形式进行呈现。比如启动了一个进程，会有与进程号同名的目录，类似于：/proc/27451，进入即可看到关于进程属性的文件

  - ps -ef | grep sshd：sshd为例
  - ls -l cwd：可以看到该进程使用的目录
  - ls -l fd：可以看到标准输入输出及其输入输出的目录，输入一般是/dev/null表示没有，因为因为终端都没有；输出如果是nohup则一般是输出到，如果是Daemon程序一般是通过socket输出给系统日志程序，系统日志程序将打印到默认的/var/log/下进程对应的目录文件下

- cd /var/log：存放系统日志，由进程通过socket通信给日志系统，写入该目录对应文件。下面是一些常见日志

  - tail -f /var/log/message：该文件被写入一些系统常规日志
  - tail -f /var/log/dmesg：内核启动日志
  - tail -f /var/log/secure：安全日志
  - tail -f /var/log/cron：cron周期任务日志信息

- 使用screen命令：因为守护进程就是为了使进程脱离终端，防止进程因终端关闭而关闭，screen工具也可以实现，进入终端操作时先进入screen的环境中，即使终端因为某些原因比如网络中断而断开，screen还是可以继续运行程序，下次连接时也能过screen恢复，
  - yum install screen
  - screen进入screen环境
  - 先按ctrl+a，然后按d，即可退出(detached) screen环境
  - screen -ls查看screen的会话，会话有sessionid唯一标识
  - screen -r sessionid恢复会话

- 服务管理工具systemctl

  - service：执行简单，但是启动停止重启的脚本完全由你自己编写，服务控制的好坏全凭编写脚本的人决定
    - cd /etc/init.d：该目录存放service启动的各种服务脚本，比如network
    - 级别：chkconfig --list，可以发现已经被systemd取代了，https://blog.csdn.net/ctthuangcheng/article/details/51219848
      - 运行级别0：系统停机状态，系统默认运行级别不能设为0，否则不能正常启动
      - 运行级别1：单用户工作状态，root权限，用于系统维护，禁止远程登陆
      - 运行级别2：多用户状态(没有NFS)
      - 运行级别3：完全的多用户状态(有NFS)，登陆后进入控制台命令行模式
      - 运行级别4：系统未使用，保留
      - 运行级别5：X11控制台，登陆后进入图形GUI模式
      - 运行级别6：系统正常关闭并重启，默认运行级别不能设为6，否则不能正常启动
  - systemctl：
    - vi /usr/lib/systemd/system：该目录存放systemctl启动的各种服务脚本，比如sshd.service
    - enable：随着开机启动
    - 级别文件：cd /lib/systemd/system，有很多.service，其中级别相关的是：ls -l runlevel*.target
    - 查看当前级别：systemctl get-default
    - 设置默认开机级别：systemctl set-default multi-user.target

- 

- 

### SELinux

- 安全增强linux。一般是利用用户、文件的权限进行安全控制，即自主访问控制；强制访问控制：给用户、进程、文件打上标签，只要用户、进程、文件的标签能对应一致即可以允许访问和控制，比如之前的通过grub进入救援模式修改etc中的password和shadow，selinux就会拒绝访问linux的密码，就需要在根目录下/.autorelabel
- 会降低服务器性能
- MAC（强制访问控制）与DAC（自主访问控制)
- 查看SELinux的命令
  - getenforce：查看selinux的状态。默认有3种状态，在/etc/selinux/config文件中，持久修改的话需要重启，临时修改可以通过如setenfortce 0设置
  - 查看标签（label）：ps -Z查看进程标签，id -Z查看用户标签，ls -Z查看文件标签
  - /usr/sbin/sestatus
  - ps -Z and ls -z and id -z
- 关闭SELinux
  - setenforce 0
  - letc/selinux/sysconfig



## 内存&磁盘管理

### 状态查看

##### top

- `top`：可以查看内存和交换



##### free

- `free`：查看内存和交换，默认单位bit，`-m`、`-g`等参数指定单位

```sh
free
```



##### fdisk

- `fdisk`：磁盘在linux中也是作为文件，如`/dev/sd?`，以扇区划分
  - start：起点扇区
  - end：止点扇区
  - system：文件系统类型，一般为linux（或其它）类型，比如ntfs就不能被linux文件系统读取，除非格式化为linux文件系统，或者编译内核时设置开启ntfs

```sh
# 查看磁盘情况
fdisk -l
```

- 操作

```sh
# 操作指定磁盘/dev/sda，将进入交互模式
fdisk -l /dev/sda
```



##### parted

- `parted`

```sh
# 与`fdisk -l`展示的信息基本一致
parted -l
```



##### df

- `df`：可以理解为`fdisk`的补充，可以看到分区挂载的目录等

```sh
df -h
```





### 文件系统

- Linux支持多种文件系统，常见的有
  - ext4（centos6默认）
  - xfs（centos7默认）
  - NTFS （需安装额外软件ntfs-3g，有版权的，windows用）

##### ext4

- 结构
  - **超级块**：会记录文件数，有副本
  - **超级块副本**：恢复数据用
  - **inode**：i 节点，记录每一个文件的信息，权限、编号等。文件名与编号不在同一inode，而是记录在其父目录的inode中
    - `ls`查看的是inode中的文件信息，`ls -i`可以查到每一个文件的inode
    - vim写文件会改变inode，vim编辑时，会在家目录下复制一份临时文件进行修改，然后保存复制到源目录，并删除原来的文件（应该是）
    - `rm testfile`是从父inode删除文件名，所以文件再大都是秒删
    - `ln afile bfile`将bfile指向afile并也加入到父inode，且不会占用额外空间
  - **datablock**：数据块，存放文件的数据内容，挂在inode上的，如果一个数据块不够就接着往后挂，链式。默认创建的数据块为4k，即使只写了1个字符也是4k，所以存储大量小文件会很费磁盘，所以网络上有一些专门用来存储小文件的文件系统
    - `du`统计的是数据块的个数用来计算大小
    - `echo >`写入文件只会改变数据块
  - **软连接**：亦称符号链接，ln -s afile cfile，ls -li afile cfile查看可以看到，其实cfile就记录了目标文件afile的路径，链接文件的权限修改对其自身是无意义的，对其权限修改将在目标文件上得到反馈。可以跨分区（跨文件系统）
  - facl：文件访问控制，getfacl afile查看文件权限，setfacl -m u:user1:r afile：u表示为用户分配权限，g表示用户组，r表示读权限，m改成x即可收回对应用户（组）权限

- 磁盘配额的使用：给多个用户之间磁盘使用做限制

### 分区

- 磁盘的分区与挂载：如果是虚拟机，可以直接在vbox上给其添加一块硬盘进行练习（比如叫sdc），可能需要关机才能添加
  - 常用命令
    - fdisk：fdisk -l查看，fdisk /dev/sdc2指定磁盘进行分区，会进入交互界面，输入m获取帮助，可以看到n是add a new partition即新建分区
    - mkfs：使用分区，输入mkfs.可以看到有很多不同后缀，都是指不同的文件系统，比如mkfd.ext4 /dev/sdc2即可格式化为ext4的文件系统。但是文件操作是文件系统之上的操作，无法直接操作，需要将其挂载到某个目录，对目录进行操作，
    - mount：mount -t auto自动检测文件系统，或者直接mount /dev/sdc2 /mnt/sdc2也会自动检测，将/dev/sd2挂载到/mnt/sd2，但是是临时挂载，vim /etc/fstab进行修改，dev/sdc1 /mnt/sdc1 ext4 defaults 0 0，即磁盘目录 挂载目录 文件系统指定 权限(defauls表示可读写) 磁盘配额相关参数1  磁盘配额相关参数2
    - parted：如果磁盘大于2T，不要用fdisk进行分区，而是parted，parted /dev/sdd
    
  - 用户磁盘配额：限制用户对磁盘的使用，比如创建文件数量（即限制i节点数）、数据块数量
    
    - xfs文件系统的用户磁盘配额quota，修改步骤如下
    
      - mkfs.xfs /dev/sdb1：创建分区，如果分区已经存在，为了防止这是一个误操作，会提示使用-f参数强制覆盖，mkfs.xfs -f /dev/sdb1
      - mkdir /mnt/disk1
      - mount -o uquota,gquota /dev/sdb1 /mnt/disk1，-o开启磁盘配额，uquota表示支持用户磁盘配额，gquota表示支持用户组磁盘配额
      - chmod 1777 /mnt/disk1：赋予1777权限
      - xfs_quota -x -c 'report -ugibh'/mnt/disk1：有参数时直接非交互配置，xfs_quota可以直接进入交互模式，但是一般非交互即可，
        - -c表示命令，report -ugibh，report表示报告（查看）磁盘配额，-u表示用户磁盘配额，g表示组磁盘配额，i表示节点，b表示块，h可以更人性化显示
    
      - xfs_quota -x -c 'limit -u isoft=5 ihard=10 user1' /mnt/disk1：root是无限制的，不要对root进行磁盘配额，没意义。这里对user1进行磁盘配额，limit表示限制磁盘配额，限制用户磁盘配额加-u，限制组磁盘配额加-g；isoft软限制i节点，ihard将硬限制，软限制比硬限制的配置的值更小，达到软限制之后，会提示用户在某一个宽限的时间条件内可以用超过软限制的值，硬限制则绝对不能超过限制的值；数据块限制即bsoft、bhard
    
  - 常见配置文件
    
    - letc/fstab
- 交换分区（虚拟内存）的查看与创建

  - free查看mem和swap，前面提到过
  - 增加交换分区的大小，使用硬盘分区扩充swap
    - mkswap：如mkswap /dev/sdd1将标记上swap
    - swapon：swapon /dev/sdd1打开swap，通过free可以看到swap被扩充了，swapoff /dev/sdd1关闭swap
  - 使用文件制作交换分区：可以直接创建一个比如10G的文件，或者创建带有空洞的文件，使其在swap的使用过程中逐渐扩大也可以
    - dd if=/dev/zero bs=4M count=1024 of=/swapfile：创建文件
    - mkswap /swapfile：即可为文件打上swap标记使其成为swap的空间
    - chmod 600 /swapfile：为了安全起见，一般修改为600权限
    - swapon、swapoff
  - 同样swap设置也是临时的，vi /etc/fstab。/swapfile swap swap defaults 0 0，即 swap文件或分区 挂载到swap目录（这是一个虚拟目录，因为swap不需要用文件目录来进行操作，挂载到这个虚拟目录即可） 文件系统格式（也是swap），第一个0表示做dump备份时要不要备份该硬盘（分区），但是现在一般都是tar进行备份，所以设置为0即可，第二个0表示开机的时候进行磁盘的自检的顺序问题，是针对之前的ext2、ext3的文件系统的设置，但是现在已经不需要了，如果发现写入是不完整的自动会对那个分区进行检查，所以也是0即可
    - 如果写错了东西，发现重启启动不起来了，通过grap进入到单一用户模式，来去修改/etc/fstab

### raid

- 软件RAID的使用：RAID（磁盘阵列）
  - RAID的常见级别及含义
    - RAID 0 striping条带方式，提高单盘吞吐率
    - RAID 1 mirroring镜像方式，提高可靠性，需要两块磁盘组成，其中有给做镜像备份
    - RAID5有奇偶校验，至少需要3块硬盘，2块硬盘写数据，还有1块做奇偶校验（存储2块数据盘的校验数据，该盘损坏，校验数据还是可以通过2块数据盘重新生成），当某一块数据硬盘损坏了，奇偶校验的硬盘就通过奇偶校验来通过未损坏的数据盘来还原数据，但是如果2块数据盘都损坏了，还是没办法了
    - RAID 10是RAID 1与RAIDO的结合，共4块，两块硬盘做raid1，两块做raid1和raid0
  - raid控制器（raid卡）：硬件设备，通过数据读写自动计算校验值，自动计算把数据放在哪块硬盘上的，甚至可以带有缓存功能加速硬盘访问
  - 软件RAID的使用：对cpu性能消耗较大，一般不实际使用，而是使用raid卡
    - 需要安装软件包yum install mdadm，练习raid建议划分3个同样大小的空白分区，做软件raid如果分区有大有小默认采用最小的空间，fdisk -l /dev/sd??
    - mdadm -C /dev/md0 -a yes -l1 -n2 /dev/sdb1 /dev/sdc1：-C /dev/md0创建raid，-a yes表示全部提示都选择yes，比如有数据、格式化等提示，所以要注意分区会被格式化，-l1指定raid级别为raid1，-n2表示2块硬盘是活动的，/dev/sdb1 /dev/sdc1即指定这两块硬盘，实际上可以简写为通配符形式/dev/sd[b,c]1。
    - 可能会提示may not suitable as a boot device什么的，因为是软件不支持

### 物理卷



### 逻辑卷

- 逻辑卷管理：在物理卷之上的虚拟卷，linux根目录就是逻辑卷的，可以将一块硬盘拆分为多个逻辑卷，也可以将多个硬盘合并为一个逻辑卷，根据场景对逻辑卷进行缩放容

###### pvcreate

- 新建逻辑卷
  - 先添加几个磁盘/dev/sdb1、/dev/sdc1、/dev/sdd1：`pvcreate /dev/sdb1 /dev/sdc1 /dev/sdd1`或者简写`pvcreate /dev/sd[b,c,d]1`，注意前面做了软件raid的磁盘如果没有停掉raid将失败
    - 停止raid：`mdadm --stop /dev/md0`，破坏超级块：`dd if=/dev/zero of=/dev/sdb1 bs=1M count=1`、`dd if=/dev/zero of=/dev/sdc1 bs=1M count=1`
    - 重新创建一下：`pvcreate /dev/sd[b,c,d]1`，`pvs`查看。信息：lvm即逻辑卷管理器、PSIZE即物理大小、PREE即物理卷的物理空间剩余量
    - `vgcreate vg1 /deb/sdb1 dev/sdc1`给物理卷分组，`pvs`查看可以发现已经被分到vg1这个卷组了
    - `vgs`查看卷组
    - `lvcreate -L 100M -n lv1 vg1`：从卷组vg1创建名字为lv1大小为100M的逻辑卷，`lvs`查看逻辑卷

```sh

```



- 使用逻辑卷：
  - 格式化：mkdir /mnt/test、mfs.xfs /dev/vg1/lv1
  - 进行挂载：fdisk /dev/sd?? pv vg1 lv1 xfs mount。命令解释：fdisk命令用来分区，用/dev/sd??磁盘建立一个pv，通过pv创建一个vg1，通过vg1创建lv1，lv1上通过xfs命令创建文件系统，mount将文件系统进行挂载。这个复杂过程相当实现了：pv vg1 lv1实现可动态扩展的功能，xfs使可以以文件形式操作的功能，mount实现内存和管理的映射。所以如果不需要扩展，可以直接fdisk /dev/sd?? xfs mount在sd磁盘设备上使用文件系统即可，如果还需要实现更复杂的功能还可以在其上进行raid功能（但就不能直接在磁盘上搭建逻辑卷了，需要在raid的基础上比如搭建一个/dev/md0，然后再在其上进行逻辑卷实现）
- 用途：可以用来扩展现有的如root、user、src 等分区
- 扩充逻辑卷
  - vgextend  centos /dev/sdd1，将/dev/sdd1这个pv划分到centos这个vg下
  - lvextend -L +50G /dev/centos/root
  - df -h查看发现文件系统的容量并没有变大，也需要告知文件系统卷已经扩大了，xfs_growfs /dev/centos/root
- 缩容
- 可以发现，卷管理实际上就是一层一层的，从物理磁盘到逻辑卷到文件系统

### 系统综合状态

##### sar

- 使用sar命令查看系统综合状态，可以对系统进行全面的体检
- 参数
  - sar -u 1 10：-u是cpu信息。1是采样时间间隔，每1秒采样10次
  - sar -r 1 10：-r显示内存情况
  - sar -b 1 10：-b显示IO信息
  - sar -d 1 10：-d磁盘信息
  - -q 1 10：进程信息

- 使用第三方命令iftop查看网络流量情况
  - yum install epel-release
  - yum install iftop
  - iftop -P：默认只监听eth0的网络接口
- 更多好用的工具自行到网络上找寻

## shell

> shell：在计算机科学中，Shell俗称壳（用来区别于核），是指"为使用者提供操作界面"的软件-命令解析（解释）器，即用于解释用户对操作系统的操作，bash即shell实现之一，其它还有cat/etc/shells等

shell会把用户所执行的命令翻译给内核，内核将命令执行的结果反馈给用户。ls为例，当输入ls时，首先由shell接收到用户执行的命令，对命令选项和参数进行分析，ls是查看文件的，将交给文件系统（属于内核层面了），然后内核把ls要查看的文件和目录翻译成硬盘对应的扇区（ssd硬盘是另外结构），硬件会把查询的结果交给内核，内核在返回给shell，最后返回给用户





### linux启动过程

- BIOS：基本输入输出系统。是主板上的功能，通过bios选择要引导的介质 - 硬盘、或光盘，如果选择了硬盘，就会有一个引导的部分-MBR
- MBR：硬盘的主引导记录部分，就进入到linux的过程了（以下）
- BootLoader(grub)：BootLoader指启动和引导内核的工具，现在用的grub2.0，可以引导linux内核（选择哪个内核启动），甚至是windows系统
- kernel：内核启动
- systemd（centos7，6中是init）：systemd启动（1号进程）。centos6时，init启动后的所有系统初始化内容都是由shell脚本完成（/etc/rc.d目录下会有大量脚本，比如用于激活软件raid、lvm等系统初始化工作），centos7中则有些内容改变为systemd的配置文件方式（到/etc/systemd/system目录下，根据启动级别来到/user/lib/systemd/system目录下，在这个目录下读取各种各样的配置文件），由应用程序引导。
- 系统初始化（）：比如通过驱动程序加载各种硬件，是由shell脚本完成的
- shell：shell开始工作



- bios：系统启动的时候按F2进入界面来选择不同的引导介质，如果选择了硬盘，就会有一个引导的部分-mbr，通过`dd`命令可以查看MBR（linux中一切皆文件，磁盘也可以当作文件被读取）
  - `dd if=/dev/sda of=mbr.bin bs=446 count=1`，if是磁盘，of指定输出文件，bs指定块大小，count指定1个块。因为mbr.bin是没有文件系统的，所以不能通过cat直接查看其中内容，这里使用`hexdump -C mbr.bin`用16进制的方式去查，-C将能够显示为字符的内容显示为字符
  - `dd if=/dev/sda of=mbr2.bin bs=512 count=1`，扩大到512字节，mbr将包括自盘分区表，最后的55 aa即证明引导扇区是正确的
- BootLoader：cd /boot/grub2
  - `grub2-editenv list`：显示默认引导的内核
  - `uname -r`：查看当前所使用的内核

### base

- 命令中也大量使用了shell脚本，以/sbin/grub2-mkconfig为例，`file /sbin/grub2-mkconfig`可以看到文件的描述shell script，vim打开可以发现也就是一个shell脚本

- shell脚本区别于py、php等脚本，无需掌握那些语言函数，完全由命令构成
- UNIX的哲学：一条命令只做一件事
- 为了组合命令和多次执行，使用脚本文件来保存需要执行的命令
- 如果是二进制可执行文件，赋予可执行（-x）权限即可，但是如果是shell脚本文件想要执行，还需要额外赋予该文件可读（-r）权限(`chmod u+rx filename`)

```sh
#比如某个目录经常反复cd进入并ls查看
cd /var ; ls ; pwd ; du -sh ; du -sh *
```

可以保存为sh文件，比如叫tmp.sh

```sh
#一般shell脚本头部上会加这样一个申明，这个申明叫做"Sha-Bang"
#!/bin/bash
cd /var ; ls ; pwd ; du -sh ; du -sh *
```

```sh
#赋予权限
chmod u+rx tmp.sh
#可以通过bash执行，在bash中"#!/bin/bash"将被当作注释
bash ./tmp.sh
#直接运行，当前系统是什么shell就将用什么shell解释，"#!/bin/bash"将是非注释的，它将告诉系统使用/bin/bash来解释执行
./tmp.sh
```

- 执行命令的方式
  - `bash ./filename.sh`：会在当前终端下fork一个叫做bash的子进程，通过子进程去运行脚本，所以执行脚本后当前终端线程并没有真正进入到/var目录，因为是子进程进到目录的。注意：通过bash执行脚本是不用赋予执行权限的，因为是通过bash的
  - `./filename.sh`：也是产生子进程然后运行，不同的是通过Sha-Bang去解释运行脚本的。直接运行脚本文件，必须有可执行权限，
  - `source ./filename.sh`：在当前进程运行，所以当前终端进入到了/var目录
  - `. ./filename.sh`：开头的`.`即source的缩写
- 内建命令和外部命令的区别
  - 内建命令：内建命令不需要创建子进程，将对当前Shell 生效，如cd、pwd、source等
  - 外部命令：需要创建子进程，不会对当前shell生效，如bash

### 管道



## 待整理

- 
- 快捷键：
  - TAB快捷键补全文件（目录）名
  - CTRL+L清屏
  - CTRL+C终止（命令）程序

- 添加执行权限：chmod a+x 文件名，a表示所有，x表示执行