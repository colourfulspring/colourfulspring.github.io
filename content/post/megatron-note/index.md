---
title: "Megatron Note"
description: "megatron-note"
keywords: "megatron,note"

date: 2024-12-24T11:15:03+08:00
lastmod: 2024-12-24T11:15:03+08:00

categories:
  - Megatron
tags:
  - Megatron
  - Tensor Parallel

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
#url: "megatron-note.html"
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

# Tensor Parallel
张量并行（Tensor Parallel）是一种模型并行的方法，它通过将模型的参数矩阵划分到多个GPU上来实现并行计算。

> 数据并行在计算需要用到某参数的时候，通过通信使每个GPU上保存完整的该参数，而张量并行在计算时，每个GPU上也只保存模型参数的一部分，通过通信传播计算结果即可。

张量并行的基础是对模型参数矩阵 $W$的切分。不妨设输入数据$X$，输出结果$Y$。

## 矩阵切分
常见的矩阵切分有按行切分、按列切分等方法。在Megatron-LM中，权重的切分操作就是由这两个基础算子组合而成的。

### 按行切分
X @ W = (X1 X2) @ (W1 W2)^T = X1 @ W1 + X2 @ W2 = Y1 + Y2 = Y。

### 按列切分
X @ W = X @ (W1 W2) = (X @ W1  X @ W2) = (Y1  Y2)= Y。


## MLP 层
GELU(X @ W1) @ W2 = Y

因为GELU是非线性的激活函数，所以GELU(X1 + X2) != GELU(X1) + GELU(X2)，GELU(X1 X2) = [GELU(X1) GELU(X2)]。

这使得W1需要按列切分，W2需要按行切分。

## Attention 层
X @ W^Q = Q
X @ W^K = K
X @ W^V = V

Softmax(Q @ K^T) @ V = Y

Z = Dropout(Y @ B)

W^Q, W^K 和 W^V 需要按列切分。
求得的Q, K, V 按列切分。
B 需要按行切分。

## Embedding 层
输入层按行切分，输出层按列切分。

Flash Attention的 bound 在softmax， tensorcore算e^x比较慢。
原版Attention的 bound 在 memory 
<!--more-->
