---
category: 博客
tags: 
    - 操作系统
    - WSL
---

# 解决 WSL2 占据 C 盘过大空间的问题

那一天我在 wsl 上跑代码，突然报错内存空间不足，我一看C盘直接红了，使用 WizTree 扫描发现是 wsl 占用了 170 多GB，给我整不会了。因此我研究了一下，提出了一种处理这种问题的办法。

----

## 删除大文件

首先打开 wsl 控制台，执行`du -sh *|sort -h`指令，然后会得到类似如下的结果。

```
4.0K    compile_commands.json
4.0K    cpp.lnk
8.0K    cuda-keyring_1.1-1_all.deb
32K     docker-env
44K     llm
136K    NEMU2021buildmatrix-mul
5.8M    cpp
17M     Ayo
26M     NEMU2021
37M     csapp
39M     nemu
79M     PRAG
125M    vllm
541M    hotpot_train_v1.1.json
642M    RoboMaster
880M    opencv-4.5.4
1008M   Anaconda3-2024.06-1-Linux-x86_64.sh
2.0G    nmcp
4.2G    cuda_12.6.3.run
20G     langchain-rag
21G     anaconda3
81G     nemuP2
```

这条命令显示了在`/home/<USER>/`下的文件占用空间大小，可以根据这个选择需要删除的文件。

可是删除后会发现C盘剩余空间并没有变大，这是因为 wsl 采用的是动态扩容的虚拟磁盘机制，当 Linux 系统内文件增加时，虚拟磁盘会自动增大，但删除文件后，Windows 不会主动回收空闲空间，因此我们需要对虚拟磁盘进行压缩无效空间。

## 压缩虚拟磁盘

首先打开 PowerShell，运行`wsl -l -v`，得到类似如下结果：

```
  NAME              STATE           VERSION
* Ubuntu            Running         2
  docker-desktop    Stopped         2
```

这里可以看到我们的发行版及其状态，通常你可以在`C:\Users\<USER>\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_79rhkp1fndgsc\LocalState\ext4.vhdx`这个路径找到 Ubuntu 的虚拟磁盘文件。输出表明我们主要的发行版 Ubuntu 正在运行，因此我们需要先关闭。为了避免失误，我们对 wsl 进行备份，执行如下命令。

```bash
wsl --shutdown
wsl --export Ubuntu D:\Desktop\Ubuntu.tar # 这里导出的发行版名称和导出路径根据实际情况调整
``` 

最后进行虚拟磁盘压缩，在 PowerShell 里运行`diskpart`命令进入 `diskpart.exe` 命令行，在命令行中输入如下指令，重启后即可完成。

```bash
# 选择虚拟磁盘文件
select vdisk file="C:\Users\<USER>\AppData\Local\Packages\CanonicalGroupLimited.Ubuntu_79rhkp1fndgsc\LocalState\ext4.vhdx"
# 压缩
compact vdisk
# 释放
detach vdisk
```

## 参考博客

- [WSL2 Ubuntu系统占用C盘空间过大的问题分析与解决](https://blog.csdn.net/XFang03/article/details/144270735)
- [wsl2释放磁盘空间——节省空间](https://zhuanlan.zhihu.com/p/23226419724)