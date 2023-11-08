## 更改安装路径
在安装Dokcer过程中，发现没有让选择安装到哪个目录下，总是安装到C盘。

解决方法：
````
mklink /J "C:\Program Files\Docker" "D:\Programming\Docker"
````
这创建了一个C盘到D盘的软连接，当软件依旧默认安装到C盘时，它会安装到软连接的文件夹中，实际文件储存在D盘。

前提： "C:\Program Files\Docker不存在（由软连接命令创建）；D:\Programming\Docker存在。