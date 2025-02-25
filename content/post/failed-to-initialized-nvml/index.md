---
title: "Failed to initialize NVML: Driver/library version mismatch"
description: "failed-to-initialize-nvml"
keywords: "failed,to,initialize,nvml"

date: 2024-11-30T21:00:53+08:00
lastmod: 2024-11-30T21:00:53+08:00

categories:
  - Bug
tags:
  - Bug
  - Nvidia

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
#url: "failed-to-initialize-nvml.html"
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
# Bug 现场
在 Ubuntu 22.04 版本的 docker 容器内，打开终端，输入命令 `nvidia-smi` ，报错如下：
```text
Failed to initialize NVML: Driver/library version mismatch
```

# Bug 原因
Ubuntu 自动更新 Nvidia Driver，导致 Nvidia Driver 和 Nvidia Library 不兼容！

由于主机中 Nvidia Driver 安装的是 Ubuntu 官方提供的版本，Ubuntu 会在后台自动更新并运行新版的 Nvidia Driver。如果启动 Docker 容器发生在更新 Nvidia Driver 之前，则 Docker 容器仍在使用旧版 Nvidia Driver，导致版本不兼容。

# 解决方法
先关闭 Docker 容器，再重启 Docker daemon。
```bash
docker stop <container-name-or-id>
sudo systemctl restart docker
```

# 参考
https://blog.csdn.net/ccsodefhy/article/details/122846921

https://blog.csdn.net/ZnS_oscar/article/details/134108758

<!--more-->
