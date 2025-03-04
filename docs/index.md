# Welcome to My Notebook

## 关于网站

芝士我的新笔记本网站，是个静态的东西，它会存一点我的垃圾和宝贝。

## 联系作者

- 邮箱：e5truswent@outlook.com

## 关于作者

- 我是一名平凡的学牲，很多东西不太能写得好；
- 若笔记存在疏漏，还望您海涵，请勘误后通过邮件联系我修改；
- 若愿意分享您的笔记或挂友链，可以通过邮箱联系；
- **我是个懒人，会拖更**。

## 网站搭建

本网站采用`mkdocs`进行搭建，这个生成器相当简洁且实用，主要使用`pymarkdown`渲染`markdown`。

### 1.安装与准备

`mkdocs`是 python 的一个库，可以用`pip install mkdocs`或`pip3 install mkdocs`下载。

```shell
# 创建一个叫 note 的文件夹
$ mkdocs new note    
# 进入笔记文件夹
$ cd note
```

若您未将 python 的库加入环境变量，请在所有`mkdocs`有关命令前加上`python -m`，例如：`python -m mkdocs new note`。

此时若您能使用`tree`命令查看结构的话，您能大致看到：
```
./
├── docs/
│     └── index.md
├── mkdocs.yml
└── site/
```

### 2.配置与查看

初始化阶段准备成功后就可以进行同步渲染了，这里请先写好 mkdocs.yml 的相关内容，关于如何写这个配置文件，请移步[配置文件——mkdocs中文文档](https://hellowac.github.io/mkdocs-docs-zh/user-guide/configuration/)

```shell
# 写好以后，在本地进行同步的渲染
$ mkdocs serve
...
# “ctrl + 单击链接”即可打开渲染的网站
INFO    -  [xx:yy:zz] Serving on http://127.0.0.1:8000/
...

# 生成静态网页代码
$ mkdocs build

# 根据配置文件 mkdocs.yml 自动生成网页有关的 site 文件夹，它自己会传到 gh-pages 分支
$ mkdocs gh-deploy
```

## 友情链接

NaNa's Notebook: https://kagunanana.site