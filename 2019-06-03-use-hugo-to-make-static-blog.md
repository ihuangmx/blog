# 使用 Hugo 快速搭建个人博客

`Hugo`,号称是世界上最快的静态网站生成工具，由 `Go` 语言开发，除了速度快之外，本身的使用也非常的简单。

## 安装

Mac 下的安装

```bash
$ brew install hugo
$ hugo version // 是否安装成功
$ hugo --help 
```

## 生成博客

创建网站

```bash
$ hugo new site ihuangmx.github.io
```

## 目录结构

由 `hugo` 生成的博客的基本目录如下

* `archetypes` - 预定义看板，在这类，你可以定制你生成的博客的基本模板。
* `content` - 网站的所有内容都保存在这里
* `data` - 保存站点可使用的配置文件
* `layouts` - 自定义的模板
* `public ` - 生成的可直接部署的博客
* `static` - 静态文件，比如你站点需要用到的 logo、头像等

目录的话，其实用到的地方不多，一般人只需要按照默认的主题来就行。

## 安装主题

生成网站之后，首先要做的就是下载 [主题](https://themes.gohugo.io/)，通常有两种方法。

第一种是下载所有主题，简单粗暴

```bash
$ cd ihuangmx.github.io
$ git clone --depth 1 --recursive https://github.com/gohugoio/hugoThemes.git themes
```

第二种是下载感兴趣的主题

```bash
$ cd themes
$ git clone https://github.com/spf13/hyde
$ git submodule add https://github.com/jacobsun/hugo-theme-cole  // 或者
```

## 使用主题

我们以 [hugo-pacman-theme](https://themes.gohugo.io/hugo-pacman-theme/) 为例简单的介绍。

安装主题

```bash
$ cd themes
$ git clone https://github.com/coderzh/hugo-pacman-theme.git --depth=1
```

按照需要配置主题页面，我自己的简单配置如下

```toml
BaseURL = "http://ihaungmx.github.io/"
LanguageCode = "zh-CN"
HasCJKLanguage = true
Title = "心智极客"
Theme = "hugo-pacman-theme"
pygmentsStyle = "default"
pygmentsUseClasses = true

[Author]
  Name = "心智极客"

[outputs]
  home = [ "RSS", "HTML" ]
    
[outputFormats] 
  [outputFormats.RSS]
    mediatype = "application/rss"
    baseName = "feed"

[Params]

  BottomIntroduce = "个人微信"
  Subtitle = "Never But Better"
  Imglogo = "img/logo.jpg"
  AuthorImg = "img/wechat.jpg"
  DateFormat = "2006年01月02日"
  MonthFormat = "2006年01月"
  FancyBox = false

  [Params.Disqus]
    ShortName = "huangmingxian"

  [Params.Strings]
    Search = "搜索"
    PageNotFound = "你访问的页面不存在"
    ShowSideBar = "显示侧边栏"
    HideSideBar = "隐藏侧边栏"
    Categories = "分类"
    Archive = "归档"
    Tags = "标签"
    Rss = "RSS 订阅"
    TableOfContents = "文章目录"

[Menu]
  [[Menu.Main]]
    Name = "首页"
    URL = "/"
    Weight = 1
  [[Menu.Main]]
    Name = "关于"
    URL = "/about"
    Weight = 2
```

配置里面对应的素材需要放到 `static` 目录下面

```
static
└── img
    ├── logo.jpg
    └── wechat.jpg
```

配置里面有一个「关于」页面，需要手动生成下

```bash
$ hugo new about.md
```

## 创建文章

首先，我们生成第一篇文章

```bash
$ hugo new post/first.md
```

生成的文章内容如下

```bash
---
title: "First"
date: 2019-06-03T17:10:40+08:00
draft: true
---
```

该文章的默认模板是根据 `archetypes` 目录下的 `default.md` 来的，可以自己修改模板，比如，修改文章的时间格式

```bash
---
title: "{{ replace .Name "-" " " | title }}"
date: {{ now.Format "2006-01-02"  }} 
draft: false
categories: [] 
tags: []
toc: false
---
```

再次生成博客文章

```bash
$ hugo new post/second.md
```

生成的文章内容变成了

```bash
---
title: "Second"
date: 2019-06-03 
draft: false
categories: [] 
tags: []
toc: false
---
```

## 生成最终网站

生成静态博客并在本地查看

```bash
$ hugo server
```

默认的话，只会显示 `draft` 为 `false` 的文章。

服务器是实时更新的，只要你的内容发生了变化，本地服务器会实时更新变化后的内容，也可以设置成不实时更新

```bash
$ hugo server --watch=false
```

`hugo server` 并不会生成真正的站点，一般在部署之前，需要运行 `hugo` 命令生成 `public` 站点

```bash
$ hugo
```

## 增加分类

增加分类的方法非常简单，只需要修改配置文件

```toml
[Menu]
   [[Menu.Main]]
    Name = "随笔"
    URL = "/categories/随笔"
    Weight = 1
   [[Menu.Main]]
    Name = "技术"
    URL = "/categories/技术"
    Weight = 2
   [[Menu.Main]]
    Name = "关于"
    URL = "/about"
    Weight = 3
```

文章增加对应的分类即可

```bash
---
title: "Second"
date: 2019-06-03 
draft: false
categories: ["随笔"] 
tags: []
toc: false
---
```

## 部署到 `Github`

只需要将 `public` 部署到 github 即可。