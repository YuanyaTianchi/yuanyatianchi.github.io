

+++

title = "Hugo"
date = "2020-11-11"
description = "hugo"
tags = ["it", "blog"]

+++



# hugo

- git、go环境
- 下载 & 解压 & 配置环境变量：https://github.com/gohugoio/hugo

```shell
#创建项目
hugo new site yuanyatianchi.github.io

#主题下载
cd yuanyatianchi.github.io
git clone https://github.com/dsrkafuu/hugo-theme-fuji.git themes/fuji
#下好后将fuji\exampleSite目录下的context、config.toml复制到项目根目录testblog替换原来的文件

#启动
hugo server
```

访问本地测试：http://localhost:1313

- config.toml修改

```shell
#修改baseURL为你自己的gitpage
baseURL = "https://yuanyatianchi.github.io"
```

- 部署

```shell
#打包主题，所有的静态页面默认生成到public目录，-d指定目录生成
hugo -t fuji -d docs

#关联远程库
git init
git remote add origin git@github.com:YuanyaTianchi/yuanyatianchi.github.io
git commit -a -m "yuanyatianchi's blog"
git push
```

远程库注意设置pages，设置到docs作为源
