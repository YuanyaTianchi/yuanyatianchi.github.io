

+++

title = "Linux.disk"
description = "Linux.disk"
tags = ["it","sys"]

+++

# Linux.disk

## 磁盘

##### lsblk

- `lsblk [-dfimpt] [device]`：list block device，列出存储设备，即磁盘
  - `-d`：仅列出磁盘本身，并不会列出该磁盘的分区数据 
  - `-f`：同时列出该磁盘内的文件系统名称
  - `-i`：使用 ASCII 的线段输出，不要使用复杂的编码 （再某些环境下很有用） 
  - `-m`：同时输出该设备在 /dev 下面的权限数据 （rwx 的数据） 
  - `-p`：列出该设备的完整文件名！而不是仅列出最后的名字而已。 
  - `-t`：列出该磁盘设备的详细数据，包括磁盘伫列机制、预读写的数据量大小等

```sh
lsblk
lsblk /dev/sdb
```

- 内容
  - NAME：设备文件名，会省略 /dev 等前导目录
  - MAJ:MIN：主要：次要设备代码。核心认识的设备都是通过这两个代码来熟悉的
  - RM：是否为可卸载设备 （removable device），如光盘、USB 磁盘等等 
  - SIZE：当然就是容量啰！ 
  - RO：是否为只读设备的意思 
  - TYPE：是磁盘 （disk）、分区 （partition） 还是只读存储器 （rom） 等输出 
  - MOUTPOINT：就是前一章谈到的挂载点！ 

##### blkid

- `blkid`：可以列出设备的uuid（全域单一识别码），与`lsblk -f`类似。Linux 会将系统内所有的设备都给予一个独一无二的识别码， 这个识别码就可以拿来作为挂载或者是使用这个设备/文件系统之用了

```sh
blkid
blkid /dev/sda*
```



##### parted

- `parted`

```sh
# 与`fdisk -l`展示的信息基本一致
parted -l

# 列出 /dev/vda 磁盘的相关数据
parted /dev/sda print
```

- 内容
  - Model：磁盘的模块名称（厂商）
  - Disk：磁盘的总容量
  - Sector size（logical/physical）：每个逻辑/物理扇区容量
  - Partition Table：分区表的格式 （MBR/GPT） 

##### df

- `df`：可以理解为`fdisk`的补充，可以看到分区挂载的目录等

```sh
df -h
```



### 分区

##### gdisk/fdisk

> gdisk/fdisk操作基本一致

- `gdisk`：

  - `-l`：查看磁盘列表

- 内容：

  - Start：起点扇区。磁盘在linux中也是作为文件，如`/dev/sd?`，以扇区划分
  - End：止点扇区
  - Size：是分区的容量
  - Code：Linux 为 8300，swap 为 8200。不过这个项目只是一个提示而已，不见得真的代表此分区内的文件系统，Linux 大概都是 8200/8300/8e00 等三种格式， Windows 几乎都用 0700 这样，gdisk下按`L`查看。文件系统类型一般为linux（或其它）类型，ntfs就不能被linux文件系统读取，除非格式化为linux文件系统，或者编译内核时设置开启ntfs

- 注意

  - 切忌操作使用中的分区

  - GPT分区表使用gdisk分区，MBR分区表使用fdisk分区，否则将分区失败，甚至干掉分区记录

  - fdisk 有时会使用柱面 （cylinder） 作为分区的最小单位，与 gdisk 默认使用 sector 不太一 

    样，另外， MBR 分区是有限制的 （Primary, Extended, Logical…）

```sh
# 操作指定磁盘/dev/sda，将进入交互模式
gdisk /dev/sda
```

- 交互：`m`查看操作，`p`查看当前分区信息，更多操作根据`m`查看的信息进行即可

```sh
Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-4140724, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-4140724, default = 4140724) or {+-}size{KMGTP}: +1g
Current type is 'Linux filesystem'
Hex code or GUID (L to show codes, Enter = 8300): 
Changed type of partition to 'Linux filesystem'
```

```sh
# 查看分区表
cat /proc/partitions
# 如果操作分区更新的是linux正在使用的磁盘，分区表不会更新，可以通过重启linux或者partprobe更新
partprobe -s
```

##### partprobe

- `partprobe`：更新 Linux 的分区表信息（重启linux亦可），
  - `-s`：将打印信息

```sh
partprobe
partprobe -s
```





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