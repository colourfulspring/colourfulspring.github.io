---
title: "tree 命令"
description: "the-tree-command"
keywords: "the,tree,command"

date: 2024-12-05T15:47:21+08:00
lastmod: 2024-12-05T15:47:21+08:00

categories:
  - Tool
tags:
  - Tool

# 原文作者
# Post's origin author name
#author:
# 原文链接
# Post's origin link URL
#link:
# 图片链接，用在open graph和twitter卡片上
# Image source link that will use in open graph and twitter card
#imgs:
# 在首页展开内容
# Expand content on the home page
#expand: true
# 外部链接地址，访问时直接跳转
# It's means that will redirecting to external links
#extlink:
# 在当前页面关闭评论功能
# Disabled comment plugins in this post
#comment:
#  enable: false
# 关闭文章目录功能
# Disable table of content
#toc: false
# 绝对访问路径
# Absolute link for visit
#url: "the-tree-command.html"
# 开启文章置顶，数字越小越靠前
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
# 开启数学公式渲染，可选值： mathjax, katex
# Support Math Formulas render, options: mathjax, katex
#math: mathjax
# 开启各种图渲染，如流程图、时序图、类图等
# Enable chart render, such as: flow, sequence, classes etc
#mermaid: true
---

# 使用需求
在编写博客时，我们需要清晰直观地向读着展示文件系统中某个目录的目录结构。一个例子如图所示：
```text
ParentPackage/
    ├── __init__.py 
    ├── main.py 
    ├── script1.py 
    ├── script2.py
    ├── ChildPackage1/
    │   ├── code1.py 
    │   ├── __init__.py
    │   └── GrandsonPackage/
    │       ├── code3.py 
    │       └── __init__.py
    └── ChildPackage2/
        ├── code2.py
        └── __init__.py
```
这种展示方法同时具有高精确性和优可读性两个特点。

# tree 命令安装
生成形如前一节展示的目录结构图需要使用 `tree` 命令。可以使用以下命令安装：
```bash
sudo apt install tree
```

# tree 命令使用
在需要展示目录下运行 `tree` 命令即可。
```bash
cd <your-directory>
tree
```
终端将会打印出目录 `<your-directory>` 的结构，形如使用需求中展示的例子。

# 更多选项
可自行使用 `--help` 参数打印 `tree` 命令的更多选项查看。

<!--more-->
