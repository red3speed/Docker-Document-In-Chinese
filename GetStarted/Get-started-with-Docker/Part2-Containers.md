# 第二部分：容器

## 先决条件
- 安装v1.13或更高版本的Docker。
- 阅读第一部分。
- 测试你的环境是否设置完毕：

	```
	docker run hello-world
	```

## 介绍
是时候开始用Docker的方式构建一个应用了。我们将从这样一个应用的层级的底部开始，什么是容器，这就是我们在这个页面涉及的内容。在这之上的层级是服务，它定义了在生产上容器该怎么运作，在第三部分设计。最后，最顶层是栈，定义了所有服务间的交互，在第五部分中涉及。
	- 栈
	- 服务
	- 容器（你在这儿）

## 您的新部署环境
过去，如果你想开始写一个Python应用，你首先要在你的机器上安装Python的运行时。但是这导致了一种状况，那就是你的机器环境必须如此以便于你的应用能够如预期般执行；对于运行你的应用的服务也是如此。

借由Docker，你可以仅获取一个便携的Python运行时镜像而不需要安装。然后你的构建时可以将Python的镜像和你的应用代码包含在一起，以确保你的应用、它的依赖、运行时都组合在一起。

这些便携的镜像通过叫做`Dockerfile`的文件定义。

## 通过`Dockerfile`定义一个容器
`Dockerfile `将定义你容器中的环境。访问资源（像是网络接口和磁盘驱动）在环境中是虚拟化的，它与你系统的其他部分隔离，所以你要与外部端口进行映射，并且要指定哪些文件你要“复制”进容器环境。但是，完成这些之后，不管在哪运行在这个`Dcokerfile`中定义的你的应用的构建，它都将保持一样的行为。
### `Dockerfile`
新建一个空的目录。跳转到该新目录，创建一个叫`Dockerfile`的文件，将下列内容复制粘贴进该文件后保存。注意这些注释解释了`Dockerfile`中的每个语句。

```
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```
这个`Dockerfile`引用了`app.py`、`requirements.txt`这两个我们还没创建的文件。接下来让我们创建它们。

## 应用自身
新建两个文件，`app.py`和`requirements.txt`，然后把它们放到和`Dockerfile`相同的目录下。这样就完成了我们的应用，如你所见非常的简单。当上述的`Dockerfile`构建成一个镜像时，由于`Dockerfile`的`ADD`命令，`app.py`和`requirements.txt`也将被放入，并且由于`EXPOSE`命令，`app.py`的输出也能通过`HTTP`访问到。

### requirements.txt
```
Flask
Redis
```
### app.py
```
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

现在我们可以看到通过`pip install -r requirements.txt`命令安装Python的Flask和Redis库，之后应用会打印环境变量`NAME`和`socket.gethostname()`。最后，由于Redis没有在运行（我们仅安装了Redis的Python依赖库，而没有安装它本身），调用Redis的部分将会失败并且返回错误信息。

> 注意: 在容器内部获取主机的名称将返回容器的ID，类似可执行程序的进程ID。

你的系统不需要安装Python或者任何在`requirements.txt`定义的东西，也不必构建或运行镜像来安装它们。你虽然没有真正安装Python和Flask，但你想当于拥有了这样的环境。

## 构建应用
我们已经可以构建应用了。确保你仍在那个新建目录下。调用`ls`命令将显示：
```
$ ls
Dockerfile		app.py			requirements.txt
```
现在执行构建的命令。这将会生成一个Docker镜像，并且我们将通过`-t`参数来指定一个友好的别名。
```
docker build -t friendlyhello .
```
构建的镜像会在你的机器的本地Docker镜像registry中：
```
$ docker images

REPOSITORY            TAG                 IMAGE ID
friendlyhello         latest              326387cea398
```
## 运行应用
运行应用，通过`-p`参数映射你本机的4000端口到容器的发布的端口80：
```
docker run -p 4000:80 friendlyhello
```
你会注意到Python服务的地址是`http://0.0.0.0:80`。但是这是在容器内部的消息，它无法知道你将容器的80端口映射到了本机的4000端口，所以你要通过`http://localhost:4000`来访问它。

通过浏览器访问该URL来查看展示在页面上的信息，包括“Hello World”文本、容器ID以及Redis错误信息。

![浏览器页面](images/app-in-browser.png)

你也可以在shell中使用`curl`命令看到相同的内容。

```
$ curl http://localhost:4000

<h3>Hello World!</h3><b>Hostname:</b> 8fc990912a14<br/><b>Visits:</b> <i>cannot connect to Redis, counter disabled</i>
```
> 注意：`4000:80`的端口重映射只是示范`Dockerfile`中定义的`EXPOSE`和你执行`docker run -p`中发布的端口的不同。在后面的步骤中，我们只需要映射本机的80端口到容器的80端口，然后使用http://localhost来访问。

