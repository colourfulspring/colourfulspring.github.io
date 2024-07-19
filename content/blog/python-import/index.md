---
title: Python导入语句的两个常见错误
date: 2024-07-06
---

## 背景
在使用Python自定义包的时候，我们常常会遇到下列两个错误：
```text
ImportError: No module named xxx
```
```text
ImportError: attempted relative import with no known parent package
```
产生这两个问题的原因往往是项目中包和文件的排布结构有误。现提出一种Python规范，按照该规范操作可以避免这两个错误。

## Python 模块
在Python中，一个`.py`文件被称为一个模块。模块是组成程序的基本单位。

## Python 包
在Python中，一个目录path满足以下两个条件，则被称为包：
1. path目录下有一个`__init__.py`文件。该文件可以为空，但不能不存在。
2. 不能将path目录下(不包括path目录的其他子目录下)的`.py`文件运行在`main`作用域中，即不能通过标准输入、脚本文件或是交互式命令读入path目录下的模块。模块可以通过检查自己的 `__name__` 是否等于 `'__main__'`来判断自己是否运行在main作用域中。

第2条较难理解，现举一个例子说明：
```text
ParentPackage/
    ├── __init__.py (此文件与入口main.py文件在同一目录，所以ParentPackage不被视为包)
    ├── main.py （执行python main.py，入口文件）
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
若在`ParentPackage`目录下运行命令
```bash
python main.py
```
则`ParentPackage`目录会因为`main.py`运行在`main`作用域而不被视为包（不满足第2条要求），即使`ParentPackage`目录下存在一个`__init__.py`（满足第1条要求）。

这时`ParentPackage`目录下的`ChildPackage1`目录则会被视为包。因为`ChildPackage1`目录下存在一个`__init__.py`（满足第1条要求），`ChildPackage1`目录下（不包括`ChildPackage`的其他子目录下）没有运行在`main`作用域的`.py`文件。`ChildPackage2`目录同理。

我们称`main.py`为入口文件。

## Python 项目代码结构规范
仍然以刚才的例子做说明。
```text
ParentPackage/
    ├── __init__.py (此文件与入口main.py文件在同一目录，所以ParentPackage不被视为包)
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
为了避免背景中两个错误的出现，现提出一种python工程文件排布规范，

该规范应满足以下几个注意点：
* 不属于任何包的`.py`脚本文件（如例子中的`script1.py`）应和入口`main.py`在同一目录下（位于`ParentPackage`目录）。
* 所有自定义包也应和入口`main.py`在同一目录下（位于`ParentPackage`目录）。
* `ParentPackage`目录下的`__init__.py`文件失效，`ParentPackage`不被视为包。因为入口`main.py`文件在`ParentPackage`目录下。

## Python 顶层包
顶层包指包目录恰好和入口`main.py`在同一目录下的包，或者通过`pip install package`安装的包。

上面的例子中，`ChildPackage1`和`ChildPackage2`为顶层包，因为和入口文件`main.py`在同一目录下。`GrandsonPackage`不是顶层包，因为不和入口文件`main.py`在同一目录下。

## Python 导入语句
Python导入其他模块和包的代码主要有绝对导入和相对导入两种方法。

绝对导入主要有以下形式，其中后两种会将导入的包改名：
```python
import A
from A import B
import A as a
from A import B as b
```

相对导入主要有以下形式：
```python
import .A
from .A import B
import ..A as a
from ..A import B as b
```
其中.表示当前`.py`文件所在的包，..表示当前`.py`文件所在包的上一级包，...表示当前`.py`文件所在包的上上一级包，以此类推。

## Python导入语句的规范
Python导入语句需要满足的规范如下：
1. 若语句为**绝对导入**，则A必须为**顶层包**。这与该导入语句所在的`.py`文件位于何目录下无关。
2. 若语句为**相对导入**，则..必须**代表一个包目录**，A为该包目录下的一个包。

## ImportError: No module named xxx
该错误表示：根据上一节描述的规则，无法找到xxx这个包。

## ImportError: attempted relative import with no known parent package
该错误表示：在相对导入语句中，..代表的不是一个包目录。