

+++

title = "Linux.cgroup"
description = "Linux.cgroup"
tags = ["it", "Linux"]

+++



摘自：https://segmentfault.com/a/1190000021196038

cgroup子系统的分析可参考下面两篇： [cgroup 分析之CPU和内存部分](https://ggaaooppeenngg.github.io/zh-CN/2017/05/07/cgroups-分析之内存和CPU/) ， [cgroup 子系统之 net_cls 和 net_prio](https://ggaaooppeenngg.github.io/zh-CN/2017/05/19/cgroup-子系统之-net-cls-和-net-prio/) 。

介绍docker的的过程中，提到lxc利用cgroup来提供资源的限额和控制，本文主要介绍cgroup的用法和操作命令，主要内容来自： https://access.redhat.com/site/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Resource_Management_Guide/ch01.html ， https://www.kernel.org/doc/Documentation/cgroups/cgroups.txt 。



## Linux.cgroup

Linux Namespace 技术帮助进程隔离出自己单独的空间，但 Docker 如何限制每个空间的大小，保证它们不会互相争抢的呢？这就要用到 Linux Cgroups 技术

Linux Cgroups (Control Groups) 提供了对一组进程及其将来子进程的资源限制、控制、统计的能力，这些资源包括 CPU、内存、存储、网络等。通过 Cgroups ，可以方便地限制某个进程的资源占用，并且可以实时地监控进程的监控和统计信息。

cgroup的功能在于将一台计算机上的资源(CPU,memory, network)进行分片，来防止*进程*间不利的资源抢占。

### Cgroups 中的 3 个组件

- cgroup：关联一组task和一组subsystem的*配置参数*。一个task对应一个进程, cgroup是资源分片的最小单位。是对进程分组管理的一种机制，一个 cgroup 包含一组进程，井可以在这个 cgroup 上增加 Linux subsystem 的各种参数配置，将一组进程和 subsystem 的系统参数关联起来。

- subsystem：资源管理器，一个subsystem对应一项资源的管理，如 cpu, cpuset, memory等。是一组资源控制的模块。每个 subsystem 会关联到定义了相应限制的 cgroup 上，并对这个 cgroup 中的进程做相应的限制和控制。这些 subsystem 是逐步合并到内核中的，要查看当前的内核支持的 subsystem，可以安装 cgroup 的命令行工具（`apt-get install cgroup-bin`），通过 `lssubsys` 看到 Kernel 支持的 subsystem。一般包含如下几项：

  - blkio：设置对块设备（比如硬盘）输入输出的访问控制
  - cpu：设置 cgroup 中进程的 CPU 被调度的策略
  - cpuacct：可以统计 cgroup 中进程的 CPU 占用
  - cpuset：在多核机器上设置 cgroup 中进程可以使用的 CPU 和内存（此处内存仅使用于 NUMA 架构）
  - devices：控制 cgroup 中进程对设备的访问
  - freezer：用于挂起（suspend）和恢复（resume) cgroup 中的进程
  - memory：用于控制 cgroup 中进程的内存占用
  - net_els：用于将 cgroup 中进程产生的网络包分类，以便 Linux
  - tc (traffic conoller）可以根据分类区分出来自某个 cgroup 的包并做限流或监控
  - net_prio：设置 cgroup 中进程产生的网络流量的优先级
  - ns：这个 subsystem 比较特殊，它的作用是使 cgroup 中的进程在新的 Namespace fork 新进程（NEWNS）时，创建出一个新的 cgroup ，这个 cgroup 包含新的 Namespace 的进程

  ```sh
  $ yum install -y libcgroup-tools.x86_64 libcgroup
  $ lssubsys
  ```

- hierarchy：关联一个到多个`subsystem`和一组树形结构的`cgroup`. 和`cgroup`不同，`hierarchy`包含的是可管理的`subsystem`而非具体参数。功能是把 cgroup 串成一个树状的结构，一个这样的树便是 hierarchy，通过这种树状结构，Cgroups 可以做到继承。比如，系统对一组定时的任务进程通过 cgroup1 限制了 CPU 的使用率，然后其中有一个定时 dump 日志的进程还需要限制磁盘 IO，为了避免限制了磁盘 IO 之后影响到其他进程，就可以创建 cgroup2 ，使其继承于 cgroup2 并限制磁盘的 IO ，这样 cgroup2 便继承了 cgroup1 中对 CPU 使用率的限制，并且增加了磁盘 IO 的限制而不影响到 cgroup1 中的其他进程。



可见，cgroup对资源的管理是一个树形结构，类似进程。

相同点 - 分层结构，子进程/cgroup继承父进程/cgroup

不同点 - 进程是一个单根树状结构(pid=0为根)，而cgroup整体来看是一个多树的森林结构(hierarchy为根)。

一个典型的`hierarchy`挂载目录如下

```
/cgroup/
├── blkio                           <--------------- hierarchy/root cgroup                   
│   ├── blkio.io_merged             <--------------- subsystem parameter
... ...
│   ├── blkio.weight
│   ├── blkio.weight_device
│   ├── cgroup.event_control
│   ├── cgroup.procs
│   ├── lxc                         <--------------- cgroup
│   │   ├── blkio.io_merged         <--------------- subsystem parameter
│   │   ├── blkio.io_queued
... ... ...
│   │   └── tasks                   <--------------- task list
│   ├── notify_on_release
│   ├── release_agent
│   └── tasks
...
```

**subsystem列表**

RHEL/centos支持的subsystem如下

- blkio — 块存储配额 >> this subsystem sets limits on input/output access to and from block devices such as physical drives (disk, solid state, USB, etc.).
- cpu — CPU时间分配限制 >> this subsystem uses the scheduler to provide cgroup tasks access to the CPU.
- cpuacct — CPU资源报告 >> this subsystem generates automatic reports on CPU resources used by tasks in a cgroup.
- cpuset — CPU绑定限制 >> this subsystem assigns individual CPUs (on a multicore system) and memory nodes to tasks in a cgroup.
- devices — 设备权限限制 >> this subsystem allows or denies access to devices by tasks in a cgroup.
- freezer — cgroup停止/恢复 >> this subsystem suspends or resumes tasks in a cgroup.
- memory — 内存限制 >> this subsystem sets limits on memory use by tasks in a cgroup, and generates automatic reports on memory resources used by those tasks.
- net_cls — 配合tc进行网络限制 >> this subsystem tags network packets with a class identifier (classid) that allows the Linux traffic controller (tc) to identify packets originating from a particular cgroup task.
- net_prio — 网络设备优先级 >> this subsystem provides a way to dynamically set the priority of network traffic per network interface.
- ns — 资源命名空间限制 >> the namespace subsystem.



##### 关系

Cgroups 是凭借这 3 个组件的相互协作实现的

- 系统在创建了新的 hierarchy 之后，系统中所有的进程都会加入这个 hierarchy 的 cgroup 根节点，这个 cgroup 根节点是 hierarchy 默认创建的， 2.2.2 节在这个 hierarchy 中创建 cgroup 都是这个 cgroup 根节点的子节点。
- 一个 subsystem 只能附加到一个 hierarchy 上面
- 一个 hierarchy 可以附加多个 subsystem
- 一个进程可以作为多个 cgroup 的成员，但是这些 cgroup 必须在不同的 hierarchy 中
- 一个进程 fork 出子进程时，子进程将与父进程在同一个 cgroup 中，也可以根据需要将其移动到其它 cgroup 中



Kernel 接口：Cgroups 中的 hierarchy 是一种树状的组织结构，Kernel 为了使对 Cgroups 的配置更直观，是通过一个虚拟的树状文件系统配置 Cgroups 的，通过层级的目录虚拟出 cgroup 树。下面就以一个配置的例子来了解如何操作 Cgroups

首先要创建并挂载一个 hierarchy（cgroup 树）

```sh
$ cd ~
# 创建一个目录作为 hierarchy 挂载点
$ mkdir cgroupdemo
# 挂载一个 hierarchy
$ mount -t cgroup -o none,name=cgroup-test cgroup-test ./cgroup-test
# 挂载后系统在目录下生成了一些默认文件
$ ls cgroup-test/
cgroup.clone_children  cgroup.procs          notify_on_release  tasks
cgroup.event_control   cgroup.sane_behavior  release_agent

```

这些文件就是这个 hierarchy 中 cgroup 根节点的配置项

- cgroup.clone_children：cpuset 的 subsystem 会读取这个配置文件，如果这个值是 1（默认认是 0），子 cgroup 才会继承父 cgroup 的 cpuset 的配置
- cgroup.procs：是树中当前节点 cgroup 中的进程组 ID，现在的位置是在根节点，这个文件中会有现在系统中所有进程组的 ID。
- notify_on_release、release agent：两项会一起使用。notify_on_release 标识当这个 cgroup 最后一个进程退出的时候是否执行了 release_agent；release_agent 则是一个路径，通常用作进程退出之后自动清理掉不再使用的 cgroup
- tasks：标识 cgroup 下面的进程 ID ，如果把一个进程 ID 写到 tasks 文件中，便会将相应的进程加入到这个 cgroup 中



然后在刚刚创建好的 hierarchy 的 cgroup 根节点中扩展出的两个子 cgroup。可以看到，在 cgroup 的目录下创建文件夹时， Kernel 会把文件夹标记为这个 cgroup 的子 cgroup ，它会继承父 cgroup 的属性。

```sh
$ cd cgroup-test
$ mkdir cgroup-1
$ mkdir cgroup-2
$ tree
.
├── cgroup-1
│   ├── cgroup.clone_children
│   ├── cgroup.event_control
│   ├── cgroup.procs
│   ├── notify_on_release
│   └── tasks
├── cgroup-2
│   ├── cgroup.clone_children
│   ├── cgroup.event_control
│   ├── cgroup.procs
│   ├── notify_on_release
│   └── tasks
├── cgroup.clone_children
├── cgroup.event_control
├── cgroup.procs
├── cgroup.sane_behavior
├── notify_on_release
├── release_agent
└── tasks
```



在 cgroup 中添加和移动进程：一个进程在一个 Cgroups 的 hierarchy 中，只能在一个 cgroup 节点上存在，系统的所有进程都会默认在根节点上存在，可以将进程移动到其他 cgroup 节点，只需要将进程 ID 写到要移动到的 cgroup 节点的 tasks 文件中即可。

```sh
$ cd cgroup1
$ echo $$
4615
# 将所在的终端进程移动到 cgroup1 中
$ sh -c "echo $$ >> tasks"
$ cat /proc/4615/cgroup
cat /proc/4615/cgroup
12:name=cgroup-test:/cgroup-1
11:hugetlb:/
10:net_prio,net_cls:/
9:pids:/user.slice
8:memory:/
7:cpuset:/
6:blkio:/
5:freezer:/
4:devices:/user.slice
3:cpuacct,cpu:/
2:perf_event:/
1:name=systemd:/user.slice/user-0.slice/session-1.scope
```



通过 subsystem 限制 cgroup 中进程的资源：在上面创建 hierarchy 的时候，这个 hierarchy 并没有关联到任何的 subsystem ，所以没办法通过那个 hierarchy 中的 cgroup 节点限制进程的资源占用，其实系统默认已经为每个 subsystem 创建了一个默认的 hierarchy ，比如 memory hierarchy

```sh
$ mount | grep memory
cgroup on /sys/fs/cgroup/memory type cgroup (rw,nosuid,nodev,noexec,relatime,seclabel,memory)
```

可以看到，/sys/fs/cgroup/memory 目录便是挂在了 memory subsystem 的 hierarchy 上。下面，就通过在这个 hierarchy 中创建 cgroup ，限制如下进程占用的内存。

```sh
$ cd /sys/fs/cgroup/memory
# 安装 stress 工具
$ yum install -y epel-release
$ yum install -y stress
# 先在不做限制的情况下，启动一个 stress 进程，看到占用了 200m 内存，然后 ctrl+c 关闭
$ stress --vm-bytes 200m --vm-keep -m 1
# 创建一个 cgroup
$ mkdir test-limit-memory
$ cd test-limit-memor
# 设置最大 cgroup 的最大内存占用为 lOOMB
$ sh -c "echo '100m' > memory.limit_in_bytes"
# 将当前进程移动到这个 cgroup
$ sh -c "echo $$ > tasks"
# 再次启动一个 200m 内存占用的 stress 进程，发现只占用了 100m 内存，说明限制内存资源成功
$ stress --vm-bytes 200m --vm-keep -m 1
```



### cgroup操作准则与方法

##### 操作准则

**1.一个hierarchy可以有多个 subsystem (mount 的时候hierarchy可以attach多个subsystem)**

> A single hierarchy can have one or more subsystems attached to it.

eg.

```
mount -t cgroup -o cpu,cpuset,memory cpu_and_mem /cgroup/cpu_and_mem
```

![cgroup-rule1](https://segmentfault.com/img/remote/1460000021196041)

**2.一个已经被挂载的 subsystem 只能被再次挂载在一个空的 hierarchy 上 (已经mount一个subsystem的hierarchy不能挂载一个已经被其它hierarchy挂载的subsystem)**

> Any single subsystem (such as cpu) cannot be attached to more than one hierarchy if one of those hierarchies has a different subsystem attached to it already.

![cgroup-rule2](https://segmentfault.com/img/remote/1460000021196043)

**3.每个task只能在一同个hierarchy的唯一一个cgroup里(不能在同一个hierarchy下有超过一个cgroup的tasks里同时有这个进程的pid)**

> Each time a new hierarchy is created on the systems, all tasks on the system are initially members of the default cgroup of that hierarchy, which is known as the root cgroup. For any single hierarchy you create, each task on the system can be a member of exactly onecgroup in that hierarchy. A single task may be in multiple cgroups, as long as each of those cgroups is in a different hierarchy. As soon as a task becomes a member of a second cgroup in the same hierarchy, it is removed from the first cgroup in that hierarchy. At no time is a task ever in two different cgroups in the same hierarchy.

![cgroup-rule3](https://segmentfault.com/img/remote/1460000021196044)

**4.子进程在被fork出时自动继承父进程所在cgroup，但是fork之后就可以按需调整到其他cgroup**

> Any process (task) on the system which forks itself creates a child task. A child task automatically inherits the cgroup membership of its parent but can be moved to different cgroups as needed. Once forked, the parent and child processes are completely independent.

![cgroup-rule4](https://segmentfault.com/img/remote/1460000021196042)

**5.其它**

- 限制一个task的唯一方法就是将其加入到一个cgroup的task里
- 多个subsystem可以挂载到一个hierarchy里, 然后通过不同的cgroup中的subsystem参数来对不同的task进行限额
- 如果一个hierarchy有太多subsystem，可以考虑重构 - 将subsystem挂到独立的hierarchy; 相应的, 可以将多个hierarchy合并成一个hierarchy
- 因为可以只挂载少量subsystem, 可以实现只对task单个方面的限额; 同时一个task可以被加到多个hierarchy中，从而实现对多个资源的控制

##### 操作方法

**1.挂载subsystem**

- 利用cgconfig服务及其配置文件 `/etc/cgconfig.conf` - 服务启动时自动挂载

  ```
  subsystem = /cgroup/hierarchy;
  ```

- 命令行操作

  ```
  mount -t cgroup -o subsystems name /cgroup/name
  ```

  取消挂载

  ```
  umount /cgroup/name
  ```

  eg. 挂载 cpuset, cpu, cpuacct, memory 4个subsystem到`/cgroup/cpu_and_mem` 目录(hierarchy)

```
  mount {
      cpuset  = /cgroup/cpu_and_mem;
      cpu    = /cgroup/cpu_and_mem;
      cpuacct = /cgroup/cpu_and_mem;
      memory  = /cgroup/cpu_and_mem;
  }
```

or

```
mount -t cgroup -o remount,cpu,cpuset,memory cpu_and_mem /cgroup/cpu_and_mem
```

**2. 新建/删除 cgroup**

- 利用cgconfig服务及其配置文件 `/etc/cgconfig.conf` - 服务启动时自动挂载

  ```
  group <name> {
      [<permissions>]    <controller> {  <param name> = <param value>;
          …
      }
      …
  }
  ```

- 命令行操作

  - 新建1 `cgcreate -t uid:gid -a uid:gid -g subsystems:path`
  - 新建2 `mkdir /cgroup/hierarchy/name/child_name`
  - 删除1 `cgdelete subsystems:path` (使用 -r 递归删除)
  - 删除2 `rm -rf /cgroup/hierarchy/name/child_name` (cgconfig service not running)

**3. 权限管理**

- 利用cgconfig服务及其配置文件 `/etc/cgconfig.conf` - 服务启动时自动挂载

  ```
  perm {
      task {
          uid = <task user>;
          gid = <task group>;
      }
      admin {
        uid = <admin name>;
        gid = <admin group>;
      }
  }
  ```

- 命令行操作

   

  ```
  chown
  ```

  eg.

```
  group daemons {
      cpuset {
          cpuset.mems = 0;
          cpuset.cpus = 0;
      }
  }
  group daemons/sql {
      perm {
          task {
              uid = root;
              gid = sqladmin;
          } admin {
              uid = root;
              gid = root;
          }
      }
      cpuset {
          cpuset.mems = 0;
          cpuset.cpus = 0;
      }
  }
```

or

```
  ~]$ mkdir -p /cgroup/red/daemons/sql
  ~]$ chown root:root /cgroup/red/daemons/sql/*
  ~]$ chown root:sqladmin /cgroup/red/daemons/sql/tasks
  ~]$ echo 0 > /cgroup/red/daemons/cpuset.mems
  ~]$ echo 0 > /cgroup/red/daemons/cpuset.cpus
  ~]$ echo 0 > /cgroup/red/daemons/sql/cpuset.mems
  ~]$ echo 0 > /cgroup/red/daemons/sql/cpuset.cpus
```

**4. cgroup参数设定**

- 命令行1 `cgset -r parameter=value path_to_cgroup`

- 命令行2 `cgset --copy-from path_to_source_cgroup path_to_target_cgroup`

- 文件

   

  ```
  echo value > path_to_cgroup/parameter
  ```

  eg.

```
  cgset -r cpuset.cpus=0-1 group1
  cgset --copy-from group1/ group2/
  echo 0-1 > /cgroup/cpuset/group1/cpuset.cpus
```

**5. 添加task**

- 命令行添加进程 `cgclassify -g subsystems:path_to_cgroup pidlist`

- 文件添加进程 `echo pid > path_to_cgroup/tasks`

- 在cgroup中启动进程 `cgexec -g subsystems:path_to_cgroup command arguments`

- 在cgroup中启动服务 `echo 'CGROUP_DAEMON="subsystem:control_group"' >> /etc/sysconfig/`

- 利用cgrulesengd服务初始化，在配置文件`/etc/cgrules.conf`中

  ```
  user<:command> subsystems control_group
  
  其中:
  +用户user的所有进程的subsystems限制的group为control_group
  +<:command>是可选项，表示对特定命令实行限制
  +user可以用@group表示对特定的 usergroup 而非user
  +可以用*表示全部
  +%表示和前一行的该项相同
  ```

  eg.

```
  cgclassify -g cpu,memory:group1 1701 1138
  echo -e "1701\n1138" |tee -a /cgroup/cpu/group1/tasks /cgroup/memory/group1/tasks
  cgexec -g cpu:group1 lynx http://www.redhat.com
  sh -c "echo \$$ > /cgroup/lab1/group1/tasks && lynx http://www.redhat.com"
```

通过/etc/cgrules.conf 对特定服务限制

```
  maria          devices        /usergroup/staff
  maria:ftp      devices        /usergroup/staff/ftp
  @student       cpu,memory     /usergroup/student/
  %              memory         /test2/
```

**6. 其他**

- cgsnapshot会根据当前cgroup情况生成/etc/cgconfig.conf文件内容

  ```
  gsnapshot  [-s] [-b FILE] [-w FILE] [-f FILE] [controller]
    -b, --blacklist=FILE  Set the blacklist configuration file (default /etc/cgsnapshot_blacklist.conf)
    -f, --file=FILE       Redirect the output to output_file
    -s, --silent          Ignore all warnings
    -t, --strict          Don't show the variables which are not on the whitelist
    -w, --whitelist=FILE  Set the whitelist configuration file (don't used by default)
  ```

- 查看进程在哪个cgroup

  ```
  ps -O cgroup
  或
  cat /proc/<PID>/cgroup
  ```

- 查看subsystem mount情况

  ```
  cat /proc/cgroups
  
  lssubsys -m <subsystems>
  ```

- 查看cgroup `lscgroup`

- 查看cgroup参数值

  ```
  cgget -r parameter list_of_cgroups
  cgget -g <controllers>:<path>
  ```

- cgclear删除hierarchy极其所有cgroup

- 事件通知API - 目前只支持memory.oom_control

- 更多

  - man 1 cgclassify — the cgclassify command is used to move running tasks to one or more cgroups.
  - man 1 cgclear — the cgclear command is used to delete all cgroups in a hierarchy.
  - man 5 cgconfig.conf — cgroups are defined in the cgconfig.conf file.
  - man 8 cgconfigparser — the cgconfigparser command parses the cgconfig.conf file and mounts hierarchies.
  - man 1 cgcreate — the cgcreate command creates new cgroups in hierarchies.
  - man 1 cgdelete — the cgdelete command removes specified cgroups.
  - man 1 cgexec — the cgexec command runs tasks in specified cgroups.
  - man 1 cgget — the cgget command displays cgroup parameters.
  - man 1 cgsnapshot — the cgsnapshot command generates a configuration file from existing subsystems.
  - man 5 cgred.conf — cgred.conf is the configuration file for the cgred service.
  - man 5 cgrules.conf — cgrules.conf contains the rules used for determining when tasks belong to certain cgroups.
  - man 8 cgrulesengd — the cgrulesengd service distributes tasks to cgroups.
  - man 1 cgset — the cgset command sets parameters for a cgroup.
  - man 1 lscgroup — the lscgroup command lists the cgroups in a hierarchy.
  - man 1 lssubsys — the lssubsys command lists the hierarchies containing the specified subsystems.

### subsystem配置

##### blkio - BLOCK IO限额

- common
  - `blkio.reset_stats`：重置统计信息，写int到此文件
  - `blkio.time`：统计cgroup对设备的访问时间 - `device_types:node_numbers milliseconds`
  - `blkio.sectors`：统计cgroup对设备扇区访问数量 - `device_types:node_numbers sector_count`
  - `blkio.avg_queue_size`：统计平均IO队列大小(需要`CONFIG_DEBUG_BLK_CGROUP=y`)
  - `blkio.group_wait_time`：统计cgroup等待总时间(需要`CONFIG_DEBUG_BLK_CGROUP=y`, 单位ns)
  - `blkio.empty_time`：统计cgroup无等待io总时间(需要`CONFIG_DEBUG_BLK_CGROUP=y`, 单位ns)
  - `blkio.idle_time`：reports the total time (in nanoseconds — ns) the scheduler spent idling for a cgroup in anticipation of a better request than those requests already in other queues or from other groups.
  - `blkio.dequeue`：此cgroup IO操作被设备dequeue次数(需要`CONFIG_DEBUG_BLK_CGROUP=y`)：`device_types:node_numbers number`
  - `blkio.io_serviced`：报告CFQ scheduler统计的此cgroup对特定设备的IO操作(read, write, sync, or async)次数 - `device_types:node_numbers operation number`
  - `blkio.io_service_bytes`：报告CFQ scheduler统计的此cgroup对特定设备的IO操作(read, write, sync, or async)数据量 - `device_types:node_numbers operation bytes`
  - `blkio.io_service_time`：报告CFQ scheduler统计的此cgroup对特定设备的IO操作(read, write, sync, or async)时间(单位ns) - `device_types:node_numbers operation time`
  - `blkio.io_wait_time`：此cgroup对特定设备的特定操作(read, write, sync, or async)的等待时间(单位ns) - `device_types:node_numbers operation time`
  - `blkio.io_merged`：此cgroup的BIOS requests merged into IO请求的操作(read, write, sync, or async)的次数 - `number operation`
  - `blkio.io_queued`：此cgroup的queued IO 操作(read, write, sync, or async)的请求次数 - `number operation`
- Proportional weight division 策略 - 按比例分配block io资源
  - `blkio.weight`：100-1000的相对权重，会被`blkio.weight_device`的特定设备权重覆盖
  - `blkio.weight_device`：特定设备的权重 - device_types:node_numbers weight
- I/O throttling (Upper limit) 策略 - 设定IO操作上限
  - 每秒读/写数据上限
    - `blkio.throttle.read_bps_device`：`device_types:node_numbers bytes_per_second`
    - `blkio.throttle.write_bps_device`：`device_types:node_numbers bytes_per_second`
  - 每秒读/写操作次数上限
    - `blkio.throttle.read_iops_device`：`device_types:node_numbers operations_per_second`
    - `blkio.throttle.write_iops_device`：`device_types:node_numbers operations_per_second`
  - 每秒具体操作(read, write, sync, or async)的控制
    - `blkio.throttle.io_serviced`：`device_types:node_numbers operation operations_per_second`
    - ``blkio.throttle.io_service_bytes`：`device_types:node_numbers operation bytes_per_second`

##### cpu - CPU使用时间限额

- CFS(Completely Fair Scheduler)策略 - CPU最大资源限制

  - `cpu.cfs_period_us`, `cpu.cfs_quota_us`：（必选，二者配合）前者规定时间周期(微秒)后者规定cgroup最多可使用时间(微秒)，实现task对单个cpu的使用上限(cfs_quota_us是cfs_period_us的两倍即可限定在双核上完全使用)。
  - `cpu.stat`：记录cpu统计信息，包含如下内容
    -  `nr_periods`：经历了几个 cfs_period_us
    - `nr_throttled`：cgroup 里的 task 被限制了几次
    - `throttled_time`：cgroup 里的 task 被限制了多少纳秒
  - `cpu.shares`：（可选）cpu轮转权重的相对值

- RT(Real-Time scheduler)策略 - CPU最小资源限制

  - `cpu.rt_period_us`, `cpu.rt_runtime_us`

    二者配合使用规定cgroup里的task每cpu.rt_period_us(微秒)必然会执行cpu.rt_runtime_us(微秒)

##### cpuacct - CPU资源报告

- `cpuacct.usage`：cgroup中所有task的cpu使用时长(纳秒)
- `cpuacct.stat`：cgroup中所有task的用户态和内核态分别使用cpu的时长
- `cpuacct.usage_percpu：cgroup中所有task使用每个cpu的时长`

##### cpuset - CPU绑定

- `cpuset.cpus`：必选 - cgroup可使用的cpu，如0-2,16代表 0,1,2,16这4个cpu
- `cpuset.mems`：必选 - cgroup可使用的memory node
- `cpuset.memory_migrate`：可选 - 当cpuset.mems变化时page上的数据是否迁移, default 0
- `cpuset.cpu_exclusive`：可选 - 是否独占cpu， default 0
- `cpuset.mem_exclusive`：可选 - 是否独占memory，default 0
- `cpuset.mem_hardwall`：可选 - cgroup中task的内存是否隔离， default 0
- `cpuset.memory_pressure`：可选 - a read-only file that contains a running average of the memory pressure created by the processes in this cpuset
- `cpuset.memory_pressure_enabled`：可选 - cpuset.memory_pressure开关，default 0
- `cpuset.memory_spread_page`：可选 - contains a flag (0 or 1) that specifies whether file system buffers should be spread evenly across the memory nodes allocated to this cpuset， default 0
- `cpuset.memory_spread_slab`：可选 - contains a flag (0 or 1) that specifies whether kernel slab caches for file input/output operations should be spread evenly across the cpuset， default 0
- `cpuset.sched_load_balance`：可选 - cgroup的cpu压力是否会被平均到cpu set中的多个cpu, default 1
- `cpuset.sched_relax_domain_level`：可选 - cpuset.sched_load_balance的策略
  - -1 = Use the system default value for load balancing
  - 0 = Do not perform immediate load balancing; balance loads only periodically
  - 1 = Immediately balance loads across threads on the same core
  - 2 = Immediately balance loads across cores in the same package
  - 3 = Immediately balance loads across CPUs on the same node or blade
  - 4 = Immediately balance loads across several CPUs on architectures with non-uniform memory access (NUMA)
  - 5 = Immediately balance loads across all CPUs on architectures with NUMA

##### device - cgoup的device限制

- 设备黑/白名单
  - `devices.allow`：允许名单
  - `devices.deny`：禁止名单
  - 语法：type device_types:node_numbers access type - b (块设备) c (字符设备) a (全部设备) access - r 读 w 写 m 创建
- `devices.list`：报告

##### freezer - 暂停/恢复 cgroup的限制

- 不能出现在root目录下
- `freezer.state`：FROZEN 停止 FREEZING 正在停止 THAWED 恢复

##### memory - 内存限制

- `memory.usage_in_bytes`：报告内存限制byte
- `memory.memsw.usage_in_bytes`：报告cgroup中进程当前所用内存+swap空间
- `memory.max_usage_in_bytes`：报告cgoup中的最大内存使用
- `memory.memsw.max_usage_in_bytes`：报告最大使用到的内存+swap
- `memory.limit_in_bytes`：cgroup - 最大内存限制，单位k,m,g. -1代表取消限制
- `memory.memsw.limit_in_bytes`：最大内存+swap限制，单位k,m,g. -1代表取消限制
- `memory.failcnt`：报告达到最大允许内存的次数
- `memory.memsw.failcnt`：报告达到最大允许内存+swap的次数
- `memory.force_empty`：设为0且无task时，清除cgroup的内存页
- `memory.swappiness`：换页策略，60基准，小于60降低换出机率，大于60增加换出机率
- `memory.use_hierarchy`：是否影响子group
- `memory.oom_control`：0 enabled，当oom发生时kill掉进程
- `memory.stat`：报告cgroup限制状态
  - `cache`：page cache, including tmpfs (shmem), in bytes
  - `rss`：anonymous and swap cache, not including tmpfs (shmem), in bytes
  - `mapped_file`：size of memory-mapped mapped files, including tmpfs (shmem), in bytes
  - `pgpgin`：number of pages paged into memory
  - `pgpgout`：number of pages paged out of memory
  - `swap`：swap usage, in bytes
  - `active_anon`：anonymous and swap cache on active least-recently-used (LRU) list, including tmpfs (shmem), in bytes
  - `inactive_anon`：anonymous and swap cache on inactive LRU list, including tmpfs (shmem), in bytes
  - `active_file`：file-backed memory on active LRU list, in bytes
  - `inactive_file`：file-backed memory on inactive LRU list, in bytes
  - `unevictable`：memory that cannot be reclaimed, in bytes
  - `hierarchical_memory_limit`：memory limit for the hierarchy that contains the memory cgroup, in bytes
  - `hierarchical_memsw_limit`：memory plus swap limit for the hierarchy that contains the memory cgroup, in bytes

##### net_cls

- `net_cls.classid`：指定tc的handle，通过tc实现网络控制

##### net_prio 指定task网络设备优先级

- `net_prio.prioidx`：a read-only file which contains a unique integer value that the kernel uses as an internal representation of this cgroup.
- `net_prio.ifpriomap`：网络设备使用优先级 - `<network_interface> <priority>`

##### 其它

- `tasks`：该cgroup的所有进程pid
- `cgroup.event_control`：event api
- `cgroup.procs`：thread group id
- `release_agent`(present in the root cgroup only)：根据notify_on_release是否在task为空时执行的脚本
- `notify_on_release`：当cgroup中没有task时是否执行release_agent

### 总结

1. 本文总结了cgroup的操作方法和详细的可配置项，为对更好的控制系统中的资源分配打下基础
2. 对于限制资源分配的两个场景，在针对特殊APP的场景中可进行非常细致的调优，而在通用的资源隔离的角度上看，可能更关注的是CPU和内存相关的主要属性

