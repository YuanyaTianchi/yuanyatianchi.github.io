

+++

title = "Linux.process"
description = "Linux.process"
tags = ["it","sys"]

+++



# Linux.process

进程：进行中的程序。运行中的程序，从程序开始运行到终止的整个生命周期是可管理的。C程序的启动是从main函数`int main(int agrc, char*argv[])`开始的，终止的方式分为正常终止、异常终止，正常终止有从main返回、调用exit等方式，异常终止有调用abort、接收信号等方式。计算机资源不足时等情况进行，进程和权限有着密不可分的关系。


## 进程查看

### ps

- `ps`：process status，进程状态
- 参数
  - `-e`：可以查看更多进程，类似于win中的系统进程
  - `-f`：额外的信息UID、PPID
  - `-L`：Light，表示轻量级，轻量级进程，实际上即线程

```sh
ps -efL
```

- 内容
  - PID：唯一标识进程（不同用户使用同样的程序也是不同的进程）
  - TTY：终端的次要装置号码（minor device number of tty）。即表示当前执行程序的终端
  - UID：效用户id。表示进程是由哪个用户启动的信息（可修改），默认显示为启动进程的用户
  - PPID：进程的父进程id。（linux的 0号进程 和 1 号进程：https://www.cnblogs.com/alantu2018/p/8526970.html ，注意centos7的systemd即以前的init）

- `pstree`：查看进程的树形结构，父子进程关系清晰，但是是静态查看的，ps是动态刷新的。非默认安装的程序，需要自行下载

### top

- `top`：系统状态，包括进程状态，是动态更新的
- 参数
  - `-p 1`：指定pid查看进程

```sh
top
```

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

### lsof

```shell
# 查看8080端口占用
lsof -i:8080
# 显示开启文件abc.txt的进程
lsof abc.txt
# 显示abc进程现在打开的文件
lsof -c abc
# 列出进程号为1234的进程所打开的文件
lsof -c -p 1234
# 显示归属gid的进程情况
lsof -g gid
# 显示目录下被进程开启的文件
lsof +d /usr/local/
# 同上，但是会搜索目录下的目录，时间较长
lsof +D /usr/local/
# 显示使用fd为4的进程
lsof -d 4
# 显示所有打开的端口和UNIX domain文件
lsof -i -U
```



## 控制命令

#### 优先级

##### nice

- `nice`：优先级调整。优先级值从-20到19，值越小优先级越高，抢占资源就越多。
  - `-n 10 ./demo.sh`：设置demo.sh的优先级为10。写一个无限循环的脚本demo.sh并运行
- `renice`：重新设置优先级
  -  `-n 15`：重新设置优先级值为15

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



## 信号-进程的通信方式

> 信号是进程间通信方式之一，典型用法是：终端用户输入中断命令，通过信号机制停止一个程序的运行。

##### kill

- `kill`
- 参数
  - `-l`：查看所有信号
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

