+++

title = "git"
description = "git"
tags = ["it", "versioncontrol"]

+++



# git

- 192.30.253.113 [github.com](http://github.com)：修改host，提高GitHub的push和pull速度
- 命令行符号
  - [ ]：可写可不写
  - <>：必须写且需要用你自己的内容替换
  - { }：必须在其中做出选择(选项之间以 | 隔开)
- 结构
  - 工作区
  - 暂存区
  - 本地库

https://www.yiibai.com/git



## api

###### clone

- 克隆项目。一般建议使用ssh，https有可能出一些不知所以的问题

```shell
#ssh
git clone git@github.com:YuanyaTianchi/yuanyatianchi.git
#https
git clone https://github.com/YuanyaTianchi/yuanyatianchi.git
```



###### remote

- 远程库操作

```shell
#查看远程库名字
git remote
#查看远程仓库详细地址
git remote -v
#
```

- git remote add [<options>] <repositoryName> <repositoryUrl>：创建repository
    - repositoryName：一般命名为origin
- git remote remove <repositoryName>：删除远程的仓库的所有跟踪分支和配置设置
- git remote rename <oldName> <newName>：重命名远程仓库在本地的简称
- git remote show <repositoryName>：查看某个远程仓库的详细信息
- 
- git pull <--rebase> <repositoryName> <branchName> <--allow-unrelated-histories>：获取远程仓库项目文件
    - --allow-unrelated-histories：可选参数，可以合并两个独立启动仓库的历史
- 提交到本地仓库后再推送到远程仓库
- git push <--set-upstream> <repositoryName> <branchName>：推送到远程仓库
- git remote rm：删除源(origin)

###### pull

- `git pull [option]... <远程主机名> [branch-name]:[local-branch-name]`
  - -q：

```shell
git pull origin remoteBranch:localBranch #获取远程origin上的分支branch1，并合并到本地的分支branch2
```

###### push



###### revert

https://juejin.im/post/6844903614767448072

- git revert [将被回退的版本]

```shell

```

###### reset

- git reset [要回退到的版本]

###### log

- git log

```shell
git log --graph --abbrev-commit --pretty=oneline #图形化，hash值简化，单行
```



###### merge

我的需求在`feature/tianchi/xxx`分支上写完，要合并到`develop`里面，需要先选择到开发分支再去merge我的分支，会是`Merge branch 'feature/tianchi/xxx' into 'develop'`，千万**不要**在我的分支上去merge开发分支，会变成`Merge branch 'develop' into 'feature/tianchi/xxx'`

```shell
#正确操作如下
git pull origin develop:develop
git checkout develop
git merge feature/tianchi/xxx
```





###### rebase



- rebase
  - -r：--rebase-merges简写。

```shell
git rebase -r
```



## cfg

仓库：每个仓库的Git配置文件都放在`.git/config`文件中

全局：家目录下.gitconfig文件

###### 别名

```
[alias]
    ch = checkout
    co = commit
    br = branch
    st = status
    lg = log --graph --abbrev-commit --pretty=oneline
```





## 本地库操作

### 本地库初始化

1. 在cmd中进到项目目录下
2. git init：本地库初始化。生成一个.git目录，该目录中存放的是本地库相关的子目录和文件
3. 设置签名：username和email用于区分不同开发人员的身份。这里设置的签名和登录远程库(代码托管中心)的账号密码没有任何关系。默认项目级别(仓库级别)仅在当前本地库范围内生效，项目级别信息保存在.git/config下。系统用户级别：指定--global，系统用户级别信息保存在系统用户家目录下的.config文件中。项目级别优先于系统用户级别。至少设置一个
   1. git config [--global] user.name <username>：设置用户名
   2. git config [--global] user.email <email>：设置Email地址
   3. cat .git/cogfig：查看本地库配置文件

### 基本操作

- git help <命令>：查看该命令文档

### 文件操作

- git status：查看状态。
  - on branch master表示在主分支上，no commits yet表示无提交内容
  - 红色文件表示未添加到暂存区中，绿色表示已添加到暂存区中
- git add <filename>：将文件从工作区添加到暂存区。unstage表示从暂存区中移除
- git rm <filename>：从暂存区删除
- git commit [-m <description>] [-a] <filename>：将文件从暂存区中提交到本地库，添加后会进入vim，写本次提交描述内容
  - -m：无需编辑vim，直接在后面写入描述内容，；
  - -a：无需git add操作，直接添加&提交，不过就不存在暂存区的撤销操作时间了
  - 结果内容：
    - root-commit：表示根提交(第一次提交)
    - 数字编号：暂时粗略认为是本次提交的版本号

### 版本移动

- git log [--pretty={oneline}] [--oneline]：查看日志版本，只显示当前及以前版本。这里有2个版本的记录，这里hash值表示该次提交的索引，HEAD是指向当前版本的指针
  - --pretty=oneline将每个版本只以一行显示
  - --oneline不仅一行显示还只显示部分hash值
- git reflog：查看日志，显示前后所有版本以及版本移动
  - HEAD@{n}表示移动到对应版本指针需要移动n步
- git reset --{hard/mixed/soft} HEAD <headHash>：索引移动(推荐)
  - --mixed在本地枯移动HEAD指针，重置暂存区，默认策略
  - --hard在本地库移动HEAD指针，重置暂存区，重置工作区（会删除文件），
  - --soft仅在本地库移动HEAD指针
- git reset  --hard HEAD^^^：^移动，只能后退，3个^即表示后退3个版本
- git reset  --hard HEAD~<n>：~移动，只能后退，n即表示后退n个版本
- git checkout <versionNumber> <filename>：选择某个版本的文件到工作区
- git diff [HEAD] [<filename>]：在修改工作区文件之后
  - 无HEAD：表示工作区与暂存区的该文件比较，显示文件变化
  - 有HEAD：表示工作区与本地库的该文件比较，显示文件变化
  - 无<filename>：将比较当前工作区中的所有文件

### 分支管理

分支：在版本控制过程中，使用多条线同时推进多个任务。同时并行推进多个功能开发，提高开发效率。各个分支在开发过程中，如果一个分支开发失败，不会对其他分支有任何影响，删除重新开始即可

- git branch [-v] [-a]：
    - -v：显示版本号
    - -a：包括远程分支
- git branch <branchName>：创建分支
- git branch -D <branchName>：删除分支
- git checkout <branchName>：切换分支
- git merge <branchName>：合并某分支，将当前所在分支与某分支合并。
  - 分支合并冲突：当两个分支都修改了同一个文件中同一行的内容，合并时取舍哪个分支的该处内容git是无法判断的，只需要vim手动编辑后再添加提交(提交不能带文件名)即可

### clean

```sh
git clean -f
git clean -d -fx #强制删除Untracked files
```



## 合并工程

- 假如有master、test、develop分支，现在有一个需求从master切出来一个demand，需求子任务从demand切出来一个demand/child：`git pull origin demand:demand`，`git ch demand`，`git ch -b demand/child`
  - 任务开发完后，需要merge demand/child into develop。切到任务分支`git ch demand/child`，拉取develop分支`git pull origin develop:develop`，会有很多改变，会因版本而merge develop into demand/child的相关提示，中止merge`git merge --abort`，不能让merge develop into demand/child发生，这反了，切到dev`git ch develop`，merge demand/child into develop`git merge demand/child`，编译检查`go build xxx`，提交之前再检查版本`git lg -5`是否是merge demand/child into develop，推送`git push origin develop:develop`
  - develop环境调试完，需要merge demand/child into demand。切到任务分支`git ch demand/child`，直接拉取远程demand并合并到demand/child`git pull origin demand:demand/child`，因为demand本来就是demand/child的源，所以直接合并之后要再合并回去`git push origin demand/child:demand`，不会有什么影响，也能将其他人的更改合并进来，如果develop也这么做的话，会将demand/child不需要的develop的内容合并进来，那就只有回退版本再重新操作了