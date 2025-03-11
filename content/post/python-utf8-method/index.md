---
title: "UnicodeEncodeError: 'gbk' codec can't encode character '\xa9' in position 2846: illegal multibyte sequence"
description: "python-utf8-method"
keywords: "python,utf8,method"

date: 2025-03-11T14:21:28+08:00
lastmod: 2025-03-11T14:21:28+08:00

categories:
  - python
tags:
  - python

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
#url: "python-utf8-method.html"
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

# 问题
运行Python程序出现下列错误：
```text
UnicodeEncodeError: 'gbk' codec can't encode character '\xa9' in position 2846: illegal multibyte sequence
```
# 解决方法
在原Python命令中增加参数`-X utf8`即可。注意X需大写
```bash
python -X utf8 source.py
```
<!--more-->
