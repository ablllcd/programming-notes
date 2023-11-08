## 简介
docker是一个类似虚拟机的程序，它可以创建很多容器（contianer），每个容器都是独立的虚拟环境，可以在其中安装程序并运行。

此外，docker还有一个重要的概念：镜像(image)。镜像是一个静态模板，它包含了应用程序和程序依赖的环境。我们可以从一个镜像中创建一个容器来布置其指定的环境并运行程序。

Docker也有自己的仓库来储存镜像。

Docker分别提供了GUI和命令行来允许用户使用。

## 命令
#### 查看已有镜像
````
docker images
````

#### 查看容器
````
docker ps   // 只查看已激活的
docker ps -a
````

#### 操作容器
````
docker run --name contianerName imageName //创建container来放置image并运行

docker start container
docker stop container
docker kill container //强制停止

docker rm  contianer    //删除container
````