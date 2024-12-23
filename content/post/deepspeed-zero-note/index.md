---
title: "Deepspeed Zero Note"
description: "deepspeed-zero-note"
keywords: "deepspeed,zero,note"

date: 2024-12-23T01:38:49+08:00
lastmod: 2024-12-23T01:38:49+08:00

categories:
  - DeepSpeed
tags:
  - DeepSpeed
  - Data Parallel

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
#url: "deepspeed-zero-note.html"
# 开启文章置顶，数字越小越靠前
# Sticky post set-top in home page and the smaller nubmer will more forward.
#weight: 1
# 开启数学公式渲染，可选值： mathjax, katex
# Support Math Formulas render, options: mathjax, katex
math: mathjax
# 开启各种图渲染，如流程图、时序图、类图等
# Enable chart render, such as: flow, sequence, classes etc
#mermaid: true
---

# Data Parallel
每张卡上放一份模型参数+梯度+优化器状态。数据分为N份，分别在N张卡训练得到N份梯度。

将得到的梯度做AllReduce操作（一种卡间 collective operation），即可在每张卡的模型上都得到所有数据训练的梯度聚合。

假设模型参数量$\Phi$，共$N$张卡参与训练。

> 整个Data Parallel的主线：存储、通信。在下文中这两大开销的优化贯穿始终。

## DP
每张卡上放一份完整的模型+梯度+优化器状态。存储开销：$K\Phi$

卡间 Worker-Server 拓扑结构直接实现 AllReduce 操作。Worker卡计算梯度并上传到Server卡，Server卡负责聚合并下发聚合梯度给Worker卡。Worker单卡通信开销：$2\Phi$。$Server单卡通信开销：$2N\Phi$。这里假设$N$张Worker卡与1张Server卡。Server卡带宽成为瓶颈。

## DDP

卡间 Ring 拓扑结构实现 ReduceScatter+AllGather 来实现 AllReduce 操作。ReduceScatter梯度和AllGather梯度的通信开销：$(N-1)\frac{2\Phi}{N}$，约为$2\Phi$。

## Zero Redundancy Optimization (ZeRO) DP 
核心思想是任何参数、梯度、优化器状态都只在需要使用的时候才通过通信构造出完整部分，用完即丢弃。这以增加通信开销为代价减少存储开销。

混合精度训练：除存储开销$K\Phi$的必存参数外，我们还需额外创建一份fp16或bf16的参数+梯度用于DP训练，存储开销分别为$2\Phi$和$2\Phi$。

> Zero DP 训练过程中，通信开销主要来自混合精度训练中构造的fp16或bf16类型的梯度。这部分梯度存储开销为$2\Phi$。

### Stage 1
将优化器状态分成$N$份，每块GPU上各自维护一份。参数和梯度不变。

存储开销：优化器状态被分成$N$份，则除以$N$。

通信开销：ReduceScatter和AllGather操作，$(N-1)\frac{2\Phi}{N}$，约为$2\Phi$。

### Stage 2
将优化器和梯度分别分成$N$份，每块GPU上各自维护一份。参数不变。

### Stage 3

https://blog.csdn.net/dpppBR/article/details/80445569

https://blog.csdn.net/weixin_43336281/article/details/139483368

https://zhuanlan.zhihu.com/p/618865052

# Model Parallel
未完待续。

<!--more-->
