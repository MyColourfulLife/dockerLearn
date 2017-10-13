## 创建图像并发布

1. 定义一个 Dockerfile文件

这东西感觉跟 js的 package.json差不多

因为官方给的是Python的例子，所以。。。你懂得，就先用Python，暂时也不必知道是文件内的Python代码是干啥的，重要的是走一遍，对整个流程有个印象

```
mkdir Dockerlearn #创建一个空文件夹 用来表示项目 目录
cd Dockerlearn/ # 切换到工作路径
vi Dockerfile # 创建Dockerfile文件，直接把代码拷贝进去保存
```

2. 创建  requirements.txt 和 app.py 文件
```
vi requirements.txt #创建requirements文件并进入vi编辑器 然后把文档中的例子拷贝进去 
vi app.py #创建app文件并进入vi编辑器 然后把文档中的例子拷贝进去 
```

3. 编译
根据提示 编译一下
```
docker build -t friendlyhello .
```

竟然成功了。😄。

查看加生成的image 使用images命令或者 image lsm命令

```
docker images
打印结果

REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
friendlyhello       latest              785d3267803b        8 minutes ago       195MB

```

4. 运行应用程序

运行应用程序，将您的计算机端口4000映射到容器发布的端口80，方法-p如下：

```
docker run -p 4000:80 friendlyhello
```

我的终端只显示了一下信息
```
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)

 ```

 访问 http://0.0.0.0:80 和 http://localhost:4000 都不行啊 😓

 好吧，退出来吧。

 后台运行 获取长容器id
 ```
 docker run -d -p 4000:80 friendlyhello
 ```
 可以使用`docker container ls`查看缩写的容器id

 可以使用`docker container stop`来结束这个过程 stop的参数 就是刚才看到的容器id
 ```
 docker container stop 7e7270bc7c4c
 ```

5. 分享您的image

分享您的image使其可以在其他地方运行。
当需要将容器部署到生产环境时，需要了解 如何推送到注册表

> 注册表是存储库的集合，存储库是图像的集合
注册表上的帐户可以创建许多存储库
我们使用Docker的公共注册表，因为它是免费和预先配置的 。

5.1 如果您没有Docker帐户，请在[cloud.docker.com](https://cloud.docker.com/)注册一个 。记下您的用户名。

顺便说下，你注册时 可能想到了很多好名字，可惜呀，可惜！都被别人占用了😄。所以用手机号或者QQ号啥的 应该就没啥问题。

然后在终端登录 
```
docker login
接下来要输入用户名和密码
登录成功会打印Login Succeeded

```

将本地映像与注册表上的存储库关联 并标记一个有意义的名字
语法·docker tag image username/repository:tag
1107660581 是我在docker注册的账号 get-started是我在上面建立的仓库
docker tag friendlyhello 1107660581/get-started:part2


打印一下看看
```
docker image ls
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
1107660581/get-started   part2               785d3267803b        2 hours ago         195MB
friendlyhello            latest              785d3267803b        2 hours ago         195MB
```

6. 发布图像
将标记的图像上传到存储库

语法`docker push username/repository:tag`

```
docker push 1107660581/get-started:part2

```

一旦完成，这个上传的结果是公开的。如果您登录[Docker Hub](https://hub.docker.com/)，则可以使用pull命令查看新图像。


7. 从远程存储库拉过来运行

`docker run -p 4000:80 username/repository:tag`

```
docker run -p 4000:80 1107660581/get-started:part2
```

然后竟然出错了 ，原因是4000端口已经被用了。这就是上面在后面运行导致的。
使用docker container ls 可以看到 这个容器还在运行 ，因此要杀掉他

```
docker container kill 7e7270bc7c4c
```

docker 的常用命令

```
docker build -t friendlyname .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyname  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyname         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```
