



+++

title = "Linux.process"
description = "Linux.process"
tags = ["it","sys"]

+++

# Linux.network

## 网卡

查看

```shell
ip addr #查看网卡及相关信息。简写，等价于ip addr ls、ip addr show
ip addr ls ens33 #查看指定网卡。不能省略ls或show
ip addr add 10.0.0.1/24 dev ens33 #设置ip

ip link #查看网卡物理连接情况。ip link ls、ip link show等都是一样的
ip link ens33 #查看指定网卡
ip link set dev ens33 up #网卡启动
ip link set dev ens33 down #网卡停止
```

修改

- 网卡接口命名修改：默认为`ens33`或`enp0s3`之类的。没有特殊需求默认即可
  - `vim /etc/default/grub`编辑grub，在`GRUB_CMDLINE_LINUX`参数内容的末尾`rhgb quiet`的后面添加` biosdevname=0 net.ifnames=0`两项，网卡命名受这两个参数影响（如下表）
  - `grub2-mkconfig -o /boot/grub2/grub.cfg`更新到真正的启动配置文件
  - `reboot`重启

|       | biosdevname | net.ifnames | 网卡名           |
| ----- | ----------- | ----------- | ---------------- |
| 默认  | 0           | 1           | ens33或enp0s3... |
| 组合1 | 1           | 0           | em1              |
| 组合2 | 0           | 0           | eth0             |



## 网关

```shell
ip route #查看网关信息。ip route ls、ip route show等都是一样的
ip route | column -t #格式化一下

#设置路由
ip route add default gw <网关ip> #设置默认路由
ip route add -host<指定ip> gw<网关ip> #为目的地址设置指定网关
ip route add -net<指定网段> netmask <子网掩码> gw<网关ip> #为目的网段设置指定网关
ip route add 192.168.56.0/24 via 192.168.56.2 dev ens33 #设置静态路由

#删除路由
ip route del 192.168.56.0/24 #删除静态路由
```



## 网络故障排除命令

- 从上到下逐步分析，基本能够解决大部分网络问题了
- ping www.baidu.com：到目标主机是否畅通
- traceroute www.baidu.com：追踪路由。centos7没有原装
- mtr www.baidu.com：检查到目标主机之间是否有数据丢失。centos7没有原装
- nslookup www.baidu.com：搜索域名的ip。centos7没有原装
- telnet www.baidu.com：检查端口连接状态，键入^]然后quit退出telnet。centos7没有原装
- tcpdump -i any -n host 10.0.0.1 and port 80 -w /tmp/filename：更细致的分析数据包，抓包工具，后面详细讲。centos7没有原装。centos7没有原装
- netstat -ntpl：查看本机服务监听地址。n显示ip地址而不是域名，t以tcp截取显示的内容（udp等就不显示了），p显示端口对应的进程，l即listen表示处于监听状态的服务。centos7没有原装，通过ss -ntpl代替
- ss -ntpl：查看本机服务监听地址。n显示ip地址而不是域名，t以tcp截取显示的内容（udp等就不显示了），p显示端口对应的进程，l即listen表示处于监听状态的服务

## 网络服务管理

- 前面的很多配置都是临时的，网卡重启后就重置了，通过配置文件修改才能实现持久配置

- 网络服务管理程序分为两种，分别为SysV和systemd（centos7中新的），最好不要同时使用两套网络管理系统，可以选择关闭其一

  - SysV
    - service network start|stop|restart|status：service的操作
    - chkconfig -list network：查看SysV服务在不同运行级别中的启用状况
    - `chkconfig --level 2345 network <on|off>`：打开或关闭运行级别为2、3、4、5的SysV服务。这样就将网络管理都交给systemd的NetworkManager了

  - systemd的NetworkManager额外功能在于：比如插入网线（或者连接无线）可以识别网卡激活状态自动进行一些网络激活，对个人电脑来说颇有用处，对服务器来说稍显鸡肋。如果都是新写一些脚本，可以使用systemd；如果有一些SysV的脚本需要延用，为了方便可以继续使用SysV
    - systemctl list-unit-files NetworkManager.service：查看是否开启
    - systemctl start|stop|restart NetworkManger.service：启动、停止、重启NetworkManger
    - systemctl enable|disable NetworkManger：启用、禁用NetworkManger


