

+++

title = "container.ufs"
description = "container.ufs"
tags = ["it", "container"]

+++



# container.ufs



容器通过一些ufs实现分层

layer：层

ufs：UnionFS，联合文件系统。把其它文件系统联合到一个联合挂载点的文件系统



## Overlay2*

https://docs.docker.com/storage/storagedriver/overlayfs-driver/#how-the-overlay2-driver-works

https://zhuanlan.zhihu.com/p/41958018



`overlay2`驱动程序本身支持多达128个较低级别的OverlayFS层。此功能可为与层相关的Docker命令（例如`docker build`和`docker commit` ）提供更好的性能，并在后备文件系统上消耗更少的inode



实验使用的 docker 信息如下

```shell
$ docker info | grep "Storage \| Version"
 Server Version: 20.10.7
 Storage Driver: overlay2
 Cgroup Version: 1
 Kernel Version: 4.18.0-305.7.1.el8_4.x86_64
```



这里拉取一个nginx，共6个layer。没有使用官方例举的ubuntu是因为ubuntu:latest版本只有一个layer了。

```shell
$ docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
b4d181a07f80: Pull complete
66b1c490df3f: Pull complete
d0f91ae9b44c: Pull complete
baf987068537: Pull complete
6bbc76cbebeb: Pull complete
32b766478bc2: Pull complete
Digest: sha256:353c20f74d9b6aee359f30e8e4f69c3d7eaea2f610681c4a95849a2fd7c497f9
Status: Downloaded newer image for nginx:latest
docker.io/library/nginx:latest
```



使用Overlay2的存储驱动，镜像和容器相关文件都存储在 `/var/lib/docker/overlay2` 目录下

```shell
$ cd /var/lib/docker/overlay2
```



### 镜像层 

可以看到nginx镜像的6个layer

```shell
$ ls
78bc97220f5645a49a1d60608114eed8b9c23de5385ee86a78340d205b8cec4c
d99c2676da17a677115c1107ddb951b7aa6821f599c8cf2e1e118ae562c17a30
dfbaafe93e3b4e1b2c1c7dc097e82b5fccc5a9ba61a7c0ab6b16870316717ba3
e23697525103cb384a2985f8e80db6d3efdc64725e377405a28fef6348b8e923
f1046d0298b8a67af1bcec0638c83c0517bb98ce76516d4ea677309afeb05234
fa64f42d6b080a0fa56f682595d442bb39fb17e04ce8811bb3acba36c22ee20f
l
```



`l`目录：包含缩短的层标识符作为符号链接，连接到各层的`diff`目录。缩短标识符通过 hash 得到，使用它是为了避免撞上`mount`命令参数的页大小限制，毕竟真实目录名很长

```shell
$ ll l/
总用量 0
lrwxrwxrwx. 1 root root 72 7月  14 23:51 6ODHHOP2BBFKHXH5JEJ3KROTA4 -> ../d99c2676da17a677115c1107ddb951b7aa6821f599c8cf2e1e118ae562c17a30/diff
lrwxrwxrwx. 1 root root 72 7月  14 23:51 7IAFM5I7VDIWABLIA75JJOFAWD -> ../dfbaafe93e3b4e1b2c1c7dc097e82b5fccc5a9ba61a7c0ab6b16870316717ba3/diff
lrwxrwxrwx. 1 root root 72 7月  14 23:51 R2UMGPEDCS6PDNKLVP74J7RQZC -> ../f1046d0298b8a67af1bcec0638c83c0517bb98ce76516d4ea677309afeb05234/diff
lrwxrwxrwx. 1 root root 72 7月  14 23:51 TPYKCH6JDXLP3PPJSJM7YBFI4F -> ../e23697525103cb384a2985f8e80db6d3efdc64725e377405a28fef6348b8e923/diff
lrwxrwxrwx. 1 root root 72 7月  14 23:51 VQIRWQRQYMM466M3CPDWQAXXN5 -> ../78bc97220f5645a49a1d60608114eed8b9c23de5385ee86a78340d205b8cec4c/diff
lrwxrwxrwx. 1 root root 72 7月  14 23:50 ZL47AMV4RSSSBV2TCLVRLWRDDO -> ../fa64f42d6b080a0fa56f682595d442bb39fb17e04ce8811bb3acba36c22ee20f/diff
```



层内文件说明如下：

- `diff`目录：装有当前 layer 的内容
- `link`文件：写有 layer 的缩短标识符
- `lower`文件：写有当前层所依赖的更低层（父层）的缩短标识符
- `work`目录：供 OverlayFS 内部使用

