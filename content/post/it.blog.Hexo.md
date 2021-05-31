

+++

title = "Hexo"
description = "Hexo"
tags = ["it","blog"]

+++



# Hexo

https://hexo.io/

## 安装部署

### git

### nodejs

### hexo

安装

```shell
npm i -g hexo #安装hexo到Nodejs\node_global
```

初始化：建一个blog项目目录：yuanya-hexo

```shell
hexo init #进入目录初始化
hexo init yuanya-hexo #或者指定目录初始化
```

- yuanya-hexo中生成的目录和文件
  - node_modules：是依赖包
  - public：存放的是生成的页面
  - scaffolds：命令生成文章等的模板
  - source：用命令创建的各种文章
  - themes：主题
  - _config.yml：整个博客的配置
  - db.json：source解析所得到的
  - package.json：项目所需模块项目的配置信息

#### 本地

一般在本地调试好再部署到GitHub

```shell
npm i hexo-server #hexo 3.0把hexo-server独立成个别模块，需要单独安装
```

```shell
hexo clean #将删除public文件夹及其内容
hexo generate #生成public文件夹及其内容
hexo server #启动服务
```

本地访问：http://localhost:4000

#### GitHub

创建Repository：名为yuanyatianchi.github.io

ssh：GitHub→头像→setting→SSH and GPG keys→New SSH key→将下面生成的ssh公钥copy进去并保存

```shell
git config --list #git bash 查看git配置列表
ssh-keygen -t rsa -C "yuanya@yuanya.com" #生成ssh
cd ~/.ssh #进到~/.ssh
cat id_rsa.pub #查看公钥
```

在blog项目目录，修改`_config.yml`的deploy内容，配置为GitHub相关内容

```yaml
deploy:
  type: git
  repo: https://github.com/YuanyaTianchi/yuanyatianchi.github.io.git #最后一段的前缀即GitHub上刚建的blog repository名
  branch: master
```

部署到GitHub

```shell
npm install hexo-deployer-git --save #安装部署器
hexo clean #将删除public文件夹及其内容
hexo generate #生成public文件夹及其内容
hexo deploy #部署到GitHub
```

访问：https://yuanyatianchi.github.io

重新部署

```shell
hexo clean #部署前clean一下保持项目干净
hexo generate #重新生成
hexo deploy #部署
```



## 主题

https://hexo.io/themes/