## 网络配置相关文件

- /etc/sysconfig/network-scripts/ifcfg-ens33（根据网卡名有变）

  - BOOTPROTO="dhcp"：遵循dhcp协议的动态ip，可以改为"none"表示静态ip

  - ONBOOT="yes"：是否开机启动，"no"时则需要手动启动网卡

  - 静态内容

    ```shell
    TYPE=Ethernet
    UUID=045d35e8-49bc-4865-b0
    NAME=eth0
    DEVICE=eth0
    ONBOOT=yes
    B0OTPROTO=none #静态
    IPADDR=10.211.55.3 #地址
    NETMASK=255.255.255.0 #子网掩码
    GATEWAY=10.211.55.1 #网关
    DNS1=114.114.114.114 #nds，可以有3个，NDS1、NDS2、NDS3
    ```

  - service network restart或systemctl restart NetworkManger.service重启网卡即可生效

- 主机

  - `hostname`查看

  - `hostname 临时主机名`

  - `hostname set-hostname 永久主机名`

  - hosts配置文件：/etc/hosts。因为有些服务绑定的可能是主机名，所以记得在hosts文件的最后面中配置映射。否则可能启动时会卡住某些服务直到超时，会很慢

    ```shell
    127.0.0.1 主机名如yuanya.tianchi
    ```

  - reboot重启主机以验证

## 端口

### 管理

新增/删除 端口 需要重启防火墙服务

```sh
#centos7启动防火墙
systemctl start firewalld.service
#centos7停止防火墙/关闭防火墙
systemctl stop firewalld.service
#centos7重启防火墙
systemctl restart firewalld.service
 
#设置开机启用防火墙
systemctl enable firewalld.service
#设置开机不启动防火墙
systemctl disable firewalld.service

#centos7查看防火墙所有信息
firewall-cmd --list-all
#centos7查看防火墙开放的端口信息
firewall-cmd --list-ports
# 重启防火墙
firewall-cmd --reload
```

```sh
# 新增开放一个端口号
firewall-cmd --zone=public --add-port=80/tcp --permanent
#说明:
#–zone #作用域
#–add-port=80/tcp #添加端口，格式为：端口/通讯协议
#–permanent 永久生效，没有此参数重启后失效
 
#多个端口:
firewall-cmd --zone=public --add-port=80-90/tcp --permanent

#查看本机已经启用的监听端口（即运行中的，开放不代表运行）
ss -ant

#删除
firewall-cmd --zone=public --remove-port=80/tcp --permanent
```

### 查看

#### netstat

- -t (tcp) 仅显示tcp相关选项
- -u (udp)仅显示udp相关选项
- -n 拒绝显示别名，能显示数字的全部转化为数字
- -l 仅列出在Listen(监听)的服务状态
- -p 显示建立相关链接的程序名

```shell
#查看端口占用
netstat -tunlp | grep 端口号
#查看当前所有tcp端口
netstat -ntlp
#查看所有80端口使用情况
netstat -ntulp | grep 80
#查看所有3306端口使用情况
netstat -ntulp | grep 3306
```

#### lsof

```sh
# 更多详见Linux.process
lsof -i:端口号
```



## 设置静态ip

```sh
$ vim /etc/sysconfig/network-scripts/ifcfg-enp0s3
# 动态dhcp修改为静态ip
BOOTPROTO="static"
# 添加ip设置
IPADDR=192.168.31.100
GATEWAY=192.168.31.1
NETMASK=255.255.255.0
DNS1=192.168.31.1
# 重启网络
$ systemctl restart network
```



## 代理

### 全局

在`~/.bashrc`或`/etc/profile`写入如下内容

```shell
export http_proxy="http://proxy_ip:proxy_port"
export https_proxy="https://proxy_ip:proxy_port"
export ftp_proxy={http_proxy}
```

```shell
source ~/.bashrc
```



### apt

```shell
# 查看代理
sudo env | grep proxy
```

`/etc/apt/apt.conf.d/proxy.conf`，没有密码就无需`user:password@`

```conf
Acquire::http::Proxy "http://user:password@proxy_ip:proxy_port";
Acquire::https::Proxy "https://user:password@proxy_ip:proxy_port";
```

