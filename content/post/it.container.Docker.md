

+++

title = "Docker"
description = "Docker"
tags = ["it", "container"]

+++



# Docker

官网：https://www.docker.com/ 

仓库：https://hub.docker.com/

文档：https://docs.docker.com/reference/

命令：https://docs.docker.com/engine/reference/commandline/docker/

## 安装

```shell
$ uname -r #查看centOS系统内核版本，Docker要求内核版本高于3.10
$ yum update #升级软件包及内核（选做）
```

使用存储库安装：https://docs.docker.com/engine/install/centos/

```sh
# 设置稳定存储库（配置docker源）
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo

# 查看 yum 源上有哪些 docker-ce 版本
yum list docker-ce --showduplicates|sort -r

# 安装最新版本的docker-ce
yum install docker-ce -y
#如果是centos8，默认使用 podman 代替 docker，会提示冲突，根据提示使用 --allowerasing 替换冲突的软件包
yum install docker-ce -y --allowerasing

#启动
systemctl enable docker #设置docker为开机启动
systemctl start docker #启动docker
```





信息

```shell
$ doAcker --help
$ docker version #版本信息
$ docker info #相信信息
```



## 镜像

### 镜像加速配置

从https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors 获取个人专属的阿里docker镜像加速地址

/etc/docker/daemon.json

```json
{
  "registry-mirrors": ["https://ohm5orzk.mirror.aliyuncs.com"]
}
```

然后重启docker即可

```shell
systemctl restart docker
```

### 镜像管理

###### 镜像搜索



###### 镜像拉取



###### 镜像导出

```shell
# 一般导出命名后缀为.tar
docker save arm64v8/centos:latest -o ./centos.arm64.tar
```



###### 镜像导入

```shell
docker load < centos.arm64.tar
```



###### 镜像构建

https://www.runoob.com/docker/docker-build-command.html

```shell
# 默认使用当前目录下的 Dockerfile 构建镜像
$ docker build -t yuanya:1.0
```

| option  | description                                                  |
| ------- | ------------------------------------------------------------ |
| -f      | 指定要使用的Dockerfile                                       |
| -t;-tag | 镜像的名字及标签，通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签。 |



###### 提交容器为镜像

```shell
docker commit 261314c94305 imagexxx
```

| option | description                    |
| ------ | ------------------------------ |
| -a     | 提交的镜像作者                 |
| -c     | 使用Dockerfile指令来创建镜像； |
| -m     | 提交时的说明文字；             |
| -p     | 在commit时，将容器暂停。       |



### 跨平台构建

qemu-user-static：https://github.com/multiarch/qemu-user-static

https://juejin.cn/post/6844903605355577358

https://zhuanlan.zhihu.com/p/106054643

