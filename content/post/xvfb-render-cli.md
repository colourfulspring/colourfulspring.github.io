---
title: "Xvfb Render Error at Ubuntu Server"
description: "xvfb-render-cli"
keywords: "xvfb,render,cli"

date: 2024-11-30T20:45:51+08:00
lastmod: 2024-11-30T20:45:51+08:00

categories:
  - Bug
tags:
  - Bug
  - OpenGL

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
#url: "xvfb-render-cli.html"
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

### 错误现场
在没有安装显示器的Ubuntu服务器上，调用gym框架中的`env.render()`方法渲染MPE环境，报错如下：
```text
ImportError:
    Error occured while running from pyglet.gl import *
    HINT: make sure you have OpenGL install. On Ubuntu, you can run 'apt-get install python-opengl'.
    If you're running on a server, you may need a virtual frame buffer; something like this should work:
    'xvfb-run -s "-screen 0 1400x900x24" python <your_script.py>'
```

### 错误原因
上述报错信息中已说明，错误原因可能有两个：
1. 没有安装`python-opengl`包。
2. 程序运行在服务器上，需要使用virtual frame buffer。

### 解决方法
1. 使用如下命令，安装`python-opengl`包。
```bash
apt-get install python-opengl
```
2. 使用如下命令，先安装`xvfb`包。
```bash
apt-get install xvfb
```
接着运行如下命令，程序便可以运行，运行效果与带显示器渲染的机器上运行相同：
```bash
xvfb-run --auto-servernum --server-num=1 python <your_script.py>
```
注：未尝试过上述报错信息中提示的命令。

<!--more-->
