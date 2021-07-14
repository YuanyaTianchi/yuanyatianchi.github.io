

+++

title = "container.unionFS"
description = "container.unionFS"
tags = ["it", "container"]

+++



# container.unionFS



## Overlay2

https://docs.docker.com/storage/storagedriver/overlayfs-driver/#how-the-overlay2-driver-works

该`overlay2`驱动程序本身支持多达 128 个较低级别的 OverlayFS 层。此功能可为与层相关的 Docker 命令（例如`docker build`和`docker commit` ）提供更好的性能，并在后备文件系统上消耗更少的 inode

```shell
$ docker info | grep "Storage \| Version"
 Server Version: 20.10.7
 Storage Driver: overlay2
 Cgroup Version: 1
 Kernel Version: 4.18.0-305.7.1.el8_4.x86_64
```

这里拉取一个 nginx，共 6 个 layer。没有使用官方例举的 ubuntu 是因为 latest 版本只有一个 layer 了。

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

使用 overlay2 的存储驱动，镜像文件存储在 `/var/lib/docker/overlay2` 目录下

```shell
$ cd /var/lib/docker/overlay2
```

可以看到 nginx 镜像的 6 个 layer

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

`l`（小写L）目录包含缩短的层标识符作为符号链接，连接到各层的`diff`目录。缩短标识符通过 hash 得到，使用它是为了避免撞上`mount`命令参数的页大小限制，毕竟真实目录名很长

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

diff 目录装有 layer 的内容；link 文件中写有 layer 的缩短标识符；lower 文件写有当前层所依赖的更低层（父层）的缩短标识符

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

fa64f42d6b080a0fa56f682595d442bb39fb17e04ce8811bb3acba36c22ee20f 这个目录中没有 lower，因为它是最低层了，是一个基础操作系统层，从 diff 目录结构也可以看到

```shell
$ ls fa64f42d6b080a0fa56f682595d442bb39fb17e04ce8811bb3acba36c22ee20f/diff/
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
$ cat fa64f42d6b080a0fa56f682595d442bb39fb17e04ce8811bb3acba36c22ee20f/link
ZL47AMV4RSSSBV2TCLVRLWRDDO
```

lower 中的缩短标识符不是无序的，从左到右是从高层到低层

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

todo: 把 nginx 运行起来继续观察



## Overlay

OverlayFS 在单个 Linux 主机上分层两个目录，并将它们呈现为单个目录。这些目录称为*层*，统一过程称为*联合挂载*。OverlayFS 指的是下层目录 as`lowerdir`和上层目录 a `upperdir`。统一视图通过其自己的名为`merged`.

