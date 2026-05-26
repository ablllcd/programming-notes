## 参考教程

https://www.runoob.com/linux/linux-shell.html

# 常用操作

## 系统操作

### 常用操作
```bash
#关机
shutdown -h now  # 立即关机
shutdown -h +10  # 10分钟后关机
reboot  # 重启系统
```

### 查看系统信息
1. 查看操作系统版本
   ```bash
   cat /etc/os-release  # 显示操作系统版本信息
   lsb_release -a        # 显示Linux标准基础信息
   hostnamectl           # 显示主机信息，包括操作系统版本
   ```

### 时间相关
1. 查看当前时间
   ```bash
   date  # 显示当前日期和时间
   date +%Y-%m-%d  # 显示当前日期
   date +%H:%M:%S  # 显示当前时间
   ```
2. 查看当前时间配置
   ```bash
   timedatectl  # 显示当前时间、时区和NTP状态
   ```
3. 列出可用时区
   ```bash
    timedatectl list-timezones  # 列出所有可用的时区
   ```
4. 修改时区
   ```bash
    sudo timedatectl set-timezone Asia/Shanghai  # 设置时区为上海
   ```
5. date进行时间格式转化
    ```bash
    # 将时间戳转换为UTC格式     
     date -u -d @<timestamp> +"%Y-%m-%dT%H:%M:%SZ"
    ```

### 查看系统日志
```bash
journalctl  # 显示系统日志
journalctl -u service_name  # 显示指定服务的日志
journalctl -f  # 实时跟踪日志输出
journalctl --since "2024-01-01" --until "2024-01-31"  # 显示指定日期范围内的日志
journalctl -n 100  # 显示最近的100条日志
journalctl -k # 显示内核日志
```

### 查看操作日志
```bash
last  # 显示最近的登录记录
last reboot  # 显示系统重启记录
history # 显示命令历史记录
```

### systemctl
systemctl 是 Linux 系统中用于 控制 systemd 系统和服务管理器 的命令行工具。而systemd 是一种初始化系统（init system），负责在系统启动时启动服务、挂载文件系统、管理日志等。systemctl 就是与 systemd 交互的工具。

通过 systemctl 管理的服务是系统服务（daemon），通常在后台运行，提供持续的功能，比如数据库、Web 服务器、网络服务等。
其特点为：
* 后台运行
* 需要系统启动时自动启动
* 由 systemd 管理生命周期（启动、停止、重启等）

**常用命令**
```bash
sudo systemctl start service_name    # 启动服务
sudo systemctl stop service_name     # 停止服务
sudo systemctl restart service_name  # 重启服务
sudo systemctl enable service_name   # 设置服务开机自启
```

## 环境变量

### CRUD环境变量
```bash
export VAR_NAME=value  # 创建环境变量 VAR_NAME 并赋值为 value
echo $VAR_NAME         # 查看环境变量 VAR_NAME 的值
export VAR_NAME=new_value  # 更新环境变量 VAR_NAME 的值为 new_value
unset VAR_NAME         # 删除环境变量 VAR_NAME
```

### 查找环境变量
```bash
printenv | grep VAR_NAME  # 查找包含 VAR_NAME 的环境变量
env | grep VAR_NAME       # 查找包含 VAR_NAME 的环境变量
```

### 应用环境变量
`export` 命令用于在当前 shell 环境中设置环境变量，使其在当前 shell 以及其子进程中可用。所谓的子进程是指由当前 shell 启动的任何程序或脚本。
```
export VAR_NAME=value  # 设置环境变量 VAR_NAME 的值为 value
```



## 文本操作

### grep命令
`grep` 是 Linux 系统中用于在文本文件中搜索特定字符串或模式的命令行工具。它可以根据用户提供的模式（通常是正则表达式）在文件中查找匹配的行，并将这些行输出到终端。
**常用选项**
- `-i`：忽略大小写进行匹配。
- `-r`：递归搜索目录中的文件。

## 磁盘操作

