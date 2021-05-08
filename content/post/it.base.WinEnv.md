

+++

title = "WinEnv"
description = "WinEnv"
tags = ["it", "base"]

+++



# Windows环境



## database

### mysql

https://dev.mysql.com/downloads/

1. 下载并解压：https://dev.mysql.com/downloads/mysql/ ，c++ 2019可再发行软件包(运行库)：https://aka.ms/vs/16/release/vc_redist.x64.exe

2. 创建my.ini：E:\it\database\mysql\Mysql，不要自己创建data目录！

   ```properties
   [mysql]
   #设置mysql客户端默认字符集
   default-character-set=utf8
   [mysqld]
   #设置端口
   port = 3306
   #设置安装目录
   basedir=E:\it\database\mysql\Mysql
   #设置数据的存放目录
   datadir=E:\it\database\mysql\Mysql\data
   #设置最大连接数
   max_connections=200
   #设置mysql服务端使用的字符集默
   character-set-server=utf8
   #设置创建新表时使用的默认存储引擎
   default-storage-engine=INNODB
   ```

3. 配置，以管理员身份进入cmd

   ```cmd
   E:
   cd it\database\mysql\Mysql\bin #进入到mysql的bin目录
   mysqld install #安装mysql服务
   mysqld  --initialize-insecure #不安全(无root密码)初始化，并自动生成了data目录
   net start mysql #启动服务
   mysql -u root -p #登录mysql，无密码，回车直接登入
   
   use mysql; #进到mysql这个database
   ALTER USER `root`@`localhost` IDENTIFIED BY '新密码'; #设置密码
   exit #退出
   ```



## front-end

### Node

1. 下载并解压：https://nodejs.org/zh-cn/download/releases/

2. 环境变量：node.exe所在目录，如E:\it\front-end\Node

3. 创建文件夹并配置：node_global、node_cache

   ```cmd
   node -v # 检查node
   npm -v # 检查npm
   npm config set cache "E:\it\front-end\Node\node_cache" # 配置缓存目录路径
   npm config set prefix "E:\it\front-end\Node\node_global" # 配置全局目录路径
   npm config set registry "https://registry.npm.taobao.org" # 淘宝镜像
   npm config -g ls # 检查config
   npm install -g windows-build-tools # 安装winows构建工具，python2.7，c++相关等
   ```





## version-control

### Git

1. 下载并安装：https://git-scm.com/download/win，叉掉GUI，叉掉创建开始菜单，然后全部Next







## 环境变量

### 增变量

| key    | value                                                        |
| ------ | ------------------------------------------------------------ |
| path   | E:\it\front-end\Node<br/>E:\it\go\Go\bin<br/>E:\it\java\Java\jdk1.8.0_231\bin<br/>E:\it\java\Java\jdk1.8.0_231\jre\bin<br/>E:\it\python\Python27<br/>E:\it\version-control\Git\cmd |
| GOPATH | E:\it\go\GoProjects                                          |

### 原变量

| key                    | value                                                        |
| ---------------------- | ------------------------------------------------------------ |
| ComSpec                | %SystemRoot%\system32\cmd.exe                                |
| DriverData             | C:\Windows\System32\Drivers\DriverData                       |
| NUMBER OF_ PROCESSORS  | 16                                                           |
| OS                     | Windows_ NT                                                  |
| path                   | %SystemRoot%<br/>%SystemRoot%\system32<br/>%SystemRoot%\System32\Wbem<br/>%SYSTEMROOT%\System32\WindowsPowerShell\v1.0<br/>%SYSTEMROOT%\System32\OpenSSH |
| PATHEXT                | .COM;.EXE,;,BAT;,CMD;,VBS;VBE;JS;JSE;:WSF;WSH;MSC            |
| PROCESSOR_ARCHITECTURE | AMD64                                                        |
| PROCESSOR_ IDENTIFIER  | AMD64 Family 23 Model 113 Stepping 0, AuthenticAMD           |
| PROCESSOR_ LEVEL       | 23                                                           |
| PROCESSOR_ REVISION    | 7100                                                         |
| PSModulePath           | %ProgramFiles%\WindowsPowerShell\Modules<br/>%SystemRoot%\system32\WindowsPowerShell\v1.0\Modules |
| TEMP                   | %SystemRoot%\TEMP                                            |
| TMP                    | %SystemRoot%\TEMP                                            |
| USERNAME               | SYSTEM                                                       |
| windir                 | %SystemRoot%                                                 |







