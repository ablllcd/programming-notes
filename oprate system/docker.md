# 安装

DOCKER官网有关于WINDOWS下安装的教程

1. 首先保证WSL的版本并且安装一个LINUX虚拟机

```
wsl --update    
wsl --install
```

2. 从官网下载docker安装器进行安装即可


# 理论
## 简介
docker是一个类似虚拟机的程序，它可以创建很多容器（contianer），每个容器都是独立的虚拟环境，可以在其中安装程序并运行。

此外，docker还有一个重要的概念：镜像(image)。镜像是一个静态模板，它包含了应用程序和程序依赖的环境。我们可以从一个镜像中创建一个容器来布置其指定的环境并运行程序。

Docker也有自己的仓库来储存镜像。

Docker分别提供了GUI和命令行来允许用户使用。

# 基本操作
## 镜像操作
````
// 查看镜像
docker images

// 下载删除
docker pull [image]
docker rmi [image]

// image打包 和 导入
docker save -o [targetPath] [image]
docker load -i [inputPath]
````

## 容器操作

### 查看容器
```
docker ps   // 只查看已激活的
docker ps -a  //查看所有容器
docker logs [container] //查看容器日志
```

### 创建/删除/启动/停止容器
```
docker run --name contianerName imageName  // 创建并启动一个容器
docker start container  //启动已存在的容器
docker rm  contianer    //删除container
docker stop container   //停止container
docker kill container   //强制停止
```

#### 启动/创建时的常见参数
```
-d  // 后台运行
-p [hostPort]:[containerPort]  // 端口映射
-v [hostPath]:[containerPath]  // 目录映射
--network [NetName]  // 指定网络
--env [key]=[value]  // 设置环境变量
```

### 进入容器

```bash
docker exec -it [container_name] /bin/bash
```


#### 其它操作
````
// 拷贝本机数据到容器里
docker cp YOUR_PATH_TO_FOLDER/DBLP-Lab2.tar.gz [containnerName]:/  

// 运行容器的shell
docker exec -it mycassandra /bin/sh

// 虚拟网络，允许container相互交流
docker network create [NetName]
docker network ls   //查看当前网络
docker network connect <Net> <contianer>  //添加到网络

````


## 修改DOCKER存储位置

https://www.jianshu.com/p/1ae32787fb14

此外可以在docker设置中查看docker image 的存储位置

## 更改安装路径
在安装Dokcer过程中，发现没有让选择安装到哪个目录下，总是安装到C盘。

解决方法：
````
mklink /J "C:\Program Files\Docker" "D:\Programming\Docker"
````
这创建了一个C盘到D盘的软连接，当软件依旧默认安装到C盘时，它会安装到软连接的文件夹中，实际文件储存在D盘。

前提： "C:\Program Files\Docker不存在（由软连接命令创建）；D:\Programming\Docker存在。

# Docker Compose
Docker Compose是一个工具，用于定义和运行多容器Docker应用程序。通过使用YAML文件来配置应用程序的服务，Compose可以轻松地管理和部署多个容器。

示例
```yaml
services:
  nginx:
    build: .
    ports:
      - "8080:80"
    depends_on:
      - app

  app:
    image: eclipse-temurin:21-jre
    working_dir: /app
    volumes:
      - ./app.jar:/app/app.jar:ro
      - ./application.yml:/app/config/application.yml:ro
    command: ["java","-jar","/app/app.jar","--spring.config.location=file:/app/config/application.yml"]
    depends_on:
      - mysql

  mysql:
    image: mysql:8.0
    environment:
      MYSQL_DATABASE: bcgw
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

## 特别注意

### depends_on 不保证服务完全启动

depends_on：指定服务之间的依赖关系，确保在启动一个服务之前，先启动它所依赖的服务。 但是，它并不保证依赖服务已经完全启动并准备好接受连接！

解决方法：在应用程序中添加重试机制，或者使用健康检查来确保依赖服务已经准备好。

例如，在app服务中添加一个重试机制，直到mysql服务准备好：

```yaml
  app:
    image: eclipse-temurin:21-jre
    working_dir: /app
    volumes:
      - ./app.jar:/app/app.jar:ro
      - ./application.yml:/app/config/application.yml:ro
    command: ["java","-jar","/app/app.jar","--spring.config.location=file:/app/config/application.yml"]
    depends_on:
      - mysql
```


### volume和容器内映射路径的关系

* 当volume为空时，volume会被容器内映射路径上的内容给初始化
* 当volume不为空时，volume会覆盖容器内映射路径上的内容

## 存储

将Docker Compose中的镜像进行存储和加载：

```bash
# 存储镜像
docker save -o [targetPath] [image]
# 加载镜像
docker load -i [inputPath]
```