### 查看磁盘用量
1. `df` 命令 - 显示文件系统磁盘空间使用情况
   ```bash
   df -h  # 以人类可读的方式显示（GB、MB等）
   df -T  # 显示文件系统类型
   df -i  # 显示inode信息而非块使用量
   ```

2. `du` 命令 - 估算文件和目录的磁盘使用量
   ```bash
   du -h /path  # 以人类可读方式显示指定目录的大小
   du -sh *     # 显示当前目录下各文件和目录占用空间
   du -sh /path # 显示指定目录总大小
   du -h --max-depth=1 /path  # 仅显示第一级子目录的大小
   ```

3. `fdisk` 命令 - 查看磁盘分区
   ```bash
   sudo fdisk -l  # 列出所有磁盘的分区表
   ```

4. `lsblk` 命令 - 以树状列出所有块设备
   ```bash
   lsblk         # 列出所有块设备
   lsblk -f      # 显示文件系统信息
   ```

5. `ncdu` 命令 - 交互式磁盘使用分析器
   ```bash
   # 安装：sudo apt-get install ncdu
   ncdu /path    # 分析指定路径的磁盘占用
   ```

## 文件操作

### 文件权限
```bash
ll  # 显示文件权限和所有者信息
chmod 755 filename  # 设置文件权限为 rwxr-xr-x
chmod -r 755 directory  # 递归设置目录及其内容的权限
chmod u+x filename  # 给文件所有者添加执行权限
chmod g-w filename  # 移除文件所属组的写权限
chmod o+r filename  # 给其他用户添加读权限
chown user:group filename  # 更改文件所有者和所属组
```
* 权限分为三类：所有者（user）、所属组（group）和其他用户（others）。每类权限可以分别设置读（r）、写（w）和执行（x）权限。
* 权限的数值表示：读（r）、写（w）和执行（x）。每个权限对应一个数字：读=4，写=2，执行=1。权限的总和决定了文件的权限设置。例如，755表示所有者有读、写、执行权限（4+2+1=7），而组和其他用户只有读和执行权限（4+1=5）。
* chmod就是用来修改文件权限的命令，chown用来修改文件的所有者和所属组。

### 统计文件夹
```bash
find /path/to/directory -type f | wc -l  # 统计指定目录下的文件数量
# 显示文件数量 
```

### 移动和重命名文件
```bash
mv oldname newname  # 重命名文件或移动文件到新位置
mv filename /path/to/directory/  # 移动文件到指定目录
```

### 复制文件
```bash
cp source_file destination_file  # 复制文件
cp -r source_directory destination_directory  # 递归复制目录
```

### 查找文件
```bash
find /path/to/search -name "filename"  # 在指定路径下查找文件
find /path/to/search -type f -name "*.txt"  # 查找指定路径下的所有 .txt 文件
```

## 下载/安装/解压操作

### 压缩文件
```bash
tar -cvf file.tar /path/to/directory  # 将目录压缩为 .tar 文件
tar -czvf file.tar.gz /path/to/directory  # 将目录压缩为 .tar.gz 文件
tar -cjvf file.tar.bz2 /path/to/directory  # 将目录压缩为 .tar.bz2 文件
zip -r file.zip /path/to/directory  # 将目录压缩为 .zip 文件
rar a file.rar /path/to/directory  # 将目录压缩为 .rar 文件 
7z a file.7z /path/to/directory  # 将目录压缩为 .7z 文件
xz -z file  # 将文件压缩为 .xz 文件
gzip file  # 将文件压缩为 .gz 文件
```

### 解压文件

1. 解压 `.tar` 文件
    ```bash
    tar -xvf file.tar
    ```

2. 解压 `.tar.gz` 或 `.tgz` 文件
    ```bash
    tar -xzvf file.tar.gz
    ```

3. 解压 `.tar.bz2` 文件
    ```bash
    tar -xjvf file.tar.bz2
    ```

