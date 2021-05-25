

+++

title = "YuanyaDocker"
description = "YuanyaDocker"
tags = ["it", "go","project"]

+++



# MyDocker

Docker 是一个使用了 Linux Namespace Cgroups 的虚拟化工具。

## Linux Namespace

Linux Namespace 是 Kernel 的一个功能，它可以隔离一系列的系统资源，比如 PIO ( Process ID ）、User ID、Network 等。可能会想到 chroot 命令，就像 chroot 允许把当前目录变成根目录一样（被隔离开来的）。Namespace 也可以在一些资源上，将进程隔离起来，这些资源包括进程树、网络接口、挂载点等。

使用 Namespace ，就可以做到 UID 级别的隔离，可以以 UID 为 n 的用户虚拟化出来一个 Namespace，在这个 Namespace 里面，用户 n 是具有 root 权限的 。但是，在真实的物理机器上，他还是那个以 UID 为 n 的用户，这样就解决了用户之间隔离的问题。

除了 User Namespace，PID 也是可以被虚拟的。命名空间建立系统的不同视图，从用户的角度来看，每一个命名空间应该像一台单独的 Linux 一样，有自己的 init 进程（PID 为 1），其他进程的 PID 依次递增。子命名空间 A 和 B 空间都有 PID 为 1 的 init 进程，子命名空间的进程映射到父空间的进程上（如：A 和 B 的 init 进程分别映射到父空间 PID为 3 和 4 的进程），父命名空间可以知道每一个子命名空间的运行状态，而子命名空间与子命名空间是隔离的。

![](./img/父子命名空间关系.png)

| Namespace 类型    | 系统调用参数  | 内核版本 |
| ----------------- | ------------- | -------- |
| Mount Namespace   | CLONE NEWNS   | 2.4.19   |
| UTS Namespace     | CLONE NEWUTS  | 2.6.19   |
| IPC Namespace     | CLONE NEWIPC  | 2.6.19   |
| PID Namespace     | CLONE NEWPID  | 2.6.24   |
| Network Namespace | CLONE NEWNET  | 2.6.29   |
| User Namespace    | CLONE NEWUSER | 3.8      |

Namespace API 主要使用如下 3 个系统调用

- clone()：创建新进程。根据系统调用参数来判断哪些类型的 Namespace 被创建，而且它们的子进程也会被包含到这些 Namespace 中。
- unshare()：将进程移出某个 Namespace 
- setns()：将进程加入到 Namespace 中。



### UTS Namespace

UTS Namespace 主要用来隔离 nodename 和 domainname 两个系统标识，在UTS Namespace中，每个 Namespace 允许有自己的 hostname



用 Go 创建 UTS Namespace 的例子：使用 `syscall.CLONE_NEWUTS` 这个标识符去创建 UTS Namespace，Go 帮我们封装了对 clone() 函数的调用。

```go
func main() {
    // 指定被 fork 来的新进程内的初始命令，默认使用sh来执行
	cmd := exec.Command("sh")
    
    // 系统调用参数
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

```sh
$ go run main.go
```



容器机：执行 main.go 后进入到 一个sh 运行环境中，是一个交互式环境

```sh
# 查看系统中执行 main.go 的进程关系
$ pstree -pl | grep main
|-gnome-terminal-(20132)-+-bash(20139)---go(23753)-+-main(23802)-+-sh(23807)-+-grep(23809)
# 查看当前 sh 环境的 PID
$ echo $$
23807
```

查看父进程 main 和子进程 sh 的 UTS Namespace，可以看到它们在两个不同的 UTS Namespace 中

```sh
$ readlink /proc/23802/ns/uts
uts:[4026531838]
$ readlink /proc/23807/ns/uts
uts:[4026532191]
```

由于 UTS Namespace 对 hostname 做了隔离 所以在这个环境内修改 hostname 也不影外部主机

```sh
$ hostname - b myhostname
$ hostname
myhostname
```



宿主机：启动另一个 shell 可以看到外部的 hostname 并没有被内部修改所影响

```sh
$ hostname
MiWiFi-R3A-srv
```



### IPC Namespace

IPC Names 用来隔离 System V IPC（） 和 POSIX message queues。每个 IPC Namespace 都有自己的 System V IPC 和 POSIX message queue。



增加 `syscall.CLONE_NEWIPC` 标识表示希望创建 IPC Namespace，

```go
func main() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
        // 增加 CLONE_NEWIPC 标识
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

```sh
$ go run main.go
```



宿主机

```sh
# 查看活动的 mq，目前没有活动的 mq
$ ipcs -q
# 创建一个 mq
$ ipcmk -Q
# 可以查看到刚刚创建的 mq
$ ipcs -q
```



容器机：看不到刚刚创建的 mq，说明 IPC Namespace 创建成功，IPC 已隔离

```sh
$ ipcs -q
```



### PID Namespace

PID Namespace 是用来隔离进程 ID 的。同一个进程在不同的 PID Namespace 里可以拥有不同的 PID 。这样就可以理解在 docker container 中 `ps -ef` 经常会发现， 在容器内，前台运行的那个进程 PID 是 1 ，但是在容器外 ，使用 `ps -ef` 会发同样的进程却有不同的 PID 这就是 PID Namespace 做的事情。



增加 `syscall.CLONE_ NEWPID` 标识表示希望创建 PID Namespace，

```go
func main() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
        // 增加 CLONE_NEWIPC 标识
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

```sh
$ go run main.go
```



宿主机：查看 main.go 执行的 pid，外部 pid 是 23121

```sh
$ pstree -pl | grep main
|-gnome-terminal-(20132)-+-bash(20139)---go(23059)-+-main(23116)-+-sh(23121)
```



容器机：内部 pid 为 1，而不像之前 UTS Namespace 时，内外部 pid 一致

```sh
$ pstree -pl | grep main
|-gnome-terminal-(20132)-+-bash(20139)---go(23059)-+-main(23116)-+-sh(23121)
$ echo $$
1
```



这里还不能使用 `ps` 来查看 因为 `ps`、 `top` 等命令会使用／proc 内容，具体内容在下面的 Mount Namespace 部分会进行讲解。



### Mount Namespace

Mount Namespace 用来隔离各个进程看到的挂载点视图。在不同 Namespace 的进程中，看到的文件系统层次是不一样的。在 Mount Namespace 中调用 mount() 和 umount() 仅仅只会影响当前 Namespace 内的文件系统，而对全局的文件系统是没有影响的。

到这里也许就会想到 chroot()。它也是将某一个子目录变成根节点。但是 Mount Namespace 不仅能实现这个功能，而且能以更灵活、安全的方式实现。

Mount Namespace 是 Linux 第一个实现 Namespace 类型，因此它的系统调用参数是 CLONE_NEWNS（New Namespace 的缩写）。



增加 `syscall.CLONE_NEWNS` 标识表示希望创建 Mount Namespace，

```go
func main() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
        // 增加 CLONE_NEWIPC 标识
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
}
```

```sh
$ go run main.go
```

容器机：查看一下 /proc 的文件内容，这是一个系统文件，提供额外的机制，可以通过内核和内核模块将信息发送给进程。这里会将 /proc 挂载到当前 Namespace 下进行查看。

```sh
# 这里的 /proc 还是宿主机的 /proc，文件很多
$ ls /proc
# 查看进程，也仍然是宿主机的进程，进程很多
$ ps -ef

