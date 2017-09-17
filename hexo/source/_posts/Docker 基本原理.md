---
title: Dokcer 基本原理
date: 2017-09-17
tags: Docker
categories: Dokcer
---

只讲解最最基本的概念。

## 什么是Docker

百度百科定义如下：

> Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器上，也可以实现虚拟化，容器是完全使用沙箱机制，相互之间不会有任何接口。

Docker可以看作是一个虚拟机，有一套运行的环境，可以简单的想象docker就是一个本地的Linux操作系统，这和KVM, Virtualbox, Vmware很像。Docker的目的就是要在宿主机上实现一个像Virtualbox这样功能的东西，但是，实现原理却不一样。KVM, Virtualbox, Vmware是虚拟出机器，让每个实例看到一个单独的机器；而Docker是虚拟出操作系统，实现应用之间的隔离，让各个应用觉得自己有一个自己的操作系统，而且彼此之间隔离。

+ 在没有虚拟化的情况下，假设有两个进程，1和2。它们共享kernel，不同的PID，共享网卡，共享IP地址，看到一样的根文件系统（不chroot的情况下）等，用Linux IPC手段进程间通信。

+ Docker的情况，进程1和进程2运行于不同的容器，进程1和进程2是两个世界的人，互相不认识，都觉得自己拥有自己的根文件系统，自己的网卡和ip等，然后进程1和进程2的PID还可以一样，比如假设2个都是100。但是，此100非彼100。但是，不同的容器在同一个宿主机的内核上。

![](/images/docker_01.png)

+ 虚拟机的情况，进程1和进程2运行于不同的虚拟机，则操作系统都是双份的，它们感觉自己在不同的虚拟电脑上面跑。

![](/images/docker_02.png)

虚拟机创建的是整个世界，包括运行时的内核；而Docker创建的是运行环境(Container)，不同的Container都运行在用一个内核上。container相当于沙箱进行资源的隔离与权限的控制。而Docker如何进行资源的隔离呢？这是借助Linux系统的LXC功能，而LXC主要依赖**kernel的namespace**来做资源隔离和**CGroups**来做资源的使用限制。

## kernel的namespace

Linux 内核从版本 2.4.19 开始陆续引入了 namespace 的概念。它提供了一种资源隔离方案，PID,IPC,Network等系统资源不再是全局性的，而是属于特定的Namespace。Linux 内核中实现了六种 namespace：

### Mount namespaces

文件系统挂接点的隔离，每个容器能看到不同的文件系统层次结构，这样不同容器中的进程就看到了不同的文件目录，**实现了文件系统的隔离**。

### UTS namespaces

隔离了nodename 和 domainname。它允许每个容器拥有自己的**hostname** 和 **domainame**。

### IPC namespaces

进程间通信的资源的隔离。每个容器有其自己的 System V IPC 和 POSIX 消息队列文件系统，因此，只有在同一个 IPC namespace 的进程之间才能互相通信

### PID namespace 

每个 PID namespace 中的进程可以有其独立的 PID； 每个容器可以有其 PID 为1的root进程；也使得容器可以在不同的 host 之间迁移，因为 namespace 中的进程 ID 和 host无关了。这也使得容器中的每个进程有两个PID：容器中的 PID 和 host 上的 PID。

### Network namespaces

每个容器用有其独立的网络设备，IP 地址，IP 路由表，/proc/net 目录，端口号等等。这也使得一个 host 上多个容器内的同一个应用都绑定到各自容器的 80 端口上。

### User namespaces

 在 user namespace 中的进程的用户和组 ID 可以和在 host 上不同； 每个 container 可以有不同的 user 和 group id；一个 host 上的非特权用户可以成为 user namespace 中的特权用户；
 
 ## Control Groups
 
 cgroups 实现了对资源的配额和度量。cgroups 的使用非常简单，提供类似文件的接口，在 /cgroup目录下新建一个文件夹即可新建一个group，在此文件夹中新建task文件，并将pid写入该文件，即可实现对该进程的资源控制。groups可以限制blkio、cpu、cpuacct、cpuset、devices、freezer、memory、net_cls、ns九大子系统的资源
 
 ## Docker的存储
 
 刚刚收到Docker使用了LXC的namespace技术和CGroups技术实现了资源的隔离和度量，那么，它的文件系统是怎样的呢？
 
 ### Linux的内核文件系统
 
一个典型的 Linux 系统要能运行的话，它至少需要两个文件系统：

+ boot file system （bootfs）：包含 boot loader 和 kernel。用户不会修改这个文件系统。实际上，在启动（boot）过程完成后，整个内核都会被加载进内存，此时 bootfs 会被卸载掉从而释放出所占用的内存。同时也可以看出，对于同样内核版本的不同的 Linux 发行版的 bootfs 都是一致的。

