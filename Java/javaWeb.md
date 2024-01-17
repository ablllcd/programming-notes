## Tomcat

### 目录结构
![Alt text](pic/tomcatDir.png)

### 启动服务器
打开bin目录下的startup.bat来启动。

### 修改端口号
修改conf/server.xml文件
````
<Connector port="9000" protocol="HTTP/1.1"
            connectionTimeout="20000"
            redirectPort="8443"
            maxParameterCount="1000"
            />
````

### 部署项目
要将项目部署到服务器上，只需要将项目放到webapps目录下即可

### 注意
springboot中的起步依赖也包含了tomcat，但那个相当于又下载了一个`内嵌tomcat`，与这个单独下载的不是同一个程序。