```shell
$ ls *
78bc97220f5645a49a1d60608114eed8b9c23de5385ee86a78340d205b8cec4c:
committed  diff  link  lower  work

d99c2676da17a677115c1107ddb951b7aa6821f599c8cf2e1e118ae562c17a30:
committed  diff  link  lower  work

dfbaafe93e3b4e1b2c1c7dc097e82b5fccc5a9ba61a7c0ab6b16870316717ba3:
diff  link  lower  work

e23697525103cb384a2985f8e80db6d3efdc64725e377405a28fef6348b8e923:
committed  diff  link  lower  work

f1046d0298b8a67af1bcec0638c83c0517bb98ce76516d4ea677309afeb05234:
committed  diff  link  lower  work

fa64f42d6b080a0fa56f682595d442bb39fb17e04ce8811bb3acba36c22ee20f:
committed  diff  link
```



`fa64f4...` 这个目录中没有`lower`，因为它是最低层了，是一个基础操作系统层，从`diff`目录结构也可以看到

```shell
$ ls fa64f42d6b080a0fa56f682595d442bb39fb17e04ce8811bb3acba36c22ee20f/diff/
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr

$ cat fa64f42d6b080a0fa56f682595d442bb39fb17e04ce8811bb3acba36c22ee20f/link
ZL47AMV4RSSSBV2TCLVRLWRDDO
```



`lower`中的缩短标识符不是无序的，从左到右是从高层到低层

```shell
$ cat d99c2676da17a677115c1107ddb951b7aa6821f599c8cf2e1e118ae562c17a30/lower
l/ZL47AMV4RSSSBV2TCLVRLWRDDO

$ cat e23697525103cb384a2985f8e80db6d3efdc64725e377405a28fef6348b8e923/lower
l/6ODHHOP2BBFKHXH5JEJ3KROTA4:l/ZL47AMV4RSSSBV2TCLVRLWRDDO

$ cat f1046d0298b8a67af1bcec0638c83c0517bb98ce76516d4ea677309afeb05234/lower
l/TPYKCH6JDXLP3PPJSJM7YBFI4F:l/6ODHHOP2BBFKHXH5JEJ3KROTA4:l/ZL47AMV4RSSSBV2TCLVRLWRDDO

$ cat 78bc97220f5645a49a1d60608114eed8b9c23de5385ee86a78340d205b8cec4c/lower
l/R2UMGPEDCS6PDNKLVP74J7RQZC:l/TPYKCH6JDXLP3PPJSJM7YBFI4F:l/6ODHHOP2BBFKHXH5JEJ3KROTA4:l/ZL47AMV4RSSSBV2TCLVRLWRDDO

$ cat dfbaafe93e3b4e1b2c1c7dc097e82b5fccc5a9ba61a7c0ab6b16870316717ba3/lower
l/VQIRWQRQYMM466M3CPDWQAXXN5:l/R2UMGPEDCS6PDNKLVP74J7RQZC:l/TPYKCH6JDXLP3PPJSJM7YBFI4F:l/6ODHHOP2BBFKHXH5JEJ3KROTA4:l/ZL47AMV4RSSSBV2TCLVRLWRDDO
```



### 容器层

把 nginx 运行起来

```shell
$ docker run -d --name nginx01 -p 8080:80 nginx:latest
```



nginx运行之后，多出来两个容器层

- `d45fb...`：无后缀，与其它层的区别是，它将包含执行容器的联合挂载目录
- `d45fb...-init`：有后缀，主要由一些配置文件构成，用于初始化

```shell
$ ls
d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2
d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2-init
...

$ ll l/
lrwxrwxrwx. 1 root root 72 7月  17 17:54 BRS3M2EWJTBFGJ3WJZW7NFYZXA -> ../d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2/diff
lrwxrwxrwx. 1 root root 77 7月  17 17:54 LCYEOG2IWKZXROCQRCGIWDFMPT -> ../d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2-init/diff


$ tree d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2-init/
d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2-init/
├── committed
├── diff
│   ├── dev
│   │   ├── console
│   │   ├── pts
│   │   └── shm
│   └── etc
│       ├── hostname
│       ├── hosts
│       ├── mtab -> /proc/mounts
│       └── resolv.conf
├── link
├── lower
└── work
    └── work
```



层内文件说明如下：

- `merged`目录包含其父层和自身的统一内容。可以在其中看到其它各个层所有内容的集合

