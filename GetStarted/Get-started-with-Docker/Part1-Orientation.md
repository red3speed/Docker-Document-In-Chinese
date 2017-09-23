# 第一部分: 定位和设置

欢迎！我们很高兴你想学习如何使用Docker。

在这六部分的教程中，你将学到：

- 1.在此页面学习和了解Docker。
- 2.构建和运行你的第一个应用。
- 3.将你的应用变成一个可伸缩的服务。
- 4.使你的服务跨机器使用。
- 5.添加一个可持久化数据的访客计数器。
- 6.部署你的集群(swarm)到生产环境。

应用本身十分简单，所以你不必为代码正在做什么而分心。毕竟，Docker的价值在于如何构建、装箱和运行应用；它完全不了解你的应用程序上实际上做了什么。

## 先决条件
过程中我们将定义各种概念，以便于在开始之前让你理解[什么是Docker](https://www.docker.com/what-docker)以及[为什么你应该使用Dokcer](https://www.docker.com/use-cases)。

在我们开始之前，我们需要假定您熟悉以下的几个概念：

- IP地址和端口
- 虚拟机
- 编辑配置文件
- 构建机器资源的用法，像是CPU占比、运存使用的字节数等

## 容器的简要说明
镜像(image)是一个轻量、独立的可执行包，它包括了运行软件的一切：代码、运行时、依赖库、环境变量和配置文件。

容器(container)是一个镜像的运行实例——镜像在内存中实际执行的状态。默认情况下，它与主机环境隔离，只能访问主机文件和端口(如果如此配置)。

容器在主机的内核上运行应用。相比于仅能通过虚拟机监控程序访问主机资源的虚拟机，它们有更好的性能表现。容积可以获得本机访问权限，每个容器都是不相关联的进程，不会比其他任何可执行进程占用更多内存。

## 容器 vs. 虚拟机
按照下图将虚拟机和容器进行比较

### 虚拟机图表

![虚拟机栈示例](images/VM@2x.png) 

虚拟机运行客户机-注意每个框中的系统层。这是资源密集型的，且由此产生的磁盘镜像和应用状态是一个系统设置、系统安装依赖、系统安全补丁以及其他易丢失且难复现的短生命周期事物的集合体。

### 容器图表
![容器栈示例](images/Container@2x.png) 

容器可以共享单个内核，而且在一个容器镜像中唯一需要的信息就是可执行的应用和它的依赖，且这些不需要被安装在主机系统上。这些进程像本地进程一样运行，而且你可以通过运行像 `docker ps`这样的命令来单独管理它们（注：类似linux上的ps命令，可以看到活动的进程）。最后，因为容器包含了它们所有的依赖，所以没有依赖主机的配置，一个容器化的应用可以做到“在任何地方运行”。

## 设置
在我们开始之前，请确保您的系统已经安装了最新版本的Docker。

<kbd>[Install Docker](https://docs.docker.com/engine/installation/)</kbd>

> **注意**: 要求版本v1.13或更高

安装好后，执行`docker run hello-world`命令，应该要能看到如下响应:

```command
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
...(snipped)...
```

现在你可以确认你的Docker版本是否在v1.13以上。执行`docker --version`来检查它。
```command
$ docker --version
Docker version 17.05.0-ce-rc1, build 2878a85
If you see messages like the ones above, you are ready to begin your journey.
```

## 小结
单位规模应用变为一个独立的、便携可执行的应用有着重大的意义。它意味着CI/CD可以推送到一个分布式应用的任何一个部分，系统依赖将不是问题，而且资源密度也能够增加。伸缩操作的调度是产生新的可执行程序而非新的虚拟机。

我们将学习以上所提及的所有，但首先让我们先学学如何走路。