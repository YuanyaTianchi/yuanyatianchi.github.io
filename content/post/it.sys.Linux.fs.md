

+++

title = "Linux.fs"
description = "Linux.fs"
tags = ["it","sys"]

+++



# Linux.fs

Linux支持多种文件系统，常见的有

- ext4（centos6默认）
- xfs（centos7默认）
- NTFS （需安装额外软件ntfs-3g，有版权的，windows用）

### ext4

结构

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
- **软连接**：亦称符号链接，类似快捷方式。`ln -s afile cfile`链接两个文件，`ls -li afile cfile`查看两个文件链接信息，其实cfile就记录了目标文件afile的路径，链接文件的权限修改对其自身是无意义的，对其权限修改将在目标文件上得到反馈。可以跨分区（跨文件系统）
- facl：文件访问控制，getfacl afile查看文件权限，setfacl -m u:user1:r afile：u表示为用户分配权限，g表示用户组，r表示读权限，m改成x即可收回对应用户（组）权限

- 磁盘配额的使用：给多个用户之间磁盘使用做限制



### mkfs

- `mkfs`：make filesystem，创建文件系统。实际上是一个综合的指令，它会去调用正确的文件系统格式化工具软件。常说的“格式化”其实就是“make filesystem”，创建的其实是 xfs 文件系统， 因此使用的是 mkfs.xfs 这个指令

- `mkfs.xfs [-b bsize] [-d parms] [-i parms] [-l parms] [-L label] [-f] \ [-r parms]`：加单位则为是Bytes值，可以用 k,m,g,t,p （小写）等来解释，s指的是 sector 个数
  - `-b`：block 容量，可由 512 到 64k，不过最大容量限制为 Linux 的 4k 
  - `-d`：data section 的相关参数值
    - `agcount=数值`：设置需要几个储存群组的意思（AG），通常与 CPU 有关
    - `agsize=数值`：每个 AG 设置为多少容量的意思，通常 agcount/agsize 只选一个设置即可
    - `file`：指的是“格式化的设备是个文件而不是个设备”的意思！（例如虚拟磁盘）
    - `size=数值`：data section 的容量，亦即你可以不将全部的设备容量用完的意思
    - `su=数值`：当有 RAID 时，那个 stripe 数值的意思，与下面的 sw 搭配使用
    - `sw=数值`：当有 RAID 时，用于储存数据的磁盘数量（须扣除备份碟与备用碟）
    - `sunit=数值`：与 su 相当，不过单位使用的是“几个 sector（512Bytes大小）”的意思
    - `swidth=数值`：就是 su*sw 的数值，但是以“几个 sector（512Bytes大小）”来设置
  - `-f`：如果设备内已经有文件系统，则需要使用这个 -f 来强制格式化才行
  - `-i`：与 inode 有较相关的设置，主要的设置值有
    - `size=数值`：最小256Bytes，最大2k，一般保留 256 就足够使用了
    - `internal=[0&#124;1]`：log 设备是否为内置？默认为 1 内置，如果要用外部设备，使用下面设置
    - `logdev=device`：log 设备为后面接的那个设备上头的意思，需设置 internal=0 才可
    - `size=数值`：指定这块登录区的容量，通常最小得要有 512 个 block，大约 2M 以上才行
  - `-L`：后面接这个文件系统的标头名称 Label name 的意思
- `-r`：指定 realtime section 的相关设置值，常见的有： 
  - `extsize=数值`：就是那个重要的 extent 数值，一般不须设置，但有 RAID 时，最好设置与 swidth的数值相同较佳！最小为 4K 最大为 1G 。

```sh
# 不带参数将使用默认值
mkfs.xfs /dev/sdb3
blkid /dev/sdb*
```

因为 xfs 可以使用多个数据流来读写系统，以增加速度，因此那个 agcount 可以跟 CPU 的核心数来做搭配！举例来说，如果我的服务器仅有一颗 4 核心，但是有启动 Intel 超线程功能，则系统会仿真 出 8 颗 CPU 时，那个 agcount 就可以设置为 8 喔

```sh
# 找出系统的 CPU 数，并据以设置 agcount 数值
grep 'processor' /proc/cpuinfo
mkfs.xfs -f -d agcount=2 /dev/sdb4
```



### 分区

如果是虚拟机，可以直接在virtualbox上给其添加一块硬盘进行练习（比如叫sdc），可能需要关机才能添加

### 挂载

如果是虚拟机，可以直接在virtualbox上给其添加一块硬盘进行练习（比如叫sdc），可能需要关机才能添加

- 磁盘的分区与挂载：
  - 常用命令
    - mkfs：使用分区，输入mkfs.可以看到有很多不同后缀，都是指不同的文件系统，比如mkfd.ext4 /dev/sdc2即可格式化为ext4的文件系统。但是文件操作是文件系统之上的操作，无法直接操作，需要将其挂载到某个目录，对目录进行操作，
    - `mount`：mount -t auto自动检测文件系统，或者直接mount /dev/sdc2 /mnt/sdc2也会自动检测，将/dev/sd2挂载到/mnt/sd2，但是是临时挂载，vim /etc/fstab进行修改，dev/sdc1 /mnt/sdc1 ext4 defaults 0 0，即磁盘目录 挂载目录 文件系统指定 权限(defauls表示可读写) 磁盘配额相关参数1  磁盘配额相关参数2
      - `-t`：指定档案系统的型态，通常不必指定，`mount` 会自动选择正确的型态，或者 `mount -t auto` 也会自动检测文件系统类型。
    - `mount -t proc proc /proc`：把proc这个虚拟文件系统挂载到/proc目录，mount的标准用法是 `mount -t type device dir `，但是内核代码中，proc filesystem根本没有处理 dev_name这个参数，所以传什么都没有影响，只是影响mount命令的输出内容，好的实践应该将设备名定义为 nodev、none 等，或者就叫 proc 亦可。
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