```shell
$ ls *
d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2:
diff  link  lower  merged  work

d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2-init:
committed  diff  link  lower  work

...

$ ls d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2/merged/
bin   docker-entrypoint.d   home   media  proc  sbin  tmp
boot  docker-entrypoint.sh  lib    mnt    root  srv   usr
dev   etc                   lib64  opt    run   sys   var
```



#### 联合挂载详情

具体挂载情况如下：

- `d45fb.../merged`：overlay2文件系统的挂载点，它将lowerdir、upperdir、workdir联合挂载到这一个目录下。
- `rw`：表示可读可写
- `lowerdir`：联合挂载的低层（父层）内容，属于只读层。正好是之前6个镜像层，以及用于初始化的容器层`d45fb...-init/diff`。
- `upperdir`：联合挂载的最高层，属于可读可写层。即该容器层本身的内容，与挂载点同目录下的`d45fb.../diff`。
- `workdir`：联合挂载的工作目录，是执行涉及修改`lowerdir`执行`copy_up`操作的中转层。例如，`upperdir`中不存在，需要从`lowerdir`中进行复制，或者两者中存在重名文件或目录，而merged中将显示`upperdir`中的文件。这部分由Overlay2管理，暂时不需要太关心，todo:以后有需求再研究

```shell
$  mount | grep overlay
overlay on /var/lib/docker/overlay2/d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2/merged type overlay (rw,relatime,seclabel,lowerdir=/var/lib/docker/overlay2/l/LCYEOG2IWKZXROCQRCGIWDFMPT:/var/lib/docker/overlay2/l/7IAFM5I7VDIWABLIA75JJOFAWD:/var/lib/docker/overlay2/l/VQIRWQRQYMM466M3CPDWQAXXN5:/var/lib/docker/overlay2/l/R2UMGPEDCS6PDNKLVP74J7RQZC:/var/lib/docker/overlay2/l/TPYKCH6JDXLP3PPJSJM7YBFI4F:/var/lib/docker/overlay2/l/6ODHHOP2BBFKHXH5JEJ3KROTA4:/var/lib/docker/overlay2/l/ZL47AMV4RSSSBV2TCLVRLWRDDO,upperdir=/var/lib/docker/overlay2/d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2/diff,workdir=/var/lib/docker/overlay2/d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2/work)
```

df 也可以看到一些信息。值得注意的是，使用有些存储驱动时，各个容器分别使用不同的存储空间，比如devicemapper；而有些则所有容器使用相同的存储空间，比如Overlay2，并且默认使用根目录上所挂载的存储设备，所以这里可以看到它们有相同的数据

```shell
$ df -h
文件系统                               容量  已用  可用 已用% 挂载点
/dev/mapper/cl_miwifi--ra69--srv-root   17G  7.4G  9.7G   44% /
overlay                                 17G  7.4G  9.7G   44% /var/lib/docker/overlay2/d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2/merged
...
```



#### 容器层提交为镜像层

进入容器创建一个文件进行实验

```shell
$ docker exec -it e57f2c454839 bash

root@e57f2c454839:/$ touch aaa
```

可以发现该文件是被写入可读可写层`d45fb.../diff`，当然`d45fb.../merged`中也可以看到

```shell
$ ls d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2/diff/
aaa  etc  run  var
$ ls d45fb20d2c3df9371b306afb90743a617315d78134d4bf2549c45686db3817d2/merged/
aaa  boot  docker-entrypoint.d   etc   lib    media  opt   root  sbin  sys  usr
bin  dev   docker-entrypoint.sh  home  lib64  mnt    proc  run   srv   tmp  var
```

当然也可以将容器可读可写层提交，生成为一个新的镜像只读层。`ccdb3...`即提交后生成的镜像只读层，可以在其中看到之前创建的`aaa`文件

```shell
$ docker commit e57f2c454839
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED         SIZE
<none>       <none>    98f672b21677   5 seconds ago   133MB
nginx        latest    4cdc5dd7eaad   10 days ago     133MB

$ ls ccdb39815beba5c418273fbc3ca1d09f3329c783b953e628f10e0314b61e09cb/diff/
aaa  etc  run  var
```



## Overlay

OverlayFS 在单个 Linux 主机上分层两个目录，并将它们呈现为单个目录。这些目录称为*层*，统一过程称为*联合挂载*。OverlayFS 指的是下层目录 as`lowerdir`和上层目录 a `upperdir`。统一视图通过其自己的名为`merged`.



## devicemapper

使用有些存储驱动时，各个容器分别使用不同的存储空间，比如devicemapper；而有些则所有容器使用相同的存储空间，比如Overlay2