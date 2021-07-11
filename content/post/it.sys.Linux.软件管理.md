

+++

title = "Linux.软件管理"
description = "Linux.软件管理"
tags = ["it","sys"]

+++



# Linux.软件管理

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
yum install epel-release -y
#epel的aliyun镜像配置
wget http://mirrors.aliyun.com/repo/epel-7.repo -O /etc/yum.repos.d/epel.repo
#清空并刷新缓存
yum clean all && yum makecache

# 查看当前源上可下载的指定软件所有版本
yum list docker-ce --showduplicates|sort -r
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