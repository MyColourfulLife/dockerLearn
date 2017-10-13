
## 服务
创建docker-compose.yml用于定义Docker容器在生产过程中的行为。

运行新的负载平衡应用程序
在我们可以使用docker stack deploy命令之前，我们先运行：

docker swarm init

报错。。。
说有多个地址，因此需要指定一个
has multiple addresses on different interfaces (10.0.2.15 on eth0 and 192.168.99.100 on eth1)

我从上面的两个ip选了一个

```
docker swarm init --advertise-addr 10.0.2.15
```

然后可以了

现在我们来运行它，你必须给你的应用程序一个名字。在这里，它设置为 getstartedlab：
```
docker stack deploy -c docker-compose.yml getstartedlab
```
 `docker service ls` 查看下 所运行的 服务 我们可以拿到服务的id
还列出了服务ID，以及副本数，图像名称和外露端口数。


```
docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                          PORTS
qex3qubm8fne        getstartedlab_web   replicated          5/5                 1107660581/get-started:part2   *:80->80/tcp
```

查看下服务中运行的任务 `docker service ps <service>`

```
docker service ps qex3qubm8fne
ID                  NAME                  IMAGE                          NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
342h35ba4xus        getstartedlab_web.1   1107660581/get-started:part2   default             Running             Running 3 minutes ago                       
nujvykr21fdb        getstartedlab_web.2   1107660581/get-started:part2   default             Running             Running 3 minutes ago                       
6oi16c888fee        getstartedlab_web.3   1107660581/get-started:part2   default             Running             Running 3 minutes ago                       
nrw85fczly70        getstartedlab_web.4   1107660581/get-started:part2   default             Running             Running 3 minutes ago                       
0q8qkqlxarie        getstartedlab_web.5   1107660581/get-started:part2   default             Running             Running 3 minutes ago  
```

我们检查这些任务之一，并将输出限制为容器ID：
`docker inspect --format='{{.Status.ContainerStatus.ContainerID}}' <task>`

```
docker inspect --format='{{.Status.ContainerStatus.ContainerID}}' 342h35ba4xus
//输出
387cf02e6dd1e36314c91772238914cad9062591e8180a8c9e4378f7298efec3
```
反之亦然，您可以检查容器ID，并提取任务ID。

首先运行docker container ls获取容器ID，
```
docker container ls
CONTAINER ID        IMAGE                          COMMAND             CREATED             STATUS              PORTS               NAMES
c99d8cf3bc8a        1107660581/get-started:part2   "python app.py"     6 minutes ago       Up 6 minutes        80/tcp              getstartedlab_web.3.6oi16c888feee18bji3k3fpes
387cf02e6dd1        1107660581/get-started:part2   "python app.py"     6 minutes ago       Up 6 minutes        80/tcp              getstartedlab_web.1.342h35ba4xus31pes5g395nox
2ac08cb7ba3a        1107660581/get-started:part2   "python app.py"     6 minutes ago       Up 6 minutes        80/tcp              getstartedlab_web.4.nrw85fczly70kvvtyu4bf2sfg
a2f7b8553585        1107660581/get-started:part2   "python app.py"     6 minutes ago       Up 6 minutes        80/tcp              getstartedlab_web.5.0q8qkqlxarie3jvjjdm3o5cw8
786b2a2969e3        1107660581/get-started:part2   "python app.py"     6 minutes ago       Up 6 minutes        80/tcp              getstartedlab_web.2.nujvykr21fdbym5x4d4kcgoz2
```

然后`docker inspect --format="{{index .Config.Labels \"com.docker.swarm.task.id\"}}" <container>`

```
docker inspect --format="{{index .Config.Labels \"com.docker.swarm.task.id\"}}" c99d8cf3bc8a

输出
6oi16c888feee18bji3k3fpes

```

现在列出所有5个容器
```
docker container ls -q
输出：
c99d8cf3bc8a
387cf02e6dd1
2ac08cb7ba3a
a2f7b8553585
786b2a2969e3

```

可以更改docker-compose.yml中replicasz值 
保存更改并重新运行docker stack deploy来缩放应用 不需要停止或杀死运行任性容器
重新运行docker container ls -q以重新配置已部署的实例

卸载应用和群组

卸载应用

docker stack rm getstartedlab

卸载了应用但是还有一个群组在运行
可以通过`docker node ls`查看

卸载群组
```
docker swarm leave --force
```

这一节常用的命令
```
docker stack ls                                            # List stacks or apps
docker stack deploy -c <composefile> <appname>  # Run the specified Compose file
docker service ls                 # List running services associated with an app
docker service ps <service>                  # List tasks associated with an app
docker inspect <task or container>                   # Inspect task or container
docker container ls -q                                      # List container IDs
docker stack rm <appname>                             # Tear down an application
```
---
