title: Mac、Win下，挂载任意目录至docker容器中
date: 2015-10-25 20:00:47
tags: [Mac,Windows,docker,挂载,文件系统]
categories: 小笔记
description: Mac、Win下，`docker run`的`-v`参数挂载非Users目录之外的目录进Docker容器
---

使用`docker run`启动容器时，可以使用`-v`参数将本机文件系统的目录挂载进容器中，例如：
``` shell
docker run -i -t -v ~/build:/root/build --name test ubuntu
```
但在Windows(Mac)下，却被限制只能挂载C:\Users(/Users)下的目录，可参看[官网文档](http://docs.docker.com/userguide/dockervolumes/)。

这对于Windows上和Mac外接硬盘放code的coder来说自然是难以忍受的事。Google下看到甚至有人在Mac上利用sshfs这种特别绕的方式来将本地文件系统挂载进去。笔者在研究后发现，实际上Docker是支持挂载其他地方的目录进容器的。远不用sshfs那么麻烦。

---
原理（以Mac为例，Win同理）：

`boot2docker up`启动docker后，使用`boo2docker ssh ls /`后就会发现，
在这里面有一个Users目录，同本机上/Users的内容是一样的，没有错，
这个就是virtualbox的共享文件夹，打开virtualbox，
查看boot2docker的设置就可以看到，Users对应的正好是本机/Users。Linux下，
`docker run`的`-v`是将本机目录挂载进容器中，而在Mac的boot2docker中，
也是一样的，它挂载的是virtualbox这个虚拟机（一般名字是boot2docker，
也是linux）的文件系统的目录。docker这个虚拟机里的文件系统挂载进容器中。
我们之所以能成功挂载Users及其子目录进容器，则是因为通过virtualbox的共享文件夹功能将本地目录映射到这个虚拟机中了。所以，`-v ~/build:/root/build`冒号左边的目录是boot2docker这个虚拟机文件系统里的目录（而不是本机），我们之所以，能写本地/Users下的目录，只是因为本地的/Users跟boot2docker中的/Users结构是完全一样的（映射）。

所以，我们就可以修改共享文件夹及`-v`参数左边的内容来实现挂载除/Users外的目录。比如我们要挂载本机的/Volumes/hello到容器的/root/hello上。

首先打开virtualbox，修改虚拟机boot2docker的设置，将共享文件夹Users指定的目录/Users改成/，保持共享文件夹Users不变。
> 当然，之前尝试过在virtualbox中增加其他的共享文件夹，但启动boot2docker后并不会生效。

之后，`boot2docker up`，这时，你再用`boot2docker ssh ls /Users/`查看时，
这里面的内容已经不在是本机/Users了，而是本机的/了。
所以，本机的/Volumes/hello对应的就是虚拟机中的/Users/Volumes/hello了。就可以这样挂载了。
``` shell
docker run -i -t -v /Users/Volumes/hello:/root/hello --name test ubuntu
```

这样只要本机目录名前加上/Users，都能挂载进容器中了。

---
后记：

不过，惟一的不足是，这样用起来有点别扭，尤其是依赖shell补全目录名时。尝试建软链接，`boot2docker ssh sudo ln -s /Users/Volumes /Volumes`。之后就可以使用
``` shell
docker run -i -t -v /Volumes/hello:/root/hello --name test ubuntu
```
来挂载，但重启boot2docker后软链接又没了，要解决这个，要自己制作boot2docker的镜像了。另外boot2docker up有一个参数可以设置共享文件夹，准备之后研究下，或许这个能更好地解决这个问题。


>>> 作者原创，转载请注明出处
