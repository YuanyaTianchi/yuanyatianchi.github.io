

+++

title = "Linux.user"
description = "Linux.user"
tags = ["it","sys"]

+++

# Linux.user

## 用户



```shell
useradd user1 #添加新用户
#创建用户后会在/home中创建用户对应的目录，目录名与用户名一致
ls -a /home/yuanya #会有一些隐藏文件
tail -10 /etc/passwd #会有用户相关内容
tail -10 /etc/shaow #会有用户相关内容，密码相关

id #查看当前用户
id root #查看指定用户。可以借此验证是否存在指定用户
#用户会有唯一id，系统是通过id识别不同用户的。root用户uid=0，如果把普通用户的uid改为0，则也会被系统当作root用户对待
#用户有用户组，创建时不指定用户组则属于其同名组，即用户组名与用户名一致，只有该用户一人
groupadd group1 #新建用户组
groupdel group1 #删除用户组
useradd user2 group1 #添加新用户并指定用户组

passwd #为当前用户自己设置密码
passwd yuanya #为指定用户设置密码

userdel yuanya#删除用户，用户的家目录/home/yuanya会被保留，文件所属用户变为数字，只有root用户可以使用，加-r则不会保留家目录。/etc/passwd、/etc/shaow中相关用户信息都将被删除

man usermod #usermod用于修改用户属性
usermod /home/yuanya yuanyatianchi #修改家目录/home/yuanya的名字为yuanyatianchi，相当于搬家
usernod -g group1 yuanya #将指定用户分配到指定用户组

man chage #更改用户密码过期信息

#root用户切换普通用户无需密码，普通用户切换其它用户需要密码
su - user1 #切换用户，"-"表示用户及用户运行环境的完全切换，这里运行环境切换指自动进入到user1的家目录/home/user1，root的家目录是/root
su user1 #不完全切换，仍在之前用户的家目录中，但是已经没有ls读取权限了，需要手动切换user1自己的家目录
```

