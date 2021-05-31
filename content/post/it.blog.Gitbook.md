

+++

title = "Gitbook"
description = "Gitbook"
tags = ["it","blog"]

+++



# Gitbook

```sh
npm install -g gitbook-cli
mkdir mybook
gitbook init
```

- README.md —— 书籍的简介
- SUMMARY.md —— 书籍的目录
- 配置方法

```markdown
# 目录

* [前言](README.md)
* [章节名称](相对路径)
  * [第1节](相对路径)
  * [第2节](相对路径)
  * [第3节](相对路径)
  * [第4节](相对路径)
* [第二章](相对路径)
* [第三章](相对路径)
```

```sh
# 重新init
gitbook init
```

```sh
gitbook serve --port 4000 #默认即4000
```



发布

```sh
# 输出成html
gitbook build [书籍路径] [输出路径]
```

然后提交到GitHub上配合gitpage即可