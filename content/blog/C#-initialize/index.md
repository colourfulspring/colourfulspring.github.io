---
title: "NullReferenceException: Object reference not set to an instance of an object"
date: 2024-08-18
---

## 背景
编写C#程序。程序运行时报了标题中的错误。

## 报错代码
```csharp
    List<Tuple<int, int>> trajList;

    //omitted

    trajList.Add(Tuple.Create(2, 2));
```

## 报错原因
Java、C#中声明一个变量且不初始化，则该变量的值为Null。调用一个Null变量的方法产生报错。

## 解决办法
```csharp
    List<Tuple<int, int>> trajList = new List<Tuple<int, double[,]>>(); // use new to initalize a variable.

    //omitted

    trajList.Add(Tuple.Create(2, 2));
```