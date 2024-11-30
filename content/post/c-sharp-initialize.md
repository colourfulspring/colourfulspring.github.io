---
title: "NullReferenceException: Object reference not set to an instance of an object"
description: "c-sharp-initialize"
keywords: "c,sharp,initialize"

date: 2024-11-30T19:27:07+08:00
lastmod: 2024-11-30T19:27:07+08:00

categories:
  - Bug
tags:
  - Bug
  - C#

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
#url: "c-sharp-initialize.html"
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
编写C#程序。程序运行时报错如下：
```text
NullReferenceException: Object reference not set to an instance of an object
```

### 报错代码
```csharp
    List<Tuple<int, int>> trajList;

    //omitted

    trajList.Add(Tuple.Create(2, 2));
```

### 报错原因
Java、C#中声明一个变量且不初始化，则该变量的值为Null。调用一个Null变量的方法产生报错。

### 解决办法
```csharp
    List<Tuple<int, int>> trajList = new List<Tuple<int, double[,]>>(); // use new to initalize a variable.

    //omitted

    trajList.Add(Tuple.Create(2, 2));
```

<!--more-->
