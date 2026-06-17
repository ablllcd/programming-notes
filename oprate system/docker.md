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
## 镜像
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

### 注意事项
* 不同版本的docker打包image的格式可能不同，导致无法在不同版本的docker之间进行image的导入和导出。

## 容器

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

### 启动/创建时的常见参数
```
-d  // 后台运行
-p [hostPort]:[containerPort]  // 端口映射
-v [hostPath]:[containerPath]  // 目录映射
--network [NetName]  // 指定网络
--env [key]=[value]  // 设置环境变量
```

### 删除容器
```
docker rm [container]  // 删除已停止的容器
docker rm -f [container]  // 强制删除正在运行的容器
docker container prune  // 删除所有已停止的容器
```

### 进入容器

```bash
docker exec -it [container_name] /bin/bash
```

### 查看容器策略
```bash
docker inspect [container_name]  // 查看容器的详细信息，包括网络设置、挂载点等
docker inspect --format='{{.HostConfig.RestartPolicy.Name}}' <容器名或ID> // 查看容器的重启策略
docker inspect --format='{{.Name}} - {{.HostConfig.RestartPolicy.Name}}' $(docker ps -aq) // 查看所有容器的重启策略
```

### 拷贝数据
````
// 拷贝本机数据到容器里
docker cp YOUR_PATH_TO_FOLDER/DBLP-Lab2.tar.gz [containnerName]:/  
````

## Docker 网络

Docker可以为多个容器创建一个虚拟网络，使它们能够相互通信。Docker默认提供了三种网络模式：
1. bridge：默认的网络模式，容器通过NAT连接到主机网络
2. host：容器直接使用主机的网络，容器内的服务可以通过主机IP访问
3. none：容器没有网络连接

### 基本操作
```
docker network create [NetName] //创建一个新的网络
docker network ls   //查看当前网络
docker network rm [NetName]  //删除网络
docker network connect <Net> <contianer>  //将容器连接到指定网络
docker network disconnect <Net> <contianer>  //将容器从指定网络断开
```



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

## Docker Compose文件结构
Docker Compose文件通常命名为docker-compose.yml，包含以下主要部分：
- version: 指定Compose文件的版本
- services: 定义应用程序的服务，每个服务对应一个容器
- volumes: 定义数据卷，用于持久化存储
- networks: 定义自定义网络，允许服务之间通信
- build: 指定构建镜像的上下文和Dockerfile路径
- image: 指定服务使用的镜像，可以是本地镜像或远程仓库中的镜像
- environment: 定义环境变量，传递给容器
- command: 覆盖容器的默认命令，指定容器启动时执行的命令

## Docker Compose和Image的关系
Docker Compose正如名字所述，是多个Docker的集合。其文件中的services部分定义了应用程序的服务，每个服务对应一个容器。

* 每个服务可以指定一个镜像（image）来运行容器，或者指定一个构建上下文（build）来构建镜像。
* 当使用image时，Docker Compose会从指定的镜像创建容器；当使用build时，Docker Compose会根据Dockerfile构建镜像，然后创建容器。
* Docker Compose只有首次运行时会构建镜像，之后会直接使用已经构建好的镜像来创建容器，除非Dockerfile发生了变化或者手动删除了镜像。也可以通过参数`docker compose build`来强制重新构建镜像。
* Docker Compose构建的Image名称默认是`<目录名>_<服务名>`，也可以通过`image`字段指定自定义的镜像名称。其中`<目录名>`是docker-compose.yml所在目录的名称，`<服务名>`是services中定义的服务名称。

## Docker Compose基本操作

### 启动Docker Compose
```
docker compose up  // 前台运行
docker compose up -d  // 后台运行
```

* 如果Service的镜像不存在，Docker Compose会自动构建或拉取镜像。

### 关闭Docker Compose
```
docker compose stop // 只停止容器，不删除
docker compose down // 停止并删除所有容器
docker compose down --rmi all  // 删除所有容器和镜像
docker compose down --v  // 删除所有容器和卷
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

