

+++

title = "Linux.shell"
description = "Linux.shell"
tags = ["it","sys"]

+++

# Linux.shell

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