4. 解压 `.zip` 文件
    ```bash
    unzip file.zip
    ```

5. 解压 `.rar` 文件
    ```bash
    unrar x file.rar
    ```

6. 解压 `.7z` 文件
    ```bash
    7z x file.7z
    ```

7. 解压 `.xz` 文件
    ```bash
    xz -d file.xz
    ```

8. 解压 `.gz` 文件
    ```bash
    gunzip file.gz
    ```

### 使用 wget 下载文件
```bash
wget http://example.com/file.zip  # 下载指定URL的文件
wget -c http://example.com/file.zip  # 断点续传下载 
wget -r http://example.com/dir/  # 递归下载目录
```

### 使用apt-get下载软件包
```bash
sudo apt-get update  # 更新软件包列表
sudo apt-get install package_name  # 安装指定软件包
sudo apt-get remove package_name  # 卸载指定软件包
sudo apt-get upgrade  # 升级已安装的软件包
sudo apt-get dist-upgrade  # 升级系统，包括内核和依赖关系
```

### 安装软件包
```bash
sudo dpkg -i package.deb  # 安装 .deb 软件包
sudo rpm -i package.rpm  # 安装 .rpm 软件包
```


## 网络操作

### 配置网络IP
```bash
dhclient # 获取动态IP地址 & 临时生效
```

#### netplan配置静态IP
在 /etc/netplan/ 目录下创建一个 .yaml 文件，内容如下：
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:[8.8.8.8, 8.8.4.4]
```
然后运行以下命令应用配置：
```bash
sudo netplan apply
```

### 查看网络配置
```bash
ifconfig          # 显示网络接口配置
ip addr show      # 显示网络接口的详细信息
ip link show      # 显示网络接口的状态
ip a              # 显示所有网络接口信息
ip route show      # 显示路由表
ip route | grep default  # 显示默认路由
```

### 查看端口占用

1. `netstat` 命令 - 显示网络连接、路由表、接口统计等
    ```bash
    netstat -tulpn  # 显示所有监听的端口
    ```
    - 参数说明:
        - `-t`: 显示TCP连接
        - `-u`: 显示UDP连接
        - `-l`: 显示监听状态的端口
        - `-p`: 显示进程ID和名称
        - `-n`: 显示数字地址而不解析主机


2. `ss` 命令 - 更快的网络状态查看工具
    ```bash
    ss -tuln       # 显示所有监听的端口
    ss -anp        # 显示所有端口及其对应的进程
    ```

3. `lsof` 命令 - 列出打开的文件，包括网络端口
    ```bash
    lsof -i :port  # 查看指定端口的占用情况
    lsof -i        # 查看所有网络连接
    ```

4. `fuser` 命令 - 显示使用指定端口的进程
    ```bash
    fuser -n tcp port  # 查看指定TCP端口的占用情况
    ```

### 查看网卡信息
```bash
ip a # 显示所有网络接口信息
ip a | grep TARGET_IP # 查找包含指定IP地址的网络接口信息
```

### 网络抓包
```bash
sudo tcpdump -i eth0  -w capture.pcap  # 在eth0接口上抓包并保存到capture.pcap文件