在x86机器上模拟arm架构指令来构建arm架构的镜像，使用 [multiarch/qemu-user-static](https://github.com/multiarch/qemu-user-static) 来实现在x86主机上模拟arm环境

**multiarch/qemu-user-static** 是通过 QEMU 和 binfmt_misc 启用不同多架构容器的执行。以下是 Docker 的示例。



通过 qemu-user-static （该程序使用内核 binfmt_misc 文件系统功能）可以模拟各种环境

```shell
# 我的环境是 centos8
$ uname -mrs
Linux 4.18.0-305.7.1.el8_4.x86_64 x86_64

# 拉取一个 arm 的 centos 镜像
$ docker pull arm64v8/centos

# 在 x86 上执行一个 arm centos 镜像时，会因无法解析 arm 指令而报错
$ docker run --rm -t arm64v8/centos uname -m
standard_init_linux.go:211: exec user process caused "exec format error"

# 此时执行 qemu-user-static 镜像，将为我们设置模拟 arm 环境
$ docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
Setting /usr/bin/qemu-alpha-static as binfmt interpreter for alpha
Setting /usr/bin/qemu-arm-static as binfmt interpreter for arm
Setting /usr/bin/qemu-armeb-static as binfmt interpreter for armeb
Setting /usr/bin/qemu-sparc-static as binfmt interpreter for sparc
Setting /usr/bin/qemu-sparc32plus-static as binfmt interpreter for sparc32plus
Setting /usr/bin/qemu-sparc64-static as binfmt interpreter for sparc64
Setting /usr/bin/qemu-ppc-static as binfmt interpreter for ppc
Setting /usr/bin/qemu-ppc64-static as binfmt interpreter for ppc64
Setting /usr/bin/qemu-ppc64le-static as binfmt interpreter for ppc64le
Setting /usr/bin/qemu-m68k-static as binfmt interpreter for m68k
Setting /usr/bin/qemu-mips-static as binfmt interpreter for mips
Setting /usr/bin/qemu-mipsel-static as binfmt interpreter for mipsel
Setting /usr/bin/qemu-mipsn32-static as binfmt interpreter for mipsn32
Setting /usr/bin/qemu-mipsn32el-static as binfmt interpreter for mipsn32el
Setting /usr/bin/qemu-mips64-static as binfmt interpreter for mips64
Setting /usr/bin/qemu-mips64el-static as binfmt interpreter for mips64el
Setting /usr/bin/qemu-sh4-static as binfmt interpreter for sh4
Setting /usr/bin/qemu-sh4eb-static as binfmt interpreter for sh4eb
Setting /usr/bin/qemu-s390x-static as binfmt interpreter for s390x
Setting /usr/bin/qemu-aarch64-static as binfmt interpreter for aarch64
Setting /usr/bin/qemu-aarch64_be-static as binfmt interpreter for aarch64_be
Setting /usr/bin/qemu-hppa-static as binfmt interpreter for hppa
Setting /usr/bin/qemu-riscv32-static as binfmt interpreter for riscv32
Setting /usr/bin/qemu-riscv64-static as binfmt interpreter for riscv64
Setting /usr/bin/qemu-xtensa-static as binfmt interpreter for xtensa
Setting /usr/bin/qemu-xtensaeb-static as binfmt interpreter for xtensaeb
Setting /usr/bin/qemu-microblaze-static as binfmt interpreter for microblaze
Setting /usr/bin/qemu-microblazeel-static as binfmt interpreter for microblazeel
Setting /usr/bin/qemu-or1k-static as binfmt interpreter for or1k

# centos7会出现报错 sh: write error: Invalid argument，因为使用 -p yes 选项至少需要内核版本 4.10 上的 binfmt_misc，https://github.com/multiarch/qemu-user-static/issues/100。可以去掉 -p yes 选项，之后还有一堆其它问题，直接放弃，换 centos8 了
$ docker run --rm --privileged multiarch/qemu-user-static --reset

# 之后就可以执行这个 arm centos 镜像了，因为 qemu-user-static将arm架构的指令解释成x86架构的指令执行
$ docker run --rm -t arm64v8/centos uname -m
aarch64

# 这里将 hello.arm 所在目录挂载进来执行
docker run --rm -t -v /it/go/gopath/src/yuanyatianchi.io/hello/bin:/usr/bin/qemu-aarch64-static arm64v8/centos ls /go/bin
```



现在 arm64v8/centos 已经可以 run 了，将其作为基础镜像编写 Dockerfile ，build 后 run 起来即可，这里用一个 hello 代码

```go
package main

import "fmt"

func main() {
	fmt.Println("hello")
}
```

```dockerfile
FROM arm64v8/centos

COPY ./bin/hello /usr/bin
```

```makefile
.PHONY: build-hello build-image export-image clean

# 构建程序
build-hello: ./cmd/hello.go
	CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -o ./bin/hello ./cmd/hello.go

# 构建镜像
build-image: build-hello
	docker build . -t hello:v1.0

# 导出镜像
export-image: build-image
	docker save hello:v1.0 -o ./bin/hello.tar

clean:
	rm -rf ./bin
```

```shell
# 构建并导出镜像
$ make export-image
CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -o ./bin/hello ./cmd/hello.go
docker build . -t hello:v1.0
Sending build context to Docker daemon  7.443MB
Step 1/2 : FROM arm64v8/centos
 ---> a0477e85b8ae
Step 2/2 : COPY ./bin/hello /usr/bin
 ---> 8a0e7d42e2f7
Successfully built 8a0e7d42e2f7
Successfully tagged hello:v1.0
docker save hello:v1.0 -o ./bin/hello.tar

# 可以看到镜像包
$ ls ./bin/
hello  hello.tar

# 验证一下执行 hello
$ docker run --rm -t hello:v1.0 hello
hello
```



## 容器

加载镜像运行容器

```shell
docker run -it --name 容器名称 镜像名称 /bin.bash
```



启动



停止



## 网络



### 代理

https://note.qidong.name/2020/05/docker-proxy/

有时因为网络原因，比如公司NAT，或其它啥的，需要使用代理。 Docker的代理配置，略显复杂，因为有三种场景。 但基本原理都是一致的，都是利用Linux的`http_proxy`等环境变量。



#### dockerd

https://docs.docker.com/config/daemon/systemd/

执行`docker pull`时，是由守护进程`dockerd`来执行。 因此，代理需要配在`dockerd`的环境中。 而这个环境，则是受`systemd`所管控，因此实际是`systemd`的配置。

```shell
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/proxy.conf
```

`proxy.conf`文件（可以是任意`*.conf`的形式）内容如下

```ini
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:8080/"
Environment="HTTPS_PROXY=https://proxy.example.com:8080/"
Environment="NO_PROXY=127.0.0.1,私有仓库ip,example.com"
```

其中，`proxy.example.com:8080`要换成可用的免密代理。 通常使用`cntlm`在本机自建免密代理，去对接公司的代理。 可参考《[Linux下安装配置Cntlm代理](https://note.qidong.name/2018/11/cntlm-proxy/)》。



重启。`dockerd`代理的修改比较特殊，它实际上是改`systemd`的配置，因此需要重载`systemd`并重启`dockerd`才能生效。

```shell
sudo systemctl daemon-reload
sudo systemctl restart docker
```



#### container

https://docs.docker.com/network/proxy/

在容器运行阶段，如果需要代理上网，则需要配置`~/.docker/config.json`。 以下配置，只在Docker 17.07及以上版本生效。

```json
{
  "proxies":
  {
    "default":
    {
      "httpProxy": "http://proxy.example.com:8080",
      "httpsProxy": "http://proxy.example.com:8080",
      "noProxy": "localhost,127.0.0.1,.example.com"
    }
  }
}
```

这个是用户级的配置，除了`proxies`，`docker login`等相关信息也会在其中。 而且还可以配置信息展示的格式、插件参数等。

此外，容器的网络代理，也可以直接在其运行时通过`-e`注入`http_proxy`等环境变量。 这两种方法分别适合不同场景。 `config.json`非常方便，默认在所有配置修改后启动的容器生效，适合个人开发环境。 在CI/CD的自动构建环境、或者实际上线运行的环境中，这种方法就不太合适，用`-e`注入这种显式配置会更好，减轻对构建、部署环境的依赖。 当然，在这些环境中，最好用良好的设计避免配置代理上网。



#### docker build

https://docs.docker.com/engine/reference/commandline/cli/#automatic-proxy-configuration-for-containers

虽然`docker build`的本质，也是启动一个容器，但是环境会略有不同，用户级配置无效。 在构建时，需要注入`http_proxy`等参数。

```sh
docker build . \
    --build-arg "HTTP_PROXY=http://proxy.example.com:8080/" \
    --build-arg "HTTPS_PROXY=http://proxy.example.com:8080/" \
    --build-arg "NO_PROXY=localhost,127.0.0.1,.example.com" \
    -t your/image:tag
```

**注意**：无论是`docker run`还是`docker build`，默认是网络隔绝的。 如果代理使用的是`localhost:3128`这类，则会无效。 这类仅限本地的代理，必须加上`--network host`才能正常使用。 而一般则需要配置代理的外部IP，而且代理本身要开启gateway模式。



### 修改Docker0默认网段

https://blog.51cto.com/lisea/1940023

Docker 服务启动后默认会创建一个 docker0 网桥（其上有一个 docker0 内部接口），它在内核层连通了其他的物理或虚拟网卡，这就将所有容器和本地主机都放到同一个物理网络。Docker 默认指定了 docker0 接口 的 IP 地址和子网掩码，让主机和容器之间可以通过网桥相互通信，它还给出了 MTU（接口允许接收的最大传输单元），通常是 1500 Bytes，或宿主主机网络路由上支持的默认值。这些值都可以在服务启动的时候进行配置。

```shell
# 查看
$ ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.17.0.1  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:61:ff:7b:91  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

修改文件 `/etc/docker/daemon.json` 添加内容 `"bip": "ip/netmask"` 即可。注意切勿与宿主机同网段

```json
{
  "bip": "192.168.100.1/24"
}
```

```shell
# 重启即可
$ systemctl restart docker
```



## client

命令：

### 镜像

docker中一个centos镜像大小200m不到，docker相当于隔离了进程，

```shell
$ docker images #本地镜像列表
$ docker search <关键字> #检索镜像
$ docker pull <镜像名>[:<tag>] #拉取镜像。tag指定版本，缺省则为latest
$ docker inspect <镜像名>#查看镜像详细信息
$ docker rmi <镜像id> #删除指定镜像

docker network ps
docker network rm <网络名> #注意几个默认的不要删除
```

### 容器

- 镜像是一种轻量级、可执行的独立软件包，用来打包软件运行环境和基于运行环境开发的软件，它包含运行某个软件所需的所有内容，包括代码、运行时、库、环境变量和配置文件。
- UnionFS (联合文件系统) : Union文件系统(UnionFS) 是一种分层、 轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是Docker镜像的基础。镜像可以通过分层来进行继承，基于基础镜像(没有父镜像)，可以制作各种具体的应用镜像。
  - 特性：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录
- Docker镜像加载原理：docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS.
  - bootfs(boot file system)主要包含bootloader和kernel，bootloader主要是引导加载kernel，Linux刚启动时会加载bootfs文件系统，在Docker镜像的最底层是bootfs。这一层与我们典型的Linux/Unix系统是一样的，包含boot加载器和内核。当boot加载完成之后整个内核就都在内存中了，此时内存的使用权己由bootfs转交给内核，此时系统也会卸载bootfs。
  - rootfs (root file system)，在bootfs之 上。 包含的就是典型Linux系统中的/dev, /proc, /bin, /etc等标准目录和文件。rootfs 就是各种不同的操作系统发行版，比如Ubuntu，Centos等 等。
  - 对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了,因为底层直接用Host的kernel,自己只需要提供rootfs就行了。由此可见对于不同的linux发行版, bootfs基本是一致的, rootfs会有差别,因此不同的发行版可以公用bootfs。
- 分层的镜像：以我们的pull为例，在下载的过程中我们可以看到docker的镜像好像是在一层一层的在下载很多镜像
- 最大的一个好处就是：共享资源
  - 比如:有多个镜像都从相同的base镜像构建而来，那么宿主机只需在磁盘上保存一-份base镜像，同时内存中也只需加载一份base镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。
- Docker镜像都是只读的。当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”，“容 器层”之下的都叫"镜像层

```shell
$ docker ps #容器列表。-a表示所有包括未运行的容器
$ docker logs <container-name | container-id> #查看指定容器日志
$ docker run [--name <container-name>] [-d|-it] [-p <container-port>:<port>] [-e <key>:<value>] <image-id>  #运行指定容器。--name为容器取名；-d守护式运行（即后台运行），-it交互式运行（即进入容器交互）；-p指定将容器内的端口映射到实体机的端口；-e配置环境参数
$ docker start <container-name | container-id> #启动
$ docker restart <container-name | container-id> #重启
$ docker stop <container-name | container-id> #停止
$ docker stop <container-name | container-id> #强制停止
$ docker rm <container-id> #删除指定容器
$ docker rm -f $(docker ps -a -q) #条件批量删除
$ docker ps-a-q | xargs docker rm #条件批量删除

$ docker top <container-id>
$ docker commit <container-id> <要创建的目标镜像名:[标签名]>  -a="<作者>" -m="<提交的描述信息>"
 #提交容器副本使之成为一个新的镜像，可以docker ps -a看到
```

停止并删除所有容器

```sh
docker stop $(docker ps -q)
docker rm $(docker ps -aq)
```



执行容器

```shell
$ docker exec -it <container-id> <bashshell> #执行伪终端。-it表示-i -t，将为容器分配伪终端，可以进行交互
$ docker exec -it <container-id> /bin/bash
$ docker exec -it <container-id> bash
$ docker exec -it <container-id> ls /
$ docker attach <container-id> #重新进入
$ exit #退出容器交互并停止容器
ctrl+q+p #退出容器交互但容器不停止

$ docker cp <container-id>:/xxx/xxx.log /app/temp #拷贝容器中的文件到本地宿主主机
```



```cmd
docker exec -it mysql8 bash #进入容器名为mysql8的容器
mysql -uroot -p123456 #进入mysql8后，可以使用mysql命令行，登录mysql

grant all privileges on *.* to 'root'@'%'; #添加权限，%表示所有ip可以登入，%可以替换为具体ip以给某ip开放root登入权限

#也可以这样到表中修改权限
use mysql; #进到mysql库
select host, user, plugin from user; #查看user表，可以看到root的host是localhost，只允许本地登入
update user set host='%' where user='root' #修改权限，%表示所有ip可以登入，%可以替换为具体ip以给某ip开放root登入权限

flush privileges; #重新加载权限
```



## 容器数据卷

- 持久化
  - 将运用与运行的环境打包形成容器运行，运行可以伴随着容器，但是我们对数据的要求希望是持久化的
  - 容器之间希望有可能共享数据
  - Docker容器产生的数据，如果不通过docker commit生成新的镜像，使得数据做为镜像的一 部分保存下来，那么当容器删除后，数据自然也就没有了。
  - 为了能保存数据在docker中我们使用卷。
- 卷就是目录或文件，存在于-一个或多个容器中，由docker挂 载到容器，但不属于联合文件系统，因此能够绕过Union File System提供一些用于持续存储或共享数据的特性:卷的设计目的就是数据的持久化，完全独立于容器的生存周期，因此Docker不会在容器删除时删除其挂载的数据卷
  - 1:数据卷可在容器之间共享或重用数据
  - 2: 卷中的更改可以直接生效
  - 3:数据卷中的更改不会包含在镜像的更新中
  - 4:数据卷的生命周期一直持续到没有容器使用它为止

### 容器卷添加

###### 直接命令添加

```shell
$ docker run <指定镜像> -it -v </宿主机绝对路径目录>:</容器内目录>  #-v指定host与container的数据同步目录，如果目录不存在则将创建目录，数据将实现实时同步，两边都可读写。-v参数可以多使用，形成一一对应的多对同步目录
$ docker run <指定镜像> -it -v </宿主机绝对路径目录>:</容器内目录:ro> #限制容器目录只读
```

###### DockerFile添加

- 新建/mydocker文件夹并进入
- 在Dockerfile中使用VOLUME指令来给镜像添加一个或多个数据卷
  - VOLUME["/dataVolumeContainer","/dataVolumeContainer2", "/dataVolumeContainer3"]
  - 出于可移植和分享的考虑，用-v主机目录:容器目录这种方法不能够直接在Dockerfile中实现。由于宿主机目录是依赖于特定宿主机的，并不能够保证在所有的宿主机上都存在这样的特定目录。但是会有默认的目录，通过`docker inspect`即可查看到

```dockerfile
#dockerfile，按序执行的脚本，每个FROM的镜像都会作为一层环境被装载
FROM centos
VOLUME [" /dataVolumeContainerA" ," /dataVolumeContainerB" ]
CMD echo "finished, -------- success1"
CMD /bin/bash
```

- docker build生成镜像

  ```shell
  $ docker build -f /mydocker/dockerfile -t yuanya/centos . #-f指定要构建的dockerfile；-t为生成的镜像取名
  $ docker run yuanya/centos -d #如果后面出现容器数据卷只能读的情况，就加上--privileged=true赋予容器扩展的特权
  $ docker inspect yuanya/centos
  ```

###### 数据卷容器

- 命名的容器挂载数据卷，其它容器通过挂载这个(父容器)实现数据共享，挂载数据卷的容器，称之为数据卷容器
- 即容器间传递共享(--volumes-from)

```shell
$ docker run yuanya/centos -it --name dc01 #启动一个01
$ docker run yuanya/centos -it --name dc02 --volumes-from dc01 #将使dc02与dc01共享VOLUME配置的数据卷
$ docker run yuanya/centos -it --name dc03 --volumes-from dc01 #将使dc03与dc01共享VOLUME配置的数据卷
#即完成dc01、dc02、dc03之间相互共享VOLUME配置的数据卷
```



## DockerFile

- Dockerfile是用来构建Docker镜像的构建文件，是由一系列命令和参数构成的脚本

```dockerfile
#相当于docker run 时加 bash
CMD /bin/bash
CMD ["/bin/bash"]
```

dockerfile解析过程

- Dockerfile内容基础知识
  - 每条保留字指令都必须为大写字母且后面要跟随至少一个参数
  - 指令按照从上到下，顺序执行
  - #表示注释
  - 每条指令都会创建一个新的镜像层，并对镜像进行提交
- 保留字指令（关键字）
  - FROM：基础镜像，当前新镜像是基于哪个镜像的
  - MAINTAINER：镜像维护者 的姓名和邮箱地址
  - RUN：容器构建时需要运行的命令
  - EXPOSE：当前容器对外暴露出的端口
  - WORKDIR：指定在创建容器后，进入容器伪终端的默认工作目录
  - ENV：用来在构建镜像过程中设置环境变量
    - 如 ENV MY_ PATH /usr/mytest。这个环境变量可以在后续的任何RUN指令中使用，这就如同在命令前面指定了环境变量前缀一-样；也可以在其它指令中直接使用这些环境变量，比如: WORKDIR $MY_ PATH
  - ADD：copy + 解压缩。将宿主机目录下的文件拷贝进镜像且ADD命令会自动处理URL和解压tar压缩包
  - COPY：类似ADD，拷贝文件和目录到镜像中。将从构建上下文目录中<源路径>的文件/目录复制到新的一层的镜像内的<目标路径>位置
    - shell格式：COPY src des
    - exec格式：COPY ["src", "dest"]
  - VOLUME：容器数据卷， 用于数据保存和持久化工作
  - CMD
    - 指定一个容器启动时要运行的命令。CMD指令的格式和RUN 相似,也是两种格式
      - shell格式：CMD <命令>
      - exec格式：CMD ["可执行文件"，”参数1"， “参数2...]
      - 参数列表格式：CMD ["参数1"，“参数2"...]。在指定了ENTRYPOINT 指令后，用CMD 指定具体的参
        数。
    - Dockerfile中可以有多个CMD指令，但只有最后一个生效，CMD会被docker run之后的参数替换.。
  - ENTRYPOINT
    - 指定一个容器启动时要运行的命令
    - ENTRYPOINT的目的和CMD一样，都是在指定容器启动程序及参数，但是命令是追加的，不会覆盖前面的命令，比如在是 ENTRYPOINT ["可执行文件"，”参数1"]，运行时docker run xxx -i，将变为ENTRYPOINT ["可执行文件"，”参数1"，"-i"]，等于可以追加参数的效果
  - ONBUILD：当构建一个被继承的Dockerfile时运行命令，父镜像在被子继承后父镜像的onbuild被触发

案例

- Base镜像(scratch)：DockerHub中99%的镜像都是通过在base镜像中安装和配置需要的软件构建出来的

  - scratch：an explicitly empty image, especially for building images "FROM scratch"

- centos的镜像很精简，我们通过dockerfile自定义一个centos镜像使其拥有vim、ifconfig等命令及其它自定义内容

  /mydocker/dockerfile01

  ```dockerfile
  from centos
  MAINTAINER yuanya<yuanyatianchi@google.com>
  
  ENV MY_PATH /tmp
  WORKDIR $MY_PATH
  
  RUN yum -y install vim
  RUN yum -y install net-tools
  
  EXPOSE 80
  
  CMD echo $MY_PATH
  CMD echo "success-ok"
  CMD /bin/bash
  ```

  ```shell
  $ docker build -f /mydocker/dockerfile01 -t mycentos:1.0 #构建
  $ docker run - it mycentos:1.0 #运行
  $ docker history <容器id> #查看历史
  ```

  

定制tomcat9

```dockerfile
FROM centos
MAINTAINER zzyy< zzyybs@126. com>
#把宿主机当前上下文的C. txt拷贝到容器/usr/local/路径下
COPY c.txt /usr/local/cincontainer.txt
#把java与tomcat添加到容器中
ADD jdk 8u171- linux -x64. tar .gz /usr/local/
ADD apache tomcat 9.0.8. tar.gz /usr/local/
#安装vim编辑器
RUN yum -y install vim
#设置工作访问时候的WORKDIR路径，登录落脚点
ENV MYPATH /usr/local
WORKDIR $MYPATH
#配置java与tomcat环境变量
ENV JAVA HOME /usr/local/jdk1.8.0_ 171
ENV CLASSPATH $JAVA HOME/lib/dt . jar :$JAVA_ HOME/1ib/ tools. jan
ENV CATALINA HOME /usr/local/apache-tomcat-9.0.8
ENV CATALINA BASE /usr/ local/ apache-tomcat-9.0.8
ENV PATH $PATH:$JAVA_ HOME/bin: $CATAL INA HOME/lib: $CATAL INA HOME/bin
#容器运行时监听的端口
EXPOSE 8080
#启动时运行tomcat
# ENTRYPOINT [" /usr/local/ apache -tomcat 9.0.8/bin/startup.sh" ]
# CMD [" /usr/local/apache -tomcat- 9.0. 8/bin/catalina. sh" ,"run"]
CMD /usr/1ocal/apache-tomcat-9.0.8/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.8/bin/1ogs/catalina.out

```

```shell
docker run -d -p 9080:8080 --name myt9
-V /zzyyuse/mydockerfile/tomcat9/tesf:/usr/local/apache-tomcat-9.0.8/webapps/ test
-V /zzyyuse/mydockerfile/tomcat9/tomcat9logs/:/usr/local/apache-tomcat-9.0.8/logs
--pr ivi leged=true
zzyytomcat9
#设置容器卷，可以使我们直接把服务部署到主机tomcat9/test即可同步到容器tomcat-9.0.8/webapps/test上运行了
```





## 软件

###### MySql

```shell
$ docker pull mysql:5.7
$ docker run mysql:5.7 --name mysql01 -d -p 3306:3306 -e MYSQL_ROOT_PASSWORD=yuanya #设置MYSQL_ROOT_PASSWORD，否则密码随机
```



###### ElesticSearch

```shell
$ docker pull elasticsearch:7.5.2

$ docker run --name elasticsearch -d -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -e "ES_JAVA_OPTS=-Xms256m -Xmx256m" elasticsearch:tag
```

###### Kibana

```shell
$ ocker pull kibana:7.5.2
$ docker run --name kibana --link elasticsearch_CONTAINER_ID:elasticsearch -d -p 5601:5601 kibana:tag
```

###### RabbitMQ

带有management的是带有web控制台的，通过15672端口访问控制台，缺省账号密码guest

```shell
$ docker pull rabbitmq:3.7.24-management 
$ docker run -d -p 5672:5672 -p 15672:15672 rabbitmq:3.7.24-management
```