# 将 /proc 挂载到当前 Namespace 下
$ mount -t proc proc /proc

# 再次查看 /proc，已经是当前 Namespace 的 /proc 了，文件很少
$ ls /proc
1	       cmdline    driver	   ioports    kpagecount  modules	    schedstat  sys		      version
8	       consoles   execdomains  irq	      kpageflags  mounts	    scsi	   sysrq-trigger  vmallocinfo
acpi	   cpuinfo    fb	       kallsyms   loadavg	  mtrr		    self	   sysvipc	      vmstat
asound	   crypto     filesystems  kcore      locks	      net		    slabinfo   timer_list	  zoneinfo
buddyinfo  devices    fs	       keys       mdstat	  pagetypeinfo	softirqs   timer_stats
bus	       diskstats  interrupts   key-users  meminfo	  partitions	stat	   tty
cgroups    dma	      iomem	       kmsg       misc	      sched_debug	swaps	   uptime
# 再次查看进程，同样是当前 Namespace 的进程了，进程很少
$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 21:30 pts/1    00:00:00 sh
root         9     1  0 21:40 pts/1    00:00:00 ps -ef
```

可以看到在当前 Namespace 中， sh 进程是 PID 为 1 的进程。这就说明当前的 Mount Namespace 中的 mount 和外部空间是隔离的，mount 操作并没有影响到外部。



宿主机：记得将 /proc 挂载回来，否则 `ps -ef` 将报"Error, do this: mount -t proc proc /proc"

```sh
$ ps -ef
Error, do this: mount -t proc proc /proc
# 将 /proc 挂载回来
$ mount -t proc proc /proc
```



### User Namespace

User Namespace 主要是隔离用户的用户组 ID，即一个进程的 User ID 和 Group ID 在 User Namespace 内外可以是不同的。

较常用的是，在宿主机上以一个非 root 用户运行创建一个 User Namespace，然后在 User Namespace 内被映射成 root 用户。这意味着这个进程在 User Namespace 内有 root 权限，但是在 User Namespace 外没有 root 权限。

从 Linux Kernel 3.8 开始。root 进程也可以创建 User Namespace，同样在 Namespace 内被映射成 root 用户，拥有 root 权限。



User namespace是目前的六个namespace中最后一个支持的，并且直到Linux内核3.8版本的时候还未完全实现（还有部分文件系统不支持）。因为user namespace实际上并不算完全成熟，很多发行版担心安全问题，在编译内核的时候并未开启USER_NS,比如我们的Centos(但是在ubuntu中是默认开启的)，最明显的，在/proc/[pid]/目录下就没有gid_map和pid_map两个文件。可以通过 grubby 修改内核参数，修改后需重启。开启后可以在 /proc/[pid]/ 中看到对应进程的 gid_map 和 pid_map 文件

```sh
# 开启
$ grubby --args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"
reboot

