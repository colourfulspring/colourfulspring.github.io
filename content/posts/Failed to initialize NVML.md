---
date: 2024-11-19
# description: ""
# image: ""
lastmod: 2024-11-19
showTableOfContents: false
tags: ["Bug", "Nvidia"]
title: "Failed to initialize NVML: Driver/library version mismatch"
type: "post"
---

### Bug现场
在Ubuntu 22.04版本的docker容器内，打开终端，输入命令`nvidia-smi`，报错如下：
```text
Failed to initialize NVML: Driver/library version mismatch
```

### Bug原因
Ubuntu自动更新Nvidia Driver，导致Nvidia Driver和Nvidia Library不兼容！

由于主机中Nvidia Driver安装的是Ubuntu官方提供的版本，Ubuntu会在后台自动更新并运行新版的Nvidia Driver。如果启动Docker容器发生在更新Nvidia Driver之前，则Docker容器仍在使用旧版Nvidia Driver，导致版本不兼容。

### 解决方法
先关闭Docker容器，再重启Docker daemon。
```bash
docker stop <container-name-or-id>
sudo systemctl restart docker
```





