---
title: "PYTHONHASHSEED: making set() container's element order same"
description: "python-hash-seed"
keywords: "python,hash,seed"

date: 2025-03-09T23:20:23+08:00
lastmod: 2025-03-09T23:20:23+08:00

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
#url: "python-hash-seed.html"
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
新建文件set.py：
```python
a = list(set(['yy', 'cc', 'aa', 'xx', 'dd', 'aa']))
print(a)
```
重复多次运行`python set.py`后，发现每次运行输出的a不同。
```text
['xx', 'cc', 'dd', 'aa', 'yy']
['aa', 'cc', 'dd', 'xx', 'yy']
['yy', 'aa', 'cc', 'xx', 'dd']
......
```
# 解决方法
在执行`python set.py`命令之前，将`PYTHONHASHSEED`环境变量设置为[0,4294967295]之间的一个值即可。

注意：在`set.py`源码中使用`os.environ`修改该环境变量无效！

# 参考文献
https://docs.python.org/3/using/cmdline.html#envvar-PYTHONHASHSEED

<!--more-->