# 关闭
$ grubby --remove-args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"
reboot

# 查看内核参数，可以发现 max_user_namespaces = 0
$ sysctl -a | grep namespace
user.max_ipc_namespaces = 15024
user.max_mnt_namespaces = 15024
user.max_net_namespaces = 15024
user.max_pid_namespaces = 15024
user.max_user_namespaces = 0
user.max_uts_namespaces = 15024
# 所以还需要设置 max_user_namespaces 为大于 0 的值
$ sysctl -w user.max_user_namespaces=4
# 刷新
$ sysctl -p
```



增加 `syscall.CLONE_NEWUSER` 标识表示希望创建 User Namespace

```go
func main() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS |
			syscall.CLONE_NEWUSER,
	}
	//cmd.SysProcAttr.Credential = &syscall.Credential{Uid: uint32(1), Gid: uint32(1)} // 目前3.10内核上不支持这么设置
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
	os.Exit(-1)
}
```

```sh
$ go run man.go
```



宿主机：查看用户id、用户组id等

```sh
$ id
uid=0(root) gid=0(root) 组=0(root) 环境=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```



容器机：uid 是不同的，说明 User N amespace 生效

```sh
$ id
uid=65534(nfsnobody) gid=65534(nfsnobody) 组=65534(nfsnobody)
```



### Network Namespace

Network Namespace 是用来隔离网络设备、 IP 地址端口等网络栈的 Namespace。Network Namespace 可以让每个容器拥有自己独立的（虚拟的）网络设备，而且容器内的应用可以绑定到自己的端口，每个 Namespace 内的端口都不会互相冲突。在宿主机上搭建网桥后，就能很方便地实现容器之间的通信，而且不同容器上的应用可以使用相同的端口



增加 `syscall.CLONE_NEWNET` 标识表示希望创建 Network Namespace

```go
func main() {
	cmd := exec.Command("sh")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWIPC | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS |
			syscall.CLONE_NEWUSER | syscall.CLONE_NEWNET,
	}
	//cmd.SysProcAttr.Credential = &syscall.Credential{Uid: uint32(1), Gid: uint32(1)}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Run(); err != nil {
		log.Fatal(err)
	}
	os.Exit(-1)
}
```

```sh
$ go run main.go
```



宿主机：查看网络设备，可以看到有 enp0s3、lo、virbr0 等网络设备

```sh
$ ifconfig
enp0s3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.31.147  netmask 255.255.255.0  broadcast 192.168.31.255
        inet6 fe80::2bd5:35c:43f9:a946  prefixlen 64  scopeid 0x20<link>
        ether 08:00:27:56:e8:02  txqueuelen 1000  (Ethernet)
        RX packets 4847  bytes 329495 (321.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 306  bytes 31373 (30.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 34  bytes 2692 (2.6 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 34  bytes 2692 (2.6 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
        ether 52:54:00:f4:91:9d  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```



容器机：查看网络设备，没有任何网络设备信息，说明 Network Namespace 生效

```sh
$ ifconfig
```



## Linux Cgroups

### docker 是如何使用 Cgroups 的

Docker 是通过 Cgroups 实现容器资源限制和监控。Docker 通过为每个容器创建 cgroup 并通过 cgroup 去配置资源限制和资源监控

```sh
$ docker pull ubuntu
# 启动一个 ubuntu，通过 -m 设置内存限制
$ docker run -itd -m 128m ubuntu
50101823a6e4d35d9d9bdc3b981b8ff5d80bd666d2bf465d77c2915698ff3309
# 进入这个 ubuntu 容器实例的 cgroup 文件
$ cd /sys/fs/cgroup/memory/docker/50101823a6e4d35d9d9bdc3b981b8ff5d80bd666d2bf465d77c2915698ff3309
# 查看 cgroup 的内存限制
$ cat memory.limit_in_bytes
134217728
# 查看 cgroup 中进程所使用的内存大小
$ cat memory.usage_in_bytes 
557056
```



### 用 Go 语言实现通过 cgroup 限制容器的资源

在 Namespace 容器的 demo 基础上， 加上 cgroup 的限制， 实现一个 demo 使其能够具有限制容器内存的功能。

```go
const cgroupMemoryHierarchyMount = "/sys/fs/cgroup/memory"

func main() {
	if os.Args[0] == "/proc/self/exe" {
		// 容器进程
		fmt.Printf("current pid %d\n", syscall.Getpid())
		cmd := exec.Command("sh", "-c", `stress --vm-bytes 200m --vm-keep -m 1`)
		cmd.SysProcAttr = &syscall.SysProcAttr{}
		cmd.Stdin = os.Stdin
		cmd.Stdout = os.Stdout
		cmd.Stderr = os.Stderr
		if err := cmd.Run(); err != nil {
			fmt.Println(err)
			os.Exit(1)
		}
	}

	// 执行这个 cmd 就会触发上面的 if 块从而运行一个 stress 进程
	cmd := exec.Command("/proc/self/exe")
	cmd.SysProcAttr = &syscall.SysProcAttr{
		Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID | syscall.CLONE_NEWNS,
	}
	cmd.Stdin = os.Stdin
	cmd.Stdout = os.Stdout
	cmd.Stderr = os.Stderr

	if err := cmd.Start(); err != nil {
		fmt.Println("ERROR", err)
		os.Exit(1)
	} else {
		// 得到 fork 出来的进程映射在外部命名空间的 pid
        fmt.Printf("cmd.Process.Pid: %v\n", cmd.Process.Pid)
		// 在系统默认创建挂载了 memory subsystem 的 Hierarchy 上创建 cgroup
		os.Mkdir(path.Join(cgroupMemoryHierarchyMount, "testmemorylimit"), 0755)
		// 将容器进程加入到这个 cgroup
		err = ioutil.WriteFile(path.Join(cgroupMemoryHierarchyMount, "testmemorylimit", "tasks"), []byte(strconv.Itoa(cmd.Process.Pid)), 0644)
		// 限制 cgroup 进程使用
		err = ioutil.WriteFile(path.Join(cgroupMemoryHierarchyMount, "testmemorylimit", "memory.limit_in_bytes"), []byte("100m"), 0644)
	}
	cmd.Process.Wait()
}
```

```sh
# 运行起来确实被限制到 100m 内存
go run main.go 
[/tmp/go-build034195069/b001/exe/main]
31964[/proc/self/exe]
current pid 1
stress: info: [6] dispatching hogs: 0 cpu, 0 io, 1 vm, 0 hdd
```



## Union File System

Union File System ，简称 UnionFS，是一种为 Linux、FreeBSD、NetBSD 操作系统设计的，把其它文件系统联合到一个联合挂载点的文件系统服务。它使用 branch 把不同文件系统的文件和目录“透明地”覆盖，形成一个单一一致的文件系统，这些 branch 或是 read-only 的，或是 read-write 的，所以当对这个虚拟后的联合文件系统进行写操作时，系统是真正写到了一个新的文件中，看起来这个虚拟后的联合文件系统是可以对任何文件进行操作的，但是其实它并没有改变原来的文件，这是因为 unionfs 用到了一个重要的资源管理技术，叫做写时复制。

写时复制（copy-on-write，下文简称 CoW），也叫隐式共享，是一种对可修改资源实现高效复制的资源管理技术。它的思想是，如果一个资源是重复的，但没有任何修改，这时并不需要立即创建一个新的资源，这个资源可以被新旧实例共享。创建新资源发生在第一次写操作，也就是对资源进行修改的时候。通过这种资源共享的方式，可以显著地减少未修改资源复制带来的消耗，但是也会在进行资源修改时增加小部分的开销。

用一个经典例子来解释。Knoppix，一个用于 Linux 演示、光盘教学和商业产品演示的 Linux 发行版，就是将 D-ROM 或 DVD 和一个存在于可读写设备（比如 U 盘）上的叫作 knoppix.img 的文件系统联合起来的系统。这样任何对 CD/DVD 上文件的改动都会被应用在 U 盘上，而不改变原来的 CD/DVD 上的内容

### AUFS

AUFS ，英文全称是 Advanced Multi-Layered Unification Filesystem 曾经也叫 Acronym Multi-Layered Unification Filesystem、Another Multi-Layered Unification Filesystem。AUFS 完全重写了早期的 UnionFS 1.x，其主要目的是为了可靠性和性能，井且引入了一些新的功能，比如可写分支的负载均衡。一些实现已经被纳入 UnionFS 2.x 版本。

### Docker 是如何使用 AUFS

AUFS 时 Docker 选用的第一种存储驱动。AUFS 具有快速启动容器、高效利用存储和内存的优点。直到现在， AUFS 仍然是 Docker 支持的一种存储驱动类型。接下来介绍Docker 是如何利用 AUFS 存储 image 和 container的

#### image layer 和 AUFS

每一个 Docker image 都是由一系列 read-only layer 组成的。image layer 的内容都存储在 Docker hosts filesystem 的 /var/lib/docker/aufs/diff 目录下。而 /var/lib/docker/aufs/layers 目录，则存储着 image layer 如何堆栈这些 layer 的 metadata。

准备一台安装了 Docker 11 .2 的机器。在没有拉取任何镜像、启动任何容器的情况下，执行如下命令

```sh
$ docker pull ubuntu:l5.04
$ ls /var/lib/docker/aufs/diff
$ ls /var/lib/docker/aufs/mnt
$ cat /var/lib/docker/aufs/layers/
```

centos7默认不支持aufs，需要升级带有 aufs 的内核：https://www.jianshu.com/p/63fdb0c0659c ，https://github.com/bnied/kernel-ml-aufs

```sh
# 查看 kenel 支持的文件系统
$ cat /proc/filesystems | grep aufs

# 下载源。这里的内核版本比较高，是最新的 5.xxx 版本，不过暂时用来做 demo 足够了
$ wget https://yum.spaceduck.org/kernel-ml-aufs/kernel-ml-aufs.repo -O /etc/yum.repo.d/kernel-ml-aufs.repo
$ yum clean all && yum makecache
$ yum install kernel-ml-aufs kernel-ml-aufs-devel -y

# 重启计算机。启动时注意选择该 5.xxx 版本的内核
$ reboot
$ cat /proc/filesystems | grep aufs
```



我的 docker version 是20.10.6了，默认使用 overlay2，所以没有 aufs 文件夹

```sh
# 查看docker 使用的存储文件系统
$ docker info | grep Storage
Storage Driver: overlay2
```

在 /etc/docker/daemon.json 增加如下配置可以修改 docker 的存储文件系统为aufs

```json
{
  "storage-driver": "overlay2"
}
```

```sh
# 重启 docker 再次查看
$ service docker restart
$ docker info | grep Storage
Storage Driver: aufs
```

拉取镜像后 docker pull 结果显示 ubuntu:18.04 镜像共有 3个 layer，在执行命令的结果中也3个对应的存储文件目录。

```sh
，Docker 1.10 之后，diff 目录下的存储镜像 layer 文件夹不再与镜像 ID 相同# 拉取一个 ubuntu
$ docker pull ubuntu:18.04
18.04: Pulling from library/ubuntu
6e0aa5e7af40: Pull complete 
d47239a868b3: Pull complete 
49cbb10cca85: Pull complete 
Digest: sha256:122f506735a26c0a1aff2363335412cfc4f84de38326356d31ee00c2cbe52171
Status: Downloaded newer image for ubuntu:18.04
docker.io/library/ubuntu:18.04
# layer 的 diff 文件夹
$ls /var/lib/docker/aufs/diff
2f70cfa4439df02ada921ed8ce1141cf9cdc9787622dd98b4a460b968c65bb14
42f094424a3c6a75bfca9569ebe885f136e52bd2874a2fa6489f4ad79253d4c9
52452e2eaaa2bc86f01b97c43b26676ca8606b13844bc78dd45596d36afed680
# layer 的 挂载 文件夹
$ ls /var/lib/docker/aufs/mnt/
2f70cfa4439df02ada921ed8ce1141cf9cdc9787622dd98b4a460b968c65bb14
42f094424a3c6a75bfca9569ebe885f136e52bd2874a2fa6489f4ad79253d4c9
52452e2eaaa2bc86f01b97c43b26676ca8606b13844bc78dd45596d36afed680
# layer 文件
$ ls /var/lib/docker/aufs/layers/
2f70cfa4439df02ada921ed8ce1141cf9cdc9787622dd98b4a460b968c65bb14
42f094424a3c6a75bfca9569ebe885f136e52bd2874a2fa6489f4ad79253d4c9
52452e2eaaa2bc86f01b97c43b26676ca8606b13844bc78dd45596d36afed680
# 在 layer 文件记录堆栈里位于某 layer 之下的其它 layer，即 layers 之间的关系
$ cat /var/lib/docker/aufs/layers/42f094424a3c6a75bfca9569ebe885f136e52bd2874a2fa6489f4ad79253d4c9 
52452e2eaaa2bc86f01b97c43b26676ca8606b13844bc78dd45596d36afed680
2f70cfa4439df02ada921ed8ce1141cf9cdc9787622dd98b4a460b968c65bb14
$ cat /var/lib/docker/aufs/layers/52452e2eaaa2bc86f01b97c43b26676ca8606b13844bc78dd45596d36afed680 
2f70cfa4439df02ada921ed8ce1141cf9cdc9787622dd98b4a460b968c65bb14
# 可以看到镜像 id 不与上面任何一个相同。因为Docker 1.10 之后，diff 目录下的存储镜像 layer 文件夹不再与镜像 ID 相同
$  docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
ubuntu       18.04     3339fde08fc3   3 weeks ago   63.3MB
```



**// todo: 实操**

接下来以 ubuntu:18.04 镜像为基础镜像，创建一个名为 changed-ubuntu 的镜像。这个镜像只是在镜像的 /tmp 文件夹中添加一个写了 "Hello world" 的文件。可以使用下面的 Dockerfile 来实现

```dockerfile
FROM ubuntu:18.04
RUN ehco "Hello world" > /tmp/newfile
```

```sh
# 在Dockefile 所在位置通过 docker build 构建镜像
$ docker build -t changed-ubuntu
# 查看镜像
$ docker images
# 查看镜像使用了哪些 image layer
$ docker history changed-ubuntu
```

从输出中可以看到 9d8602c9aee1 image layer 位于最上层，只有 12B 大小，由如下命令创建 `/bin/sh -c echo ” Hello world" > /tmp/newfile` 。也就是说，changed-ubuntu 镜像只占用了12B 的磁盘空间，这证明了 AUFS 是如何高效使用磁盘空间的。而下面 4 层 image layer，则是共享地构成 ubuntu:18.04 镜像的 4 个 image layer。"missing" 标记的 layer，是自 Docker 1.10 之后，一个镜像的 image layer 的 image history 数据都存储在一个文件中导致的，这是 Docker官方认为的正常行为

```sh
# 再次查看 diff、mnt 等
$ ls /var/lib/docker/aufs/diff
$ ls /var/lib/docker/aufs/mnt
```

可以看到 diff、mnt 目录均多了一个文件夹 xxxxxx...。当使用如下命令来查看它的 metadata 时，可以看到，它前面的 layer 就是 ubuntu:18.04 镜像所使用的 image layer。进一步探查 diff 发现多了一个 tmp/newfile 文件，文件中只有一行 "Hello world"。至此就完整分析出                                了 image layer 和 AUFS 之间是如何通过共享文件和文件夹来实现镜像存储的

```sh
$ cat /var/lib/docker/aufs/layers/xxxxxx...
$ cat /var/lib/docker/aufs/diff/xxxxxx...
$ cat /var/lib/docker/aufs/diff/xxxxxx.../tmp/newfile
```



### 自己动手写 AUFS







