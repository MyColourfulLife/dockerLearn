这是Docker文档的第四部分的翻译,原文请点击[这里](https://docs.docker.com/get-started/part4/#prerequisites)

# 必要条件
- 安装Docker1.13版本或者更高的版本
- 在第3部分使用Docker Compose作为描述
- 使用Docker Machine，这一步在 Docker for Mac 和 Docker for Windows 部分已经安装过了，但是在Linux系统上，你需要直接安装，在没有Hyper-V的Windows10预览版以及WIndow10家庭版，使用Docker toolbox
- 阅读第一部分指南
- 在第二部分学习如何创建容器 
- 确保你已经把创建的 名为 friendlyhello 图像推送到了注册表，我们将在这里使用共享的这个图像
- 确保你的图像作为已部署的容器，运行这个命令，键入你的用户名，仓库 和标记 `docker run -p 80:80 username/repo:tag` 然后访问 `http://localhost/`
- 顺便从第三部分拷贝你的 `docker-compose.yml`

# 介绍
在第三部分，你获得了第二部分写的应用，通过将其转为服务，定义了它在生产中的运行方式，并将其扩展到五倍。

在这第四部分，你把这个应用部署到集群上，让它在多个机器上运行。通过使用群组使多容器多设备应用成为可能，群组就是把多个设备连接成Docker化的集群。

# 了解群集

一个群组使是运行docker的一组机器，并把这些机器链接成一个集群。自那以后，你可以继续像以前那样运行docker命令，但是现在是被群组管理执行在一个集群上。

群组管理员可以使用几个策略运行容器，比如空节点-将会使用容器填充最少可用的机器，或者 全部 这会保证每一个机器都会得到一个特定容器的实例。你可以在 Compose文件中编辑这些策略，就像之前你已经使用的那个。

群组管理员就是群组中的一个机器，你可以用它执行你的命令，授权其他机器作为工作人员加入群组，工作人员仅仅提供能力，没有权限去告诉其他机器什么可以做什么不可以做。

到目前为止，你已经在你的本机上使用单一主机模式使用了Docker，但是Docker也可以切换到群组模式。那就是使用群组。立即使用群组，让你当前的机器成为一个群组管理员。
这样，Docker将会在你管理的群组上执行命令，而不是只在当前的机器上。

# 启动群组

一个群组由多个节点组成，这些节点可以是物理机也可以是虚拟机。最基本的原理相当简单:执行`docker swarm init` 来启动群组模式并且让当前的机器成为群组管理者，
然后在其他机器上运行 `docker swarm join` 使他们作为工作人员加入群组。在下面选择一个标签来看它是如何在不同的环境中运行的。我们将使用虚拟机快速创建两个机器群集并且把它转换成群组。

## 创建一个集群

以 Mac Linux Windows7 和 Windows8 为例

本机上的虚拟机(Mac Linux Windows7 和 Windows8 )

第一步， 你需要一个可以安装虚拟机的管理程序，因此可以为您的机器的操作系统安装Oracle VirtualBox。
> 注意：如果您在Windows系统上安装了Hyper-V，例如Windows 10，则不需要安装VirtualBox，您应该使用Hyper-V。 通过单击上面的Hyper-V选项卡查看Hyper-V系统的说明。 如果您使用Docker Toolbox，您应该已经安装了VirtualBox作为其一部分，所以你可以开始了。

现在 使用`docker-machine`创建几个虚拟机，使用VirtualBox作为驱动

```
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2
```

## 列出虚拟机 并且获取 ip地址
你已经创建了两个虚拟机了，名字为`myvm1` 和 `myvm2`
```
docker-machine ls
```
这是这个命令的输出
```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   -        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce   
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce 
```


## 初始化群组 添加 节点

第一个机器将会作为一个管理员，用它来执行管理员命令，授权其他工作人员加入群组，第二个机器将会成为一个工作人员。

使用使用 `docker-machine ssh` 向你的虚拟机发送指令。使用 `docker swarm init` 指示 `myvm1` 成为一个群组管理员，你将会看到像下面那样的输出：

```
$ docker-machine ssh myvm1 "docker swarm init --advertise-addr <myvm1 ip>"
Swarm initialized: current node <node ID> is now a manager.

To add a worker to this swarm, run the following command:

  docker swarm join \
  --token <token> \
  <myvm ip>:<port>

To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

> 端口：2377 和 2376
总是使用2377端口(群组管理端口)运行`docker swarm init`和`docker swarm join`,或者根本不给出端口号，让他采用默认值。
使用`docker-machine ls`将会返回包括2376端口的机器ip地址，2376端口是Docker后台驻留端口。不要使用这个端口，否则可能会出错。

正如你所看到的， `docker swarm init` 的响应 为你 包含了一个 预置命令 `docker swarm join` 让你添加想要的节点。拷贝这个命令，然后通过`docker-machine ssh`把它发送给 `myvm2`让  `myvm2` 作为工作人员加入你的新群组:

```
$ docker-machine ssh myvm2 "docker swarm join \
--token <token> \
<ip>:2377"

This node joined a swarm as a worker.
```
恭喜你，你已经创建了你的第一个群组。

使用管理员运行`docker node ls`来查看这个集群上的节点。

```
$ docker-machine ssh myvm1 "docker node ls"
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS
brtu9urxwfd5j0zrmkubhpkbd     myvm2               Ready               Active
rihwohkh3ph38fhillhhb84sk *   myvm1               Ready               Active              Leader
```

> 离开群组
如果你想要重新开始，你可以是从每个节点运行`docker swarm leave`

## 将你的应用部署到群集

最难的部分已经结束了。现在你可以重复之前在第三部分的操作，来在新的群组上部署。只要记住只有像myvm1那样的管理员才可以执行Docker命令，工作人员只是提供能力的。

## 给群组管理员配置一个docker-machine 
到目前为止，你一直在 `docker-machine ssh` 打包docker命令 来告诉 虚拟机。另外一个选项就是 运行 `docker-machine env <machine>` 来获取和运行你在当前shell配置的命令 来告诉虚拟机上的Docker守护程序。下面的方法更好用。因为他允许你使用你本地的`docker-compose.yml`文件来远程部署应用，而无需将其拷贝到任何位置。

键入`docker-machine env myvm1` 然后赋值粘贴 输出的最后一行命令 来配置你的 shell 和 群组管理员 myvm1 进行通讯。

环境不同 配置shell的命令也是不一样的 这取决于你的电脑是 Mac  Linux 或是 Windows 下面这个标签有例子。

我只列出了 Mac 和 Linux 的命令

运行 `docker-machine env myvm1` 来获取命令，用来配置你的shell 和 myvm1通讯
```
$ docker-machine env myvm1
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://192.168.99.100:2376"
export DOCKER_CERT_PATH="/Users/sam/.docker/machine/machines/myvm1"
export DOCKER_MACHINE_NAME="myvm1"
# Run this command to configure your shell:
# eval $(docker-machine env myvm1)
```
运行给出的命令来配置shell和myvm1通讯
```
eval $(docker-machine env myvm1)
```

运行 `docker-machine ls` 来检验下 myvm1 是当前激活的机器，后面后面会有*作为标示。

```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER        ERRORS
myvm1   *        virtualbox   Running   tcp://192.168.99.100:2376           v17.06.2-ce   
myvm2   -        virtualbox   Running   tcp://192.168.99.101:2376           v17.06.2-ce   
```

## 在集群管理器上部署应用

既然你已经有了 myvm1 ，你可以使用它作为管理员的能力来部署你的应用。你可以使用和第三部同样的命令 `docker stack deploy`
以及本地复制的 `docker-stack.yml`.

你已经通过`docker-machine` shell配置 连接到了myvm1,你也有可以获取本地主机上的文件。确保你和以前一样，处在同一个目录中。这个目录包含在第三部分创建的` docker-compose.yml`

就像以前那样，运行下面的命令 在 myvm1上部署你的应用

`
docker stack deploy -c docker-compose.yml getstartedlab
`
这样，应用就部署到了一个群组集群上。

现在 你可以使用和第三部分一样的命令。只是这一次，你会看到服务(和相关的容器)已经分配到myvm1和myvm2

```
$ docker stack ps getstartedlab

ID            NAME                  IMAGE                   NODE   DESIRED STATE
jq2g3qp8nzwx  getstartedlab_web.1   john/get-started:part2  myvm1  Running
88wgshobzoxl  getstartedlab_web.2   john/get-started:part2  myvm2  Running
vbb1qbkb0o2z  getstartedlab_web.3   john/get-started:part2  myvm2  Running
ghii74p9budx  getstartedlab_web.4   john/get-started:part2  myvm1  Running
0prmarhavs87  getstartedlab_web.5   john/get-started:part2  myvm2  Running
```

> 使用`docker-machine env`和`docker-machine ssh` 链接虚拟机
>- 要将shell设置为与不同的机器通话 比如myvm2，只需在相同或不同的shell中重新运行docker-machine env，
然后运行给定的命令来指向myvm2。 这只是限定在当前的shell。 如果更改为未配置的shell或打开新的shell，则需要重新运行命令。 
使用docker-machine ls来列出机器，看看它们处于什么状态，获取IP地址，并找出连接到那个（如果存在的话）。 要了解更多信息，请参阅Docker Machine入门主题。
>- 此外，您可以用`docker-machine ssh <machine>“<command>”`的形式封装Docker命令，该命令直接登录到VM中，但不能立即访问本地主机上的文件。
>- 在 Mac 和 Linux 电脑上，你可以使用 `docker-machine scp <file> <machine>:~` 来在机器之间复制文件，但是Windows用户需要用Git Bash这样的Linux终端模拟器才可以。
>
>本教程演示了docker-machine ssh和docker-machine env，因为这些都可以通过docker-machine CLI在所有平台上使用。

## 连接你的集群

您可以从myvm1或myvm2的IP地址访问您的应用程序。

您创建的网络在它们之间共享，并且负载平衡.运行docker-machine ls来获取您的虚拟机的IP地址，并在浏览器上访问它们，点击刷新（或是在终端curl）。
![](https://docs.docker.com/get-started/images/app-in-browser-swarm.png)

您会看到五个可能的容器ID，它们都随机循环，显示了负载平衡。

IP地址工作的原因是群集中的节点参与入口路由网格。这样可以确保在群集中某个端口部署的服务始终将该端口保留给自己，无论实际运行什么节点的容器
下图是个示意图。一个路由网络是如何为服务工作的，这个服务名字叫 my-web 它是一个三节点群组 部署在8080端口上。

![](https://docs.docker.com/engine/swarm/images/ingress-routing-mesh.png)

> 链接有问题？
> 请记住，为了在群集中使用入口网络，在启用群组模式之前，需要在群集节点之间打开以下端口：
> - 端口7946 TCP / UDP用于容器网络发现。
> - 端口4789 UDP用于容器入口网络。

## 迭代和缩放你的应用

在这里，你可以做你在第二部分和第三部分学到的一切东西。

使用更改 `docker-compose.yml` 文件来缩放你的应用。

通过编辑代码，然后重新编译，然后发布新的图像，来改变应用的行为(要做到这一点，请按照您之前采用的步骤来构建应用程序并发布图像）。

无论哪种情况，只需运行docker stack再次部署这些更改。

你可以使用在myvm2上使用的`docker swarm join`命令 来加入任意的机器(物理机，虚拟机)到群组，能力会添加到集群中。

稍后运行docker stack部署，您的应用程序将使用新的资源。

## 清理和重启

### 堆叠和群组

您可以用docker stack rm移除堆叠。 例如：

```
docker stack rm getstartedlab
```

### 卸载 docker-machine shell 变量设置

您可以在当前shell中使用以下命令取消设置docker-machine环境变量：
```
eval $(docker-machine env -u)
```

这和虚拟机创建的docker-machine中是shell断开了链接。并且允许你继续在当前的shell中工作，现在使用本地原生的docker 命令。
要了解更多信息，请参阅关于取消设置环境变量的机器主题。

### 重启 docker 机器
如果你关闭了本地主机，Docker机器就会停止运行。你可以通过 `docker-machine ls`来检查机器的状态

```
$ docker-machine ls
NAME    ACTIVE   DRIVER       STATE     URL   SWARM   DOCKER    ERRORS
myvm1   -        virtualbox   Stopped                 Unknown
myvm2   -        virtualbox   Stopped                 Unknown
```
要重新启动已经停止运行的机器，运行下面的命令：

```
docker-machine start <machine-name>
```

例如：
```
$ docker-machine start myvm1
Starting "myvm1"...
(myvm1) Check network to re-create if needed...
(myvm1) Waiting for an IP...
Machine "myvm1" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.

$ docker-machine start myvm2
Starting "myvm2"...
(myvm2) Check network to re-create if needed...
(myvm2) Waiting for an IP...
Machine "myvm2" was started.
Waiting for SSH to be available...
Detecting the provisioner...
Started machines may have new IP addresses. You may need to re-run the `docker-machine env` command.
```
# 回顾

在第4部分中，您知道了群体是什么，群组中的节点是如何成为管理员或工作人员的，创建群组，并在其上部署应用程序。 

你看到Docker的核心命令没有从第3部分改变，他们只需要在群组主机上运行。 

您也看到了Docker网络的强大功能，即使它们在不同的机器上运行，这些功能也可以在容器之间进行负载平衡。 

最后，您学习了如何在集群上迭代和扩展应用程序。

这里有一些你和群组交互时可能会用到的命令:

```
docker-machine创建--driver virtualbox myvm1＃创建一个虚拟机（Mac，Win7，Linux）
docker-machine create -d hyperv --hyperv-virtual-switch“myswitch”myvm1＃Win10
docker-machine env myvm1＃查看有关您节点的基本信息
docker-machine ssh myvm1“docker node ls”＃列出你的群集中的节点
docker-machine ssh myvm1“docker node inspect <node ID>”＃检查节点
docker-machine ssh myvm1“docker swarm join-token -q worker”＃查看连接令牌
docker-machine ssh myvm1＃与虚拟机打开SSH会话;键入“退出”结束
docker-machine ssh myvm2“docker swarm leave”＃让工作人员离开群集
docker-machine ssh myvm1“docker swarm leave -f”＃让主人离开，关闭群组
docker-machine启动myvm1＃启动当前未运行的虚拟机
docker-machine stop $（docker-machine ls -q）＃停止所有正在运行的虚拟机
docker-machine rm $（docker-machine ls -q）＃删除所有虚拟机及其磁盘映像
docker-machine scp docker-compose.yml myvm1：〜＃将文件复制到节点的home目录
docker-machine ssh myvm1“docker stack deploy -c <file> <app>”＃部署应用程序

```