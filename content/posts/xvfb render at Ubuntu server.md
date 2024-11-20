---
date: 2024-11-20
# description: ""
# image: ""
lastmod: 2024-11-20
showTableOfContents: false
tags: ["Bug", "opengl"]
title: "Xvfb Render at Ubuntu Server"
type: "post"
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