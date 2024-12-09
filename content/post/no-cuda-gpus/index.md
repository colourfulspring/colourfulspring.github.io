---
title: "RuntimeError: No CUDA GPUs are available"
description: "no-cuda-gpus"
keywords: "no,cuda,gpus"

date: 2024-11-30T21:43:02+08:00
lastmod: 2024-11-30T21:43:02+08:00

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
#url: "no-cuda-gpus.html"
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
使用 Nvidia GPU 运行某深度学习程序，显示如下报错：
```text
RuntimeError: No CUDA GPUs are available
```

# Bug 原因
上述 Bug 可能由两个不同的原因导致：
1. 当前机器不存在可被使用的 Nvidia GPU。
2. 当前机器存在可被使用的 Nvidia GPU，但通过环境变量 `CUDA_VISIBLE_DEVICES` 指定了不存在的 GPU 编号。

# 解决方法
## 原因一
打开终端，输入命令 `nvidia-smi`，观察结果，若显示如下结果，可判断是原因一：
```text
NVIDIA-SMI has failed because it couldn't communicate with the NVIDIA driver. Make sure that the latest NVIDIA driver is installed and running.
```
解决方法一般为检查显卡和显卡驱动的安装情况。

## 原因二
若显示类似如下格式的结果，可判断是原因二：
```text
Sat Nov 30 21:28:01 2024       
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 556.12                 Driver Version: 556.12         CUDA Version: 12.5     |
|-----------------------------------------+------------------------+----------------------+
| GPU  Name                  Driver-Model | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce RTX 4070 ...  WDDM  |   00000000:01:00.0 Off |                  N/A |
| N/A   37C    P8              1W /   80W |       0MiB /   8188MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI        PID   Type   Process name                              GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
+-----------------------------------------------------------------------------------------+

```
则属于情况一。需检查**报错位置**的 `CUDA_VISIBLE_DEVICES` 环境变量。

>注意：面对他人编写的代码时，请一定检查**报错位置**的 `CUDA_VISIBLE_DEVICES` 环境变量值，因为永远无法预
>料他人会在代码的哪个位置随便修改环境变量值。

可能修改 `CUDA_VISIBLE_DEVICES` 环境变量的语句有：
```bash
CUDA_VISIBLE_DEVICES=0 python main.py
```
```python
import os
os.environ['CUDA_VISIBLE_DEVICE']='0'
```
```python
import torch
 
torch.cuda.set_device(0)
```

# 参考
https://www.cnblogs.com/nannandbk/p/18144622

<!--more-->
