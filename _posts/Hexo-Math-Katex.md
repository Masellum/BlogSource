---
title: '为 Hexo 更换数学渲染引擎为 KaTeX'
date: 2018-10-08 08:16:24
tags:
  - Hexo
categories: 其他
summary: ' '
---

<!-- more -->

# 引言

在使用 Hexo 写学习笔记的时候经常要用到数学公式功能。绝大多数 Hexo 博客使用的数学公式渲染引擎是 [MathJax](https://www.mathjax.org/)，但相对于我的需求而言，MathJax 显得过于笨重而缓慢。相较之下，[KaTeX](https://katex.org/) 渲染引擎速度更快。虽然在获得了性能的同时牺牲的是一部分的功能。MathJax 具有而 KaTeX 不具有的功能有：查看公式源码、更换渲染方式、缩放公式等，对我来说都并非硬性需求；与此同时 KaTeX 支持的 TeX 数学符号相对而言略少（可以在 [这个地方](https://katex.org/docs/supported.html) 查看 KaTeX 支持的符号列表），但可以看到绝大多数的符号也都已经被支持。而我不太能忍受我的博客中的公式加载缓慢，因此权衡利弊后我决定使用 KaTeX。

古语有云：不要重复造轮子。搜索一下可以发现已经有一个名为 [hexo-katex](https://github.com/thcd/hexo-katex) 的 Hexo 插件，照着文档安装就完事了。

# 具体过程

Hexo 默认的渲染引擎 hexo-renderer-marked 功能上对数学公式的实现有些丑陋——或者说是不够优美，而且难以定制。大概是这个原因让 hexo-katex 插件与其不合，所以 hexo-katex 插件依赖的渲染引擎是 [hexo-renderer-pandoc](https://github.com/wzpan/hexo-renderer-pandoc)。[Pandoc](http://pandoc.org/index.html) 是一个功能非常强大的格式转换工具，可以进行数十种文档格式之间的相互转换，这个渲染引擎只利用了其中 Markdown 转换为 HTML 的一部分，但是仍需要我们安装 Pandoc。

## 安装 Pandoc

参照 [Pandoc 官网的安装指南](http://pandoc.org/installing.html) 进行安装，注意 hexo-renderer-pandoc 要求 Pandoc 版本大于等于 2.0。Pandoc 支持主流三大系统，对于 Windows 可以下载 MSI 安装包或 ZIP 压缩文件或通过 Chocolatey 安装，macOS 可以使用 .pkg 安装包、下载压缩文件、homebrew 等方式进行安装，Linux 可以通过系统自带的软件包管理器安装（虽然版本一般较为老旧），如果软件源中没有或版本过于老旧无法接受也可以下载预编译好的二进制文件，也提供 deb 包供 Debian 系发行版使用（RH 系没人权（逃））。

> 对于 macOS 和 Linux，你还可以选择从源码编译。由于 Pandoc 使用 Haskell 语言编写，所以想要编译安装的话你还需要下载并安装 Haskell 平台。

> 对于 Windows 用户，请在安装完后确保 pandoc.exe 可执行文件所在目录被添加到了你的 Path 环境变量中，也就是说可以直接在命令行中通过 `pandoc` 来调用。

## 安装并配置 hexo-renderer-pandoc

首先确保已经正确安装了 Pandoc。在命令行中进入博客根目录，执行

```shell
npm install hexo-renderer-pandoc --save
```

来安装。虽然官方安装文档没有提到，但是我之后卸载了默认的 hexo-renderer-marked 插件：

```shell
npm uninstall hexo-renderer-marked --save
```

更改博客根目录下的 `_config.yml` 文件，在其中加入如下文本：

```yaml
pandoc:
  mathEngine: katex
```

来启用 KaTeX。

## 安装并配置 hexo-katex

```shell
npm install hexo-katex --save
```

根据这个插件的项目主页所说，它可以自动帮你插入 KaTeX 所需的 CSS 文件与 JavaScript 文件，不过这个功能好像在我这里不太好用，因此我选择关闭这个功能并自己手动添加。更改博客根目录下的 `_config.yml` 文件，在其中加入如下文本：

```yaml
katex:
  css: false
```

来关闭这个功能。

自己更改你的主题中的模板，让你的主题在每篇博客中适当的位置加入如下语句：

```html
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.10.0-rc.1/dist/katex.min.css" integrity="sha384-D+9gmBxUQogRLqvARvNLmA9hS2x//eK1FhVb9PiU86gmcrBrJAQT8okdJ4LMp2uv" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.10.0-rc.1/dist/katex.min.js" integrity="sha384-483A6DwYfKeDa0Q52fJmxFXkcPCFfnXMoXblOkJ4JcA8zATN6Tm78UNL72AKk+0O" crossorigin="anonymous"></script>
```

之后 KaTeX 应该就可以正常使用了。

# 注意事项

我在使用 KaTeX 代替 MathJax 的过程中遇到的一些需要注意的问题。

1. KaTeX 的公式块内不支持 Unicode 字符（比如汉字）。
2. KaTeX 的行内公式中，\$ 符号与里面的公式间不能有空格，否则会无法正常渲染。
3. `\def` 语句只会在一个公式块内起作用，无法贯彻整个文档。



Finita la comedia.