# 抓目的为 xxx、端口为 yy 的包，并把 payload 也打印（适合 HTTP）
sudo tcpdump -i any -nn -s0 -A "dst host xxx and dst port yy"
```

## 进程操作

### 查看进程以及资源占用

1. `ps` 命令 - 显示当前进程信息
    ```bash
    ps            # 显示当前用户的进程
    ps aux        # 显示所有进程的详细信息
    ps -ef        # 显示所有进程的完整格式
    ```

2. `top` 命令 - 实时显示系统资源使用情况
    ```bash
    top           # 显示实时进程和资源使用情况
    htop          # 更友好的交互式界面（需安装：sudo apt-get install htop）
    ```

3. `pidstat` 命令 - 显示进程的资源使用情况
    ```bash
    pidstat -u    # 显示进程的CPU使用情况
    pidstat -r    # 显示进程的内存使用情况
    pidstat -d    # 显示进程的I/O使用情况
    ```

4. `vmstat` 命令 - 显示系统性能统计
    ```bash
    vmstat 1      # 每秒更新一次系统性能统计
    ```

5. `iostat` 命令 - 显示CPU和磁盘I/O使用情况
    ```bash
    iostat -x     # 显示详细的CPU和磁盘I/O使用情况
    ```

6. `sar` 命令 - 收集、报告系统活动
    ```bash
    sar -u 1      # 每秒显示一次CPU使用情况
    sar -r 1      # 每秒显示一次内存使用情况
    ```

7. `pmap` 命令 - 显示进程的内存映射
    ```bash
    pmap pid      # 查看指定进程的内存使用情况
    ```

### 查找进程
1. 根据端口号查找进程
    ```bash
    lsof -i :port  # 查找占用指定端口的进程
    netstat -tulpn | grep :port  # 查找占用指定端口的进程
    ss -tuln | grep :port  # 查找占用指定端口的进程
    ```

### 终止进程
1. 使用 `kill` 命令终止进程
    ```bash
    kill pid              # 发送默认的TERM信号终止进程
    kill -9 pid           # 强制终止进程
    ```
2. 使用`pkill`命令根据进程名称终止进程
    ```bash
    pkill process_name     # 根据进程名称终止进程
    pkill -9 process_name  # 强制根据进程名称终止进程
    ```

## GPU相关操作

### 查看GPU信息
```bash
nvidia-smi  # 显示NVIDIA GPU的状态和使用情况
```
openebs-3.3.1
## 文本编辑器

### nano
```bash
Ctrl + K  # 剪切当前行
Ctrl + U  # 粘贴剪切的行
Ctrl + O  # 保存文件
Ctrl + X  # 退出nano
alt + G  #  输入行号，跳转到指定行
```

#### 查找关键词
```
Ctrl + W  # 查找关键词
```

# Shell相关

## 环境变量
### CRUD环境变量
```bash
export VAR_NAME=value  # 创建环境变量 VAR_NAME 并赋值为 value
echo $VAR_NAME         # 查看环境变量 VAR_NAME 的值
export VAR_NAME=new_value  # 更新环境变量 VAR_NAME 的值为 new_value
unset VAR_NAME         # 删除环境变量 VAR_NAME
```

这些只是在当前 shell 会话中生效，如果想要永久生效，需要将 export 命令添加到用户的 shell 配置文件中，如 ~/.bashrc。

# 概念讲解

## 下载

### APT

APT（Advanced Package Tool）是Debian及其衍生发行版（如Ubuntu）中用于管理软件包的工具。它提供了一套命令行工具，用于安装、升级、删除和管理软件包。简单来说，APT是ubuntu系统中用来管理软件的软件包管理器,方便用户从`可靠来源`下载和安装软件。

APT的主要功能包括：
1. 软件包安装：通过APT，用户可以轻松地从软件仓库中安装所需的软件包。
2. 软件包升级：APT可以自动检查并升级已安装的软件包，确保系统保持最新状态。
3. 依赖管理：APT会自动处理软件包之间的依赖关系，确保安装的软件包能够正常运行。

#### apt的软件从哪来的？

APT从软件仓库（repository）中获取软件包。软件仓库是一个集中存储和分发软件包的服务器，通常由操作系统的维护者或第三方组织管理。APT通过配置文件（如`/etc/apt/sources.list`）指定要使用的软件仓库地址。

#### apt的软件下载/安装到哪？

APT下载的软件包通常存储在本地的缓存目录中，默认情况下，这个目录是`/var/cache/apt/archives/`。当用户使用APT安装软件包时，APT会先从指定的软件仓库下载软件包并将其存储在这个缓存目录中，然后再进行安装。安装位置则取决于软件包的类型和配置，通常会安装在系统的标准目录中，如`/usr/bin/`、`/usr/lib/`等。