+ root file system （rootfs）：包含典型的目录结构，包括 /dev, /proc, /bin, /etc, /lib, /usr, and /tmp 等再加上要运行用户应用所需要的所有配置文件，二进制文件和库文件。这个文件系统在不同的Linux 发行版中是不同的。而且用户可以对这个文件进行修改。

Linux 系统在启动时，roofs 首先会被挂载为只读模式，然后在启动完成后被修改为读写模式，随后它们就可以被修改了。对于以上两种文件系统的理解，可以是bootfs是Linux内核的文件系统，而rootfs是不同linux发行版本的文件系统，如Ubuntu roofs, centos rootfs

### AUFS

AUFS是一种联合文件系统，它可以**把不同物理位置的目录合并mount到同一个目录中**.举个简单的例子：把 /tmp， /var， /opt三个目录联合挂着到 /aufs目录下，则 /aufs目录可见 /tmp，/var， /opt目录下的所有文件。而每个成员目录，则称为虚拟文件系统的一个branch

```
mount -t aufs -o br=/tmp=rw: /var=ro: /opt=ro none /aufs
```

一般的，命令从左到右的目录，只有最左边的（最上层）是读写的，其余的为只读。但我们对/aufs进行修改时，改变会反应到挂载的目录中。

### Docker中的文件系统

有了AUFS的基本了解，理解Docker的镜像就比较容易了。首先看什么是镜像

#### Docker images

Docker的镜像跟virtualbox的镜像不一样。在虚拟机中，镜像是一个系统的完整体，包括了系统、用户在上面做的操作等等。而在Docker中，**镜像是一组文件的叠加**。Docker Image是用AUFS系统组织起来的，如上面的命令，每一个branch就代表着每一个镜像层，一个镜像层可以构建另一个镜像层，一般，最上面的镜像层是读写的，其余的是只读的。

为什么Docker image可以创建多个不同的容器？

> 创建多个容器时，只需创建多个容器运行目录，使用aufs把容器运行目录挂载在Image目录之上，即实现一个Image运行多个Container。

![](/images/docker_03.jpg)

如图所示，在 Docker 中，基础镜像中的 roofs 会一直保持只读模式，Docker 会利用 union mount 来在这个 rootfs 上增加更多的只读文件系统，最后它们看起来就像一个文件系统即容器的 rootfs。这就是一个linux操作系统的基础镜像。

+ 所有 Docker 容器都共享主机系统的 bootfs 即 Linux 内核
+ 每个容器有自己的 rootfs，它来自不同的 Linux 发行版的基础镜像，包括 Ubuntu，Debian 和 SUSE 等
所有基于一种基础镜像的容器都共享这种 rootfs

但我们通过dockerfiles 添加一些东西时，其实实际上是在Base image上增加镜像层，注意，这些镜像层都是只读的。镜像层对应的目录在 `/var/lib/docker/aufs/diff/文件夹内`。所以我们发现，罚我们重复运行dockerfile时，第二次运行时，不用再次创建镜像层，而是直接类似从cache中读取。

我们有了Docker image后，不能够直接使用，而是要用`docker run iamge` 来生成容器。当生成容器时，会则镜像层上生成一个容器层，这个容器层是可读写的，同样位于`/var/lib/docker/aufs/diff/文件夹内`，而在容器内对文件的修改，增加，删除，都会在这个容器层里操作，而当container停止运行时，这个容器层也不在存在了。

关于 Docker的分层镜像，除了 aufs，docker还支持btrfs, devicemapper和vfs，你可以使用 -s 或 –storage-driver= 选项来指定相关的镜像存储。在Ubuntu 14.04下，Docker 默认 Ubuntu的 AUFS。

#### 拓展

刚刚提到在容器内对文件的操作都在容器层，容器停止了就消失了，很想linux里的/tmp目录，每次重启，上次的数据就没了，那么如果想要保存容器运行时生成的数据呢？

##### 目录挂载

docker可以支持把一个宿主机上的目录挂载到镜像里

```
docker run -it -v /home/dock/Downloads:/usr/Downloads ubuntu64 /bin/bash
```
通过-v参数，冒号前为宿主机目录，必须为绝对路径，冒号后为镜像内挂载的路径。这样，可以在容器内操作本地文件了，存储直接存到宿主机的文件系统。

##### 数据卷

1. 宿主机的文件作为数据卷

```
dockerfile中指定VOLUME后的目录，如VOLUME /var/lib/test
或者
docker run 命令中加-v选项。如 
docker run -d -P -v /webapp training/webapp python app.py
```

2. 容器数据卷

专门用于挂载数据卷的容器，供其他容器引用，多用于多个容器需要共享同一段数据。用法：–volumes-from

具体是先创建一个数据卷容器，在创建一个新的容器来引用数据卷容器。 
```
docker run -d -v /dbdata --name=dbdata training/postgres//创建一个数据卷容器 
docker run -d --volumes-from dbdata --name=db1 training/postgres//创建一个名为db1的容器并引用
```