在终端中按下`CTRL+C`来终止curl。

现在让我们通过后台模式(-d detached)在后台运行应用，

```
docker run -d -p 4000:80 friendlyhello
```
你会看到返回一长串的容器ID然后退回到你的终端。你的容器就运行在后台了。你也可以通过`docker container ls`命令看到缩写的容器ID(该命令在非后台模式运行情况下也有效)。

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND             CREATED
1fa4ab2cf395        friendlyhello       "python app.py"     28 seconds ago
```
你可以发现`CONTAINER ID`和在`http://localhost:4000`上展示的ID是对应的。

现在我们使用`docker stop`命令和`CONTAINER ID`来终止进程：

```
docker stop 1fa4ab2cf395
```
## 分享你的镜像
现在让我们上传我们刚才构建的镜像而后在其他地方运行它，来展示它的便携性。毕竟，你需要学习怎么推送到注册表，以便部署容器到生产上。

一个registry是仓库(repository)的集合，一个仓库是镜像的集合(像是GitHub上的代码仓库)。一个账号在registry可以创建多个仓库。docker的命令行接口默认使用Docker的公共registry。

> 注意：我们将使用Docker的公共registry，它是免费且预设置好的。有许多的公有仓库可供选择，你甚至可以通过[`Docker Trusted Registry`](https://docs.docker.com/datacenter/dtr/2.2/guides/)设置你的私有的registry。

### 通过你的Docker ID登录
你过你还没有Docker账号，请在[cloud.docker.com](https://cloud.docker.com)上注册。记住你的用户名。

在本机上登录到Docker的公有registry。

```
docker login
```
### 给镜像打标签
要把一个本地镜像和在registry上的仓库关联起来的标记是`username/repository:tag`。tag是可选的，但我们建议设置它，因为它是registry判断镜像版本的途径。给repository和tag赋一个有意义的名称，例如`get-started:part1`。它将会将镜像放到`get-started`仓库中，并打上`part1`标签。

现在给你的镜像打上标签。执行`docker tag image`设置username、repository和tag，上传到你期望的位置。命令语法如下：

```
docker tag image username/repository:tag
```
例如：

```
docker tag friendlyhello john/get-started:part1
```

运行`docker images`来观察你刚打过标签的镜像(或者`docker image ls`)。

```
$ docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
friendlyhello            latest              d9e555c53008        3 minutes ago       195MB
john/get-started         part1               d9e555c53008        3 minutes ago       195MB
python                   2.7-slim            1c7128a655f6        5 days ago          183MB
...
```
### 发布镜像
上传打过标签的镜像到仓库中：

```
docker push username/repository:tag
```
完成后，这个镜像就变成了公有的。如果你登录[Docker Hub](https://hub.docker.com)，你可以看到新上传的镜像和它的pull命令。

### 拉取并执行远程仓库上的镜像
现在，你可以在任意一台机器上执行`docker run`命令来运行你的应用。

```
docker run -p 4000:80 username/repository:tag
```
如果该镜像不在你的机器上，Docker将会从仓库上下载它。

```
docker image rm <image id>
```

```
$ docker run -p 4000:80 john/get-started:part1

Unable to find image 'john/get-started:part1' locally
part1: Pulling from john/get-started
10a267c67f42: Already exists
f68a39a6a5e4: Already exists
9beaffc0cf19: Already exists
3c1fe835fb6b: Already exists
4c9f1fa8fcb8: Already exists
ee7d8f576a14: Already exists
fbccdcced46e: Already exists
Digest: sha256:0601c866aab2adcc6498200efd0f754037e909e5fd42069adeff72d1e2439068
Status: Downloaded newer image for john/get-started:part1
 * Running on http://0.0.0.0:80/ (Press CTRL+C to quit)
```
> 注意：如果你没有指定`:tag`，那在你构建和运行镜像时将会使用`:latest`标签。当没有指定标签时，Docker会使用镜像的最后一个版本(不一定是最近的镜像)。

不管`docker run`在哪里执行，它都会拉取你的镜像、Python和`requirements.txt`中定义的所有依赖，然后执行你的代码。它们都被整合在一个包中，主机不用安装任何东西但Docker仍能运行它。

## 小结
在下一节中，我们将学习如何通过在服务中运行容器来伸缩我们的应用规模。

## 备忘单
下面是这个页面中提及的基础的Docker命令，在继续下一节之前,你可以研究其中一些相关的命令。

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